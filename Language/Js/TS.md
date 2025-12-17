# TypeScript — Setup & Basics (Easy & Example)

Quick guide to get started with TypeScript: installing the compiler, creating tsconfig.json, and compiling TS → JS.

---

## 1. Installing TypeScript (tsc)

- Globally:

```bash
npm install -g typescript
# then `tsc --version`
```

- Locally (recommended per project):

```bash
npm init -y
npm install --save-dev typescript
# run: npx tsc --version
```

- Helpful tools:
  - ts-node (run .ts directly): npm install --save-dev ts-node
  - types for node/browser libs: npm install --save-dev @types/node

---

## 2. Configure tsconfig.json

Create a default config:

```bash
npx tsc --init
```

Minimal useful tsconfig.json example:

```json
{
  "compilerOptions": {
    "target": "ES2019", // output JS version
    "module": "CommonJS", // module system
    "strict": true, // enable strict checks
    "esModuleInterop": true, // interop with CommonJS modules
    "skipLibCheck": true,
    "outDir": "dist", // compiled files go here
    "rootDir": "src", // source files root
    "sourceMap": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

Notes:

- target: ES version for emitted JS.
- module: CommonJS (Node) or ES2020/ESNext for ESM.
- strict: turn on for stronger type checks.
- outDir/rootDir: organize compiled output.

---

## 3. Compiling TS → JS

- Compile single file:

```bash
npx tsc src/example.ts
```

- Compile whole project (uses tsconfig.json):

```bash
npx tsc
```

- Watch mode (recompile on changes):

```bash
npx tsc -w
```

- Run without compiling using ts-node (dev):

```bash
npx ts-node src/example.ts
```

---

## 4. Easy examples (code + compiled output)

src/example.ts

```ts
function greet(name: string): string {
  return `Hello, ${name.toUpperCase()}`;
}

interface Person {
  name: string;
  age?: number;
}

const p: Person = { name: "Alice" };
console.log(greet(p.name));
```

Compile:

```bash
npx tsc src/example.ts --outDir dist
```

dist/example.js (emitted JS, simplified)

```js
function greet(name) {
  return "Hello, " + name.toUpperCase();
}
var p = { name: "Alice" };
console.log(greet(p.name));
```

Console output when run:

```
Hello, ALICE
```

---

## 5. Quick tips

- Use "strict": true for best safety.
- Keep source in src/ and compiled output in dist/.
- Use type definitions (@types/...) for third-party libs.
- Use build scripts in package.json:

```json
{
  "scripts": {
    "build": "tsc",
    "watch": "tsc -w",
    "start:ts": "ts-node src/index.ts"
  }
}
```

That's the minimal, easy-to-remember TypeScript setup and compile flow with examples.// filepath: /media/awolad/MEDIA1.1/learning/Language/Js/TS.md

# TypeScript — Setup & Basics (Easy & Example)

Quick guide to get started with TypeScript: installing the compiler, creating tsconfig.json, and compiling TS → JS.

---

## 1. Installing TypeScript (tsc)

- Globally:

```bash
npm install -g typescript
# then `tsc --version`
```

- Locally (recommended per project):

```bash
npm init -y
npm install --save-dev typescript
# run: npx tsc --version
```

- Helpful tools:
  - ts-node (run .ts directly): npm install --save-dev ts-node
  - types for node/browser libs: npm install --save-dev @types/node

---

## 2. Configure tsconfig.json

Create a default config:

```bash
npx tsc --init
```

Minimal useful tsconfig.json example:

```json
{
  "compilerOptions": {
    "target": "ES2019", // output JS version
    "module": "CommonJS", // module system
    "strict": true, // enable strict checks
    "esModuleInterop": true, // interop with CommonJS modules
    "skipLibCheck": true,
    "outDir": "dist", // compiled files go here
    "rootDir": "src", // source files root
    "sourceMap": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

Notes:

- target: ES version for emitted JS.
- module: CommonJS (Node) or ES2020/ESNext for ESM.
- strict: turn on for stronger type checks.
- outDir/rootDir: organize compiled output.

---

## 3. Compiling TS → JS

- Compile single file:

```bash
npx tsc src/example.ts
```

- Compile whole project (uses tsconfig.json):

```bash
npx tsc
```

- Watch mode (recompile on changes):

```bash
npx tsc -w
```

- Run without compiling using ts-node (dev):

```bash
npx ts-node src/example.ts
```

---

## 4. Easy examples (code + compiled output)

src/example.ts

```ts
function greet(name: string): string {
  return `Hello, ${name.toUpperCase()}`;
}

interface Person {
  name: string;
  age?: number;
}

const p: Person = { name: "Alice" };
console.log(greet(p.name));
```

Compile:

```bash
npx tsc src/example.ts --outDir dist
```

dist/example.js (emitted JS, simplified)

```js
function greet(name) {
  return "Hello, " + name.toUpperCase();
}
var p = { name: "Alice" };
console.log(greet(p.name));
```

Console output when run:

```
Hello, ALICE
```

---

## 5. Quick tips

- Use "strict": true for best safety.
- Keep source in src/ and compiled output in dist/.
- Use type definitions (@types/...) for third-party libs.
- Use build scripts in package.json:

```json
{
  "scripts": {
    "build": "tsc",
    "watch": "tsc -w",
    "start:ts": "ts-node src/index.ts"
  }
}
```

## Types

Clear, easy notes with short TypeScript examples and expected outputs.

### Primitive types

- Basic built-in types.

```ts
const s: string = "hello";
const n: number = 42;
const b: boolean = true;
const nl: null = null;
const undef: undefined = undefined;
const sym: symbol = Symbol("id");
const big: bigint = 10n;

console.log(s, n, b); // hello 42 true
console.log(nl, undef); // null undefined
console.log(typeof sym); // "symbol"
console.log(big + 1n); // 11n
```

Notes:

- Use bigint for integers beyond Number.MAX_SAFE_INTEGER (append `n`).
- null and undefined are distinct values; with strict mode they are not freely assignable.

---

### Arrays & Tuples

- Arrays hold many values of a type. Tuples hold a fixed number of elements with known types.

```ts
// Arrays
const nums: number[] = [1, 2, 3];
const strs: Array<string> = ["a", "b"];

console.log(nums[0]); // 1
console.log(strs.map((s) => s.toUpperCase())); // ["A","B"]

// Tuples (fixed-length, typed positions)
const pair: [string, number] = ["age", 30];
const [key, value] = pair;
console.log(key, value); // "age" 30

// Tuple with optional/rest positions
type NameParts = [first: string, last?: string];
const n1: NameParts = ["Alice"];
const n2: NameParts = ["Bob", "Smith"];
```

Notes:

- Array generics (Array<T>) and T[] are equivalent.
- Tuples enforce element types/positions; methods like push can still mutate tuples (careful).

---

### any, unknown, void, never

- any: opt-out of type checking (use sparingly).
- unknown: safer alternative to any — must be narrowed before use.
- void: absence of meaningful return (usually functions).
- never: function never returns (throws or infinite loop).

```ts
let a: any = 5;
a = "now a string";
a.doSomething(); // allowed at compile time (no safety)

// unknown requires type check
let x: unknown = "text";
// x.toUpperCase(); // Error: Object is of type 'unknown'
if (typeof x === "string") {
  console.log(x.toUpperCase()); // safe after narrowing
}

// void
function logger(msg: string): void {
  console.log(msg);
}
logger("hi"); // prints "hi"

// never
function fail(msg: string): never {
  throw new Error(msg);
}
// function infinite(): never { while(true) {} }
try {
  // fail("fatal"); // throws Error
} catch (e) {}
```

Cheat:

- Use any only when you must bypass types.
- Prefer unknown to force safe narrowing.
- void for functions that don't return a value.
- never for functions that cannot complete normally.



---

## Objects

Simple, practical notes with examples and expected outputs.

### Object type annotations

- Inline annotation for object shapes, use ? for optional, readonly for immutable fields.

```ts
const person: {
  readonly id: number;
  name: string;
  age?: number;
  greet: () => string;
} = {
  id: 1,
  name: "Alice",
  greet() {
    return `Hi, I'm ${this.name}`;
  },
};

console.log(person.name); // Alice
console.log(person.greet()); // Hi, I'm Alice
// person.id = 2;                // Error: Cannot assign to 'id' because it is a read-only property
```

- Index signature for dynamic keys:

```ts
const dictionary: { [key: string]: string } = { hello: "world" };
dictionary.foo = "bar";
console.log(dictionary.hello, dictionary.foo); // world bar
```

- Structural typing (duck typing): types match by shape, extra props allowed unless explicitly prohibited

```ts
function greet(obj: { name: string }) {
  console.log(`Hello ${obj.name}`);
}
greet({ name: "Bob", age: 30 }); // Hello Bob (extra age property is okay)
```

---

### Type aliases (type)

- Create reusable named type shapes, unions, intersections, and more.
- Use for unions or complex composition.

```ts
type ID = number | string; // union alias
type User = { name: string; age?: number };

const id1: ID = 42;
const id2: ID = "abc-123";

type Admin = User & { role: "admin" }; // intersection: User + extra fields
const admin: Admin = { name: "Alice", role: "admin" };

console.log(id1, id2, admin.name); // 42 abc-123 Alice
```

- Type aliases can also describe functions:

```ts
type Comparator<T> = (a: T, b: T) => number;
const numComp: Comparator<number> = (a, b) => a - b;
```

---

### Interfaces (interface)

- Interfaces describe object shapes and can be implemented by classes and extended by other interfaces.
- Interfaces support declaration merging (multiple declarations add up).

```ts
interface User {
  name: string;
  age?: number;
  greet?(): string;
}

interface Employee extends User {
  company: string;
}

class Person implements Employee {
  constructor(
    public name: string,
    public company: string,
    public age?: number
  ) {}
  greet() {
    return `Hi ${this.name} from ${this.company}`;
  }
}

const p = new Person("Carol", "Acme", 28);
console.log(p.greet()); // Hi Carol from Acme
```

- Index signature in interfaces:

```ts
interface StringMap {
  [key: string]: string;
}
const map: StringMap = { a: "1", b: "2" };
```

- Declaration merging (interface can be expanded):

```ts
interface Window {
  MY_APP?: string;
}
interface Window {
  OTHER?: number;
}
// both MY_APP and OTHER are available on Window
```

---

### Differences & when to use

- interface
  - Can be implemented by classes.
  - Supports declaration merging (extend by re-declaring).
  - Preferred for public API shapes and object contracts.
- type
  - Can alias unions, intersections, primitives, tuples — more flexible.
  - Cannot be re-opened (no merging).
  - Preferred for complex unions/compositions or function/type aliases.

Quick example:

```ts
type Shape =
  | { kind: "circle"; r: number }
  | { kind: "rect"; w: number; h: number };

function area(s: Shape) {
  if (s.kind === "circle") return Math.PI * s.r * s.r;
  return s.w * s.h;
}
```

Cheat:

- Use `interface` for objects and extensible API contracts.
- Use `type` for unions, intersections, and non-object aliases.
- Prefer explicit object shapes and readonly where appropriate to avoid accidental mutations.



---

### Union & Intersection Types (Easy & Examples)

- Union (A | B): value may be A or B. You must narrow before using type-specific members.
- Intersection (A & B): value is both A and B (combines properties).

#### Union — basics (string | number)

```ts
let id: string | number;
id = 42;
id = "abc";

// Can't use string methods directly — must narrow
// id.toUpperCase(); // Error: Object is not a string

if (typeof id === "string") {
  console.log(id.toUpperCase()); // safe after narrowing
} else {
  console.log(id.toFixed(0)); // safe as number
}
```

- Arrays vs union arrays:

```ts
const mixed: (string | number)[] = [1, "a", 2]; // array of union
const either: string[] | number[] = Math.random() > 0.5 ? ["a"] : [1];

// either.push("b"); // Error: either might be number[]
```

- Literal unions and discriminated unions:

```ts
type Shape =
  | { kind: "circle"; r: number }
  | { kind: "rect"; w: number; h: number };

function area(s: Shape) {
  if (s.kind === "circle") return Math.PI * s.r * s.r;
  return s.w * s.h; // safe: discriminated union narrowed by kind
}

console.log(area({ kind: "circle", r: 2 })); // 12.566...
```

- Exhaustive checks (helps future-proofing):

```ts
function assertNever(x: never): never {
  throw new Error("Unexpected: " + x);
}

function areaSwitch(s: Shape) {
  switch (s.kind) {
    case "circle":
      return Math.PI * s.r * s.r;
    case "rect":
      return s.w * s.h;
    default:
      return assertNever(s); // compile-time error if a case is missing
  }
}
```

#### Intersection — basics (A & B)

```ts
type A = { a: number };
type B = { b: string };
type C = A & B; // { a: number; b: string }

const obj: C = { a: 1, b: "x" };
console.log(obj.a, obj.b); // 1 "x"
```

- Intersection with unions (distributive effect):

```ts
type U1 = { x: number } | { y: string };
type U2 = { z: boolean };
type I = U1 & U2;
// I is { x:number; z:boolean } | { y:string; z:boolean }

const v1: I = { x: 1, z: true }; // valid
const v2: I = { y: "h", z: false }; // valid
```

- Practical use: mixin / augmentation of objects

```ts
type WithId = { id: string };
type Timestamp = { createdAt: Date };
type Entity = WithId & Timestamp & { name: string };

const e: Entity = { id: "1", createdAt: new Date(), name: "Item" };
console.log(e.id, e.createdAt, e.name);
```

#### Type Guards & Custom Narrowing

```ts
function isString(x: unknown): x is string {
  return typeof x === "string";
}

function process(u: string | number) {
  if (isString(u)) {
    console.log("string length", u.length); // safe
  } else {
    console.log("number", u.toFixed(2)); // safe
  }
}
```

Cheat summary:

- Use unions (|) to accept multiple possible types — narrow before using type-specific members.
- Use intersections (&) to combine multiple type shapes (useful for mixins/augmentation).
- Prefer discriminated unions (common literal key) for safe branching and exhaustive checks.
- For arrays, (A | B)[] is different from A[] | B[] — choose based on intent.



---

## Enums & Type Assertions (Easy & Examples)

Short notes with examples, outputs, and quick tips.

### Enums

- Numeric enums (default numeric values, reverse mapping exists):

```ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

console.log(Direction.Up); // 0
console.log(Direction.Down); // 1
console.log(Direction[0]); // "Up"  (reverse mapping)
```

- String enums (no reverse mapping, safer for logs/serialization):

```ts
enum Status {
  Active = "ACTIVE",
  Pending = "PENDING",
  Done = "DONE",
}

console.log(Status.Active); // "ACTIVE"
console.log(Status["ACTIVE"]); // undefined (no reverse mapping)
```

- const enums (inlined at compile time — smaller output, no object generated):

```ts
const enum Role {
  Admin,
  User,
  Guest,
}
const r = Role.User;
console.log(r); // 1
```

- When to prefer enums vs unions:
  - Enums useful for named constants (especially with shared runtime values).
  - Literal unions (type Color = 'red' | 'blue') are lightweight and fully type-only (no runtime object).

Example using enums in function signatures:

```ts
function move(dir: Direction) {
  if (dir === Direction.Up) console.log("move up");
}
move(Direction.Left); // "move up" not logged
```

---

### Type Assertions (as keyword and angle-bracket syntax)

- Type assertions tell the compiler "treat this value as this type" — no runtime check.
- Use when you know more than TypeScript's inference.

```ts
let input: unknown = "hello";
const s = input as string; // 'as' keyword
console.log(s.length); // 5

// angle-bracket style (not allowed in TSX/JSX)
const s2 = (<string>input).toUpperCase();
console.log(s2); // "HELLO"
```

- Note: assertions do not convert or validate at runtime. They only affect compile-time type checking.

- Use 'as' in TSX/React to avoid JSX/angle-bracket conflicts:

```tsx
const el = document.getElementById("root") as HTMLDivElement;
el.style.backgroundColor = "pink";
```

---

### Non-null assertion operator (!)

- Use the non-null assertion to tell TypeScript an expression is not null or undefined:
- Syntactic form `value!` — suppresses null/undefined checks for that value.

```ts
const app = document.getElementById("app")!; // assertion: not null
app.textContent = "Ready"; // OK for TS, but runtime will throw if null
```

- Caution: if the value is actually null at runtime, you'll get a runtime error. Prefer explicit checks:

```ts
const maybe = document.getElementById("app");
if (maybe) {
  maybe.textContent = "Safe";
} else {
  console.warn("No #app element");
}
```

---

### Quick Tips

- Enums create runtime values — use them when you need both types and runtime constants.
- Prefer string enums when you need clear serialized values.
- Use literal unions when you don't need runtime objects.
- Type assertions are compile-time only — avoid them to mask real typing issues.
- Non-null assertion (!) is handy but risky — prefer runtime checks or strict types if possible.



---

## Generics

Short, easy notes and examples to remember: generic functions, generic classes, and constraints.

### Generic functions

- Generics let you write reusable, type-safe functions.

```ts
// identity: returns same type that was passed
function identity<T>(arg: T): T {
  return arg;
}

console.log(identity(5)); // 5
console.log(identity<string>("hi")); // "hi"

// multiple generics & tuple swap
function swap<T, U>([a, b]: [T, U]): [U, T] {
  return [b, a];
}
console.log(swap([1, "a"])); // ["a", 1]

// constraint with extends (must have length)
function logLength<T extends { length: number }>(x: T): T {
  console.log(x.length);
  return x;
}
logLength("hello"); // 5
logLength([1, 2, 3]); // 3
// logLength(123);        // Error: number doesn't have length

// keyof + property access
function pluck<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
const person = { name: "Alice", age: 30 };
console.log(pluck(person, "name")); // "Alice"
```

### Generic classes

- Parameterize classes for type-safe containers.

```ts
class Stack<T> {
  private items: T[] = [];
  push(v: T) {
    this.items.push(v);
  }
  pop(): T | undefined {
    return this.items.pop();
  }
  size(): number {
    return this.items.length;
  }
}

const s = new Stack<number>();
s.push(1);
s.push(2);
console.log(s.pop()); // 2
console.log(s.size()); // 1

// Constraint example: items must have id
type WithId = { id: string };
class Repo<T extends WithId> {
  private store = new Map<string, T>();
  save(item: T) {
    this.store.set(item.id, item);
  }
  get(id: string): T | undefined {
    return this.store.get(id);
  }
}
const repo = new Repo<{ id: string; name: string }>();
repo.save({ id: "1", name: "X" });
console.log(repo.get("1")?.name); // "X"
```

### Generic utilities & defaults

- Merge objects with generics + intersection
- Default generic value

```ts
function merge<T extends object, U extends object>(a: T, b: U): T & U {
  return { ...a, ...b };
}
console.log(merge({ a: 1 }, { b: 2 })); // { a:1, b:2 }

// default generic
function createArray<T = number>(len: number, v: T): T[] {
  return Array.from({ length: len }, () => v);
}
console.log(createArray(3, 0)); // [0,0,0]  (T inferred as number)
console.log(createArray<string>(2, "x")); // ["x","x"]
```

### Notes & tips

- Type parameters are erased at runtime (type erasure).
- Prefer constraints (extends) to narrow acceptable types and access properties safely.
- Use keyof and discriminated unions to enable safe access and exhaustive checks.
- Combine generics with interfaces/classes for reusable typed APIs.



---

## Modules

- Named export:

```ts
// src/user.ts
export interface User {
  id: string;
  name: string;
}
export function greet(u: User) {
  return `Hi ${u.name}`;
}
```

- Default export:

```ts
// src/logger.ts
export default function log(msg: string) {
  console.log("[LOG]", msg);
}
```

- Importing:

```ts
import log from "./logger";
import { greet, type User } from "./user"; // "type" import is erased at runtime

const u: User = { id: "1", name: "Alice" };
log(greet(u)); // [LOG] Hi Alice
```

- Re-export:

```ts
// src/index.ts
export * from "./user";
export { default as log } from "./logger";
```

- Dynamic import (runtime, Promise-based):

```ts
async function loadAndRun() {
  const mod = await import("./heavy"); // code-splitting / lazy-load
  mod.heavyTask();
}
```

Notes:

- Use `import type { ... }` to import only types (no runtime code).
- Use named exports for tree-shaking, default for single primary export.

---

## Type Narrowing

- typeof (primitives)

```ts
function format(x: string | number) {
  if (typeof x === "string") return x.toUpperCase(); // narrowed to string
  return x.toFixed(2); // narrowed to number
}
```

- instanceof (classes)

```ts
class A {
  a = 1;
}
class B {
  b = 2;
}
function check(x: A | B) {
  if (x instanceof A) return x.a; // safe: A
  return x.b; // safe: B
}
```

- in operator (object unions)

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(a: Fish | Bird) {
  if ("swim" in a) a.swim(); // narrowed to Fish
  else a.fly(); // narrowed to Bird
}
```

- Custom type guard (user-defined)

```ts
type Person = { name: string; age?: number };

function isPerson(x: any): x is Person {
  return typeof x === "object" && x !== null && "name" in x;
}

function welcome(x: unknown) {
  if (isPerson(x)) {
    console.log("Hello", x.name); // x typed as Person here
  } else {
    console.log("Not a person");
  }
}
```

- Discriminated unions (recommended)

```ts
type Shape =
  | { kind: "circle"; r: number }
  | { kind: "rect"; w: number; h: number };

function area(s: Shape) {
  if (s.kind === "circle") return Math.PI * s.r * s.r; // narrowed
  return s.w * s.h;
}
```

---

## Utility Types

- Partial<T> — all properties optional:

```ts
interface User {
  id: string;
  name: string;
  age?: number;
}
function updateUser(u: User, patch: Partial<User>) {
  return { ...u, ...patch };
}
const u1 = updateUser({ id: "1", name: "A" }, { name: "B" });
// u1: { id: '1', name: 'B' }
```

- Pick<T, K> — subset of properties:

```ts
type NameOnly = Pick<User, "name">;
const n: NameOnly = { name: "Alice" };
```

- Omit<T, K> — remove keys:

```ts
type WithoutAge = Omit<User, "age">;
const u2: WithoutAge = { id: "2", name: "Bob" };
```

- Readonly<T> — all fields readonly:

```ts
const readonlyUser: Readonly<User> = { id: "1", name: "X" };
// readonlyUser.name = 'Y'; // Error: cannot assign to readonly property
```

- Record<K extends keyof any, T> — map keys to a type:

```ts
type Scores = Record<"alice" | "bob", number>;
const scores: Scores = { alice: 10, bob: 8 };
```

Quick extras:

- Required<T> — all properties required.
- ReturnType<T>, Parameters<T>, Exclude/Extract, NonNullable<T> are useful built-in utility types.



---

## Advanced Generics

Short, practical examples to memorize: default generic types, conditional types, and mapped types.

### Default generic types

- Provide a default for a generic parameter when the caller doesn't supply one.

```ts
function createArray<T = number>(len: number, v: T): T[] {
  return Array.from({ length: len }, () => v);
}

console.log(createArray(3, 0)); // [0,0,0]  (T inferred as number)
console.log(createArray<string>(2, "x")); // ["x","x"]

class Box<T = unknown> {
  constructor(public value: T) {}
}
const b1 = new Box(5); // Box<number> inferred
const b2 = new Box<string>("hi"); // Box<string>
```

Tip: defaults keep generics ergonomic when a typical type is common.

---

### Conditional types (T extends ? :)

- Conditional types select one type or another based on a condition. They are evaluated at compile time and can be distributive across unions.

```ts
// simple check
type IsString<T> = T extends string ? true : false;
type A = IsString<"a">; // true
type B = IsString<42>; // false

// infer element type from arrays or pass-through
type ElementType<T> = T extends (infer U)[] ? U : T;
type E1 = ElementType<number[]>; // number
type E2 = ElementType<string>; // string

// distributive behavior: applied to unions
type Wrapped<T> = T extends any ? T[] : never;
type W = Wrapped<number | string>; // number[] | string[]

// utility-style example: NonNull (similar to built-in NonNullable)
type NonNull<T> = T extends null | undefined ? never : T;
type NN = NonNull<string | null | undefined>; // string
```

Caveat: conditional types distribute over unions by default unless wrapped in a tuple type like [T] to prevent distribution.

---

### Mapped types

- Transform each property in a type using keyof and a mapping expression.

```ts
type User = { id: string; name: string; age?: number };

// make all properties optional (manual Partial)
type MyPartial<T> = { [K in keyof T]?: T[K] };
type PartialUser = MyPartial<User>; // { id?: string; name?: string; age?: number }

// make all properties readonly
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };
type ReadonlyUser = MyReadonly<User>;

// add getter methods (key remapping with `as` and string manipulation)
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};
type UserGetters = Getters<User>;
// UserGetters -> { getId: () => string; getName: () => string; getAge?: () => number }

// conditional in mapped types: apply rule only to string props
type NullableStrings<T> = {
  [K in keyof T]: T[K] extends string ? T[K] | null : T[K];
};
type NullableName = NullableStrings<User>; // name: string | null; id: string; age?: number

// recursive mapped type (DeepPartial)
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};
type DeepUser = DeepPartial<{ profile: { bio: string; tags: string[] } }>;
// DeepUser -> { profile?: { bio?: string; tags?: string[] } }
```

Notes:

- Use mapped types to automate repetitive transformations (Partial, Readonly, Pick are mapped-type patterns).
- Combine with built-in utilities (keyof, as, conditional types, infer) to build powerful, type-safe helpers.

---

Cheat summary:

- Default generics: provide a sensible default type (T = X).
- Conditional types: T extends U ? X : Y; watch distribution over unions.
- Mapped types: [K in keyof T] to transform shape; use `as` to remap names and conditional logic for selective transformations.



---

## Advanced Interfaces & Decorators (Easy & Examples)

Short, clear explanations and small examples to memorize.

### Extending interfaces

- An interface can extend another to add more fields or behaviors.

```ts
interface Animal {
  name: string;
  speak(): string;
}

interface Dog extends Animal {
  breed: string;
  bark(): string;
}

class MyDog implements Dog {
  constructor(public name: string, public breed: string) {}
  speak() {
    return `${this.name} makes a sound`;
  }
  bark() {
    return `${this.name} barks: Woof!`;
  }
}

const d: Dog = new MyDog("Rex", "Beagle");
console.log(d.speak()); // Rex makes a sound
console.log(d.bark()); // Rex barks: Woof!
```

Cheat: "extends" merges assumptions so Dog includes Animal fields.

---

### Hybrid types

- A hybrid type is callable (function) but also has properties. Useful for counters, factories.

```ts
interface Counter {
  (start: number): string; // callable signature
  count: number; // property
  reset(): void; // method
}

function createCounter(): Counter {
  const f = ((start: number) => `started at ${start}`) as Counter;
  f.count = 0;
  f.reset = () => {
    f.count = 0;
  };
  return f;
}

const c = createCounter();
console.log(c(5)); // started at 5
c.count += 2;
console.log(c.count); // 2
c.reset();
console.log(c.count); // 0
```

Cheat: cast function to Counter and attach properties to it.

---

### Decorators (experimental)

- Enable in tsconfig.json:

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

- Decorators let you annotate/modify classes, methods, accessors, or properties at design time.

#### Class decorator (add or change constructor behavior)

```ts
function addTimestamp<T extends { new (...args: any[]): {} }>(ctor: T) {
  return class extends ctor {
    createdAt = new Date();
  };
}

@addTimestamp
class User {
  constructor(public name: string) {}
}

const u = new User("Alice") as any;
console.log(u.name, u.createdAt); // Alice <Date>
```

Cheat: class decorator can return a new constructor to extend behavior.

#### Method decorator (wrap method to log arguments/results)

```ts
function logMethod(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${key} with`, args);
    const res = original.apply(this, args);
    console.log(`Result from ${key}:`, res);
    return res;
  };
  return descriptor;
}

class Calc {
  @logMethod
  add(a: number, b: number) {
    return a + b;
  }
}

const calc = new Calc();
console.log(calc.add(2, 3));
// Console:
// Calling add with [2, 3]
// Result from add: 5
// 5
```

Cheat: method decorator receives target, method name, descriptor — you can wrap and replace descriptor.value.

#### Property decorator (metadata or modify property behavior)

```ts
function nonEnumerable(target: any, prop: string) {
  // Set the property on the prototype as non-enumerable
  Object.defineProperty(target, prop, {
    enumerable: false,
    configurable: true,
    writable: true,
  });
}

class Product {
  @nonEnumerable
  secret = "hidden";

  public name: string;
  constructor(name: string) {
    this.name = name;
  }
}

const p = new Product("Widget") as any;
console.log(p.secret); // hidden
console.log(Object.keys(p)); // ["name"] -> secret not shown (non-enumerable)
```

Notes:

- Property decorators are applied to the prototype; behavior can differ between prototype property vs instance property. For complex runtime metadata or injection use reflect-metadata.
- For true runtime enforcement (readonly, validation), consider using accessors or initialization code that runs in the constructor.

---

### Practical tips & safety

- Decorators are experimental: enable "experimentalDecorators" in tsconfig.
- Use class decorators for cross-cutting concerns (logging, DI, adding defaults).
- Use method decorators to add instrumentation, retries, caching, or logging.
- Be careful with property decorators — they can be subtle and are limited compared to method/accessor decorators.
- Prefer readable, testable patterns; keep decorators small and focused.

Cheat summary:

- Extending interfaces = combine shapes.
- Hybrid types = callable + properties (cast function and add props).
- Class/method/property decorators = annotate and modify behavior at runtime (enable experimentalDecorators).


---

## Namespaces, Declaration Files & Type System Basics (Easy & Examples)

### Namespaces

- Group related types/values when you want a single global script (older pattern). For modern projects prefer ES Modules.

```ts
namespace Utils {
  export function log(msg: string) {
    console.log(msg);
  }
  export interface Person {
    name: string;
    age?: number;
  }
}

const p: Utils.Person = { name: "Alice" };
Utils.log(p.name); // Console: Alice
```

Tip: avoid `namespace` for module-based projects; use `export`/`import` instead.

---

### Declaration files (.d.ts) & writing custom types

- Use .d.ts to describe the types of untyped JavaScript libraries or global script APIs.

Example: declare a module for a JS lib

```ts
// filepath: types/legacy-lib.d.ts (example)
declare module "legacy-lib" {
  export function compute(x: number): number;
  export const VERSION: string;
  export default function run(conf?: { debug?: boolean }): void;
}
```

- For global script augmentations:

```ts
// global.d.ts
declare global {
  interface Window {
    MY_APP?: string;
  }
}
export {};
```

Notes:

- Put .d.ts under a "types" folder or configure "typeRoots" in tsconfig.json.
- Use `declare module "..."` for package-level types.

---

### Type Inference & Contextual Typing

- TypeScript infers types so you often don’t need explicit annotations; contextual typing narrows types using expected type information.

```ts
let n = 10; // inferred as number
const s = "hi"; // inferred as "hi" (literal) with const

function add(a: number, b: number) {
  return a + b;
} // return inferred number

// contextual typing: map callback gets inferred element type
const nums = [1, 2, 3];
const strs = nums.map((n) => n.toString()); // n inferred as number

type Handler = (e: { value: number }) => void;
const h: Handler = (e) => console.log(e.value); // e is typed from Handler (contextual)
```

Tips:

- Inferred return types are usually fine; annotate public API functions for clarity and stable contracts.

---

### Return Type Inference & When to Annotate

- Inference is helpful, but public functions and library APIs benefit from explicit return types.

```ts
// inferred return type: { ok: boolean }
function check(n: number) {
  return { ok: n > 0 };
}

// explicit annotation recommended in public API
export function parseAndDouble(s: string): number {
  const n = Number(s);
  return n * 2;
}
```

---

## Advanced Utility Types

### Exclude<T, U>

- Removes union members that are assignable to U.

```ts
type Letter = "a" | "b" | "c";
type WithoutB = Exclude<Letter, "b">; // "a" | "c"
```

### Extract<T, U>

- Keeps only members of T assignable to U.

```ts
type Some = Extract<"a" | "b" | "c", "b" | "c">; // "b" | "c"
```

### NonNullable<T>

- Removes null and undefined from T.

```ts
type Maybe = string | null | undefined;
type NotNull = NonNullable<Maybe>; // string
```

### ReturnType<T>

- Gets the return type of a function type.

```ts
function mk(n: number) {
  return { id: `${n}`, value: n * 2 };
}
type R = ReturnType<typeof mk>; // { id: string; value: number }
```

### InstanceType<T>

- Gets the instance type from a constructor type.

```ts
class User {
  constructor(public name: string) {}
}
type UserInstance = InstanceType<typeof User>; // User
const u: UserInstance = new User("Alice");
console.log(u.name); // "Alice"
```

---

## Common Patterns with Types & Utilities

- Use Exclude/Extract for safe union transformations and narrowing.
- Use NonNullable to remove nullable variants for internal safe handling.
- ReturnType + InstanceType are handy when building factory types or wrapper APIs.
- Prefer inference for local functions; annotate exports/public APIs for stable interfaces.

---

## Quick Example: Combining Utilities

```ts
type ReqUser = { id: string; name: string; age?: number };
type PublicUser = Omit<ReqUser, "age">; // remove private field
type NullableUser = ReqUser | null;

type NonNullUser = NonNullable<NullableUser>; // ReqUser

type UserFactory = (name: string) => ReqUser;
type UserFactoryReturn = ReturnType<UserFactory>; // ReqUser
```

Cheat-sheet:

- `declare module` / `.d.ts` to add typings for JS packages or global objects.
- `namespace` for global grouping (legacy); use modules for modern code.
- Let TypeScript infer locally; annotate public exports.
- Utility types: Exclude, Extract, NonNullable, ReturnType, InstanceType for flexible, safe transforms.



---

## Type System Mastery

Concise notes and examples for advanced TypeScript features.

### Distributive conditional types

- Conditional types distribute over unions by default.

```ts
type Box<T> = T extends any ? { value: T } : never;
type Boxed = Box<number | string>; // { value: number } | { value: string }

// Prevent distribution by wrapping in tuple
type NoDist<T> = [T] extends [any] ? { value: T } : never;
type NotDistributed = NoDist<number | string>; // { value: number | string }
```

- Example usage: mapping union members to separate shapes or keeping them combined as one type.

---

### Template literal types

- Compose string-like types at compile-time.

```ts
type Role = "user" | "admin";
type Event = `${Role}:created` | `${Role}:deleted`; // 'user:created' | 'admin:created' | 'user:deleted' | 'admin:deleted'

type HandlerName<T extends string> = `on${Capitalize<T>}`;
type Handlers = {
  [K in HandlerName<"click" | "input">]?: () => void;
};
// Equivalent keys: onClick, onInput

const h: Handlers = { onClick: () => console.log("click") };
h.onClick?.(); // prints "click"
```

---

### Recursive types

- Create deep transforms for nested objects / arrays.

```ts
type DeepReadonly<T> = T extends Function
  ? T
  : T extends Array<infer U>
  ? ReadonlyArray<DeepReadonly<U>>
  : T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;

// Example
type Nested = { a: { b: number[] } };
const obj: DeepReadonly<Nested> = { a: { b: [1, 2] } };
// obj.a = {}; // Error: readonly
// obj.a.b.push(3); // Error: push does not exist on readonly array
```

Cheat: Recursive mapped & conditional types let you express deep, structural constraints.

---

## Advanced Patterns

### Nominal typing (branding types)

- TypeScript uses structural typing; nominal typing can be emulated with branding.

```ts
type Brand<K, T> = K & { readonly __brand: T };

type UserId = Brand<string, "UserId">;
const mkUserId = (s: string) => s as UserId;

function getUser(id: UserId) {
  /* ... */
}

const uid = mkUserId("user-1");
getUser(uid); // ok
// getUser('user-1');   // Error: string is not assignable to UserId
```

Tip: branding prevents mixing different ID string types while still being runtime `string`.

---

### Type-safe builders

- Track required fields via generics; build only once all required fields are set.

```ts
type User = { name: string; age: number };

class UserBuilder<T extends Partial<User> = {}> {
  private data = {} as T;

  setName(name: string): UserBuilder<T & { name: string }> {
    (this.data as any).name = name;
    return this as any;
  }
  setAge(age: number): UserBuilder<T & { age: number }> {
    (this.data as any).age = age;
    return this as any;
  }

  build(): T extends User ? T : never {
    return this.data as any;
  }
}

// Usage:
const ok = new UserBuilder().setName("Alice").setAge(30).build(); // type is User
// const bad = new UserBuilder().setName('Alice').build(); // build() returns never -> compile-time error if you assign to User
```

- The build method's conditional return uses `T extends User ? T : never` to enforce required fields be present.

---

### Fluent APIs with generics

- Collect data safely in chainable APIs; typed `build()` reflects accumulated keys.

```ts
class ConfigBuilder<T extends Record<string, any> = {}> {
  private cfg = {} as T;
  set<K extends string, V>(key: K, value: V): ConfigBuilder<T & Record<K, V>> {
    (this.cfg as any)[key] = value;
    return this as any;
  }
  build(): T {
    return this.cfg;
  }
}

// Usage:
const c = new ConfigBuilder()
  .set("host", "localhost")
  .set("port", 8080)
  .build();
// 'c' inferred as { host: string; port: number }
```

- Fluent APIs + generics allow compile-time guarantees about the final object's shape.

---

Cheat summary:

- Distributive conditional types convert unions into mapped unions; wrap type in tuple to prevent.
- Template literal types let you compute string-literal types and key names.
- Recursive types model deep transforms (readonly, partial) across nested objects/arrays.
- Branding emulates nominal types; generic builders & fluent APIs enforce correctness using type-state generics.

# Advanced TypeScript Integration & Performance Guide

> A comprehensive guide to integrating TypeScript with modern stacks, ensuring type safety across boundaries, and optimizing for scale.

## 📋 Table of Contents

- [Integration](#-integration)
  - [React with TypeScript](#react-with-typescript)
  - [Node.js & Express](#nodejs--express)
- [Type-Safe APIs](#-type-safe-apis)
  - [REST (Zod / OpenAPI)](#rest-apis)
  - [GraphQL (Codegen)](#graphql-apis)
- [Performance & Scalability](#-performance--scalability)
  - [Large-scale tsconfig Optimization](#large-scale-tsconfig-optimization)
  - [Incremental Builds](#incremental-builds)
  - [Project References](#project-references)
- [Testing with TypeScript](#-testing-with-typescript)
  - [Type-safe Unit Tests (Jest + TS)](#type-safe-unit-tests)
  - [Mocking with Types](#mocking-with-types)

---

## 🔌 Integration

### React with TypeScript

Best practices for typing components, hooks, and contexts.

```tsx
// Example: Strongly typed functional component with Props
import React, { ReactNode } from "react";

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary";
  children: ReactNode;
}

export const Button = ({
  variant = "primary",
  children,
  ...props
}: ButtonProps) => {
  return (
    <button className={`btn-${variant}`} {...props}>
      {children}
    </button>
  );
};
```

### Node.js & Express

Best practices for typing server-side code with Express.

```ts
// Example: Typed Express router
import { Router, Request, Response } from "express";

const router = Router();

router.get("/users", (req: Request, res: Response) => {
  res.json({ users: [] });
});

export default router;
```
