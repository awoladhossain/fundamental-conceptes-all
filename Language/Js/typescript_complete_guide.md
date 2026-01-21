# Complete TypeScript Guide: Junior to Senior Level
## With Detailed Explanations & Real-World Examples

---

## Table of Contents
1. [What is TypeScript?](#what-is-typescript)
2. [Basic Concepts](#basic-concepts)
3. [Type System](#type-system)
4. [Interfaces & Types](#interfaces--types)
5. [Classes & OOP](#classes--oop)
6. [Generics](#generics)
7. [Advanced Types](#advanced-types)
8. [Decorators](#decorators)
9. [Modules](#modules)
10. [Utility Types](#utility-types)
11. [Error Handling](#error-handling)
12. [Design Patterns](#design-patterns)

---

## WHAT IS TYPESCRIPT?

### Definition
TypeScript is a **superset of JavaScript** that adds **static type checking** before code execution. It means TypeScript takes JavaScript and adds a layer that checks data types at compile time (before running) instead of runtime (while running). This helps catch bugs early.

### Why TypeScript?
- **Catches errors before runtime**: If you try to call a string method on a number, TypeScript will tell you immediately
- **Better IDE support**: Your editor can autocomplete and provide suggestions
- **Self-documenting code**: Types act as documentation
- **Refactoring safety**: When you change code, TypeScript tells you what breaks
- **Large project organization**: Type system helps scale projects

### How it works
```typescript
// TypeScript code
let name: string = "John";
name = 123; // ERROR! Cannot assign number to string

// Compiles to JavaScript (only this runs in browser/Node.js)
var name = "John";
name = 123; // Works fine in plain JavaScript
```

### Real-world analogy
Think of TypeScript as a **spell checker for code**. JavaScript is like writing without checking - it might work or crash. TypeScript checks before you publish to catch mistakes.

---

## BASIC CONCEPTS

### What is a Type?
**Definition**: A type is a **description of what kind of data something is**. It tells TypeScript what values are allowed.

Think of it like a label on a box:
- A box labeled "String" can only hold text
- A box labeled "Number" can only hold numbers
- A box labeled "Boolean" can only hold true/false

```typescript
// These are type annotations - telling TypeScript what type each variable is

let age: number = 25;        // Can only store numbers
let name: string = "John";   // Can only store text
let isActive: boolean = true; // Can only store true or false

// ❌ These would cause errors:
age = "twenty-five";  // ERROR! age is a number, not string
name = 123;           // ERROR! name is string, not number
isActive = "yes";     // ERROR! isActive is boolean, not string
```

### Type Inference
**Definition**: **Type inference** means TypeScript automatically figures out the type without you explicitly writing it.

```typescript
// TypeScript automatically knows 'count' is a number
let count = 5;
count = "five"; // ERROR! TypeScript inferred it's a number

// TypeScript automatically knows 'message' is a string
let message = "Hello";
message = 42; // ERROR! TypeScript inferred it's a string

// You can see what TypeScript inferred by hovering in your editor
let price = 29.99; // TypeScript infers this as 'number'
let active = true; // TypeScript infers this as 'boolean'
```

### Why Type Safety Matters
```typescript
// JavaScript (dangerous) - no types
function calculateTotal(price, quantity) {
  return price * quantity;
}

calculateTotal("10", "5"); // Returns "10" repeated 5 times (NaN), not 50!

// TypeScript (safe) - with types
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}

calculateTotal("10", "5"); // ERROR! Must pass numbers, not strings
calculateTotal(10, 5); // Returns 50 ✓
```

---

## TYPE SYSTEM

### Primitive Types Explained

#### String
**Definition**: Text data enclosed in quotes (single, double, or backticks).

```typescript
let firstName: string = "John";
let lastName: string = 'Doe';
let fullName: string = `${firstName} ${lastName}`; // Template literals

// Real-world example: User email validation
let userEmail: string = "john@example.com";
function validateEmail(email: string): boolean {
  return email.includes("@");
}
```

#### Number
**Definition**: Any numeric value including integers, decimals, positive, negative, Infinity, NaN.

```typescript
let age: number = 25;
let price: number = 19.99;
let temperature: number = -5;
let infinity: number = Infinity;

// Real-world example: Calculate age from birthdate
function getAge(birthYear: number): number {
  return new Date().getFullYear() - birthYear;
}

const johnAge = getAge(1995); // Returns 29 (in 2024)
```

#### Boolean
**Definition**: True or false values used for conditional logic.

```typescript
let isLoggedIn: boolean = true;
let hasPermission: boolean = false;

// Real-world example: User authentication
class User {
  private isVerified: boolean = false;

  verify(): void {
    this.isVerified = true;
  }

  canAccess(): boolean {
    return this.isVerified;
  }
}
```

#### Null & Undefined
**Definition**:
- **null** = intentional absence of value
- **undefined** = variable declared but no value assigned

```typescript
let emptyValue: null = null;        // Intentionally empty
let notDefined: undefined;           // Declared but not initialized

// Real-world example: API response
interface ApiResponse {
  data: string | null;     // Might be null
  error: string | undefined; // Might not exist
}

const response: ApiResponse = {
  data: null,
  error: undefined
};
```

### Union Types
**Definition**: A type that can be **one of multiple types**. Use the pipe `|` symbol.

Think of it like saying: "This variable can be EITHER a string OR a number, but not both."

```typescript
// This variable can be string or number
let age: string | number;

age = 25;      // ✓ OK - it's a number
age = "25";    // ✓ OK - it's a string
age = true;    // ❌ ERROR - boolean not allowed

// Real-world example: API status response
type ApiStatus = "loading" | "success" | "error";

function handleApiResponse(status: ApiStatus): void {
  if (status === "loading") {
    console.log("Fetching data...");
  } else if (status === "success") {
    console.log("Data loaded!");
  } else if (status === "error") {
    console.log("Something went wrong!");
  }
}

handleApiResponse("loading"); // ✓ OK
handleApiResponse("pending"); // ❌ ERROR - not a valid status
```

### Type Narrowing
**Definition**: The process of **refining a variable's type** inside a code block. When you have a union type, narrowing helps TypeScript understand which specific type you're working with.

```typescript
// Before narrowing - could be string or number
function processValue(value: string | number) {
  // TypeScript doesn't know if it's string or number yet
  // value.toUpperCase(); // ❌ ERROR! numbers don't have toUpperCase()

  // After narrowing - we check the type
  if (typeof value === "string") {
    // Inside this block, TypeScript knows it's a string
    console.log(value.toUpperCase()); // ✓ OK - strings have toUpperCase()
  } else {
    // Inside this block, TypeScript knows it's a number
    console.log(value * 2); // ✓ OK - numbers can be multiplied
  }
}

// Real-world example: Handle both API response and error
function handleResponse(response: string | { error: string }) {
  if (typeof response === "string") {
    console.log("Success:", response);
  } else {
    console.log("Error:", response.error);
  }
}
```

### Literal Types
**Definition**: A type that can **only be one specific value**, not just any value of that type.

```typescript
// Regular number type - can be any number
let regularNumber: number = 5;
regularNumber = 10; // ✓ OK
regularNumber = -5; // ✓ OK

// Literal type - can ONLY be exactly 5
let literalNumber: 5 = 5;
literalNumber = 10; // ❌ ERROR - must be exactly 5

// Useful with unions - restrict to specific choices
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

function makeRequest(method: HttpMethod): void {
  console.log(`Making ${method} request`);
}

makeRequest("GET"); // ✓ OK
makeRequest("POST"); // ✓ OK
makeRequest("PATCH"); // ❌ ERROR - PATCH not allowed

// Real-world example: Order status
type OrderStatus = "pending" | "shipped" | "delivered" | "cancelled";

class Order {
  status: OrderStatus = "pending";

  updateStatus(newStatus: OrderStatus): void {
    this.status = newStatus;
  }
}

const order = new Order();
order.updateStatus("shipped"); // ✓ OK
order.updateStatus("lost"); // ❌ ERROR - not a valid status
```

### Arrays
**Definition**: A collection of multiple values of the **same type**.

```typescript
// Array of numbers
let numbers: number[] = [1, 2, 3, 4, 5];

// Alternative syntax
let moreNumbers: Array<number> = [10, 20, 30];

// Array of strings
let names: string[] = ["John", "Jane", "Bob"];

// Array with union types (can contain strings OR numbers)
let mixed: (string | number)[] = [1, "two", 3, "four"];

// Multi-dimensional arrays
let matrix: number[][] = [[1, 2], [3, 4]];

// Real-world example: Shopping cart
interface Product {
  id: number;
  name: string;
  price: number;
}

let cart: Product[] = [
  { id: 1, name: "Laptop", price: 999 },
  { id: 2, name: "Mouse", price: 25 }
];

function calculateTotal(products: Product[]): number {
  return products.reduce((sum, product) => sum + product.price, 0);
}

const total = calculateTotal(cart); // 1024
```

### Tuples
**Definition**: An array with a **fixed length** where each position has a **specific type**.

Unlike regular arrays that can have any length, tuples have exactly the right number of elements with specific types.

```typescript
// Regular array - any length, all strings
let fruits: string[] = ["apple", "banana"];
fruits.push("orange"); // ✓ OK - can add more

// Tuple - exactly 2 elements: first is number, second is string
let coordinate: [number, string] = [10, "meters"];
coordinate.push("extra"); // ❌ ERROR - tuples have fixed length

// Real-world example: Latitude and Longitude
type GeoLocation = [latitude: number, longitude: number];

function getNearbyPlaces(location: GeoLocation): void {
  const [lat, lng] = location;
  console.log(`Finding places near ${lat}, ${lng}`);
}

getNearbyPlaces([40.7128, -74.0060]); // New York coordinates

// Tuples with optional elements
type ApiResponse = [status: number, message: string, data?: any];

const success: ApiResponse = [200, "OK"];
const withData: ApiResponse = [200, "OK", { userId: 1 }];

// Tuple with rest element (can have multiple of last type)
type StringNumberPair = [string, ...number[]];

const pair: StringNumberPair = ["IDs", 1, 2, 3, 4];
```

### Enums
**Definition**: An **enumeration** is a way to give names to a set of specific values. Instead of using magic strings/numbers, you use meaningful names.

It's like creating a special type with predefined options.

```typescript
// Numeric enum - values are 0, 1, 2...
enum Status {
  Active = 0,
  Inactive = 1,
  Pending = 2
}

let userStatus: Status = Status.Active;
console.log(userStatus); // 0

// String enum - more readable
enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE"
}

let userColor: Color = Color.Red;
console.log(userColor); // "RED"

// Enum with auto-increment
enum Priority {
  Low = 1,
  Medium = 2,    // Explicitly set
  High = 3
}

// Real-world example: User roles
enum UserRole {
  Admin = "ADMIN",
  Moderator = "MODERATOR",
  User = "USER"
}

class User {
  name: string;
  role: UserRole;

  constructor(name: string, role: UserRole) {
    this.name = name;
    this.role = role;
  }

  canDelete(): boolean {
    return this.role === UserRole.Admin;
  }
}

const admin = new User("John", UserRole.Admin);
const normalUser = new User("Jane", UserRole.User);

console.log(admin.canDelete()); // true
console.log(normalUser.canDelete()); // false

// Reverse mapping (numeric only)
enum Direction {
  North = 1,
  South = 2,
  East = 3,
  West = 4
}

console.log(Direction[1]); // "North" - you can look up the name by number
```

### Any & Unknown
**Definition**:
- **any** = TypeScript will NOT check the type (turns off type safety - USE WITH CAUTION)
- **unknown** = TypeScript WILL check before you use it (safer alternative)

```typescript
// any - turns off type checking (DON'T USE THIS!)
let anything: any = "hello";
anything = 123; // ✓ OK
anything.toUpperCase(); // ✓ No error even though it's a number now!
anything(); // ✓ No error

// unknown - safer, requires type checking
let unknownValue: unknown = "hello";
unknownValue = 123;
// unknownValue.toUpperCase(); // ❌ ERROR - must check type first

if (typeof unknownValue === "string") {
  console.log(unknownValue.toUpperCase()); // ✓ OK - TypeScript knows it's a string
}

// Real-world example: API response
interface ApiResponse {
  data: unknown; // Could be anything from server
}

function handleData(response: ApiResponse): void {
  // ❌ Wrong - TypeScript won't allow this
  // console.log(response.data.name);

  // ✓ Correct - check type first
  if (typeof response.data === "object" && response.data !== null) {
    console.log("Got an object:", response.data);
  }
}
```

### Never Type
**Definition**: Represents a type that **never occurs**. Used for functions that never return (always throw or infinite loop).

```typescript
// Function that always throws
function throwError(message: string): never {
  throw new Error(message);
}

// Function with infinite loop
function infiniteLoop(): never {
  while (true) {
    // never returns
  }
}

// Real-world example: Exhaustive checking
type UserRole = "admin" | "user" | "guest";

function getPermissions(role: UserRole): string[] {
  switch (role) {
    case "admin":
      return ["read", "write", "delete"];
    case "user":
      return ["read", "write"];
    case "guest":
      return ["read"];
    default:
      // If you forget a case, TypeScript catches it
      const exhaustive: never = role; // ERROR if case is missing
      return exhaustive;
  }
}
```

---

## INTERFACES & TYPES

### Type Aliases
**Definition**: A **name you give to a type**. It's like creating a variable for a type that you can reuse.

```typescript
// Simple type alias
type Age = number;
let myAge: Age = 25;

// Object type alias
type Person = {
  name: string;
  age: number;
  email?: string; // Optional property (can be undefined)
  readonly id: number; // Cannot be changed after creation
};

let person: Person = {
  name: "John",
  age: 30,
  id: 1
};

person.name = "Jane"; // ✓ OK - can change
// person.id = 2; // ❌ ERROR - id is readonly

// Function type alias
type Callback = (data: string) => void;

const handleData: Callback = (data) => {
  console.log("Received:", data);
};

// Real-world example: API request
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

type ApiRequest = {
  url: string;
  method: HttpMethod;
  headers?: Record<string, string>;
  body?: any;
};

function sendRequest(request: ApiRequest): void {
  console.log(`${request.method} ${request.url}`);
}

sendRequest({
  url: "/api/users",
  method: "POST",
  body: { name: "John" }
});
```

### Interfaces
**Definition**: A way to **describe the shape of an object**. Like a blueprint that says "any object with this interface must have these properties and methods."

Interfaces are similar to types but have some different rules (you'll learn later).

```typescript
// Basic interface
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // Optional
  readonly createdAt: Date; // Readonly
}

const user: User = {
  id: 1,
  name: "John",
  email: "john@example.com",
  createdAt: new Date()
};

// Real-world example: Database models
interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
  description?: string;
}

interface Order {
  id: number;
  userId: number;
  products: Product[];
  totalPrice: number;
  createdAt: Date;
}

// Function that accepts interface
function createOrder(order: Order): void {
  console.log(`Order #${order.id} created for user ${order.userId}`);
  console.log(`Total: $${order.totalPrice}`);
}
```

### Interface Inheritance
**Definition**: When an interface **extends** another interface, it inherits all its properties and can add new ones.

```typescript
// Base interface
interface Animal {
  name: string;
  age: number;
  makeSound(): void;
}

// Dog interface inherits from Animal
interface Dog extends Animal {
  breed: string;
  fetchBall(): void;
}

// Dog must have everything from Animal PLUS breed and fetchBall
const dog: Dog = {
  name: "Buddy",
  age: 5,
  breed: "Golden Retriever",
  makeSound() {
    console.log("Woof!");
  },
  fetchBall() {
    console.log("Fetching...");
  }
};

// Real-world example: User hierarchy
interface User {
  id: number;
  name: string;
  email: string;
}

interface Admin extends User {
  adminLevel: number;
  canDeleteUsers: boolean;
}

interface Moderator extends User {
  moderationLevel: number;
  canBanUsers: boolean;
}

const admin: Admin = {
  id: 1,
  name: "John",
  email: "john@example.com",
  adminLevel: 5,
  canDeleteUsers: true
};
```

### Multiple Interface Inheritance
**Definition**: An interface can inherit from **multiple interfaces** at once.

```typescript
interface Named {
  name: string;
}

interface Timed {
  createdAt: Date;
  updatedAt: Date;
}

interface Active {
  isActive: boolean;
}

// Post has properties from all three interfaces
interface Post extends Named, Timed, Active {
  content: string;
}

const post: Post = {
  name: "My First Post",
  createdAt: new Date(),
  updatedAt: new Date(),
  isActive: true,
  content: "Hello world!"
};
```

### Interface vs Type - When to Use What
**Definition**: Both describe types, but they have different purposes:

```typescript
// ❌ Type CANNOT do this (declaration merging)
type Bird = {
  fly(): void;
};
type Bird = { // ERROR! Cannot redeclare
  sing(): void;
};

// ✓ Interface CAN do this (declaration merging)
interface Bird {
  fly(): void;
}
interface Bird { // ✓ OK! Merges with previous
  sing(): void;
}

// Now Bird has both methods
const bird: Bird = {
  fly() { console.log("Flying"); },
  sing() { console.log("Singing"); }
};

// ✓ Type CAN do union (interface CANNOT)
type Status = "active" | "inactive" | "pending";

// ❌ Interface cannot use union
// interface Status = "active" | "inactive"; // ERROR!

// Recommendation:
// - Use 'interface' for object shapes (classes, API responses)
// - Use 'type' for unions, primitives, and complex types

// Example: Object shape -> use interface
interface UserProfile {
  username: string;
  bio: string;
}

// Example: Union type -> use type
type RequestStatus = "pending" | "success" | "error";
```

### Declaration Merging
**Definition**: When you define the same interface multiple times, TypeScript **combines them** into one.

```typescript
// First definition
interface Window {
  myCustomProperty: string;
}

// Second definition - MERGES with first
interface Window {
  myCustomMethod(): void;
}

// Result: Window now has both
const windowObj: Window = {
  myCustomProperty: "hello",
  myCustomMethod() {
    console.log("Custom method");
  }
};

// Real-world example: Extending built-in types
interface Window {
  myApp: {
    version: string;
    start(): void;
  };
}

// In your code
window.myApp.start();
```

---

## CLASSES & OOP

### Classes Introduction
**Definition**: A **class** is a blueprint for creating objects with properties and methods. It's like a template.

```typescript
class Car {
  // Properties
  brand: string;
  color: string;
  speed: number = 0;

  // Constructor - runs when you create a new instance
  constructor(brand: string, color: string) {
    this.brand = brand;
    this.color = color;
  }

  // Methods - functions inside the class
  accelerate(): void {
    this.speed += 10;
    console.log(`${this.brand} is going ${this.speed} km/h`);
  }

  brake(): void {
    this.speed = 0;
    console.log(`${this.brand} stopped`);
  }
}

// Creating instances (objects) from the class
const myCar = new Car("Toyota", "blue");
myCar.accelerate(); // Output: Toyota is going 10 km/h
myCar.accelerate(); // Output: Toyota is going 20 km/h
myCar.brake(); // Output: Toyota stopped

// Real-world example: Bank Account
class BankAccount {
  accountNumber: string;
  balance: number = 0;
  owner: string;

  constructor(accountNumber: string, owner: string) {
    this.accountNumber = accountNumber;
    this.owner = owner;
  }

  deposit(amount: number): void {
    if (amount > 0) {
      this.balance += amount;
      console.log(`Deposited $${amount}. New balance: $${this.balance}`);
    }
  }

  withdraw(amount: number): void {
    if (amount > 0 && amount <= this.balance) {
      this.balance -= amount;
      console.log(`Withdrew $${amount}. New balance: $${this.balance}`);
    } else {
      console.log("Insufficient funds!");
    }
  }

  getBalance(): number {
    return this.balance;
  }
}

const account = new BankAccount("ACC123", "John");
account.deposit(500); // Deposited $500. New balance: $500
account.withdraw(200); // Withdrew $200. New balance: $300
```

### Access Modifiers - Public, Private, Protected
**Definition**: Controls **who can access** a property or method.

- **public** = anyone can access (default)
- **private** = only this class can access
- **protected** = only this class and classes that inherit from it

```typescript
class User {
  // Public - anyone can access
  public name: string;

  // Private - only User class can access
  private password: string;

  // Protected - User and classes extending User can access
  protected email: string;

  constructor(name: string, password: string, email: string) {
    this.name = name;
    this.password = password;
    this.email = email;
  }

  // Public method - anyone can call
  public displayName(): void {
    console.log(`Name: ${this.name}`);
  }

  // Private method - only User can call
  private validatePassword(pwd: string): boolean {
    return pwd === this.password;
  }

  // Protected method - User and subclasses can call
  protected getEmail(): string {
    return this.email;
  }

  login(pwd: string): boolean {
    return this.validatePassword(pwd); // ✓ Can access own private method
  }
}

// Using the User class
const user = new User("John", "secret123", "john@example.com");

user.displayName(); // ✓ OK - public
user.name = "Jane"; // ✓ OK - public

// user.password = "hack"; // ❌ ERROR - private
// user.validatePassword("secret"); // ❌ ERROR - private
// user.getEmail(); // ❌ ERROR - protected

// Real-world example: Authentication system
class AuthService {
  private secretKey: string = "super-secret-key";
  protected tokenExpiry: number = 3600; // 1 hour
  public appName: string = "MyApp";

  private generateToken(): string {
    return `token-${Math.random()}`;
  }

  public authenticate(username: string, password: string): boolean {
    // Can access private method
    const token = this.generateToken();
    console.log(`User authenticated with token: ${token}`);
    return true;
  }
}

class AdminService extends AuthService {
  manageUsers(): void {
    // Can access protected but not private
    console.log(`Token expires in ${this.tokenExpiry} seconds`);
    // console.log(this.secretKey); // ❌ ERROR - cannot access private
  }
}
```

### Getters and Setters
**Definition**: Special methods that let you **control access** to properties. Getters retrieve values, setters change values.

Think of them as custom functions for reading/writing properties.

```typescript
class Person {
  private _age: number = 0; // underscore convention for private fields

  // Getter - like a property that calls a function
  get age(): number {
    return this._age;
  }

  // Setter - runs validation before changing value
  set age(value: number) {
    if (value < 0 || value > 150) {
      console.log("Invalid age!");
      return;
    }
    this._age = value;
  }

  get ageGroup(): string {
    if (this._age < 18) return "Minor";
    if (this._age < 65) return "Adult";
    return "Senior";
  }
}

const person = new Person();

// Using getter - looks like accessing a property
console.log(person.age); // Output: 0

// Using setter - looks like setting a property
person.age = 25;
console.log(person.age); // Output: 25
console.log(person.ageGroup); // Output: Adult

person.age = 150; // OK
person.age = 200; // Output: Invalid age! (setter blocked it)

// Real-world example: Temperature converter
class Temperature {
  private _celsius: number = 0;

  get celsius(): number {
    return this._celsius;
  }

  set celsius(value: number) {
    this._celsius = value;
  }

  get fahrenheit(): number {
    return (this._celsius * 9) / 5 + 32;
  }

  set fahrenheit(value: number) {
    this._celsius = (value - 32) * (5 / 9);
  }

  get kelvin(): number {
    return this._celsius + 273.15;
  }

  set kelvin(value: number) {
    this._celsius = value - 273.15;
  }
}

const temp = new Temperature();
temp.celsius = 25;
console.log(temp.fahrenheit); // 77
console.log(temp.kelvin); // 298.15

// Change via Fahrenheit
temp.fahrenheit = 32;
console.log(temp.celsius); // 0
```

### Static Members
**Definition**: Properties/methods that belong to the **class itself**, not to individual instances.

You access them using the class name, not an instance.

```typescript
class MathUtils {
  static PI = 3.14159;
  static readonly E = 2.71828; // Cannot be changed

  static add(a: number, b: number): number {
    return a + b;
  }

  static multiply(a: number, b: number): number {
    return a * b;
  }
}

// Access static members using class name
console.log(MathUtils.PI); // 3.14159
console.log(MathUtils.add(5, 3)); // 8

// You DON'T create an instance
// const math = new MathUtils(); // Would be wrong

// Real-world example: Singleton pattern (only one instance)
class Database {
  private static instance: Database;
  private connection: string = "";

  private constructor() {
    console.log("Database initialized");
    this.connection = "Connected";
  }

  static getInstance(): Database {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }

  query(sql: string): any {
    console.log(`Executing: ${sql}`);
    return [];
  }
}

// Always get the same instance
const db1 = Database.getInstance();
const db2 = Database.getInstance();
console.log(db1 === db2); // true - same object!
```

### Inheritance
**Definition**: When a class **inherits** from another class, it gets all its properties and methods. The inheriting class can add new properties/methods or override existing ones.

```typescript
// Parent class
class Animal {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  makeSound(): void {
    console.log("Some generic sound");
  }

  sleep(): void {
    console.log(`${this.name} is sleeping`);
  }
}

// Child class inherits from Animal
class Dog extends Animal {
  breed: string;

  constructor(name: string, age: number, breed: string) {
    super(name, age); // Call parent constructor
    this.breed = breed;
  }

  // Override parent method
  makeSound(): void {
    console.log("Woof! Woof!");
  }

  // New method only in Dog
  fetch(): void {
    console.log(`${this.name} is fetching the ball`);
  }
}

const dog = new Dog("Buddy", 5, "Golden Retriever");
dog.makeSound(); // Output: Woof! Woof! (overridden)
dog.sleep(); // Output: Buddy is sleeping (inherited)
dog.fetch(); // Output: Buddy is fetching the ball (new method)

// Real-world example: Employee hierarchy
class Employee {
  name: string;
  salary: number;

  constructor(name: string, salary: number) {
    this.name = name;
    this.salary = salary;
  }

  work(): void {
    console.log(`${this.name} is working`);
  }

  getSalary(): number {
    return this.salary;
  }
}

class Manager extends Employee {
  teamSize: number;

  constructor(name: string, salary: number, teamSize: number) {
    super(name, salary);
    this.teamSize = teamSize;
  }

  // Override parent method
  work(): void {
    console.log(`${this.name} is managing ${this.teamSize} people`);
  }

  // New method
  giveRaise(employeeName: string, amount: number): void {
    console.log(`${this.name} gives ${amount} raise to ${employeeName}`);
  }
}

class Developer extends Employee {
  language: string;

  constructor(name: string, salary: number, language: string) {
    super(name, salary);
    this.language = language;
  }

  work(): void {
    console.log(`${this.name} is coding in ${this.language}`);
  }
}

const manager = new Manager("John", 100000, 5);
const developer = new Developer("Jane", 80000, "TypeScript");

manager.work(); // John is managing 5 people
developer.work(); // Jane is coding in TypeScript
```

### Abstract Classes
**Definition**: A class that **cannot be created directly** but serves as a blueprint for other classes to inherit from. It can have abstract methods that subclasses MUST implement.

Use abstract classes when you want to force subclasses to implement certain methods.

```typescript
// Abstract class - cannot be instantiated
abstract class Shape {
  abstract getArea(): number; // Subclasses must implement this

  describe(): void {
    console.log(`This shape has area of ${this.getArea()}`);
  }
}

// const shape = new Shape(); // ❌ ERROR! Cannot instantiate abstract class

// Must extend and implement abstract methods
class Circle extends Shape {
  radius: number;

  constructor(radius: number) {
    super();
    this.radius = radius;
  }

  getArea(): number {
    return Math.PI * this.radius * this.radius;
  }
}

class Rectangle extends Shape {
  width: number;
  height: number;

  constructor(width: number, height: number) {
    super();
    this.width = width;
    this.height = height;
  }

  getArea(): number {
    return this.width * this.height;
  }
}

const circle = new Circle(5);
circle.describe(); // Output: This shape has area of 78.54...

const rectangle = new Rectangle(4, 5);
rectangle.describe(); // Output: This shape has area of 20

// Real-world example: Payment processors
abstract class PaymentProcessor {
  abstract processPayment(amount: number): Promise<boolean>;

  abstract refund(amount: number): Promise<boolean>;

  logTransaction(amount: number, type: string): void {
    console.log(`[${type}] Transaction of ${amount}`);
  }
}

class StripeProcessor extends PaymentProcessor {
  async processPayment(amount: number): Promise<boolean> {
    console.log(`Processing ${amount} with Stripe`);
    return true;
  }

  async refund(amount: number): Promise<boolean> {
    console.log(`Refunding ${amount} from Stripe`);
    return true;
  }
}

class PayPalProcessor extends PaymentProcessor {
  async processPayment(amount: number): Promise<boolean> {
    console.log(`Processing ${amount} with PayPal`);
    return true;
  }

  async refund(amount: number): Promise<boolean> {
    console.log(`Refunding ${amount} from PayPal`);
    return true;
  }
}
```

### Implementing Interfaces
**Definition**: When a class **implements** an interface, it **must have** all the properties and methods defined in that interface.

```typescript
interface Drawable {
  draw(): void;
}

interface Erasable {
  erase(): void;
}

// Canvas must implement both interfaces
class Canvas implements Drawable, Erasable {
  draw(): void {
    console.log("Drawing on canvas...");
  }

  erase(): void {
    console.log("Erasing canvas...");
  }
}

const canvas = new Canvas();
canvas.draw(); // Drawing on canvas...
canvas.erase(); // Erasing canvas...

// Real-world example: User roles with different permissions
interface Readable {
  read(): void;
}

interface Writable {
  write(): void;
}

interface Deletable {
  delete(): void;
}

class AdminUser implements Readable, Writable, Deletable {
  read(): void { console.log("Reading data"); }
  write(): void { console.log("Writing data"); }
  delete(): void { console.log("Deleting data"); }
}

class RegularUser implements Readable, Writable {
  read(): void { console.log("Reading data"); }
  write(): void { console.log("Writing data"); }
}

class GuestUser implements Readable {
  read(): void { console.log("Reading data"); }
}
```

---

## GENERICS

### What are Generics?
**Definition**: **Generics** let you write code that works with **any type** while still maintaining type safety. It's like creating a flexible function that can work with strings, numbers, objects, etc., but still knows what type it's working with.

Think of it as using a **variable for types** instead of hardcoding one specific type.

```typescript
// Without generics - only works with strings
function getFirstString(arr: string[]): string {
  return arr[0];
}

// Without generics - only works with numbers
function getFirstNumber(arr: number[]): number {
  return arr[0];
}

// With generics - works with ANY type!
function getFirst<T>(arr: T[]): T {
  return arr[0];
}

console.log(getFirst(["apple", "banana"])); // "apple" (T is string)
console.log(getFirst([1, 2, 3])); // 1 (T is number)
console.log(getFirst([true, false])); // true (T is boolean)
```

### Generic Functions
**Definition**: A function where the type is **determined when you call it**, not when you define it.

```typescript
// T is a placeholder for any type
function identity<T>(value: T): T {
  return value;
}

identity<string>("hello"); // T is string
identity<number>(42); // T is number
identity("hello"); // T is inferred as string

// Multiple generic types
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const merged = merge(
  { name: "John" },
  { age: 30 }
);
// merged has type { name: string } & { age: number }

// Real-world example: API response wrapper
function fetchData<T>(url: string): Promise<T> {
  return fetch(url).then(res => res.json());
}

interface User {
  id: number;
  name: string;
}

interface Post {
  id: number;
  title: string;
}

// Returns Promise<User>
const userPromise = fetchData<User>("/api/users/1");

// Returns Promise<Post>
const postPromise = fetchData<Post>("/api/posts/1");
```

### Generic Constraints
**Definition**: **Limiting** what types a generic can be. You can say "T must be a certain type or have certain properties."

```typescript
// T can be any type
function log<T>(value: T): void {
  console.log(value);
}

// T must be a string or number
function add<T extends string | number>(a: T, b: T): T {
  return a + b as T; // Works for both
}

// T must have a 'length' property
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}

getLength("hello"); // 5 (strings have length)
getLength([1, 2, 3]); // 3 (arrays have length)
// getLength(123); // ❌ ERROR - numbers don't have length

// T must be a key of object U
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: "John", age: 30 };
const name = getProperty(person, "name"); // OK
// getProperty(person, "email"); // ❌ ERROR - person doesn't have email property

// Real-world example: Database query builder
function query<T extends { id: number }>(arr: T[], id: number): T | undefined {
  return arr.find(item => item.id === id);
}

interface User {
  id: number;
  name: string;
}

interface Post {
  id: number;
  title: string;
}

const users: User[] = [{ id: 1, name: "John" }];
const posts: Post[] = [{ id: 1, title: "Hello" }];

query(users, 1); // Returns User | undefined
query(posts, 1); // Returns Post | undefined
```

### Generic Classes
**Definition**: A class where properties/methods can work with **any type** specified when creating the instance.

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }

  print(): void {
    console.log(this.items);
  }
}

// Stack for numbers
const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
console.log(numberStack.pop()); // 2

// Stack for strings
const stringStack = new Stack<string>();
stringStack.push("hello");
stringStack.push("world");
console.log(stringStack.peek()); // "world"

// Real-world example: Generic Repository for database
class Repository<T> {
  private items: T[] = [];

  save(item: T): void {
    this.items.push(item);
  }

  findById(predicate: (item: T) => boolean): T | undefined {
    return this.items.find(predicate);
  }

  getAll(): T[] {
    return this.items;
  }

  delete(predicate: (item: T) => boolean): void {
    this.items = this.items.filter(item => !predicate(item));
  }
}

interface User {
  id: number;
  name: string;
}

interface Product {
  id: number;
  title: string;
}

const userRepo = new Repository<User>();
userRepo.save({ id: 1, name: "John" });
const user = userRepo.findById(u => u.id === 1);

const productRepo = new Repository<Product>();
productRepo.save({ id: 1, title: "Laptop" });
const product = productRepo.findById(p => p.id === 1);
```

### Generic Interfaces
**Definition**: Interfaces with **generic type parameters** that are filled in when the interface is used.

```typescript
interface Pair<T, U> {
  first: T;
  second: U;
}

const stringNumber: Pair<string, number> = {
  first: "age",
  second: 30
};

const userProduct: Pair<User, Product> = {
  first: { id: 1, name: "John" },
  second: { id: 1, title: "Laptop" }
};

// Real-world example: API Response wrapper
interface ApiResponse<T> {
  status: number;
  message: string;
  data: T;
}

interface UserData {
  id: number;
  name: string;
  email: string;
}

interface PostData {
  id: number;
  title: string;
  content: string;
}

function handleUserResponse(response: ApiResponse<UserData>) {
  console.log(`User: ${response.data.name}`);
}

function handlePostResponse(response: ApiResponse<PostData>) {
  console.log(`Post: ${response.data.title}`);
}

const userResponse: ApiResponse<UserData> = {
  status: 200,
  message: "Success",
  data: { id: 1, name: "John", email: "john@example.com" }
};

handleUserResponse(userResponse);
```

---

## ADVANCED TYPES

### Intersection Types
**Definition**: Combine multiple types into one using `&`. The resulting type has properties from **all** combined types.

Think of it as "AND" - must satisfy all types.

```typescript
interface Admin {
  canManageUsers: boolean;
  canDeletePosts: boolean;
}

interface Moderator {
  canDeleteComments: boolean;
  canBanUsers: boolean;
}

// SuperModerator must have ALL properties from both types
type SuperModerator = Admin & Moderator;

const superMod: SuperModerator = {
  canManageUsers: true,
  canDeletePosts: true,
  canDeleteComments: true,
  canBanUsers: true
};

// Real-world: User who is both Admin and Premium subscriber
interface PremiumMember {
  membershipLevel: "gold" | "platinum";
  premiumFeatures: string[];
}

interface AdminUser {
  adminLevel: number;
  permissions: string[];
}

type AdminPremiumUser = AdminUser & PremiumMember;

const adminPrem: AdminPremiumUser = {
  adminLevel: 5,
  permissions: ["delete", "ban", "edit"],
  membershipLevel: "platinum",
  premiumFeatures: ["advanced-analytics", "api-access"]
};
```

### Type Guards
**Definition**: Code that **checks the type** of a value so TypeScript knows which type it is in that code block.

```typescript
// typeof guard - for primitive types
function processValue(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase()); // OK - TypeScript knows it's string
  } else {
    console.log(value * 2); // OK - TypeScript knows it's number
  }
}

// instanceof guard - for classes/objects
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}

// Custom type guard - create your own check
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNumber(value: unknown): value is number {
  return typeof value === "number";
}

let value: unknown = "hello";
if (isString(value)) {
  console.log(value.toUpperCase()); // OK
}

// in operator - check if property exists
interface Bird {
  fly(): void;
}

interface Fish {
  swim(): void;
}

function move(animal: Bird | Fish) {
  if ("fly" in animal) {
    animal.fly();
  } else {
    animal.swim();
  }
}

// Real-world: API response handling
type ApiResult = { success: true; data: User } | { success: false; error: string };

function handleResult(result: ApiResult) {
  if (result.success) {
    console.log(`User: ${result.data.name}`);
  } else {
    console.log(`Error: ${result.error}`);
  }
}
```

### Mapped Types
**Definition**: Create a **new type by transforming** each property of an existing type.

Think of it as a loop that goes through each property and changes it.

```typescript
type Person = {
  name: string;
  age: number;
  email: string;
};

// Make all properties readonly
type ReadonlyPerson = {
  readonly [K in keyof Person]: Person[K];
};

// Make all properties optional
type OptionalPerson = {
  [K in keyof Person]?: Person[K];
};

// Convert all properties to getters
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type PersonGetters = Getters<Person>;
// Result: {
//   getName: () => string;
//   getAge: () => number;
//   getEmail: () => string;
// }

// Real-world: Track which API fields are loading
interface User {
  id: number;
  name: string;
  email: string;
}

type LoadingStates<T> = {
  [K in keyof T]: boolean;
};

type UserLoading = LoadingStates<User>;
// Result: { id: boolean; name: boolean; email: boolean }

const userLoading: UserLoading = {
  id: false,
  name: true,
  email: false
};
```

### Conditional Types
**Definition**: Types that **check a condition** and return different types based on the result.

It's like an `if-else` statement but for types.

```typescript
// If T is string, return string; otherwise return number
type StringOrNumber<T> = T extends string ? string : number;

type A = StringOrNumber<"hello">; // string
type B = StringOrNumber<42>; // number

// Extract the array element type
type Flatten<T> = T extends Array<infer U> ? U : T;

type Str = Flatten<string[]>; // string
type Num = Flatten<number>; // number (not an array)

// Real-world: Return type depends on input type
function process<T extends string | number>(
  value: T
): T extends string ? string : number {
  if (typeof value === "string") {
    return value.toUpperCase() as any;
  } else {
    return value * 2 as any;
  }
}

// Real-world: API response type depends on status
type ApiResponse<T, IsSuccess extends boolean> = IsSuccess extends true
  ? { status: 200; data: T }
  : { status: 400; error: string };

const successResponse: ApiResponse<User, true> = {
  status: 200,
  data: { id: 1, name: "John", email: "john@example.com" }
};

const errorResponse: ApiResponse<User, false> = {
  status: 400,
  error: "Not found"
};
```

### Discriminated Unions
**Definition**: Using a common property (the **discriminator**) to distinguish between different union types.

```typescript
// Each response type has a 'status' property that differs
type SuccessResponse = {
  status: "success";
  data: User;
};

type LoadingResponse = {
  status: "loading";
};

type ErrorResponse = {
  status: "error";
  error: string;
};

type Response = SuccessResponse | LoadingResponse | ErrorResponse;

// TypeScript narrows based on 'status'
function handleResponse(response: Response) {
  switch (response.status) {
    case "success":
      console.log(`User: ${response.data.name}`);
      break;
    case "loading":
      console.log("Loading...");
      break;
    case "error":
      console.log(`Error: ${response.error}`);
      break;
  }
}

// Real-world: Different request states
type RequestState<T> =
  | { state: "idle" }
  | { state: "loading" }
  | { state: "success"; data: T }
  | { state: "error"; error: Error };

function render<T>(requestState: RequestState<T>) {
  switch (requestState.state) {
    case "idle":
      return "Not started";
    case "loading":
      return "Loading...";
    case "success":
      return `Data: ${requestState.data}`;
    case "error":
      return `Error: ${requestState.error.message}`;
  }
}
```

---

## DECORATORS

**What are Decorators?**
**Definition**: Functions that **modify or enhance** classes, methods, properties, or parameters. They're like annotations that add extra functionality.

Enable in `tsconfig.json`: `"experimentalDecorators": true`

```typescript
// Basic class decorator
function Logger(constructor: Function) {
  console.log(`Creating instance of ${constructor.name}`);
}

@Logger
class User {
  constructor(public name: string) {}
}

// When you create: new User("John")
// Output: Creating instance of User
```

### Method Decorators with Practical Use
```typescript
// Decorator that measures function execution time
function Timing(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    const start = performance.now();
    const result = originalMethod.apply(this, args);
    const end = performance.now();
    console.log(`${propertyKey} took ${(end - start).toFixed(2)}ms`);
    return result;
  };

  return descriptor;
}

class DataProcessor {
  @Timing
  processLargeDataset(data: number[]): number {
    let sum = 0;
    for (let i = 0; i < data.length; i++) {
      sum += data[i];
    }
    return sum;
  }
}

const processor = new DataProcessor();
processor.processLargeDataset([1, 2, 3, 4, 5]);
// Output: processLargeDataset took 0.XX ms
```

---

## UTILITY TYPES

### Partial<T>
**Definition**: Makes **all properties optional** in a type.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// All properties become optional
type PartialUser = Partial<User>;

const user: PartialUser = { name: "John" }; // OK - other properties are optional

// Real-world: Update user where only some fields need to change
function updateUser(user: User, updates: Partial<User>): User {
  return { ...user, ...updates };
}

const originalUser: User = { id: 1, name: "John", email: "john@example.com" };
const updated = updateUser(originalUser, { name: "Jane" }); // Only need name
```

### Required<T>
**Definition**: Makes **all optional properties required**.

```typescript
interface Config {
  host?: string;
  port?: number;
}

// All optional properties become required
type RequiredConfig = Required<Config>;

const config: RequiredConfig = {
  host: "localhost",
  port: 3000
};
```

### Record<K, T>
**Definition**: Create an object type where **keys are K and values are T**.

```typescript
type Role = "admin" | "user" | "guest";

// Each role maps to an array of permissions
type RolePermissions = Record<Role, string[]>;

const permissions: RolePermissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"]
};

// Real-world: Color palette
type ColorName = "primary" | "secondary" | "danger";
type ColorPalette = Record<ColorName, string>;

const colors: ColorPalette = {
  primary: "#0066cc",
  secondary: "#666666",
  danger: "#cc0000"
};
```

### Pick<T, K>
**Definition**: Create a type with **only specific properties** from another type.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Only pick id, name, email (exclude password for security)
type UserPreview = Pick<User, "id" | "name" | "email">;

const preview: UserPreview = {
  id: 1,
  name: "John",
  email: "john@example.com"
  // password is not allowed
};

// Real-world: Different API response shapes
type PublicUser = Pick<User, "id" | "name">;
type PrivateUser = Pick<User, "id" | "name" | "email">;
```

### Omit<T, K>
**Definition**: Create a type by **removing specific properties** from another type.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Omit password (everything except password)
type UserData = Omit<User, "password">;

const userData: UserData = {
  id: 1,
  name: "John",
  email: "john@example.com"
  // password not allowed
};

// Real-world: Database response without sensitive data
type SafeUser = Omit<User, "password">;
```

---

## ERROR HANDLING

### Custom Error Classes
**Definition**: Create your own error types to **handle specific error scenarios**.

```typescript
class ValidationError extends Error {
  constructor(public field: string, message: string) {
    super(message);
    this.name = "ValidationError";
  }
}

class DatabaseError extends Error {
  constructor(public code: string, message: string) {
    super(message);
    this.name = "DatabaseError";
  }
}

function validateEmail(email: string): void {
  if (!email.includes("@")) {
    throw new ValidationError("email", "Invalid email format");
  }
}

try {
  validateEmail("invalid-email");
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(`Field: ${error.field}, Error: ${error.message}`);
  }
}
```

### Result Pattern
**Definition**: Instead of throwing errors, **return a result object** that indicates success or failure.

This is safer and more predictable than exceptions.

```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function divide(a: number, b: number): Result<number> {
  if (b === 0) {
    return { ok: false, error: new Error("Division by zero") };
  }
  return { ok: true, value: a / b };
}

const result = divide(10, 2);
if (result.ok) {
  console.log(`Result: ${result.value}`);
} else {
  console.log(`Error: ${result.error.message}`);
}

// Real-world: Safe parsing
function parseJson<T>(json: string): Result<T, SyntaxError> {
  try {
    return { ok: true, value: JSON.parse(json) };
  } catch (error) {
    return { ok: false, error: error as SyntaxError };
  }
}
```

---

## DESIGN PATTERNS

### Singleton Pattern
**Definition**: Ensures **only one instance** of a class exists throughout the application.

Useful for databases, configuration, logger services.

```typescript
class Database {
  private static instance: Database;

  private constructor() {
    console.log("Database initialized");
  }

  static getInstance(): Database {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }

  query(sql: string): any[] {
    console.log(`Executing: ${sql}`);
    return [];
  }
}

const db1 = Database.getInstance();
const db2 = Database.getInstance();
console.log(db1 === db2); // true - same instance!

// Real-world: Logger service
class Logger {
  private static instance: Logger;
  private logs: string[] = [];

  private constructor() {}

  static getInstance(): Logger {
    if (!Logger.instance) {
      Logger.instance = new Logger();
    }
    return Logger.instance;
  }

  log(message: string): void {
    this.logs.push(message);
    console.log(message);
  }

  getLogs(): string[] {
    return this.logs;
  }
}

// Always same logger instance
const logger1 = Logger.getInstance();
const logger2 = Logger.getInstance();
logger1.log("First message");
logger2.log("Second message");
console.log(logger2.getLogs()); // Both messages there!
```

### Factory Pattern
**Definition**: Create objects **without specifying their exact classes**. A factory decides which type to create.

```typescript
interface Animal {
  makeSound(): void;
  move(): void;
}

class Dog implements Animal {
  makeSound() { console.log("Woof!"); }
  move() { console.log("Running..."); }
}

class Cat implements Animal {
  makeSound() { console.log("Meow!"); }
  move() { console.log("Jumping..."); }
}

class Bird implements Animal {
  makeSound() { console.log("Tweet!"); }
  move() { console.log("Flying..."); }
}

// Factory - creates the right animal type
class AnimalFactory {
  static createAnimal(type: "dog" | "cat" | "bird"): Animal {
    switch (type) {
      case "dog":
        return new Dog();
      case "cat":
        return new Cat();
      case "bird":
        return new Bird();
    }
  }
}

const dog = AnimalFactory.createAnimal("dog");
dog.makeSound(); // Woof!

// Real-world: Payment processor factory
class StripePayment {
  pay(amount: number) { console.log(`Stripe: ${amount}`); }
}

class PayPalPayment {
  pay(amount: number) { console.log(`PayPal: ${amount}`); }
}

class PaymentFactory {
  static create(provider: "stripe" | "paypal") {
    if (provider === "stripe") return new StripePayment();
    return new PayPalPayment();
  }
}

const payment = PaymentFactory.create("stripe");
payment.pay(100);
```

### Observer Pattern
**Definition**: Create a **subscription system** where listeners are notified when something changes.

One-to-many relationship: one subject notifies many observers.

```typescript
interface Observer {
  update(data: any): void;
}

class Subject {
  private observers: Observer[] = [];

  subscribe(observer: Observer): void {
    this.observers.push(observer);
  }

  unsubscribe(observer: Observer): void {
    this.observers = this.observers.filter(o => o !== observer);
  }

  notify(data: any): void {
    this.observers.forEach(o => o.update(data));
  }
}

class EmailObserver implements Observer {
  update(data: any): void {
    console.log(`Email sent: ${data.message}`);
  }
}

class SMSObserver implements Observer {
  update(data: any): void {
    console.log(`SMS sent: ${data.message}`);
  }
}

const subject = new Subject();
subject.subscribe(new EmailObserver());
subject.subscribe(new SMSObserver());
subject.notify({ message: "Hello!" }); // Both notified

// Real-world: Event emitter
class EventEmitter {
  private listeners: Map<string, Function[]> = new Map();

  on(event: string, callback: Function): void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event)!.push(callback);
  }

  emit(event: string, data: any): void {
    const callbacks = this.listeners.get(event);
    if (callbacks) {
      callbacks.forEach(cb => cb(data));
    }
  }
}

const emitter = new EventEmitter();
emitter.on("message", (data) => console.log(`Got: ${data}`));
emitter.on("message", (data) => console.log(`Also got: ${data}`));
emitter.emit("message", "hello"); // Both callbacks fire
```

### Dependency Injection
**Definition**: Pass **dependencies into a class** instead of having the class create them. Makes code flexible and testable.

```typescript
interface Logger {
  log(message: string): void;
}

class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }
}

class FileLogger implements Logger {
  log(message: string): void {
    console.log(`[FILE] Writing: ${message}`);
  }
}

class UserService {
  constructor(private logger: Logger) {}

  createUser(name: string): void {
    this.logger.log(`Creating user: ${name}`);
    // Create user logic...
  }
}

// Easy to switch implementations
const consoleLogger = new ConsoleLogger();
const userService1 = new UserService(consoleLogger);
userService1.createUser("John");

const fileLogger = new FileLogger();
const userService2 = new UserService(fileLogger);
userService2.createUser("Jane");

// Real-world: Easier testing
class MockLogger implements Logger {
  logs: string[] = [];
  log(message: string): void {
    this.logs.push(message);
  }
}

const mockLogger = new MockLogger();
const testService = new UserService(mockLogger);
testService.createUser("Test");
console.log(mockLogger.logs); // Check what was logged
```

### Builder Pattern
**Definition**: Create **complex objects step-by-step**. Makes construction readable and flexible.

```typescript
interface QueryOptions {
  select?: string[];
  from?: string;
  where?: string;
  limit?: number;
}

class QueryBuilder {
  private options: QueryOptions = {};

  select(...fields: string[]): this {
    this.options.select = fields;
    return this; // Return 'this' for chaining
  }

  from(table: string): this {
    this.options.from = table;
    return this;
  }

  where(condition: string): this {
    this.options.where = condition;
    return this;
  }

  limit(count: number): this {
    this.options.limit = count;
    return this;
  }

  build(): string {
    let query = "";
    if (this.options.select) {
      query += `SELECT ${this.options.select.join(", ")} `;
    }
    if (this.options.from) {
      query += `FROM ${this.options.from} `;
    }
    if (this.options.where) {
      query += `WHERE ${this.options.where} `;
    }
    if (this.options.limit) {
      query += `LIMIT ${this.options.limit}`;
    }
    return query;
  }
}

// Method chaining makes it readable
const query = new QueryBuilder()
  .select("id", "name", "email")
  .from("users")
  .where("age > 18")
  .limit(10)
  .build();

console.log(query);
// SELECT id, name, email FROM users WHERE age > 18 LIMIT 10

// Real-world: Request builder
class RequestBuilder {
  private config: any = {};

  method(method: string): this {
    this.config.method = method;
    return this;
  }

  url(url: string): this {
    this.config.url = url;
    return this;
  }

  header(key: string, value: string): this {
    if (!this.config.headers) this.config.headers = {};
    this.config.headers[key] = value;
    return this;
  }

  body(data: any): this {
    this.config.body = JSON.stringify(data);
    return this;
  }

  async send() {
    return fetch(this.config.url, this.config);
  }
}

const request = new RequestBuilder()
  .method("POST")
  .url("/api/users")
  .header("Content-Type", "application/json")
  .body({ name: "John" })
  .send();
```

---

## MODULES & IMPORTS/EXPORTS

### ES6 Modules
**Definition**: Share code between files using `export` and `import`.

```typescript
// math.ts - exporting
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export const PI = 3.14159;

export default class Calculator {
  constructor() {}
}

// main.ts - importing
import Calculator, { add, subtract, PI } from "./math";

const result = add(5, 3);
console.log(result);

const calc = new Calculator();

// Importing everything
import * as Math from "./math";
Math.add(5, 3);

// Real-world structure:
// src/
//   models/
//     User.ts
//   services/
//     UserService.ts
//   index.ts

// models/User.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

// services/UserService.ts
import { User } from "../models/User";

export class UserService {
  createUser(name: string, email: string): User {
    return { id: 1, name, email };
  }
}

// index.ts
export { User } from "./models/User";
export { UserService } from "./services/UserService";
```

---

## KEY CONCEPTS TO REMEMBER

### Type Safety Benefits
1. **Catch errors early** - Before code runs
2. **Better IDE support** - Autocomplete, jump to definition
3. **Refactoring confidence** - TypeScript tells you what breaks
4. **Self-documenting** - Types serve as documentation

### Common Interview Questions

**Q: What's the difference between `any` and `unknown`?**
A: `any` disables type checking (unsafe), `unknown` requires type checking (safe).

**Q: When should you use interfaces vs types?**
A: Use interfaces for object shapes, types for unions/primitives. Interfaces support declaration merging.

**Q: What are generics?**
A: They let you write flexible code that works with any type while maintaining type safety.

**Q: What's the difference between `public`, `private`, and `protected`?**
A: public (anyone), private (only this class), protected (this class + subclasses).

**Q: What is a type guard?**
A: Code that checks a value's type so TypeScript narrows it to the correct type in that scope.

**Q: What are decorators?**
A: Functions that modify or enhance classes, methods, properties. Useful for logging, timing, validation.

### Best Practices
1. **Avoid `any`** - Use `unknown` or specific types instead
2. **Prefer interfaces for objects** - They're clearer for object shapes
3. **Use generics for reusable code** - Makes code DRY and flexible
4. **Add type constraints** - Narrow down what types a generic can be
5. **Use error handling properly** - Either throw errors or return Result types
6. **Use access modifiers** - Encapsulate with private/protected
7. **Leverage readonly** - Prevent accidental mutations



