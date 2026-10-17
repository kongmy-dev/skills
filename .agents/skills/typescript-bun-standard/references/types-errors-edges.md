# Types, errors and edges

The compiler is the proof that the code means what it says. Each rule here moves a
fact from a sentence someone must remember into a check that runs.

## Model the states (T1)

```ts
type Payout =
  | { readonly state: 'pending' }
  | { readonly state: 'settled'; readonly settledAt: Date; readonly amountMinor: number }
  | { readonly state: 'rejected'; readonly reason: RejectReason };
```

If you could write a sentence explaining when a combination of fields is valid, the
type is too loose.

Exhaustive matching, so a new variant breaks every place that must handle it:

```ts
export function unreachable(value: never): never {
  throw new Error(`Unreachable: ${JSON.stringify(value)}`);
}

function payoutLabel(p: Payout): string {
  switch (p.state) {
    case 'pending': return 'Waiting';
    case 'settled': return 'Paid';
    case 'rejected': return 'Refused';
    default: return unreachable(p);
  }
}
```

When every branch is a value, a table is the same proof with less code:
`const PAYOUT_LABEL: Record<Payout['state'], string> = { … }`.

## Declare meaningful strings once (T4)

```ts
export const OrderStatus = { Draft: 'draft', Placed: 'placed', Cancelled: 'cancelled' } as const;
export type OrderStatus = (typeof OrderStatus)[keyof typeof OrderStatus];
```

Not `enum`: it isn't erasable, and numeric enums reverse-map. The declaration lives
where the concept is owned. Test data, log messages and user-facing copy are not
covered; copy lives with the screen or the message catalogue.

## Brand what must not be mixed

```ts
type Brand<T, B extends string> = T & { readonly __brand: B };
export type CustomerId = Brand<string, 'CustomerId'>;

export function parseCustomerId(raw: string): CustomerId | null {
  return /^cus_[a-z0-9]{12}$/.test(raw) ? (raw as CustomerId) : null;
}
```

The `as` inside the parse function is the one sanctioned cast: that function is
the boundary. Brand ids that cross modules; an id used inside one module can stay
`string`. Don't brand money: give it a type with its currency instead.

## Cast nothing; derive everything (T2, T3)

- `unknown` at the edge, narrowed by a schema. No `any`.
- No `!`. `noUncheckedIndexedAccess` says the value may be missing; decide what
  happens when it is.
- No `as` on data from outside, no `as unknown as` anywhere. `satisfies` checks a
  literal without widening it.
- Types come from their owner: `z.infer<typeof schema>`, the ORM's row type,
  `Pick`/`Omit`, `ReturnType`. Row schemas derive from tables; wire schemas for a
  vendor's protocol are written once, beside the adapter that speaks it.

## Parse at the edge (E1)

Everything that enters from outside is parsed once:

| Edge | Parse with |
|---|---|
| HTTP body, query, path params | the route's zod schema |
| env | the env group's schema (`env-and-config.md`) |
| vendor response, webhook payload | the adapter's wire schema |
| `JSON.parse`, file contents, browser storage | a schema, straight away |
| LLM output | a schema, with a fallback when it fails |

Inside, the types are trusted: no second validation and no defensive `if (!x)` on
values the type says exist.

Reads of stored config can be fail-soft (log the issue, never the value, fall back
to a tested default). Writes are fail-hard.

## Returned failures vs thrown errors (E2)

| Failure | Shape |
|---|---|
| A domain outcome the caller must decide on | returned union: `{ ok: false; reason: 'not-billable' \| 'sku-unknown' }` |
| Input that failed its schema | returned at the boundary as field errors |
| A refused permission | thrown typed error, mapped once by the transport (403) |
| An outage or broken contract with an outside system | thrown typed error from the adapter |
| A programming error | thrown `Error` |

```ts
type SaveMappingOutcome =
  | { readonly ok: true; readonly mappingId: string }
  | { readonly ok: false; readonly reason: 'sku-unknown' | 'item-disabled' | 'already-mapped' };
```

- `reason` is a literal union, so the caller matches exhaustively and the words a
  person reads are chosen at the edge, not deep inside.
- Each use case declares its own union: no shared generic `Outcome<T, E>` and no
  `Result` class or library. Plain data crosses a queue payload or a
  server/client boundary unchanged.
- `try`/`catch` lives at the transport (route handler, server action, job runner,
  CLI `main`) and in an adapter translating a vendor error into ours. A `catch`
  anywhere else that swallows, returns an empty list or logs and carries on is a
  bug report waiting to be filed.

## One shape on the wire (E3)

HTTP errors are `application/problem+json` (RFC 9457): `status`, `title`, and a
`detail` written for the person who will read it, saying what to do. An unexpected
error is logged with the request id and answered with a fixed 500 body, never the
thrown message and never a stack.

## Dependencies point inward, and are parameters (E4)

```
app / transport  →  service (use cases)  →  core (pure)
                         ↓
                    adapters (db, vendors)
```

- Core knows no framework, database, HTTP client, clock or vendor. Pass `now`, ids
  and config in.
- A use case is `(input, deps)`. One wiring file per module builds the real deps;
  tests pass plain fakes. No DI container and no service locator: a missing
  dependency should be a compile error, not a runtime one.
- An app composes and never constructs a module's adapters. If an app needs an
  adapter, the module is missing a use case.
- Each package has one public entry, sealed by its `exports` map. Nothing imports
  another package's internals. A deliberate second entry (for example `./jobs`)
  is listed in `exports` and named in the lint rule that allows it.

## One vendor, one adapter (E5)

A vendor's words, ids and wire shapes stay in `adapters/<vendor>/`; core and
services speak the business's words. Every outbound call has a timeout. A vendor
failure degrades the feature (manual entry, a retry job), and never blocks the
flow it isn't essential to. Don't call a third party inside a database
transaction: queue a job in the transaction instead.

## Logs (E6)

One structured logger (pino or the platform's), with a request or job id on every
line. No `console.*` outside CLI scripts. Never log a password, token, API key,
session id, cookie or a whole request body; log the field you need.
