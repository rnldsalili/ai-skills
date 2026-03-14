---
name: zod
description: Write, review, or debug TypeScript/JavaScript validation code using Zod 4. Use this skill whenever the user mentions Zod, schema validation, z.object, z.string, z.infer, parsing with Zod, ZodError, or wants to validate data, define schemas, or infer TypeScript types from schemas. Also triggers when migrating from Zod 3 to Zod 4, debugging parse errors, or asking about best practices for data validation in TypeScript.
---

# Zod 4 Skill

Zod is a TypeScript-first schema validation library. This skill covers **Zod 4** — the current major version. Always use Zod 4 APIs and avoid deprecated/removed methods from Zod 3.

## Setup

```bash
npm install zod
```

```ts
// Recommended import style
import * as z from "zod";
```

**tsconfig.json** — `"strict": true` is required.

---

## Core Concepts

### Defining schemas

```ts
const User = z.object({
  name: z.string(),
  age: z.number().int().min(0),
  email: z.email(),              // top-level format (preferred over z.string().email())
  role: z.enum(["admin", "user"]),
});
```

### Parsing data

```ts
// Throws ZodError on failure
const user = User.parse(input);

// Safe: returns { success, data } | { success: false, error }
const result = User.safeParse(input);
if (result.success) {
  console.log(result.data);
} else {
  console.error(result.error.issues);
}

// Async (required when schema uses async refinements/transforms)
const user = await User.parseAsync(input);
const result = await User.safeParseAsync(input);
```

### Type inference

```ts
type User = z.infer<typeof User>;

// For schemas with diverging input/output types (e.g. transforms):
type UserInput  = z.input<typeof User>;
type UserOutput = z.output<typeof User>;
```

---

## Primitives

```ts
z.string()
z.number()
z.bigint()
z.boolean()
z.date()
z.symbol()
z.undefined()
z.null()
z.void()
z.any()
z.unknown()
z.never()
```

---

## Strings

### Validations & transforms

```ts
z.string().min(1).max(100)
z.string().length(10)
z.string().regex(/^[a-z]+$/)
z.string().includes("foo")
z.string().startsWith("foo")
z.string().endsWith("bar")
z.string().trim()          // transform: trims whitespace
z.string().toLowerCase()   // transform: lowercases
z.string().toUpperCase()   // transform: uppercases
z.string().nonempty()      // shorthand for .min(1)
```

### String formats — use top-level APIs (preferred in Zod 4)

```ts
z.email()          // NOT z.string().email() (deprecated)
z.url()            // NOT z.string().url() (deprecated)
z.uuid()           // strict RFC 9562/4122 — use z.guid() for permissive
z.guid()           // permissive UUID-like identifier
z.cuid()
z.cuid2()
z.ulid()
z.nanoid()
z.ipv4()           // NOT z.string().ip() (dropped)
z.ipv6()           // NOT z.string().ip() (dropped)
z.cidrv4()         // NOT z.string().cidr() (dropped)
z.cidrv6()         // NOT z.string().cidr() (dropped)
z.mac()
z.jwt()
z.base64()
z.base64url()      // unpadded by default
z.iso.datetime()   // ISO 8601
z.iso.date()       // YYYY-MM-DD
z.iso.time()       // HH:MM[:SS[.s+]]
```

To accept both IPv4 and IPv6:
```ts
z.union([z.ipv4(), z.ipv6()])
```

---

## Numbers

```ts
z.number()                  // any finite number (no Infinity)
z.number().min(0).max(100)
z.number().positive()
z.number().negative()
z.number().nonnegative()
z.number().nonpositive()
z.number().multipleOf(5)
z.number().finite()
z.number().safe()           // integer within safe range (Zod 4: same as .int())
z.int()                     // preferred alias for safe integers
z.float32()
z.float64()
```

---

## Objects

```ts
// Default: strips unknown keys
z.object({ name: z.string() })

// Strict: errors on unknown keys (instead of .strict() — deprecated)
z.strictObject({ name: z.string() })

// Loose: passes unknown keys through (instead of .passthrough() — deprecated)
z.looseObject({ name: z.string() })
```

### Object utilities

```ts
const Base = z.object({ id: z.string(), name: z.string() });

// Extend (preferred over .merge() which is deprecated)
const Extended = Base.extend({ age: z.number() });

// Even better for large schemas (more tsc-efficient):
const Extended = z.object({ ...Base.shape, age: z.number() });

// Pick / Omit
const NameOnly  = Base.pick({ name: true });
const WithoutId = Base.omit({ id: true });

// Partial / Required
const PartialUser = Base.partial();
const RequiredUser = Base.required();

// Access shape
Base.shape.name  // => ZodString

// Keyof
Base.keyof()     // => ZodEnum<["id", "name"]>
```

### Recursive objects

```ts
type Category = { name: string; subcategories: Category[] };
const Category: z.ZodType<Category> = z.object({
  name: z.string(),
  get subcategories() {
    return z.array(Category);
  },
});
```

---

## Arrays

```ts
z.array(z.string())
z.array(z.string()).min(1).max(10)
z.array(z.string()).length(5)
z.array(z.string()).nonempty()   // behaves like .min(1)
```

---

## Enums

```ts
// String enum (preferred)
const Role = z.enum(["admin", "user", "guest"]);
type Role = z.infer<typeof Role>;  // "admin" | "user" | "guest"

// Access enum object
Role.enum.admin  // => "admin"

// Pick/exclude values
Role.extract(["admin"])
Role.exclude(["guest"])

// TypeScript native enum (z.nativeEnum is DEPRECATED — use z.enum instead)
enum Direction { Up = "UP", Down = "DOWN" }
const DirectionSchema = z.enum(Direction);
```

---

## Optionals, Nullables, Defaults

```ts
z.string().optional()         // string | undefined
z.string().nullable()         // string | null
z.string().nullish()          // string | null | undefined

// Default — short-circuits if input is undefined; value must match OUTPUT type
z.string().default("hello")
z.string().toLowerCase().default("hello")

// Prefault — parses the default through the schema; value must match INPUT type
z.string().toLowerCase().prefault("HELLO")  // result: "hello"

// Catch — returns fallback on any validation failure
z.number().catch(0)
```

---

## Unions & Intersections

```ts
// Regular union — first match wins
z.union([z.string(), z.number()])

// Discriminated union — more efficient for large unions
z.discriminatedUnion("status", [
  z.object({ status: z.literal("ok"),    data: z.string() }),
  z.object({ status: z.literal("error"), message: z.string() }),
])

// Exclusive union (XOR) — exactly one option must match
z.xor(z.object({ a: z.string() }), z.object({ b: z.number() }))

// Intersection
z.intersection(SchemaA, SchemaB)
// Or prefer: z.object({ ...SchemaA.shape, ...SchemaB.shape })
```

---

## Records, Maps, Sets, Tuples

```ts
// Record — now does exhaustiveness checking with enum keys
z.record(z.string(), z.number())
z.record(z.enum(["a", "b"]), z.number())   // exhaustive: both keys required

// Partial record — skips exhaustiveness (replicates Zod 3 behavior)
z.partialRecord(z.enum(["a", "b"]), z.number())

// Loose record — passes non-matching keys through
z.looseRecord(z.string(), z.number())

// Map / Set
z.map(z.string(), z.number())
z.set(z.string()).min(1).max(5)

// Tuple
z.tuple([z.string(), z.number()])
z.tuple([z.string(), z.number()]).rest(z.boolean())  // variadic rest
```

---

## Transforms & Pipes

```ts
// .transform() pipes into a transformation
const schema = z.string().transform(val => val.length);
// inferred type: number

// z.preprocess() — transform BEFORE validation
const schema = z.preprocess(val => Number(val), z.number());

// Manual pipe
const schema = z.string().pipe(z.coerce.number());
```

Transforms should never throw — push issues onto `ctx.issues` to report errors:

```ts
z.string().transform((val, ctx) => {
  const parsed = JSON.parse(val);
  if (!parsed.id) ctx.issues.push({ code: "custom", message: "Missing id" });
  return parsed;
});
```

---

## Refinements

```ts
// .refine() — custom validation
z.string().refine(val => val.includes("@"), {
  error: "Must contain @",   // use `error`, NOT `message` (deprecated)
});

// Abort on first failure
z.string().refine(val => val.length > 3, { abort: true, error: "Too short" });

// Custom error path (useful on objects)
z.object({ pw: z.string(), confirm: z.string() }).refine(
  data => data.pw === data.confirm,
  { path: ["confirm"], error: "Passwords must match" }
);

// .superRefine() — access full ctx, emit multiple issues
z.string().superRefine((val, ctx) => {
  if (val.length < 3) ctx.issues.push({ code: "too_small", minimum: 3, type: "string", inclusive: true });
});
```

---

## Error Handling

```ts
import * as z from "zod";

const result = schema.safeParse(input);
if (!result.success) {
  const err = result.error;

  // Human-readable string
  console.log(z.prettifyError(err));

  // Tree structure (replaces deprecated .format())
  const tree = z.treeifyError(err);

  // Flat structure (replaces deprecated .flatten())
  const flat = z.flattenError(err);
  // { formErrors: string[], fieldErrors: { [key]: string[] } }

  // ZodError-shaped nested object (replaces deprecated .format())
  const formatted = z.formatError(err);

  // Raw issues array
  err.issues;
}
```

**Customize error messages** using the `error` param (not `message`, not `errorMap`):

```ts
z.string({ error: "Must be a string" })
z.string().min(3, { error: "At least 3 chars" })

// Dynamic error (error map function)
z.string({ error: (issue) => issue.code === "too_small" ? "Too short!" : undefined })
```

---

## Coercion

```ts
z.coerce.string()   // String(input)
z.coerce.number()   // Number(input)
z.coerce.boolean()  // Boolean(input)
z.coerce.date()     // new Date(input)
z.coerce.bigint()   // BigInt(input)
```

Input type of coerced schemas is `unknown`.

---

## JSON Schema Conversion

```ts
// Zod → JSON Schema
const jsonSchema = z.toJSONSchema(MySchema);

// JSON Schema → Zod
const zodSchema = z.fromJSONSchema(jsonSchema);
```

---

## Miscellaneous

```ts
// Literal
z.literal("admin")
z.literal(42)
z.literal(true)
z.literal([null, undefined])   // multiple literals (Zod 4)

// String boolish values
z.stringbool()   // "true"/"1"/"yes" → true, "false"/"0"/"no" → false

// Branded types (nominal typing)
const UserId = z.string().brand<"UserId">();
type UserId = z.infer<typeof UserId>;

// Readonly
z.array(z.string()).readonly()

// Custom schema
z.custom<`${string}@${string}`>(val => typeof val === "string" && val.includes("@"))

// Instanceof
z.instanceof(Error)

// Function validation
const fn = z.function({
  input: z.tuple([z.string(), z.number()]),
  output: z.string(),
}).implement((name, age) => `${name} is ${age}`);

// JSON value
z.json()    // any JSON-encodable value

// Metadata
z.string().meta({ description: "User's email address" })
z.string().describe("User's email")  // shorthand
```

---

## Deprecated / Removed in Zod 4 — Never Use These

| ❌ Zod 3 (avoid)                    | ✅ Zod 4 (use instead)                          |
|-------------------------------------|--------------------------------------------------|
| `z.string().email()`                | `z.email()`                                     |
| `z.string().url()`                  | `z.url()`                                       |
| `z.string().uuid()`                 | `z.uuid()`                                      |
| `z.string().ip()`                   | `z.union([z.ipv4(), z.ipv6()])`                 |
| `z.string().cidr()`                 | `z.union([z.cidrv4(), z.cidrv6()])`             |
| `z.nativeEnum(Enum)`               | `z.enum(Enum)`                                  |
| `z.object({}).strict()`             | `z.strictObject({})`                            |
| `z.object({}).passthrough()`        | `z.looseObject({})`                             |
| `z.object({}).strip()`              | `z.object({})` (default behavior)               |
| `A.merge(B)`                        | `A.extend(B.shape)` or spread syntax            |
| `z.record(valueSchema)`             | `z.record(z.string(), valueSchema)`             |
| `z.promise()`                       | `await` before parsing; rarely needed           |
| `z.deepPartial()`                   | (removed — no direct replacement)               |
| `schema.refine(fn, { message: "" })`| `schema.refine(fn, { error: "" })`              |
| `{ errorMap: ... }`                 | `{ error: ... }`                                |
| `{ invalid_type_error: "" }`        | `{ error: "" }`                                 |
| `{ required_error: "" }`            | `{ error: "" }`                                 |
| `error.format()`                    | `z.treeifyError(error)` / `z.formatError()`     |
| `error.flatten()`                   | `z.flattenError(error)`                         |
| `error.addIssue()`                  | push directly to `err.issues`                   |
| `z.ZodTypeAny`                      | `z.ZodType`                                     |
| `z.ostring()`, `z.onumber()`, etc.  | `z.string().optional()`, etc.                   |

---

## Best Practices

1. **Prefer `safeParse`** over `parse` for user input — avoid try/catch noise.
2. **Use top-level string format APIs** (`z.email()`, `z.url()`, etc.) — they're tree-shakable and less verbose.
3. **Colocate types with schemas** — use `z.infer<typeof Schema>` rather than maintaining separate type definitions.
4. **Use `z.discriminatedUnion`** instead of `z.union` when schemas share a literal discriminator key — it's significantly faster.
5. **Use spread syntax for merging** large schemas: `z.object({ ...A.shape, ...B.shape })` — it's more efficient than `.extend()` chains and avoids TypeScript's quadratic slowdown.
6. **Validate at boundaries** (API routes, form inputs, env vars) and trust the inferred types everywhere else.
7. **Use `.default()` for post-parse defaults** (value is output type); use `.prefault()` when you want the default to go through parsing (value is input type).
8. **Never throw inside `.refine()` or `.transform()`** — push to `ctx.issues` instead.
9. **Use `async` parsing methods** (`.parseAsync`, `.safeParseAsync`) whenever your schema contains async refinements or transforms.
10. **For environment variables**, use `z.coerce` or `z.stringbool()` to handle string inputs from `process.env`.

```ts
// Example: environment variable schema
const Env = z.object({
  PORT:     z.coerce.number().default(3000),
  DEBUG:    z.stringbool().default(false),
  DATABASE_URL: z.url(),
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
});

const env = Env.parse(process.env);
```
