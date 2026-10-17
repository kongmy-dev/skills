# Env and config

## What goes where

| Kind of value | Lives in |
|---|---|
| secrets (keys, passwords, DSNs) and process wiring (ports, hostnames) | env, validated |
| a value that varies per tenant, or is switched on over time | runtime config in the database, typed and validated on read |
| a tested contract (a role map, a state machine) | code, with runtime overrides layered on top if it must vary |
| a fixed fact (a page size a vendor imposes) | a named constant beside its user |

Env is the wrong place for a feature flag: changing it needs a redeploy, and it
can't differ per tenant.

## Validate env per group, lazily, and fail at boot

```ts
import { z } from 'zod';

export function defineEnv<S extends z.ZodObject<z.ZodRawShape>>(
  name: string,
  schema: S,
  read: () => Record<string, string | undefined> = () => process.env
) {
  let cached: z.infer<S> | undefined;
  const get = (): z.infer<S> => {
    if (cached !== undefined) return cached;
    const result = schema.safeParse(read());
    if (!result.success) {
      const problems = result.error.issues
        .map((issue) => `  ${issue.path.join('.')}: ${issue.message}`)
        .join('\n');
      throw new Error(`Invalid environment for "${name}":\n${problems}\nSee .env.example.`);
    }
    cached = result.data;
    return cached;
  };
  return Object.assign(get, { envName: name, keys: Object.keys(schema.shape) });
}

export const dbEnv = defineEnv('db', z.object({ DATABASE_URL: z.url() }));
```

- **Per group**, because different entrypoints need different subsets. A build or
  a worker that never touches the database shouldn't need `DATABASE_URL`.
- **Lazy and memoised**: parsed on first read, once.
- **Every problem at once.** Throwing on the first missing variable makes a fresh
  deploy discover its config one restart at a time.
- **Values never appear in the error.** Env holds secrets.
- **Fail at boot.** Each app calls an `assertEnv([dbEnv, authEnv])` from its entry
  point for exactly the groups it needs, so a misconfigured deploy dies on start.
- `read` is a parameter so tests pass a plain object instead of mutating
  `process.env`.

## `.env.example`

Every key any schema reads is listed, with a placeholder, never a real value. A
test compares the schemas' `keys` with the file, so a new variable can't ship
without its documentation. `.env` and `.env.*` (except `.env.example`) are
git-ignored.

Bun loads `.env` automatically. Scripts that need it name it
(`bun --env-file=.env scripts/migrate.ts`); tests never depend on the real one.

## Secrets

- Never in a file that is committed, a log line, an error message, a job payload
  or an API response.
- A secret stored in the database is encrypted on write and decrypted on read,
  inside one module.
- Notes and docs say where a secret lives (which host, which file), never its value.

## Money and time (C4)

- **Money** is never a JS `number` with a fractional part. Use integer minor units
  (`amountMinor: 1999` plus a currency) or decimal strings worked by a decimal
  library, one representation per project, through one helper module. A sum over
  money groups by currency or filters to one. Convert at the write boundary, once.
- **Instants** are `Date`/`timestamptz` and end in `At`. **Calendar dates** are
  ISO `YYYY-MM-DD` strings or `date` columns and end in `On`.
- **"Today" is the business's day in its time zone**, from one function. Never
  `new Date().toISOString().slice(0, 10)` or `getFullYear()` on a machine clock:
  in UTC+8 that reads yesterday until 08:00.
- Tests write instants with their offset (`'2026-09-24T14:32:00+08:00'`). `bun test`
  runs in UTC, so code that leans on the machine's zone fails on every machine,
  not just the server.
