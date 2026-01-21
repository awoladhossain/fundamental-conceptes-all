# JavaScript In-Depth: Junior to Senior Level Knowledge

This comprehensive guide covers what mid-level and senior JavaScript engineers know in depth. It's organized from foundational concepts to advanced patterns.

---

## 1. EXECUTION CONTEXT & CALL STACK

### What It Is
Every time JavaScript code runs, it executes within an execution context. There are three types: global, function, and eval context.

### The Call Stack
The call stack is a mechanism that keeps track of multiple function calls. JavaScript is single-threaded, meaning it can only execute one piece of code at a time.

```javascript
function first() {
  console.log("First function");
  second();
}

function second() {
  console.log("Second function");
  third();
}

function third() {
  console.log("Third function");
}

first();
```

Execution order:
1. Global execution context is created
2. `first()` is called → pushed to call stack
3. Inside `first()`, `second()` is called → pushed to call stack
4. Inside `second()`, `third()` is called → pushed to call stack
5. `third()` completes → popped from stack
6. `second()` completes → popped from stack
7. `first()` completes → popped from stack
8. Global context completes

**Why this matters**: Understanding the call stack helps you debug code, understand asynchronous behavior, and prevent stack overflow errors.

### Execution Context Components

Each execution context has three components:

1. **Variable Environment**: Where variables and function declarations are created
2. **Scope Chain**: The chain of scopes accessible from this context
3. **this Binding**: What the `this` keyword refers to

```javascript
function myFunction(a, b) {
  // Variable Environment: a, b, c are created
  // Scope Chain: myFunction scope → global scope
  // this Binding: depends on how function was called

  const c = a + b;
  return c;
}

myFunction(1, 2);
```

---

## 2. HOISTING

### Variable Hoisting

**var** is hoisted and initialized with `undefined`:
```javascript
console.log(x); // undefined (not error)
var x = 5;
console.log(x); // 5
```

Internally, JavaScript treats this as:
```javascript
var x;
console.log(x); // undefined
x = 5;
console.log(x); // 5
```

**let** and **const** are hoisted but not initialized (Temporal Dead Zone):
```javascript
console.log(y); // ReferenceError: Cannot access 'y' before initialization
let y = 5;
```

### Function Hoisting

Function declarations are fully hoisted:
```javascript
console.log(myFunc()); // Works fine - returns 5

function myFunc() {
  return 5;
}
```

Function expressions are NOT hoisted:
```javascript
console.log(myFunc()); // TypeError: myFunc is not a function

var myFunc = function() {
  return 5;
};
```

**Why this matters**: Hoisting can cause unexpected bugs. Understanding it helps you write predictable code and know why some patterns work and others don't.

---

## 3. SCOPE & CLOSURES

### Types of Scope

**Global Scope**: Variables accessible everywhere
```javascript
const global = "I am global";

function myFunc() {
  console.log(global); // Can access
}
```

**Function Scope**: Variables only accessible inside the function
```javascript
function myFunc() {
  const local = "I am local";
  console.log(local); // Works
}

console.log(local); // ReferenceError
```

**Block Scope**: Variables only accessible within a block (let/const only)
```javascript
if (true) {
  let blockVar = "I am block scoped";
  console.log(blockVar); // Works
}

console.log(blockVar); // ReferenceError
```

### Scope Chain

When JavaScript looks for a variable, it searches:
1. Current scope
2. Parent scope
3. Parent's parent scope
4. ... until global scope
5. If not found, ReferenceError

```javascript
const global = "global";

function outer() {
  const outerVar = "outer";

  function inner() {
    const innerVar = "inner";

    console.log(innerVar); // Found in local scope
    console.log(outerVar); // Found in parent scope
    console.log(global);   // Found in global scope
  }

  inner();
}

outer();
```

### Closures

A closure is a function that has access to variables from its outer (enclosing) scope even after the outer function has returned.

```javascript
function outer() {
  const secretValue = "secret";

  function inner() {
    return secretValue;
  }

  return inner;
}

const myFunction = outer();
console.log(myFunction()); // "secret"
```

**The key insight**: When `inner()` is returned from `outer()`, it doesn't lose access to `secretValue`. The function "closes over" the variable.

### Practical Closure Examples

**Data Privacy**:
```javascript
function createCounter() {
  let count = 0;

  return {
    increment: function() {
      count++;
      return count;
    },
    decrement: function() {
      count--;
      return count;
    },
    getCount: function() {
      return count;
    }
  };
}

const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.decrement(); // 1
// No way to access count directly - it's private!
```

**Function Factory**:
```javascript
function createMultiplier(multiplier) {
  return function(number) {
    return number * multiplier;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

**Why closures matter**: They enable data privacy, function factories, callbacks, and are fundamental to many design patterns. Senior engineers use closures to create robust, encapsulated code.

---

## 4. THE this KEYWORD

### How this is Determined

`this` is determined by **how** a function is called, not where it's defined.

### 1. Regular Function Call
When called as a regular function, `this` is the global object (or undefined in strict mode).

```javascript
function myFunc() {
  console.log(this);
}

myFunc(); // window (browser) or global (Node.js), or undefined in strict mode
```

### 2. Method Call
When called as a method of an object, `this` refers to that object.

```javascript
const person = {
  name: "John",
  greet: function() {
    console.log(this.name);
  }
};

person.greet(); // "John" - this refers to person
```

### 3. Constructor Call
When called with `new`, `this` refers to the newly created object.

```javascript
function Person(name) {
  this.name = name;
}

const john = new Person("John");
console.log(john.name); // "John"
```

### 4. Explicit Binding with call, apply, bind

**call**: Execute function with specific `this` and arguments
```javascript
function introduce(greeting) {
  console.log(greeting + ", I am " + this.name);
}

const person = { name: "Alice" };
introduce.call(person, "Hello"); // "Hello, I am Alice"
```

**apply**: Same as call, but arguments as array
```javascript
introduce.apply(person, ["Hi"]); // "Hi, I am Alice"
```

**bind**: Return new function with bound `this`
```javascript
const boundIntroduce = introduce.bind(person);
boundIntroduce("Hey"); // "Hey, I am Alice"
```

### 5. Arrow Functions (Special Case)
Arrow functions don't have their own `this`. They inherit it from the enclosing scope.

```javascript
const person = {
  name: "John",
  greet: function() {
    const inner = () => {
      console.log(this.name); // "John" - inherits from greet's this
    };
    inner();
  }
};

person.greet(); // "John"
```

**Why this matters**: Misunderstanding `this` causes bugs. Understanding it deeply helps you write correct object-oriented code and avoid pitfalls with callbacks.

---

## 5. PROTOTYPES & PROTOTYPAL INHERITANCE

### Prototype Chain

Every object in JavaScript has a `[[Prototype]]` property (internal, accessed via `__proto__` or `Object.getPrototypeOf()`).

```javascript
const obj = {};
console.log(Object.getPrototypeOf(obj) === Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype)); // null
```

When you access a property on an object, JavaScript searches:
1. The object itself
2. The object's prototype
3. The prototype's prototype
4. ... up the chain until null

```javascript
const parent = {
  greet: function() {
    return "Hello from parent";
  }
};

const child = Object.create(parent);
console.log(child.greet()); // "Hello from parent"
```

### Constructor Functions

Before ES6 classes, constructor functions were the standard way to create objects with shared methods.

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(this.name + " makes a sound");
};

const dog = new Animal("Dog");
dog.speak(); // "Dog makes a sound"

const cat = new Animal("Cat");
cat.speak(); // "Cat makes a sound"
```

When you call `new Animal()`:
1. A new empty object is created
2. The object's `[[Prototype]]` is set to `Animal.prototype`
3. `Animal` is called with `this` bound to the new object
4. The new object is returned

### Prototype-based Inheritance

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(this.name + " makes a sound");
};

function Dog(name, breed) {
  Animal.call(this, name); // Call parent constructor
  this.breed = breed;
}

// Set up prototype chain
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
  console.log(this.name + " barks");
};

const myDog = new Dog("Buddy", "Golden Retriever");
myDog.speak(); // "Buddy makes a sound"
myDog.bark();  // "Buddy barks"
```

### ES6 Classes

Classes are syntactic sugar over the prototype system. They work the same way underneath.

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(this.name + " makes a sound");
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }

  bark() {
    console.log(this.name + " barks");
  }
}

const myDog = new Dog("Buddy", "Golden");
myDog.speak(); // "Buddy makes a sound"
myDog.bark();  // "Buddy barks"
```

**Important**: `extends` sets up the prototype chain. `super()` calls the parent constructor.

### Property Lookup

```javascript
const person = {
  name: "John",
  greet() {
    return "Hello";
  }
};

const employee = Object.create(person);
employee.role = "Engineer";

console.log(employee.name);  // "John" - from prototype
console.log(employee.role);  // "Engineer" - own property
console.log(employee.greet()); // "Hello" - from prototype

// Check if property is own property
console.log(employee.hasOwnProperty("name")); // false
console.log(employee.hasOwnProperty("role")); // true
```

**Why this matters**: Understanding prototypes is crucial for inheritance, performance optimization, and avoiding common bugs. Many JavaScript patterns rely on prototype understanding.

---

## 6. THE EVENT LOOP & ASYNCHRONOUS JAVASCRIPT

### JavaScript is Single-Threaded

JavaScript runs on a single thread, but we can perform asynchronous operations. How?

### The Event Loop

The event loop continuously checks:
1. Call stack - synchronous code
2. Callback queue - callbacks from completed async operations

```javascript
console.log("Start");

setTimeout(() => {
  console.log("Timeout");
}, 0);

console.log("End");

// Output:
// Start
// End
// Timeout
```

Why? Even though timeout is 0, it goes to the callback queue, not the call stack.

### How Async Operations Work

```javascript
console.log("1");

setTimeout(() => {
  console.log("2");
}, 1000);

console.log("3");

// Output:
// 1
// 3
// 2 (after 1 second)
```

**Timeline**:
1. `console.log("1")` executes → prints "1"
2. `setTimeout()` is registered with browser/Node.js (goes off-thread)
3. `console.log("3")` executes → prints "3"
4. Call stack is empty
5. After 1 second, callback goes to callback queue
6. Event loop sees empty call stack, moves callback from queue to stack
7. Callback executes → prints "2"

### Microtask Queue

There are actually TWO queues:
1. **Microtask Queue**: Promises, async/await, MutationObserver
2. **Callback Queue (Macrotask Queue)**: setTimeout, setInterval, I/O

The event loop prioritizes microtasks:

```javascript
console.log("Start");

setTimeout(() => {
  console.log("setTimeout");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("Promise");
  });

console.log("End");

// Output:
// Start
// End
// Promise
// setTimeout
```

**Why?** Promises are microtasks and have higher priority.

### Order of Execution (Important for Interviews)

1. Synchronous code (call stack)
2. Microtasks (Promises, async/await)
3. Render (if needed)
4. Macrotasks (setTimeout, setInterval)
5. Back to step 2

```javascript
console.log("Script start");

setTimeout(() => {
  console.log("setTimeout");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("Promise 1");
  })
  .then(() => {
    console.log("Promise 2");
  });

console.log("Script end");

// Output:
// Script start
// Script end
// Promise 1
// Promise 2
// setTimeout
```

**Why this matters**: Understanding the event loop is critical for debugging async code, preventing race conditions, and optimizing performance. Senior engineers use this knowledge to write performant, non-blocking code.

---

## 7. PROMISES

### Promise States

A Promise can be in one of three states:
1. **Pending**: Initial state, operation hasn't completed
2. **Fulfilled**: Operation completed successfully, has a value
3. **Rejected**: Operation failed, has a reason

Once a promise is settled (fulfilled or rejected), it cannot change states.

### Creating Promises

```javascript
const promise = new Promise((resolve, reject) => {
  // Executor function

  setTimeout(() => {
    resolve("Success!");
  }, 1000);
});

promise
  .then(result => {
    console.log(result); // "Success!"
  })
  .catch(error => {
    console.log("Error:", error);
  });
```

### Promise Chaining

```javascript
fetch('/user/1')
  .then(response => response.json())
  .then(user => {
    console.log(user.name);
    return fetch(`/user/${user.id}/posts`);
  })
  .then(response => response.json())
  .then(posts => {
    console.log(posts);
  })
  .catch(error => {
    console.log("Error:", error);
  });
```

### Key Promise Methods

**Promise.all()**: Wait for all promises, or fail if any fails
```javascript
Promise.all([promise1, promise2, promise3])
  .then(results => {
    // results is array of values
  });
```

**Promise.race()**: Return result of first settled promise
```javascript
Promise.race([promise1, promise2])
  .then(result => {
    // result from whichever promise settles first
  });
```

**Promise.allSettled()**: Wait for all promises to settle (ES2020)
```javascript
Promise.allSettled([promise1, promise2])
  .then(results => {
    // results includes both fulfilled and rejected
  });
```

**Promise.any()**: Return first fulfilled promise (ES2021)
```javascript
Promise.any([promise1, promise2])
  .then(result => {
    // result from first fulfilled promise
  });
```

### Async/Await

Async/await is syntactic sugar over promises, making async code look synchronous.

```javascript
async function getUser() {
  try {
    const response = await fetch('/user/1');
    const user = await response.json();
    console.log(user.name);
    return user;
  } catch (error) {
    console.log("Error:", error);
  }
}

getUser();
```

**Important**: An `async` function always returns a Promise, even if you don't explicitly return one.

```javascript
async function myFunc() {
  return 5;
}

myFunc(); // Returns Promise { 5 }

myFunc().then(result => console.log(result)); // 5
```

**Why this matters**: Promises and async/await are fundamental to modern JavaScript. Understanding them deeply helps you handle asynchronous operations correctly and avoid common pitfalls like unhandled rejections or race conditions.

---

## 8. MEMORY MANAGEMENT & GARBAGE COLLECTION

### How JavaScript Memory Works

JavaScript allocates memory in two places:
1. **Stack**: Stores primitive values and function calls
2. **Heap**: Stores objects and functions

```javascript
let age = 25;           // Stack: age → 25
const person = {        // Stack: person → reference
  name: "John",         // Heap: object with properties
  age: 25
};
```

### Garbage Collection

JavaScript automatically removes objects that are no longer referenced:

```javascript
let person = { name: "John" };
console.log(person.name); // "John"

person = null; // No reference to object anymore
// Object is garbage collected
```

### Memory Leaks

A memory leak occurs when objects are no longer needed but still referenced.

**Global Variables**:
```javascript
function badFunc() {
  leakyVar = "I'm global"; // Accidentally global
}

badFunc();
// leakyVar stays in memory forever
```

**Event Listeners**:
```javascript
const button = document.querySelector('button');

button.addEventListener('click', () => {
  console.log('clicked');
});

// Later, if element is removed from DOM but listener isn't removed,
// it stays in memory
```

**Fix**:
```javascript
button.removeEventListener('click', handler);
```

**Timers**:
```javascript
const timerId = setInterval(() => {
  console.log('running');
}, 1000);

// Later, if not cleared:
clearInterval(timerId);
```

**Circular References** (less of an issue in modern browsers, but still important):
```javascript
const person = {};
const company = {};

person.company = company;
company.person = person;

// Modern garbage collectors handle this, but it's still good practice
// to avoid circular references
```

**Why this matters**: Memory leaks cause applications to slow down and eventually crash. Senior engineers actively prevent memory leaks and use tools to identify them.

---

## 9. TYPE SYSTEM & TYPE COERCION

### JavaScript's Type System

JavaScript has 7 primitive types and Objects:

**Primitives**:
- String
- Number
- Boolean
- null
- undefined
- Symbol (ES6)
- BigInt (ES2020)

**Objects**: Everything else (arrays, functions, objects, dates, etc.)

### typeof Operator

```javascript
typeof "hello"      // "string"
typeof 42          // "number"
typeof true        // "boolean"
typeof undefined   // "undefined"
typeof Symbol()    // "symbol"
typeof 123n        // "bigint"
typeof {}          // "object"
typeof []          // "object" (quirk!)
typeof function(){} // "function"
typeof null        // "object" (quirk! null is actually a primitive)
```

### Type Coercion

JavaScript automatically converts types in certain situations.

**String Coercion**:
```javascript
"5" + 2;           // "52" - concatenation
"5" - 2;           // 3 - arithmetic forces number
"hello" + null;    // "hellonull"
null + undefined;  // NaN
```

**Truthy and Falsy**:
```javascript
// Falsy values:
false, 0, -0, 0n, "", null, undefined, NaN

// All other values are truthy
if ("0") { console.log("truthy"); } // "0" string is truthy!
```

**Equality Coercion**:
```javascript
5 == "5"    // true - type coercion
5 === "5"   // false - no coercion

0 == false  // true
0 === false // false

null == undefined   // true
null === undefined  // false
```

### Best Practice: Use === and Avoid Coercion

```javascript
// Bad
if (value == null) { }

// Good
if (value === null || value === undefined) { }

// Better
if (value == null) { } // This is actually safe because == null checks both
```

### Strict Equality Rules

```javascript
// Numbers
5 === 5               // true
5 === 5.0             // true
NaN === NaN           // false (NaN is special!)
0 === -0              // true (in practice, they're equal)

// Objects
{} === {}             // false (different instances)
const obj = {};
obj === obj           // true (same reference)

// Arrays
[1,2,3] === [1,2,3]   // false (different instances)
```

**Why this matters**: Type coercion bugs are common. Understanding the type system and knowing when to use `===` vs `==` prevents bugs and improves code quality.

---

## 10. FUNCTIONAL PROGRAMMING CONCEPTS

### First-Class Functions

Functions are values. You can assign them to variables, pass them as arguments, and return them.

```javascript
// Assign to variable
const greet = function(name) {
  return "Hello, " + name;
};

// Pass as argument
function executeFunc(fn, value) {
  return fn(value);
}

executeFunc(greet, "John"); // "Hello, John"

// Return from function
function createGreeter(greeting) {
  return function(name) {
    return greeting + ", " + name;
  };
}

const sayHi = createGreeter("Hi");
sayHi("Alice"); // "Hi, Alice"
```

### Higher-Order Functions

A function that takes a function as an argument or returns a function.

```javascript
// Takes function as argument
function map(array, fn) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    result.push(fn(array[i]));
  }
  return result;
}

map([1, 2, 3], x => x * 2); // [2, 4, 6]

// Returns function
function compose(f, g) {
  return function(x) {
    return f(g(x));
  };
}

const addTwo = x => x + 2;
const timesThree = x => x * 3;

const addThenMultiply = compose(timesThree, addTwo);
addThenMultiply(5); // (5 + 2) * 3 = 21
```

### Pure Functions

A pure function always produces the same output for the same input and has no side effects.

```javascript
// Pure - same input always gives same output
function add(a, b) {
  return a + b;
}

// Not pure - depends on external state
let multiplier = 2;
function multiply(x) {
  return x * multiplier; // Depends on external variable
}

// Not pure - has side effects
let total = 0;
function addToTotal(x) {
  total += x; // Modifies external state
  return total;
}
```

### Immutability

Avoiding mutation of existing data.

```javascript
// Mutating (not recommended)
const user = { name: "John", age: 30 };
user.age = 31; // Mutation

// Non-mutating (better)
const user = { name: "John", age: 30 };
const updatedUser = { ...user, age: 31 }; // Spread operator

// Array mutation vs non-mutation
const arr = [1, 2, 3];
arr.push(4); // Mutation

const newArr = [...arr, 4]; // Non-mutation
const newArr2 = arr.concat([4]); // Non-mutation
```

### Map, Filter, Reduce

These are essential functional programming methods.

**map**: Transform each element
```javascript
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(x => x * 2); // [2, 4, 6, 8]
```

**filter**: Keep elements that pass test
```javascript
const numbers = [1, 2, 3, 4, 5];
const evens = numbers.filter(x => x % 2 === 0); // [2, 4]
```

**reduce**: Accumulate values
```javascript
const numbers = [1, 2, 3, 4];
const sum = numbers.reduce((acc, curr) => acc + curr, 0); // 10

const grouped = users.reduce((acc, user) => {
  const key = user.department;
  if (!acc[key]) acc[key] = [];
  acc[key].push(user);
  return acc;
}, {});
```

### Currying

Converting a function that takes multiple arguments into a series of functions each taking one argument.

```javascript
// Regular function
function add(a, b, c) {
  return a + b + c;
}

// Curried version
function curriedAdd(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

curriedAdd(1)(2)(3); // 6

// Or with arrow functions
const curriedAdd = a => b => c => a + b + c;
```

**Why this matters**: Functional programming patterns make code more predictable, testable, and easier to reason about. Many libraries use these patterns extensively.

---

## 11. OBJECT-ORIENTED PROGRAMMING IN JAVASCRIPT

### Encapsulation

Bundling data and methods together, hiding internal details.

```javascript
class BankAccount {
  #balance = 0; // Private field (ES2022)

  constructor(owner) {
    this.owner = owner;
  }

  deposit(amount) {
    if (amount > 0) {
      this.#balance += amount;
      return this.#balance;
    }
  }

  withdraw(amount) {
    if (amount > 0 && amount <= this.#balance) {
      this.#balance -= amount;
      return this.#balance;
    }
  }

  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount("John");
account.deposit(100);
console.log(account.getBalance()); // 100
// account.#balance; // SyntaxError - can't access private field
```

### Inheritance

One class inheriting properties and methods from another.

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(this.name + " makes a sound");
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }

  bark() {
    console.log(this.name + " barks");
  }

  // Override parent method
  speak() {
    console.log(this.name + " says woof");
  }
}

const dog = new Dog("Rex", "Labrador");
dog.speak(); // "Rex says woof"
dog.bark();  // "Rex barks"
```

### Polymorphism

Different objects responding to the same method call differently.

```javascript
class Shape {
  area() {
    throw new Error("area() must be implemented");
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  area() {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  area() {
    return this.width * this.height;
  }
}

const shapes = [new Circle(5), new Rectangle(4, 5)];
shapes.forEach(shape => {
  console.log(shape.area()); // Different implementation for each
});
```

### Composition over Inheritance

Sometimes composition (combining objects) is better than inheritance.

```javascript
// Bad inheritance hierarchy
class Vehicle {}
class Car extends Vehicle {}
class Bike extends Vehicle {}
class Plane extends Vehicle {}
// Problem: Plane shouldn't inherit from Vehicle

// Better: Use composition
const canFly = {
  fly() { console.log("Flying"); }
};

const canDrive = {
  drive() { console.log("Driving"); }
};

const Plane = Object.assign({}, canFly);
const Car = Object.assign({}, canDrive);
const FlyingCar = Object.assign({}, canFly, canDrive);
```

**Why this matters**: OOP patterns make large codebases maintainable. Understanding when to use inheritance vs composition is crucial for good design.

---

## 12. TEMPLATE LITERALS & STRING MANIPULATION

### Template Literals

Using backticks for string interpolation.

```javascript
const name = "John";
const age = 30;

const message = `Hello, my name is ${name} and I'm ${age} years old`;
// "Hello, my name is John and I'm 30 years old"

// Expressions
const result = `2 + 2 = ${2 + 2}`; // "2 + 2 = 4"

// Multi-line strings
const text = `This is
a multi-line
string`;
```

### Tagged Template Literals

Functions that process template literals.

```javascript
function highlight(strings, ...values) {
  let result = "";
  for (let i = 0; i < strings.length; i++) {
    result += strings[i];
    if (i < values.length) {
      result += `<mark>${values[i]}</mark>`;
    }
  }
  return result;
}

const name = "Alice";
const age = 25;
const result = highlight`Hello ${name}, you are ${age} years old`;
// "Hello <mark>Alice</mark>, you are <mark>25</mark> years old"
```

---

## 13. DESTRUCTURING

### Object Destructuring

```javascript
const person = {
  name: "John",
  age: 30,
  city: "New York",
  job: "Engineer"
};

// Extract specific properties
const { name, age } = person;
console.log(name); // "John"
console.log(age);  // 30

// Rename properties
const { name: personName, age: personAge } = person;

// Default values
const { name, country = "USA" } = person;
console.log(country); // "USA"

// Nested destructuring
const person = {
  name: "John",
  address: {
    city: "New York",
    zip: "10001"
  }
};

const { address: { city, zip } } = person;
```

### Array Destructuring

```javascript
const colors = ["red", "green", "blue"];

// Extract elements
const [first, second, third] = colors;
console.log(first); // "red"

// Skip elements
const [first, , third] = colors; // Skip "green"

// Rest operator
const [first, ...rest] = colors;
console.log(rest); // ["green", "blue"]

// Swap values
let a = 1, b = 2;
[a, b] = [b, a];
console.log(a, b); // 2, 1

// Function parameters
function logCoords([x, y]) {
  console.log(`(${x}, ${y})`);
}

logCoords([10, 20]); // "(10, 20)"
```

**Why destructuring matters**: It makes code more concise and readable. It's a modern JavaScript essential.

---

## 14. REST & SPREAD OPERATORS

### Spread Operator (...)

Expands iterables into individual elements.

```javascript
// Array spreading
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Object spreading
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3 };
const merged = { ...obj1, ...obj2 }; // { a: 1, b: 2, c: 3 }

// Override properties
const user = { name: "John", age: 30 };
const updated = { ...user, age: 31 }; // { name: "John", age: 31 }

// Function arguments
function sum(a, b, c) {
  return a + b + c;
}

const numbers = [1, 2, 3];
sum(...numbers); // 6
```

### Rest Operator (...)

Collects remaining elements into an array.

```javascript
// Function parameters
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

sum(1, 2, 3, 4, 5); // 15

// Destructuring
const [first, ...rest] = [1, 2, 3, 4];
console.log(rest); // [2, 3, 4]

// Object destructuring
const { name, ...otherProps } = { name: "John", age: 30, city: "NY" };
console.log(otherProps); // { age: 30, city: "NY" }
```

**Why this matters**: Spread and rest operators make code cleaner and more expressive. They're essential for modern JavaScript patterns.

---

## 15. ARRAY METHODS (DEEP DIVE)

### Array.prototype.map()
```javascript
// Not just for transformation
const users = [
  { id: 1, name: "John" },
  { id: 2, name: "Jane" }
];

const names = users.map(u => u.name); // ["John", "Jane"]

// Creates new array - original unchanged
const doubled = [1, 2, 3].map(x => x * 2); // [2, 4, 6]
```

### Array.prototype.filter()
```javascript
const numbers = [1, 2, 3, 4, 5];
const evens = numbers.filter(n => n % 2 === 0); // [2, 4]

// Can filter based on index
const words = ["a", "ab", "abc"];
const short = words.filter((w, i) => w.length > 1); // ["ab", "abc"]
```

### Array.prototype.reduce()
```javascript
// Sum
const sum = [1, 2, 3].reduce((acc, curr) => acc + curr, 0); // 6

// Grouping
const items = [
  { category: "A", value: 1 },
  { category: "B", value: 2 },
  { category: "A", value: 3 }
];

const grouped = items.reduce((acc, item) => {
  acc[item.category] = (acc[item.category] || 0) + item.value;
  return acc;
}, {}); // { A: 4, B: 2 }

// Flattening
const nested = [[1, 2], [3, 4]];
const flat = nested.reduce((acc, arr) => [...acc, ...arr], []); // [1, 2, 3, 4]
```

### Array.prototype.find() & findIndex()
```javascript
const users = [
  { id: 1, name: "John" },
  { id: 2, name: "Jane" }
];

const user = users.find(u => u.id === 2);
// { id: 2, name: "Jane" }

const index = users.findIndex(u => u.id === 2); // 1
```

### Array.prototype.some() & every()
```javascript
const numbers = [1, 2, 3, 4, 5];

numbers.some(n => n > 3); // true - at least one number > 3
numbers.every(n => n > 0); // true - all numbers > 0
```

### Array.prototype.includes()
```javascript
const fruits = ["apple", "banana", "orange"];
fruits.includes("banana"); // true

// Works with objects if same reference
const users = [{id: 1}];
users.includes({id: 1}); // false - different object
```

### Array.prototype.flat() & flatMap()
```javascript
const nested = [1, [2, [3, [4]]]];
nested.flat(); // [1, 2, [3, [4]]]
nested.flat(2); // [1, 2, 3, [4]]
nested.flat(Infinity); // [1, 2, 3, 4]

// flatMap
const words = ["hello", "world"];
words.flatMap(word => word.split("")); // ["h","e","l","l","o","w"...]
```

**Why this matters**: Array methods are fundamental to modern JavaScript. Understanding them deeply makes you write cleaner, more efficient code.

---

## 16. OBJECT METHODS & PROPERTY DESCRIPTORS

### Object.assign()
```javascript
const source = { a: 1, b: 2 };
const target = {};
Object.assign(target, source); // { a: 1, b: 2 }

// Multiple sources
Object.assign(target, { a: 1 }, { b: 2 }, { c: 3 });
```

### Object.create()
```javascript
const parent = {
  greet() {
    return "Hello from parent";
  }
};

const child = Object.create(parent);
child.greet(); // "Hello from parent"
```

### Object.keys(), values(), entries()
```javascript
const obj = { a: 1, b: 2, c: 3 };

Object.keys(obj); // ["a", "b", "c"]
Object.values(obj); // [1, 2, 3]
Object.entries(obj); // [["a", 1], ["b", 2], ["c", 3]]

// Convert to object from entries
Object.fromEntries([["a", 1], ["b", 2]]); // { a: 1, b: 2 }
```

### Object.defineProperty() & Property Descriptors
```javascript
const obj = {};

Object.defineProperty(obj, "name", {
  value: "John",
  writable: false,    // Can't be changed
  enumerable: true,   // Shows up in Object.keys()
  configurable: false // Can't be reconfigured
});

obj.name = "Jane"; // Fails silently (or throws error in strict mode)
```

### Object.freeze() & Object.seal()
```javascript
const user = { name: "John", age: 30 };

Object.freeze(user); // Can't add, delete, or modify
user.name = "Jane"; // Fails

Object.seal(user); // Can modify but can't add/delete
user.name = "Jane"; // Works
user.email = "john@example.com"; // Fails
```

**Why this matters**: Understanding property descriptors and object mutation control helps prevent bugs and protect data integrity.

---

## 17. MODERN JAVASCRIPT FEATURES (ES6+)

### Arrow Functions

```javascript
// Regular function
function add(a, b) {
  return a + b;
}

// Arrow function
const add = (a, b) => {
  return a + b;
};

// Concise arrow function
const add = (a, b) => a + b;

// Single parameter (parentheses optional)
const square = x => x * x;

// No parameters
const random = () => Math.random();
```

**Important**: Arrow functions don't have their own `this`, they don't have a `prototype`, and they can't be used as constructors.

### Default Parameters

```javascript
function greet(name = "Guest", greeting = "Hello") {
  return `${greeting}, ${name}`;
}

greet(); // "Hello, Guest"
greet("John"); // "Hello, John"
greet("John", "Hi"); // "Hi, John"
```

### for...of Loop

```javascript
const arr = [1, 2, 3];
for (const value of arr) {
  console.log(value); // 1, 2, 3
}

const str = "hello";
for (const char of str) {
  console.log(char); // h, e, l, l, o
}
```

### Enhanced Object Literals

```javascript
const name = "John";
const age = 30;

// Before
const person = {
  name: name,
  age: age,
  greet: function() {
    return "Hello";
  }
};

// After
const person = {
  name,
  age,
  greet() {
    return "Hello";
  }
};
```

### Computed Property Names

```javascript
const key = "dynamicKey";
const obj = {
  [key]: "value",
  [1 + 2]: "three"
};

console.log(obj.dynamicKey); // "value"
console.log(obj[3]); // "three"
```

### Optional Chaining (?.)

```javascript
const user = {
  name: "John",
  address: {
    city: "New York"
  }
};

console.log(user?.address?.city); // "New York"
console.log(user?.phone?.number); // undefined (no error)

// Works with arrays and functions
const arr = [1, 2, 3];
console.log(arr?.[0]); // 1
console.log(arr?.[10]); // undefined

const obj = {};
console.log(obj.greet?.()); // undefined (no error)
```

### Nullish Coalescing Operator (??)

```javascript
// Returns right value only if left is null or undefined
const value = null ?? "default";
console.log(value); // "default"

// Different from ||
const zero = 0 || "default";
console.log(zero); // "default"

const zero2 = 0 ?? "default";
console.log(zero2); // 0
```

### Logical Assignment Operators (ES2021)

```javascript
let x = 0;
x ||= 10; // x = 0 || 10 = 10

let y = null;
y ??= 5; // y = null ?? 5 = 5

let obj = { a: 1 };
obj.b &&= obj.a + 1; // obj.b = obj.b && (obj.a + 1) = true && 2
```

---

## 18. MODULES (IMPORT/EXPORT)

### CommonJS (Node.js, older)

```javascript
// Export
module.exports = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b
};

// Import
const math = require('./math');
math.add(5, 3); // 8
```

### ES6 Modules (Modern Standard)

```javascript
// Export default
export default function add(a, b) {
  return a + b;
}

// Import default
import add from './math.js';

// Named exports
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// Named imports
import { add, subtract } from './math.js';

// Alias imports
import { add as addition } from './math.js';

// Import everything
import * as math from './math.js';
math.add(5, 3);

// Mixed
export default function main() { }
export const helper = () => { };

import main, { helper } from './module.js';
```

### Circular Dependencies

```javascript
// a.js
import { funcB } from './b.js';

export const funcA = () => {
  funcB();
};

// b.js
import { funcA } from './a.js';

export const funcB = () => {
  funcA();
};

// This works but can cause issues - avoid circular dependencies
```

**Why this matters**: Understanding module systems is crucial for code organization and avoiding namespace pollution. Modern codebases use ES6 modules.

---

## 19. DESIGN PATTERNS

### Singleton Pattern

A class with only one instance.

```javascript
const Singleton = (() => {
  let instance;

  function createInstance() {
    return {
      value: Math.random()
    };
  }

  return {
    getInstance: () => {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

const instance1 = Singleton.getInstance();
const instance2 = Singleton.getInstance();
console.log(instance1 === instance2); // true
```

### Observer Pattern

Allows objects to subscribe to events.

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }

  emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }
}

const emitter = new EventEmitter();

emitter.on('message', (data) => {
  console.log("Received:", data);
});

emitter.emit('message', 'Hello');
```

### Factory Pattern

Creating objects without specifying exact classes.

```javascript
class Animal {
  speak() {
    throw new Error("speak() must be implemented");
  }
}

class Dog extends Animal {
  speak() {
    return "Woof!";
  }
}

class Cat extends Animal {
  speak() {
    return "Meow!";
  }
}

class AnimalFactory {
  static createAnimal(type) {
    switch(type) {
      case 'dog': return new Dog();
      case 'cat': return new Cat();
      default: throw new Error("Unknown animal");
    }
  }
}

const dog = AnimalFactory.createAnimal('dog');
console.log(dog.speak()); // "Woof!"
```

### Decorator Pattern

Adding functionality to objects dynamically.

```javascript
class Coffee {
  cost() {
    return 5;
  }

  description() {
    return "Coffee";
  }
}

function addMilk(coffee) {
  const originalCost = coffee.cost;
  const originalDescription = coffee.description;

  coffee.cost = function() {
    return originalCost.call(this) + 2;
  };

  coffee.description = function() {
    return originalDescription.call(this) + ", Milk";
  };

  return coffee;
}

let coffee = new Coffee();
coffee = addMilk(coffee);
console.log(coffee.cost()); // 7
console.log(coffee.description()); // "Coffee, Milk"
```

**Why this matters**: Design patterns are reusable solutions to common problems. Senior engineers use them to write scalable, maintainable code.

---

## 20. THIS BINDING & CALL, APPLY, BIND (DEEP DIVE)

### The Problem with this

```javascript
const person = {
  name: "John",
  greet: function() {
    console.log(this.name); // What is this?
  }
};

person.greet(); // "John" - this is person

const greet = person.greet;
greet(); // undefined - this is global/undefined
```

### call()

Calls a function with a specific `this` value.

```javascript
function introduce(greeting, punctuation) {
  console.log(greeting + ", I'm " + this.name + punctuation);
}

const person = { name: "Alice" };

introduce.call(person, "Hello", "!"); // "Hello, I'm Alice!"
```

### apply()

Like `call()` but arguments as array.

```javascript
function sum(a, b, c) {
  return a + b + c;
}

const numbers = [1, 2, 3];
sum.apply(null, numbers); // 6

// Useful with built-in functions
const max = Math.max.apply(null, [5, 2, 8, 1]); // 8
```

### bind()

Returns a new function with bound `this`.

```javascript
const person = {
  name: "Bob",
  greet: function() {
    return "Hello, " + this.name;
  }
};

const greet = person.greet.bind(person);
greet(); // "Hello, Bob"

// Can bind partial arguments (currying)
function add(a, b) {
  return a + b;
}

const addFive = add.bind(null, 5);
addFive(3); // 8
```

### Practical Scenarios

**Event Handlers**:
```javascript
class Button {
  constructor(label) {
    this.label = label;
  }

  click() {
    console.log(this.label + " clicked");
  }
}

const btn = new Button("Submit");

// Problem: loses this
document.querySelector('button').addEventListener('click', btn.click);
// Output: "undefined clicked"

// Solution 1: bind
document.querySelector('button').addEventListener('click', btn.click.bind(btn));

// Solution 2: arrow function
document.querySelector('button').addEventListener('click', () => btn.click());
```

**Partial Application**:
```javascript
function curry(fn) {
  return function(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    }
    return (...nextArgs) => curry(fn)(...args, ...nextArgs);
  };
}

function add(a, b, c) {
  return a + b + c;
}

const curriedAdd = curry(add);
curriedAdd(1)(2)(3); // 6
curriedAdd(1, 2)(3); // 6
```

**Why this matters**: Many bugs stem from misunderstanding `this`. Mastering call/apply/bind helps you write correct, reusable code.

---

## 21. REGULAR EXPRESSIONS

### Basic Pattern Matching

```javascript
const regex1 = /hello/;
const regex2 = new RegExp('hello');

"hello world".match(/hello/); // ["hello"]
"hello world".match(/goodbye/); // null

// Flags
/hello/g   // global - find all matches
/hello/i   // case-insensitive
/hello/m   // multiline
/hello/gi  // combine multiple flags
```

### Common Methods

```javascript
const text = "The price is 100 dollars";

// test() - returns boolean
/\d+/.test(text); // true

// match() - returns array of matches
text.match(/\d+/); // ["100"]
text.match(/\d+/g); // ["100"]

// replace() - replace first match
text.replace(/\d+/, "200"); // "The price is 200 dollars"

// replaceAll() - replace all matches
text.replaceAll(/\d+/g, "200");

// split() - split by pattern
"a1b2c3".split(/\d/); // ["a", "b", "c", ""]
```

### Common Patterns

```javascript
// Email (basic)
/^[^\s@]+@[^\s@]+\.[^\s@]+$/

// Phone (US)
/^\d{3}-\d{3}-\d{4}$/

// URL
/^https?:\/\/.+/

// Alphanumeric
/^[a-zA-Z0-9]+$/

// Password (min 8 chars, uppercase, lowercase, number)
/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/
```

**Why this matters**: Regular expressions are powerful for validation and text processing. Understanding them deeply helps with data validation and parsing.

---

## 22. JSON

### JSON Methods

```javascript
// Parse JSON string to object
const json = '{"name":"John","age":30}';
const obj = JSON.parse(json);
console.log(obj.name); // "John"

// Convert object to JSON string
const person = { name: "Alice", age: 25 };
const str = JSON.stringify(person);
console.log(str); // '{"name":"Alice","age":25}'

// Pretty print
const formatted = JSON.stringify(person, null, 2);
// {
//   "name": "Alice",
//   "age": 25
// }
```

### Replacer Function

```javascript
const user = {
  name: "John",
  password: "secret123",
  email: "john@example.com"
};

// Filter sensitive data
const json = JSON.stringify(user, (key, value) => {
  if (key === 'password') return undefined;
  return value;
});

console.log(json); // '{"name":"John","email":"john@example.com"}'
```

### Reviver Function

```javascript
const json = '{"date":"2024-01-20T00:00:00Z","value":100}';

const obj = JSON.parse(json, (key, value) => {
  if (key === 'date') {
    return new Date(value);
  }
  return value;
});

console.log(obj.date instanceof Date); // true
```

**Why this matters**: JSON is everywhere. Understanding parsing, serialization, and edge cases is essential for working with APIs and data.

---

## 23. PERFORMANCE & OPTIMIZATION

### Debouncing

Delay function execution until after a specified wait time.

```javascript
function debounce(func, wait) {
  let timeout;
  return function(...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func(...args), wait);
  };
}

const search = debounce((query) => {
  console.log("Searching for:", query);
}, 300);

// Will only execute once after 300ms of no calls
input.addEventListener('input', (e) => search(e.target.value));
```

### Throttling

Execute function at most once every specified interval.

```javascript
function throttle(func, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func(...args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

const handleScroll = throttle(() => {
  console.log("Scrolling");
}, 1000);

window.addEventListener('scroll', handleScroll);
```

### Memoization

Cache function results.

```javascript
function memoize(func) {
  const cache = {};
  return function(...args) {
    const key = JSON.stringify(args);
    if (key in cache) {
      return cache[key];
    }
    const result = func(...args);
    cache[key] = result;
    return result;
  };
}

const expensiveFunc = (n) => {
  console.log("Computing...");
  return n * n;
};

const memoized = memoize(expensiveFunc);
memoized(5); // Computes
memoized(5); // Returns from cache
```

### Performance Tips

1. **Avoid global scope pollution**
2. **Use const/let instead of var**
3. **Minimize DOM manipulation**
4. **Use event delegation** instead of multiple listeners
5. **Lazy load images and content**
6. **Bundle and minify code**
7. **Use Web Workers for heavy computation**
8. **Optimize loops**:

```javascript
// Bad
for (let i = 0; i < array.length; i++) {
  // Do something
}

// Better (cache length)
const len = array.length;
for (let i = 0; i < len; i++) {
  // Do something
}
```

**Why this matters**: Performance directly impacts user experience. Optimized code is crucial for production systems.

---

## 24. ERROR HANDLING & DEBUGGING

### Try-Catch-Finally

```javascript
try {
  riskyOperation();
} catch (error) {
  console.log("Error caught:", error.message);
  console.log("Stack trace:", error.stack);
} finally {
  console.log("This always runs");
}
```

### Custom Errors

```javascript
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}

try {
  throw new ValidationError("Invalid email");
} catch (error) {
  if (error instanceof ValidationError) {
    console.log("Validation failed:", error.message);
  }
}
```

### Async Error Handling

```javascript
// Promises
fetch('/data')
  .then(response => response.json())
  .catch(error => {
    console.log("Error:", error);
  });

// Async/await
async function getData() {
  try {
    const response = await fetch('/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.log("Error:", error);
    throw error;
  }
}
```

### Unhandled Promise Rejection

```javascript
// Bad - unhandled rejection
Promise.reject(new Error("Oops"));

// Good - handle rejection
Promise.reject(new Error("Oops"))
  .catch(error => {
    console.log("Handled:", error);
  });

// Global handler
window.addEventListener('unhandledrejection', (event) => {
  console.log("Unhandled rejection:", event.reason);
  event.preventDefault(); // Prevent crash
});
```

**Why this matters**: Proper error handling makes applications robust and user-friendly. Senior engineers build comprehensive error handling strategies.

---

## 25. SECURITY BEST PRACTICES

### XSS Prevention

```javascript
// Bad - vulnerable to XSS
element.innerHTML = userInput;

// Good
element.textContent = userInput;

// Or escape HTML
function escapeHTML(html) {
  const div = document.createElement('div');
  div.textContent = html;
  return div.innerHTML;
}
```

### SQL Injection Prevention

```javascript
// Bad
const query = "SELECT * FROM users WHERE id = " + userId;

// Good - use parameterized queries
const query = "SELECT * FROM users WHERE id = ?";
db.query(query, [userId]);
```

### Sensitive Data

```javascript
// Bad - storing passwords
const user = {
  name: "John",
  password: "myPassword123"
};

// Good - hash passwords
const hashedPassword = bcrypt.hashSync(password, 10);

// Never log sensitive data
console.log(userData); // Could expose passwords/tokens

// Use environment variables
const API_KEY = process.env.API_KEY;
```

### CORS & CSRF

```javascript
// Check origin (server-side)
if (req.headers.origin !== 'https://trusted-domain.com') {
  res.status(403).send('Forbidden');
}

// CSRF tokens
const csrfToken = req.session.csrfToken;
// Validate token on form submission
```

**Why this matters**: Security breaches can be catastrophic. Understanding common vulnerabilities helps you write secure code.

---

## 26. TESTING CONCEPTS

### Unit Testing

```javascript
// Simple example
function add(a, b) {
  return a + b;
}

// Test
function testAdd() {
  console.assert(add(2, 3) === 5, "add(2,3) should equal 5");
  console.assert(add(-1, 1) === 0, "add(-1,1) should equal 0");
}

testAdd();
```

### Test Frameworks (Jest, Mocha, etc.)

```javascript
describe('Calculator', () => {
  it('should add two numbers', () => {
    expect(add(2, 3)).toBe(5);
  });

  it('should handle negative numbers', () => {
    expect(add(-1, 1)).toBe(0);
  });
});
```

### Testing Async Code

```javascript
it('should fetch data', async () => {
  const data = await fetchData();
  expect(data).toBeDefined();
});

it('should handle errors', (done) => {
  fetchData().catch(error => {
    expect(error).toBeDefined();
    done();
  });
});
```

### Mocking

```javascript
jest.mock('./api', () => ({
  fetchUser: jest.fn(() => Promise.resolve({ name: 'John' }))
}));

it('should fetch user', async () => {
  const user = await fetchUser();
  expect(user.name).toBe('John');
});
```

**Why this matters**: Tests catch bugs, provide documentation, and enable refactoring with confidence. Senior engineers write comprehensive test suites.

---

## 27. WEB APIS

### DOM Manipulation

```javascript
// Query
const el = document.querySelector('.class');
const els = document.querySelectorAll('.class');

// Create and append
const div = document.createElement('div');
div.textContent = 'Hello';
parent.appendChild(div);

// Remove
el.remove();

// Classes
el.classList.add('active');
el.classList.remove('active');
el.classList.toggle('active');

// Attributes
el.setAttribute('data-id', '123');
el.getAttribute('data-id'); // '123'
el.removeAttribute('data-id');
```

### Events

```javascript
// Listen
element.addEventListener('click', (event) => {
  console.log(event.target);
  console.log(event.currentTarget);
});

// Stop propagation
event.stopPropagation();

// Prevent default
event.preventDefault();

// Event delegation
parent.addEventListener('click', (event) => {
  if (event.target.matches('.item')) {
    // Handle click on .item
  }
});

// Remove listener
function handler() {}
element.addEventListener('click', handler);
element.removeEventListener('click', handler);
```

### Fetch API

```javascript
// GET
fetch('/api/users')
  .then(response => response.json())
  .then(data => console.log(data));

// POST
fetch('/api/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ name: 'John' })
})
  .then(response => response.json());

// Async version
async function getUsers() {
  const response = await fetch('/api/users');
  return response.json();
}
```

### LocalStorage

```javascript
// Store
localStorage.setItem('key', 'value');

// Retrieve
const value = localStorage.getItem('key');

// Remove
localStorage.removeItem('key');

// Clear all
localStorage.clear();

// With JSON
localStorage.setItem('user', JSON.stringify({ name: 'John' }));
const user = JSON.parse(localStorage.getItem('user'));
```

**Why this matters**: Web APIs are essential for building interactive applications. Understanding them deeply enables you to build rich user experiences.

