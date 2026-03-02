# Coding Style

## Immutability (CRITICAL)

ALWAYS create new objects, NEVER mutate:

```javascript
// WRONG: Mutation
function updateUser(user, name) {
  user.name = name  // MUTATION!
  return user
}

// CORRECT: Immutability
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

## File Organization

MANY SMALL FILES > FEW LARGE FILES:
- High cohesion, low coupling
- 200-400 lines typical, 800 max
- Extract utilities from large components
- Organize by feature/domain, not by type

## Error Handling with neverthrow (CRITICAL)

Use `neverthrow` for type-safe error handling. Errors are **values**, not exceptions.
`throw` is reserved for truly unrecoverable bugs — all expected failures MUST be expressed as `Result<T, E>`.

### Setup

```bash
npm install neverthrow
npm install eslint-plugin-neverthrow  # Ensures Results are always consumed
```

### Define Domain Error Types

Use discriminated unions with a `type` field for composable, narrowable errors:

```typescript
// errors.ts
type NotFoundError = { type: 'not_found'; resource: string; id: string }
type ValidationError = { type: 'validation'; field: string; message: string }
type DatabaseError = { type: 'database'; cause: Error }
type NetworkError = { type: 'network'; cause: Error }

type AppError = NotFoundError | ValidationError | DatabaseError | NetworkError
```

### Synchronous — Return `Result<T, E>`

```typescript
import { ok, err, Result } from 'neverthrow'

function divide(a: number, b: number): Result<number, ValidationError> {
  if (b === 0) {
    return err({ type: 'validation', field: 'divisor', message: 'Cannot divide by zero' })
  }
  return ok(a / b)
}
```

### Asynchronous — Return `ResultAsync<T, E>`

```typescript
import { ResultAsync, errAsync } from 'neverthrow'

function findUser(id: string): ResultAsync<User, NotFoundError | DatabaseError> {
  if (!id) {
    return errAsync({ type: 'not_found', resource: 'user', id })
  }
  return ResultAsync.fromPromise(
    db.query('SELECT * FROM users WHERE id = $1', [id]),
    (e): DatabaseError => ({ type: 'database', cause: e instanceof Error ? e : new Error(String(e)) })
  )
}
```

### Wrapping Third-Party Code

Use `fromThrowable` / `ResultAsync.fromThrowable` to convert throwing libraries into `Result`:

```typescript
import { Result, ResultAsync } from 'neverthrow'

// Sync: wrap JSON.parse
const safeJsonParse = Result.fromThrowable(
  JSON.parse,
  (e): ValidationError => ({
    type: 'validation',
    field: 'json',
    message: e instanceof Error ? e.message : 'Invalid JSON'
  })
)

const parsed = safeJsonParse('{"valid": true}') // Result<any, ValidationError>

// Async: wrap a function that returns Promise and may throw synchronously
const safeFetchUser = ResultAsync.fromThrowable(
  (id: string) => fetchUserFromApi(id),
  (e): NetworkError => ({
    type: 'network',
    cause: e instanceof Error ? e : new Error(String(e))
  })
)
```

### Chaining with `map` / `andThen`

```typescript
function processOrder(orderId: string): ResultAsync<Invoice, AppError> {
  return findOrder(orderId)                    // ResultAsync<Order, NotFoundError | DatabaseError>
    .andThen(validateStock)                    // ResultAsync<Order, AppError>
    .andThrough(reserveInventory)              // side-effect that can fail, passes Order through
    .andThen(createInvoice)                    // ResultAsync<Invoice, AppError>
    .andTee((invoice) => logger.info(invoice)) // side-effect that won't affect the chain
}
```

### Consuming Results

```typescript
// Pattern 1: match (preferred — forces both paths)
const message = result.match(
  (value) => `Success: ${value}`,
  (error) => `Failed: ${error.message}`
)

// Pattern 2: Type-guard narrowing
if (result.isErr()) {
  switch (result.error.type) {
    case 'not_found':
      return res.status(404).json({ error: result.error.message })
    case 'validation':
      return res.status(400).json({ error: result.error.message })
    case 'database':
      return res.status(500).json({ error: 'Internal server error' })
  }
}
const value = result.value // TypeScript knows this is the Ok value

// Pattern 3: unwrapOr (when a default is acceptable)
const count = result.unwrapOr(0)
```

### Combining Multiple Results

```typescript
import { Result, ResultAsync } from 'neverthrow'

// Like Promise.all — short-circuits on first error
const combined = Result.combine([
  validateName(input.name),
  validateEmail(input.email),
  validateAge(input.age)
])
// Result<[string, string, number], ValidationError>

// Collect ALL errors instead of short-circuiting
const allValidated = Result.combineWithAllErrors([
  validateName(input.name),
  validateEmail(input.email),
  validateAge(input.age)
])
// Result<[string, string, number], ValidationError[]>
```

### Error Recovery with `orElse`

```typescript
const result = findUserInCache(id)
  .orElse((cacheError) =>
    cacheError.type === 'not_found'
      ? findUserInDatabase(id)  // recover: fall back to DB
      : err(cacheError)         // propagate other errors
  )
```

### Rules

- NEVER use `try/catch` for business logic — only at system boundaries to wrap third-party code into `Result`
- NEVER use `_unsafeUnwrap` in production code
- ALWAYS define error types as discriminated unions with a `type` field
- ALWAYS consume every `Result` via `match`, `isOk`/`isErr` check, or `unwrapOr`
- Use `andTee` / `orTee` for side-effects (logging) that should not affect the chain
- Use `andThrough` for side-effects that CAN fail and should abort the chain

## Input Validation

ALWAYS validate user input with Zod, and integrate with neverthrow:

```typescript
import { z } from 'zod'
import { ok, err, Result } from 'neverthrow'

// Define schema
const CreateUserSchema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

type CreateUserInput = z.infer<typeof CreateUserSchema>

// Zod → Result adapter
type ZodParseError = { type: 'validation'; errors: z.ZodError }

function safeZodParse<T extends z.ZodSchema>(
  schema: T
): (data: unknown) => Result<z.infer<T>, ZodParseError> {
  return (data) => {
    const result = schema.safeParse(data)
    return result.success
      ? ok(result.data)
      : err({ type: 'validation', errors: result.error })
  }
}

// Usage — composable with other Result chains
const parseCreateUser = safeZodParse(CreateUserSchema)

function handleCreateUser(raw: unknown): ResultAsync<User, AppError> {
  return parseCreateUser(raw)         // Result<CreateUserInput, ZodParseError>
    .asyncAndThen(insertUser)          // ResultAsync<User, DatabaseError>
}
```

## Code Quality Checklist

Before marking work complete:
- [ ] Code is readable and well-named
- [ ] Functions are small (<50 lines)
- [ ] Files are focused (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Error handling uses `Result<T, E>` — no raw `try/catch` in business logic
- [ ] Error types are discriminated unions with `type` field
- [ ] All `Result` values are consumed (`match` / `isOk` / `unwrapOr`)
- [ ] No `console.log` statements
- [ ] No hardcoded values
- [ ] No mutation (immutable patterns used)
