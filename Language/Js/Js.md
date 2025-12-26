# JavaScript — Syntax & Basics (Easy & Example)

Quick reference to core JS syntax with short examples to memorize.

---

## Variables (var, let, const)

- var: function-scoped, can be re-declared.
- let: block-scoped, can be reassigned.
- const: block-scoped, cannot be reassigned (reference can mutate).

```javascript
// var (function scoped)
function f(){
  var x = 1;
  if(true){ var x = 2; console.log(x); } // 2
  console.log(x); // 2
}

// let/const (block scoped)
{
  let a = 1;
  a = 2;           // OK
  const b = 3;
  // b = 4;        // Error
}
```

Tip: Prefer let/const. Use const by default.

---

## Data types

Primitive types: string, number, boolean, null, undefined, symbol, bigint
Composite: object (arrays, functions, objects)

```javascript
typeof "hi"       // "string"
typeof 12         // "number"
typeof true       // "boolean"
typeof null       // "object"   // historical JS quirk
typeof undefined  // "undefined"
typeof Symbol()   // "symbol"
typeof 10n        // "bigint"

const obj = {a:1}; // "object"
const arr = [1,2]; // "object"
```

---

## Operators

Arithmetic:

```javascript
5 + 2    // 7
5 - 2    // 3
5 * 2    // 10
5 / 2    // 2.5
5 % 2    // 1
2 ** 3   // 8  (exponent)
```

Comparison:

```javascript
5 == "5"   // true  (loose)
5 === "5"  // false (strict)
5 > 3      // true
3 <= 3     // true
```

Logical:

```javascript
true && false   // false
true || false   // true
!true           // false
// Short-circuit:
null || "default" // "default"
```

Other useful:

```javascript
++i, i++, --i, i--   // increment/decrement
a += 2               // shorthand assignment
```

---

## Type coercion & conversions

Implicit coercion examples:

```javascript
"5" + 2      // "52"   (string concatenation)
"5" - 2      // 3      (numbers coerced)
true + 1     // 2
[] + {}      // "[object Object]"  (weird)
```

Equality pitfalls:

```javascript
0 == false    // true
0 === false   // false
null == undefined // true
null === undefined // false
```

Explicit conversions:

```javascript
Number("42")      // 42
String(42)        // "42"
Boolean("")       // false
parseInt("12px")  // 12
BigInt(123)       // 123n
```

Best practice: use === and explicit conversions.

---

## Small cheatsheet to memorize

- var = function scope, let/const = block scope.
- const for constants, let for changeable values.
- typeof null === "object" (quirk).
- - with a string => concatenation; -, *, / force numeric coercion.
- Use === to avoid unexpected coercion.

---

Examples to try in console:

```javascript
let s = "10";
console.log(s + 5);       // "105"
console.log(s - 5);       // 5
console.log(Boolean("0"));// true
console.log(typeof null); // "object"
```

---

## Control Flow

### if / else

- Use for conditional branching. Condition is truthy/falsy.

```javascript
const x = 10;
if (x > 10) {
  console.log("greater");
} else if (x === 10) {
  console.log("equal"); // printed
} else {
  console.log("less");
}
```

- Ternary operator for short conditions:

```javascript
const status = x > 10 ? "gt" : "leq";
```

### switch

- Use when checking one value against many cases.

```javascript
const day = 3;
switch (day) {
  case 1: console.log("Mon"); break;
  case 2: console.log("Tue"); break;
  case 3: console.log("Wed"); break; // printed
  default: console.log("Unknown");
}
```

- Remember to use break to avoid fall-through (or fall-through intentionally).

---

## Loops

### for

- Classic counted loop.

```javascript
for (let i = 0; i < 5; i++) {
  console.log(i); // 0..4
}
```

### while

- Loop while condition is true.

```javascript
let i = 0;
while (i < 3) {
  console.log(i);
  i++;
}
```

### do...while

- Executes body at least once.

```javascript
let j = 0;
do {
  console.log(j);
  j++;
} while (j < 2);
```

### for...of

- Iterate over iterable values (arrays, strings, maps values).

```javascript
const arr = ["a", "b", "c"];
for (const v of arr) {
  console.log(v); // a b c
}

for (const ch of "hi") console.log(ch); // h i
```

### for...in

- Iterate over enumerable property keys (objects, array indices). Use carefully for arrays.

```javascript
const obj = {a:1, b:2};
for (const key in obj) {
  console.log(key, obj[key]); // a 1  |  b 2
}

const arr = [10, 20];
for (const idx in arr) {
  console.log(idx, arr[idx]); // "0" 10 | "1" 20
}
```

### break & continue

- break exits loop; continue skips to next iteration.

```javascript
for (let i = 0; i < 5; i++) {
  if (i === 2) continue; // skip 2
  if (i === 4) break;    // stop at 4
  console.log(i);
}
```

Cheats:

- Use for...of for arrays when you need values.
- Use for...in for object keys (or array indices if intended).
- Prefer clear loop exits and avoid mutating loop variable externally.

---

## Functions

Concise guide with examples to memorize.

### Function declarations

- Named function. Hoisted (can be called before definition).

```javascript
function add(a, b) {
  return a + b;
}
console.log(add(2, 3)); // 5
```

### Function expressions

- Function assigned to a variable. Not hoisted the same way.

```javascript
const multiply = function (a, b) {
  return a * b;
};
console.log(multiply(2, 3)); // 6
```

### Arrow functions

- Shorter syntax, lexical this, no arguments object.

```javascript
const square = x => x * x;          // implicit return
const sum = (a, b) => { return a + b; }; // block body needs return

console.log(square(4)); // 16
```

Note: Arrow functions do not have their own this/bind and cannot be used as constructors.

### Parameters & default values

- Default values, rest parameters, and parameter destructuring.

```javascript
function greet(name = "Guest") {
  return `Hello, ${name}!`;
}
console.log(greet());         // Hello, Guest!
console.log(greet("Alice"));  // Hello, Alice!

function total(...nums) {               // rest collects args
  return nums.reduce((s, n) => s + n, 0);
}
console.log(total(1,2,3)); // 6

function coord({x = 0, y = 0} = {}) {   // destructuring + default
  return `x:${x}, y:${y}`;
}
console.log(coord({x:5})); // x:5, y:0
```

### Return values

- Functions return undefined if no return. Arrow concise bodies return expression value implicitly.

```javascript
function noReturn() {}
console.log(noReturn()); // undefined

const inc = n => n + 1; // implicit return
console.log(inc(4)); // 5
```

Quick tips:

- Use function declarations when you need hoisting; use expressions/arrow for predictable scoping.
- Prefer arrow functions for short callbacks; prefer normal functions when you need dynamic this or arguments.
- Use default and rest params for safer, clearer APIs.

## Scope

Quick, easy-to-memorize notes with examples.

### Global vs Local scope

- Global: declared outside any function/block — accessible everywhere (in Node use globalThis; in browser use window/globalThis).
- Local (function) scope: variables declared inside a function are not visible outside.

```javascript
// global
var gVar = "globalVar";
let gLet = "globalLet";

function demo() {
  // local to demo()
  var localVar = "local";
  console.log(gVar, gLet, localVar); // globalVar globalLet local
}

demo();
console.log(typeof localVar); // "undefined" (localVar not visible here)
```

### Block scope (let, const)

- let and const are block-scoped: only visible inside { ... }.
- var is NOT block-scoped (it is function-scoped).

```javascript
if (true) {
  let b = 1;
  const c = 2;
  var a = 3; // function-scoped
}
console.log(typeof b); // "undefined"
console.log(typeof c); // "undefined"
console.log(a);        // 3  (var leaked out of block)
```

Tip: prefer let/const to avoid accidental leaks; use const by default.

### Hoisting

- Declarations are moved to the top of their scope at runtime (hoisted).
  - Function declarations are hoisted (you can call them before they appear).
  - var declarations are hoisted but initialized to undefined.
  - let and const are hoisted but not initialized — accessing them before declaration causes a ReferenceError (Temporal Dead Zone).

```javascript
// Function declaration hoisted
console.log(sum(2,3)); // 5
function sum(a,b){ return a + b; }

// var hoisting
console.log(x); // undefined
var x = 10;

// let/const and TDZ
console.log(y); // ReferenceError: Cannot access 'y' before initialization
let y = 5;
```

Quick rules to remember:

- Functions (declarations) — hoisted and usable before declaration.
- var — declaration hoisted, value = undefined until assignment.
- let/const — declared in TDZ until evaluated; do not access before declaration.
- Use let/const to write safer, clearer scope-aware code.

---

## Objects

Easy notes + small examples to memorize.

- Object literal: quick way to group related data and behavior.

```javascript
const person = {
  name: "Alice",
  age: 30,
  city: "NY",
  // method (short syntax)
  greet() {
    console.log(`Hi, I'm ${this.name}`);
  }
};

person.greet(); // Hi, I'm Alice
```

- Property access: dot vs bracket
  - Dot: use when you know the identifier (safe, concise).
    person.name  // "Alice"
  - Bracket: use for dynamic keys or keys with spaces/symbols.

```javascript
const key = "city";
console.log(person[key]);     // "NY"
person["home address"] = "Street 1";
```

- Methods: functions stored in objects. Use shorthand for readability.

```javascript
const calculator = {
  x: 10,
  y: 5,
  add() { return this.x + this.y; },    // uses this
  mul: function() { return this.x * this.y; } // older syntax
};
console.log(calculator.add()); // 15
```

- this keyword in objects:
  - In a regular method, this refers to the owning object (the receiver).
  - Arrow functions do NOT have their own this — they use the surrounding (lexical) this. Avoid arrow functions as object methods when you need this to refer to the object.

```javascript
const counter = {
  count: 0,
  inc() { this.count++; console.log(this.count); },        // works: this -> counter
  incArrow: () => { console.log(this.count); }            // NOT recommended: this is lexical (not counter)
};

counter.inc();      // 1
counter.incArrow(); // likely undefined or not the object count — arrow won't bind to counter
```

- Useful operations:

```javascript
// add/update
person.age = 31;

// delete
delete person.city;

// check existence
console.log("age" in person);         // true/false

// iterate keys/values
Object.keys(person).forEach(k => console.log(k, person[k]));
```

Memorize:

- Use object literals for grouped state + behavior.
- Dot access for known keys, bracket for dynamic keys.
- Use regular methods when you need this to refer to the object; avoid arrow methods for that case.

---

## Arrays

Quick, easy examples showing what each method returns and console output.

```javascript
const nums = [1, 2, 3, 4, 5];

// map: returns a new array with function applied to each item
const doubled = nums.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]
// original unchanged
console.log(nums);    // [1, 2, 3, 4, 5]

// forEach: executes a function for each item, returns undefined
let sumForEach = 0;
const ret = nums.forEach(n => { sumForEach += n; });
console.log(sumForEach); // 15
console.log(ret);        // undefined

// filter: returns a new array with items that pass the predicate
const evens = nums.filter(n => n % 2 === 0);
console.log(evens); // [2, 4]

// find: returns the first element that matches, or undefined
const firstGt2 = nums.find(n => n > 2);
console.log(firstGt2); // 3
const notFound = nums.find(n => n > 10);
console.log(notFound); // undefined

// some: returns true if any element matches predicate
const anyEven = nums.some(n => n % 2 === 0);
console.log(anyEven); // true

// every: returns true if all elements match predicate
const allPositive = nums.every(n => n > 0);
console.log(allPositive); // true

// reduce: reduces array to single value (accumulator). Common for sums, products, etc.
const sum = nums.reduce((acc, n) => acc + n, 0); // 0 is initial value
console.log(sum); // 15

// reduce for more complex result (grouping)
const grouped = nums.reduce((acc, n) => {
  const key = n % 2 === 0 ? 'even' : 'odd';
  acc[key] = acc[key] || [];
  acc[key].push(n);
  return acc;
}, {});
console.log(grouped); // { odd: [1,3,5], even: [2,4] }

// findIndex: returns index of first match or -1
console.log(nums.findIndex(n => n === 3)); // 2
console.log(nums.findIndex(n => n === 42)); // -1

// includes: checks presence (uses ===)
console.log(nums.includes(3)); // true
console.log(nums.includes(42)); // false

// slice vs splice
const a = [0,1,2,3,4];
console.log(a.slice(1,3)); // [1,2]  (non-mutating)
const b = a.slice();       // copy
console.log(b);            // [0,1,2,3,4]

// splice mutates array: remove 2 items from index 1
const c = [0,1,2,3];
c.splice(1,2);
console.log(c); // [0,3]

// concat: returns new array
console.log([1,2].concat([3,4])); // [1,2,3,4]

// indexOf / lastIndexOf
console.log([1,2,3,2].indexOf(2));     // 1
console.log([1,2,3,2].lastIndexOf(2)); // 3
```

### Spread and Rest

```javascript
// Spread: expand elements (creates shallow copies)
const arr1 = [1,2];
const arr2 = [...arr1, 3, 4];
console.log(arr2); // [1,2,3,4]

// Spread copy (shallow)
const original = [{x:1}];
const copy = [...original];
console.log(copy === original);        // false (different arrays)
console.log(copy[0] === original[0]);  // true  (same referenced object)

// Use spread to merge arrays or pass as args
function sum3(a,b,c){ return a+b+c; }
console.log(sum3(...[1,2,3])); // 6

// Rest: collect remaining args into an array (function params or destructuring)
function restExample(first, ...rest) {
  return { first, rest };
}
console.log(restExample(1,2,3,4)); // { first: 1, rest: [2,3,4] }

// Rest in destructuring
const [head, ...tail] = [10,20,30];
console.log(head); // 10
console.log(tail); // [20,30]
```

Notes:

- map/filter/concat/slice return new arrays (non-mutating).
- forEach and splice mutate or return undefined (forEach) / removed elements (splice).
- reduce returns any value (depends on reducer).
- spread/rest are shallow (object references preserved).

---

## ES6+ Features

### Template literals

- Backtick strings with interpolation and multi-line support.

```javascript
const name = "Alice";
const a = 5;
console.log(`Hi ${name}, ${a + 2}`); // Output: Hi Alice, 7

// Multi-line
console.log(`Line1
Line2`);
// Output:
// Line1
// Line2
```

### Destructuring (arrays & objects)

- Extract values concisely, with defaults and rest.

```javascript
// Array destructuring
const arr = [10, 20, 30, 40];
const [first, second, ...rest] = arr;
console.log(first, second, rest); // Output: 10 20 [30, 40]

// Object destructuring with defaults and renaming
const obj = { x: 1, y: 2 };
const { x, y: valueY, z = 0 } = obj;
console.log(x, valueY, z); // Output: 1 2 0

// Parameter destructuring
function draw({ x = 0, y = 0 } = {}) {
  return `x:${x}, y:${y}`;
}
console.log(draw({ x: 5 })); // Output: x:5, y:0
```

### Modules (import / export)

- Named export:

```javascript
// file: /path/to/math.js
export function add(a, b) { return a + b; }
export const PI = 3.14159;
```

- Default export:

```javascript
// file: /path/to/logger.js
export default function log(msg) { console.log(msg); }
```

- Importing:

```javascript
// default + named import
import log from './logger.js';
import { add, PI } from './math.js';

log(add(2, 3)); // Output: 5
console.log(PI); // Output: 3.14159

// renaming imports
import { add as sum } from './math.js';
console.log(sum(1,2)); // 3
```

### Default, named, and dynamic imports

- You can have one default and many named exports per module.

- Dynamic import (loads module at runtime, returns a promise):

```javascript
(async () => {
  const mod = await import('./math.js'); // relative path or URL
  // mod has named exports and default (if any)
  console.log(mod.add(3,4)); // Output: 7
})();
```

- Use dynamic imports for code-splitting or conditional loading.

Cheat:

- Use template literals for readable strings and multi-line.
- Use destructuring to simplify extraction and defaults.
- Prefer named exports for utilities; default export for main module value.
- Use dynamic import for lazy loading.

---

### JavaScript Function Methods: `call`, `apply`, and `bind`

In JavaScript, functions are first-class objects, which means they can be passed around like any other object. This includes the ability to modify their behavior using various methods. Three important methods for manipulating function behavior are `call`, `apply`, and `bind`.

## `call`

The `call` method allows a function to be invoked with a specified `this` value and arguments provided as separate parameters.

```javascript
function greet(name) {
  console.log(`Hello, ${name}!`);
}

const user = { name: 'John' };

greet.call(user, 'Alice'); // Output: Hello, Alice!
```

- The apply method is similar to call, but it allows arguments to be provided as an array instead of individual parameters.

```javascript
function sum(a, b) {
  return a + b;
}

const numbers = [1, 2];

console.log(sum.apply(null, numbers)); // Output: 3
```

- The bind method creates a new function that, when invoked, has its this value set to the provided value and any additional arguments pre-applied.

```javascript
function greet(name) {
  console.log(`Hello, ${name}!`);
}

const user = { name: 'John' };

const greetUser = greet.bind(user);

greetUser('Alice'); // Output: Hello, John!
```

---

## Events

Quick, easy notes with short examples to memorize.

### Event listeners

- Use addEventListener to attach handlers; supports multiple listeners and options (capture, once, passive).
- Remove with removeEventListener (same function reference).

```html
<!-- HTML -->
<button id="btn">Click</button>

<script>
const btn = document.getElementById('btn');
function onClick(e) { console.log('button clicked'); }

// attach
btn.addEventListener('click', onClick);

// later remove
// btn.removeEventListener('click', onClick);
</script>
```

Console: button clicked

### Event bubbling & capturing

- Event flow: capturing phase (top → target), target phase, bubbling phase (target → top).
- addEventListener(..., { capture: true }) listens in capturing phase; default is bubbling.
- Example shows order of logs.

```html
<div id="outer" style="padding:10px; border:1px solid">
  Outer
  <div id="inner" style="padding:10px; border:1px solid">Inner
    <button id="b">Go</button>
  </div>
</div>

<script>
const outer = document.getElementById('outer');
const inner = document.getElementById('inner');
const btn = document.getElementById('b');

outer.addEventListener('click', () => console.log('outer bubble'));
inner.addEventListener('click', () => console.log('inner bubble'));
btn.addEventListener('click', () => console.log('button target'));

// capturing listeners
outer.addEventListener('click', () => console.log('outer capture'), { capture: true });
inner.addEventListener('click', () => console.log('inner capture'), { capture: true });
</script>
```

Possible console order when clicking the button:
outer capture
inner capture
button target
inner bubble
outer bubble

### preventDefault & stopPropagation

- preventDefault(): prevent browser default action (e.g., link navigation, form submit).
- stopPropagation(): stop event from reaching other listeners (no further capturing/bubbling).

```html
<a id="link" href="https://example.com">Go</a>
<div id="wrap"><button id="btn2">Click</button></div>

<script>
const link = document.getElementById('link');
link.addEventListener('click', (e) => {
  e.preventDefault(); // link won't navigate
  console.log('link clicked but navigation prevented');
});

const wrap = document.getElementById('wrap');
wrap.addEventListener('click', () => console.log('wrap handler'));

const btn2 = document.getElementById('btn2');
btn2.addEventListener('click', (e) => {
  e.stopPropagation(); // prevents wrap handler from running
  console.log('button clicked, propagation stopped');
});
</script>
```

Console when clicking the link:
link clicked but navigation prevented

Console when clicking the button:
button clicked, propagation stopped
// (wrap handler not called)

Tips:

- Use preventDefault for custom handling of default browser behavior.
- Use stopPropagation to isolate an event from parent handlers; prefer targeted logic over widespread stopping where possible.

## DOM Manipulation

Quick, easy examples showing how to select elements, modify content/styles, and create/remove elements.

### Selecting elements

- querySelector / querySelectorAll — CSS selectors, returns first Element or NodeList.
- getElementById / getElementsByClassName / getElementsByTagName — older DOM APIs.

```javascript
// HTML example:
// <div id="root" class="box">Hello <span class="name">Alice</span></div>

// querySelector / querySelectorAll
const root = document.querySelector('#root');         // Element or null
const names = document.querySelectorAll('.name');     // NodeList (iterable)
console.log(root);            // <div id="root" class="box">...</div>
console.log(names.length);    // 1

// getElementById
const r2 = document.getElementById('root');
console.log(r2 === root);     // true

// getElementsByClassName / getElementsByTagName
const boxes = document.getElementsByClassName('box'); // HTMLCollection (live)
console.log(boxes[0]);        // same element
```

Notes:

- querySelectorAll returns a static NodeList; getElementsByClassName returns a live HTMLCollection.
- Use querySelector when you need CSS selector flexibility.

### Modifying content & styles

- textContent, innerText, innerHTML for content.
- value for form inputs.
- classList to add/remove/toggle classes.
- element.style or getComputedStyle for inline/Computed styles.

```javascript
// Assume: <p id="p">Old</p> <input id="i" value="a">

const p = document.querySelector('#p');
p.textContent = 'New text';           // replaces text content
// p.innerHTML = '<strong>Bold</strong>'; // sets HTML (be careful with user input)

const input = document.querySelector('#i');
console.log(input.value);              // "a"
input.value = 'b';

p.classList.add('highlight');          // add CSS class
p.classList.toggle('hidden');          // toggle class
p.classList.remove('old');

p.style.color = 'red';                 // inline style
p.style.backgroundColor = '#eef';
const comp = getComputedStyle(p);
console.log(comp.color);               // "rgb(255, 0, 0)" or similar
```

Tips:

- Use textContent to set plain text (safe).
- Use innerHTML only for trusted HTML.
- Prefer classList to change classes instead of mutating style directly when possible.

### Creating & removing elements

- createElement, append / appendChild, insertBefore, cloneNode, remove / removeChild.

```javascript
// create element and append
const ul = document.createElement('ul');
const li = document.createElement('li');
li.textContent = 'Item 1';
ul.appendChild(li);           // appendChild returns appended node
document.body.append(ul);     // append can append nodes or strings

// insert before
const li2 = document.createElement('li');
li2.textContent = 'Item 0';
ul.insertBefore(li2, ul.firstChild); // insert li2 before firstChild

// clone node
const cloned = li.cloneNode(true); // deep clone if true
cloned.textContent = 'Item 1 (copy)';
ul.append(cloned);

// remove element
cloned.remove();               // preferred modern API
// or: ul.removeChild(li2);

// create from HTML string (quick, but be careful with XSS)
const tpl = document.createElement('div');
tpl.innerHTML = '<button id="btn">Click</button>';
document.body.append(tpl.firstElementChild); // append the button
```

Outputs (example when run in browser console after HTML above):

- document.body now contains the new <ul> with items.
- console.log shows created elements and computed style values.

Safety:

- Avoid innerHTML with untrusted content to prevent XSS.
- Prefer createElement + textContent for dynamic user data.

## Error Handling

Quick notes + examples for try...catch...finally and throwing errors.

- try { } catch (err) { } finally { } — catch handles runtime errors; finally always runs (cleanup).
- throw new Error('msg') — create and throw errors.
- Use custom Error subclasses for typed errors.
- Async: use try/catch with await; promises need .catch or await inside try.

Examples with outputs:

```javascript
// Simple try/catch/finally
try {
  JSON.parse("invalid");          // throws SyntaxError
} catch (err) {
  console.log("Caught:", err.message); // Caught: Unexpected token i in JSON at position 0
} finally {
  console.log("Finally runs");   // Finally runs
}

// Throwing errors and using a function that validates
function divide(a, b) {
  if (b === 0) throw new Error("Divide by zero");
  return a / b;
}

try {
  console.log(divide(6, 2));     // 3
  console.log(divide(1, 0));     // throws
} catch (err) {
  console.log("Error:", err.message); // Error: Divide by zero
} finally {
  console.log("Done divide attempts"); // Done divide attempts
}

// Custom error class and selective handling
class ValidationError extends Error {
  constructor(msg) { super(msg); this.name = "ValidationError"; }
}

try {
  throw new ValidationError("Invalid input");
} catch (err) {
  if (err instanceof ValidationError) {
    console.log("Validation:", err.message); // Validation: Invalid input
  } else {
    throw err; // rethrow unexpected errors
  }
}

// Async example: try/catch with await
async function asyncFail() {
  try {
    await Promise.reject(new Error("async oops"));
  } catch (err) {
    console.log("Async caught:", err.message); // Async caught: async oops
  } finally {
    console.log("Async cleanup"); // Async cleanup
  }
}
asyncFail();
```

Best practices:

- Throw Error (or subclasses) with clear messages.
- Catch only what you can handle; rethrow otherwise.
- Use finally for cleanup (closing resources, stopping timers).
- For Promise-based code, prefer async/await + try/catch.

## Asynchronous JavaScript

Quick, easy notes + short examples with outputs.

### Callbacks

- A function passed to another function to run later. Can cause "callback hell" if nested deeply.

```javascript
function fetchData(cb) {
  setTimeout(() => {
    cb(null, { id: 1, name: 'Alice' });
  }, 100);
}

fetchData((err, data) => {
  if (err) return console.error('Error:', err);
  console.log('Got:', data); // Got: { id: 1, name: 'Alice' }
});
```

Prefer flat structure or Promises to avoid deep nesting.

### Promises

- Object representing a future value. Use then/catch/finally.

```javascript
function fetchNumber() {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(42), 100);
  });
}

fetchNumber()
  .then(n => {
    console.log('Number:', n); // Number: 42
    return n + 1;
  })
  .then(n2 => console.log('Plus one:', n2)) // Plus one: 43
  .catch(err => console.error('Error:', err))
  .finally(() => console.log('Done')); // Done
```

- Promise combinators:

```javascript
const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);
Promise.all([p1, p2]).then(arr => console.log(arr)); // [1,2]
Promise.allSettled([p1, Promise.reject('oops')]).then(r => console.log(r));
```

### async / await

- Syntactic sugar over Promises; write async code like sync. Use try/catch for errors.

```javascript
async function getData() {
  const n = await fetchNumber(); // waits for Promise
  return n * 2;
}

(async () => {
  try {
    const res = await getData();
    console.log('Result:', res); // Result: 84
  } catch (err) {
    console.error('Async error:', err);
  }
})();
```

- Parallel awaits:

```javascript
async function parallel() {
  const [a, b] = await Promise.all([fetchNumber(), Promise.resolve(10)]);
  console.log(a, b); // 42 10
}
parallel();
```
### Higher-Order Function

- A higher-order function is a function that either:
 > Takes another function as an argument, or
 > Returns a function as its result.

**Example 1: Function as Argument**
```js
function greet(name) {
  return "Hello, " + name;
}

function processUserInput(callback) {
  const name = "Awolad";
  return callback(name);
}

console.log(processUserInput(greet));
// Output: Hello, Awolad
```
- ✅ Explanation: processUserInput is a higher-order function because it takes another function (greet) as input.

**Example 2: Function Returning Function**
```js
function multiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = multiplier(2);
console.log(double(5)); // Output: 10
```
- ✅ Explanation: multiplier is a higher-order function because it returns another function.

**Array methods like map, filter, reduce are higher-order functions:**
```js
const numbers = [1, 2, 3, 4];

const doubled = numbers.map(num => num * 2);
// [2, 4, 6, 8]

const evens = numbers.filter(num => num % 2 === 0);
// [2, 4]
```

### Error handling in async code

- Always handle rejections: use .catch() for Promises and try/catch for async/await.
- Rethrow if you can't handle the error.

```javascript
function willFail() {
  return Promise.reject(new Error('fail'));
}

// Promise style
willFail().catch(err => console.log('Handled:', err.message)); // Handled: fail

// async/await style
async function run() {
  try {
    await willFail();
  } catch (err) {
    console.log('Caught in async:', err.message); // Caught in async: fail
    // optionally rethrow: throw err;
  }
}
run();
```

Best practices:

- Prefer Promises / async-await over nested callbacks.
- Always handle Promise rejections (use lint rules or unhandledRejection listener).
- Use Promise.allSettled for independent tasks where some may fail.
- Keep async functions small and testable.

## Closures

- Definition: A closure is a function that "remembers" the lexical environment where it was created — i.e., it retains access to outer-scope variables even after the outer function has returned.
- Lexical scope: inner functions access variables from their defining (lexical) scope, not from where they are called.

### Easy examples & outputs

```javascript
// Simple counter (data privacy)
function makeCounter() {
  let count = 0;                 // private variable
  return function() {            // closure: remembers `count`
    count += 1;
    return count;
  };
}

const c = makeCounter();
console.log(c()); // 1
console.log(c()); // 2
// `count` is not accessible directly from outside
```

```javascript
// Function factory (specialized functions)
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}

const add5 = makeAdder(5);
console.log(add5(3)); // 8
console.log(add5(10)); // 15
```

```javascript
// Memoization (performance via closure)
function memoize(fn) {
  const cache = {}; // preserved across calls
  return function (arg) {
    if (cache[arg] !== undefined) return cache[arg];
    return (cache[arg] = fn(arg));
  };
}

const slowDouble = n => { /* expensive work */ return n * 2; };
const fastDouble = memoize(slowDouble);
console.log(fastDouble(10)); // 20  (computed)
console.log(fastDouble(10)); // 20  (from cache)
```

```javascript
// IIFE module pattern (encapsulate state)
const counterModule = (function () {
  let value = 0;
  return {
    inc() { value++; },
    get() { return value; }
  };
})();

counterModule.inc();
console.log(counterModule.get()); // 1
```

### Practical use-cases (memorize)

- Data privacy / encapsulation: hide variables from global scope.
- Function factories: create parameterized helpers (e.g., adders, validators).
- Memoization / caching: keep caches private to the wrapper.
- Event handlers & callbacks: preserve context/state without global variables.

Cheat: "Closure = inner function + preserved outer variables" — practice with makeCounter and makeAdder to remember.

## Prototypes & Inheritance

Quick, easy notes with short examples and outputs.

### Prototype chain (concept)

- Every object has an internal prototype link ([[Prototype]] / __proto__) to another object.
- Property lookup: own properties → prototype → prototype's prototype → ... → Object.prototype → undefined.
- Use Object.getPrototypeOf(obj) to inspect prototype.

```javascript
const grand = { greet() { return 'hi from grand'; } };
const parent = Object.create(grand);
parent.sayParent = () => 'parent';
const child = Object.create(parent);
child.name = 'C';

console.log(child.name);                     // "C"    (own property)
console.log(child.sayParent());              // "parent" (from parent proto)
console.log(child.greet());                  // "hi from grand" (from grand proto)
console.log(Object.getPrototypeOf(child) === parent); // true
```

### Object.create

- Create a new object with a specific prototype; can define properties with descriptors.
- Good for simple prototypal inheritance without constructors.

```javascript
const proto = { hello() { return `hi ${this.name}`; } };
const obj = Object.create(proto, { name: { value: 'Bob', enumerable: true } });
console.log(obj.hello()); // "hi Bob"
```

### Constructor functions + prototype

- Before classes, constructors + prototype functions were common.

```javascript
function Person(name) { this.name = name; }
Person.prototype.say = function() { return `Hi ${this.name}`; };

const p = new Person('Alice');
console.log(p.say());            // "Hi Alice"
console.log(p instanceof Person); // true
```

### ES6 Classes & extends (syntactic sugar over prototypes)

- class / extends / super provide clearer syntax but use prototypes under the hood.

```javascript
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a noise`; }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);              // calls Animal constructor
    this.breed = breed;
  }
  speak() {
    return `${this.name} (${this.breed}) barks`; // overrides speak
  }
}

const d = new Dog('Rex', 'Beagle');
console.log(d.speak());                 // "Rex (Beagle) barks"
console.log(d instanceof Dog, d instanceof Animal); // true true
console.log(Object.getPrototypeOf(Dog) === Animal); // true (constructor inheritance)
```

Cheat summary:

- Prototype chain = how objects inherit properties.
- Object.create(proto) sets prototype directly.
- Constructor functions + prototypes or ES6 classes both set up prototype-based inheritance.
- Use super(...) in subclass constructors and super.method() in overridden methods.

## Execution Context, Call Stack & Event Loop (Concise)

### Execution Context (EC)

- An EC is the environment in which JS code runs. Types: Global EC, Function EC, Eval EC.
- Creation phase (hoisting): scope & variables set up, functions hoisted, let/const in TDZ.
- Execution phase: code runs, values assigned, this bound.
- Each function call creates a new Function EC with its own variable environment and scope chain.

### Call Stack

- LIFO stack of execution contexts. The currently running function is on top.
- Example:

```javascript
function a(){ b(); }
function b(){ c(); }
function c(){ console.log('in c'); }

a();
// Call stack when c runs:
// (top) c -> b -> a -> global (bottom)
```

Output:
in c

### Event Loop (Task vs Microtask)

- JS is single-threaded. The event loop coordinates:
  1. Execute the current stack (run-to-completion).
  2. Process microtasks queue (all microtasks) before rendering and before next macrotask.
  3. Render (if needed).
  4. Take next macrotask (task) from task queue (aka macrotask queue) and repeat.
- Microtasks: Promise callbacks (.then/.catch/.finally), queueMicrotask, MutationObserver, async/await continuations.
- Tasks (macrotasks): setTimeout, setInterval, I/O, UI events.

Example demonstrating order:

```javascript
console.log('script start');

setTimeout(() => console.log('timeout'), 0); // macrotask

Promise.resolve()
  .then(() => console.log('promise1'))       // microtask
  .then(() => console.log('promise2'));

queueMicrotask(() => console.log('qMicro'));

console.log('script end');
```

Expected output order:
script start
script end
promise1
promise2
qMicro
timeout

(Explanation: sync code runs first; then microtasks run fully in order; finally the macrotask from setTimeout runs.)

### Short cheats

- Microtasks run immediately after the current stack finishes and before the next macrotask.
- Use microtasks for urgent async callbacks; use macrotasks for timers/IO.
- async/await uses microtasks for resumptions (awaited continuations run as microtasks).

## Advanced Functions

Short, easy notes + examples with outputs to memorize.

### Higher-order functions

- Definition: functions that accept functions as arguments or return functions.
- Use-cases: callbacks, decorators, factories, map/filter/reduce utilities.

```javascript
// returns a function that multiplies by `n`
function makeMultiplier(n) {
  return function (x) {
    return x * n;
  };
}
const double = makeMultiplier(2);
console.log(double(5)); // 10

// `once` decorator: call a function only once
function once(fn) {
  let done = false;
  let result;
  return function (...args) {
    if (!done) { result = fn.apply(this, args); done = true; }
    return result;
  };
}
const init = once(() => 'initialized');
console.log(init()); // "initialized"
console.log(init()); // "initialized" (same result, inner fn not run again)
```

### Currying

- Definition: transform a function with multiple args into a chain of single-arg functions.
- Benefits: partial application, clearer composition.

```javascript
// manual curried function
const add = a => b => a + b;
console.log(add(2)(3)); // 5

// general curry helper (works for fixed-arity functions)
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn(...args);
    return (...more) => curried(...args, ...more);
  };
}
function sum3(a, b, c) { return a + b + c; }
const curriedSum3 = curry(sum3);
console.log(curriedSum3(1)(2)(3)); // 6
console.log(curriedSum3(1, 2)(3)); // 6
```

### Partial application

- Definition: fix some arguments of a function returning a new function for the rest.
- Methods: Function.prototype.bind, custom partial helper.

```javascript
function add2(a, b) { return a + b; }
// using bind (bind first arg to 5)
const add5 = add2.bind(null, 5);
console.log(add5(3)); // 8

// custom partial
function partial(fn, ...fixed) {
  return (...rest) => fn(...fixed, ...rest);
}
const greet = (greeting, name) => `${greeting}, ${name}!`;
const sayHelloTo = partial(greet, 'Hello');
console.log(sayHelloTo('Alice')); // "Hello, Alice!"
```

Cheat-sheet:

- Higher-order = accept/return functions (factories, decorators).
- Currying = break multi-arg fn into unary chain: f(a)(b)(c).
- Partial = fix some args now, supply rest later.
- Use curry/partial for composition and clearer reusable code.

## Modules & Bundling

Quick, practical notes with short examples.

### CommonJS vs ES Modules

- CommonJS (Node.js classic): synchronous, uses require / module.exports.
  - Good for Node scripts; not tree-shakeable.

```javascript
// CommonJS (file: math.cjs)
function add(a,b){ return a + b; }
module.exports = { add };

// usage
const { add } = require('./math.cjs');
console.log(add(1,2)); // 3
```

- ES Modules (ESM): import / export, static structure (supports tree-shaking), works in modern browsers and Node (with "type":"module").

```javascript
// ESM (file: math.mjs)
export function add(a,b){ return a + b; }
export function sub(a,b){ return a - b; }

// usage
import { add } from './math.mjs';
console.log(add(1,2)); // 3
```

### Tree-shaking

- Tree-shaking removes unused exports at bundle time. Requires static ESM imports (named exports).
- Example: if a module exports add and sub but bundle only imports add, a bundler with tree-shaking can drop sub from final output.
- Tip: prefer pure functions and named exports for better tree-shaking.

### Dynamic imports

- ESM supports dynamic import(...) which returns a Promise and enables code-splitting.

```javascript
// lazy load
async function loadUtil(){
  const { heavy } = await import('./heavy.js');
  heavy();
}
```

### Bundlers (Webpack, Rollup, Parcel) — quick compare

- Webpack
  - Very configurable, supports loaders and plugins, code-splitting, widely used for apps.
  - Good for complex projects where you need custom transforms.
- Rollup
  - Optimized for libraries and ES modules; produces smaller bundles and excellent tree-shaking.
  - Preferred for publishing libraries.
- Parcel
  - Zero-config bundler, fast to start, auto-transforms common assets.
  - Good for small to medium projects and prototypes.

Cheats:

- Use ESM + named exports for tree-shaking.
- Use dynamic import for lazy-loading routes/features.
- Choose Rollup for libraries, Webpack for complex app pipelines, Parcel for zero-config setups.

## Regular Expressions — Pattern matching & Validation (Easy & Examples)

Quick notes, common methods, flags, and validation examples.

### Basics

- RegExp literal: /pattern/flags
- Constructor: new RegExp('pattern', 'flags')
- Flags: i (ignore case), g (global), m (multiline), s (dotAll), u (unicode), y (sticky)

### Common methods & outputs

```javascript
const re = /(\w+)@(\w+)\.(\w+)/i;
const str = "Contact: alice@example.com";

// test -> boolean
console.log(re.test(str)); // true

// exec -> first match array (with groups)
console.log(re.exec(str));
// ["alice@example.com","alice","example","com", index:9, input: "Contact: alice@example.com", groups: undefined]

// String methods
console.log(str.match(re));      // same as exec (non-global)
console.log(str.match(/a/g));    // ["a", "a"]  (global returns array of matches)
console.log(str.replace(re, "[redacted]")); // "Contact: [redacted]"

// matchAll (with groups, returns iterator)
for (const m of "a1 b2 c3".matchAll(/(\w)(\d)/g)) console.log(m[0], m[1], m[2]);
// outputs:
// "a1" "a" "1"
// "b2" "b" "2"
// "c3" "c" "3"
```

### Groups & lookarounds

- Capturing: (..), Non-capturing: (?:...)
- Named groups: (?<name>...)
- Positive lookahead: (?=...), Negative lookahead: (?!...)
- Lookbehind (supported in modern engines): (?<=...), (?<!...)

Example: named groups + lookahead

```javascript
const re = /(?<user>[^\s@]+)@(?<domain>[^\s@]+\.[^\s@]+)/;
const m = "bob@mail.com".match(re);
console.log(m.groups.user, m.groups.domain); // bob mail.com
```

Password rule (min 8, one upper, one digit) using lookaheads:

```javascript
const pwdRe = /^(?=.*[A-Z])(?=.*\d).{8,}$/;
console.log(pwdRe.test("Abcdef12")); // true
console.log(pwdRe.test("abcdef12")); // false (no uppercase)
```

### Validation use-cases (simple, practical patterns)

- Email (simple, practical): /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  - Good for basic checks; use stricter rules or libraries for full validation.
- URL: prefer URL constructor; for quick check: /^(https?:\/\/)?([\w-]+\.)+[\w-]{2,}(\/\S*)?$/
- International phone (digits, optional +, 7–15 digits): /^\+?\d{7,15}$/
- Integer / decimal: /^-?\d+$/  and  /^-?\d+(\.\d+)?$/
- Hex color: /^#?([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/

Examples:

```javascript
const emailRe = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
console.log(emailRe.test("x@x.co")); // true
console.log(emailRe.test("bad@"));   // false

const phoneRe = /^\+?\d{7,15}$/;
console.log(phoneRe.test("+1234567890")); // true
```

### Tips & pitfalls

- Avoid overly strict single regex for complex validation (use parsing / dedicated libs).
- Use anchors ^ and $ to match whole string.
- Use non-capturing groups (?:) when you don't need captured values.
- Use flags and escape special chars: . * + ? ^ $ { } ( ) | [ ] \ /
- Remember global flag changes behavior of exec() (lastIndex), so reset or avoid mixing test/exec with g.

## Performance Optimization

Concise notes, causes, fixes and small examples to memorize.

### Memory leaks & garbage collection

- GC frees memory for objects no longer reachable. Leaks happen when references persist unintentionally.
- Common causes: forgotten timers, event listeners, closures holding large objects, globals, caches with strong refs, DOM nodes kept in JS.

Examples & fixes:

```javascript
// Leak: forgotten interval keeps references alive
let large = new Array(1e6).fill('*');
const id = setInterval(() => {
  // work...
  // large remains referenced -> cannot GC
}, 1000);

// fix: clear interval when done
clearInterval(id);
large = null; // allow GC
```

```javascript
// Leak: event listener holding closure state
function attach() {
  const big = new Array(1e6).fill(0);
  function onClick() { console.log(big[0]); }
  document.body.addEventListener('click', onClick);
  // if listener not removed -> `big` cannot be GC'd
  return () => document.body.removeEventListener('click', onClick); // remover
}
const remove = attach();
// later:
remove(); // prevents leak
```

Use WeakMap/WeakRef for caches keyed by objects:

```javascript
const cache = new WeakMap();
function getCached(obj) {
  if (!cache.has(obj)) cache.set(obj, compute(obj));
  return cache.get(obj);
}
// WeakMap keys can be GC'd when no other refs exist.
```

Tips:

- Remove listeners/timers on teardown.
- Avoid large globals; null references when no longer needed.
- Use browser DevTools memory profiler to find retained objects.

---

### Debouncing & Throttling

- Debounce: delay action until events stop. Good for input/search.
- Throttle: limit action rate. Good for scroll/resize.

Debounce implementation:

```javascript
function debounce(fn, wait = 200) {
  let t;
  return (...args) => {
    clearTimeout(t);
    t = setTimeout(() => fn(...args), wait);
  };
}

// usage
const onType = debounce((q) => console.log('Search', q), 300);
// rapid keystrokes -> only one "Search" after 300ms of silence
```

Throttle implementation:

```javascript
function throttle(fn, limit = 200) {
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= limit) {
      last = now;
      fn(...args);
    }
  };
}

// usage
window.addEventListener('scroll', throttle(() => {
  console.log('scroll handler at', Date.now());
}, 100));
```

---

### Lazy loading

- Load code/resources only when needed to reduce initial load.
- Patterns: dynamic import for code, IntersectionObserver for images/components.

Dynamic import example (lazy code):

```javascript
// on-demand
button.addEventListener('click', async () => {
  const mod = await import('./heavyModule.js');
  mod.run();
});
```

Image lazy-load with IntersectionObserver:

```javascript
const imgs = document.querySelectorAll('img[data-src]');
const io = new IntersectionObserver((entries, obs) => {
  entries.forEach(e => {
    if (e.isIntersecting) {
      const img = e.target;
      img.src = img.dataset.src;
      obs.unobserve(img);
    }
  });
});
imgs.forEach(i => io.observe(i));
```

---

### Code splitting

- Split bundle into chunks (routes, vendors, commons) so initial bundle is smaller.
- Use dynamic import(...) to create chunks automatically with bundlers (Webpack/Rollup/Parcel).
- Strategy: entry split (per route), vendor split (third-party libs), and lazy components.

Example (route split):

```javascript
// router load
const Home = () => import('./Home.js'); // creates separate chunk for Home
```

Tips:

- Measure bundle size (source-map-explorer, webpack-bundle-analyzer).
- Tree-shake (use ESM + named exports) to remove dead code.
- Defer non-critical work; prefetch/preload important subsequent chunks.

---

### Practical checklist

- Profile first (DevTools CPU / Memory / Network).
- Remove unnecessary listeners & timers.
- Use debouncing/throttling for frequent events.
- Lazy-load heavy modules and assets.
- Split bundles and enable tree-shaking.
- Prefer passive event listeners for scroll to improve responsiveness.

## Design Patterns in JS (Easy & Examples)

Short definitions + minimal examples with expected console output.

### Singleton

- Ensure a single instance (shared state).

```javascript
const Logger = (function () {
  let instance;
  function create() {
    return {
      log: (m) => console.log('[LOG]', m)
    };
  }
  return {
    getInstance() {
      if (!instance) instance = create();
      return instance;
    }
  };
})();

const l1 = Logger.getInstance();
const l2 = Logger.getInstance();
console.log(l1 === l2); // true
l1.log('hello'); // [LOG] hello
```

### Factory

- Create objects without exposing concrete classes.

```javascript
class Circle { draw(){ console.log('draw circle'); } }
class Square { draw(){ console.log('draw square'); } }

function shapeFactory(type) {
  switch(type) {
    case 'circle': return new Circle();
    case 'square': return new Square();
    default: throw new Error('Unknown');
  }
}

const s = shapeFactory('circle');
s.draw(); // draw circle
```

### Observer (Publish/Subscribe)

- Observers subscribe to a subject; subject notifies them on changes.

```javascript
function Subject() {
  const observers = new Set();
  return {
    subscribe(o){ observers.add(o); },
    unsubscribe(o){ observers.delete(o); },
    notify(data){ observers.forEach(o => o.update(data)); }
  };
}

const subj = Subject();
const obs1 = { update: d => console.log('obs1 got', d) };
subj.subscribe(obs1);
subj.notify('event'); // obs1 got event
```

### Strategy

- Swap algorithms (behaviors) at runtime.

```javascript
const payByCard = { pay: amt => console.log('Paid by card', amt) };
const payByPayPal = { pay: amt => console.log('Paid by PayPal', amt) };

function Checkout(strategy){ this.strategy = strategy; }
Checkout.prototype.doPay = function(amt){ this.strategy.pay(amt); };

const c = new Checkout(payByCard);
c.doPay(20); // Paid by card 20
c.strategy = payByPayPal;
c.doPay(30); // Paid by PayPal 30
```

### Module Pattern

- Encapsulate private state, expose a public API.

```javascript
const Counter = (function () {
  let value = 0; // private
  return {
    inc() { value += 1; },
    get() { return value; }
  };
})();

Counter.inc();
console.log(Counter.get()); // 1
// value is not accessible directly
```

Cheat: Singleton = single instance, Factory = create by type, Observer = publish/subscribe, Strategy = interchangeable algorithms, Module = encapsulated private state + public API.

## Functional Programming Concepts

Short, easy notes with small examples and outputs to memorize.

### Pure functions

- Definition: same input → same output, no side effects (do not modify external state).
- Benefits: easier testing, reasoning, caching.

Examples:

```javascript
// Pure
function add(a, b) { return a + b; }
console.log(add(2, 3)); // 5

// Impure (side effect: mutates input)
function pushToArray(arr, v) { arr.push(v); return arr; }
const a = [1, 2];
console.log(pushToArray(a, 3)); // [1,2,3]
console.log(a);                 // [1,2,3]  (original mutated)
```

Return behavior:

- add(2,3) returns 5 (pure).
- pushToArray returns the same array reference (impure) and mutates original.

---

### Immutability

- Principle: avoid changing existing data; create new versions instead.
- Benefits: predictable state, safer concurrency, easier undo/redo.

Examples:

```javascript
// Arrays (use spread to create new array)
const nums = [1, 2];
const nums2 = [...nums, 3];
console.log(nums2); // [1,2,3]
console.log(nums);  // [1,2]  (unchanged)

// Objects (use spread to create new object)
const obj = { x: 1 };
const obj2 = { ...obj, y: 2 };
console.log(obj2); // { x:1, y:2 }
console.log(obj);  // { x:1 } (unchanged)

// Note: Object.freeze is shallow
const nested = Object.freeze({ a: { b: 1 } });
nested.a.b = 2;
console.log(nested.a.b); // 2  (inner object still mutable)
```

Return behavior:

- Spread returns new arrays/objects; original stays the same.
- Object.freeze prevents top-level mutations only.

---

### Composition

- Idea: build complex behavior by composing small functions (f(g(x))).
- Benefits: modularity, reuse, clearer pipelines.

Helpers and examples:

```javascript
const compose = (f, g) => (x) => f(g(x));
const pipe = (...fns) => (x) => fns.reduce((v, fn) => fn(v), x);

// small functions
const inc = x => x + 1;
const double = x => x * 2;

// compose: double(inc(x)) => inc then double
const incThenDouble = compose(double, inc);
console.log(incThenDouble(3)); // 8  (inc -> 4, double -> 8)

// pipe: left-to-right
const pipeline = pipe(inc, double);
console.log(pipeline(3)); // 8

// practical: map with composed function
const arr = [1,2,3];
console.log(arr.map(pipe(inc, double))); // [4,6,8]
```

Return behavior:

- compose/pipe return new functions that return transformed values; originals unchanged.

---

Cheat summary:

- Pure = no side effects, predictable output.
- Immutable = do not mutate; return new data structures.
- Composition = build logic by combining small pure functions.

## Advanced Asynchronous Patterns

Concise notes and small examples with expected outputs.

### Generators & Iterators

- Generator: function* that can pause (yield) and resume. It returns an iterator compliant with the iterator protocol (next() → { value, done }).

```javascript
// generator function
function* genNumbers() {
  yield 1;
  yield 2;
  yield* [3, 4]; // delegate to iterable
  return 5;      // return is accessible only via .next().value when done=true
}

const it = genNumbers();
console.log(it.next()); // { value: 1, done: false }
console.log(it.next()); // { value: 2, done: false }
console.log([...genNumbers()]); // [1,2,3,4]  (return value not included in spread)
```

- Custom iterator (manual implementation of Symbol.iterator):

```javascript
const counter = {
  from: 1,
  to: 3,
  [Symbol.iterator]() {
    let current = this.from;
    const end = this.to;
    return {
      next() {
        if (current <= end) return { value: current++, done: false };
        return { value: undefined, done: true };
      }
    };
  }
};
for (const n of counter) console.log(n); // 1 2 3
```

Cheat: Generators are great for lazy sequences and implementing iterables without manual state management.

---

### Async Iteration (for await...of)

- Async generator: async function* yields Promises or awaited values. Consume with for await...of (runs in an async context).

```javascript
async function* asyncGen() {
  yield await Promise.resolve('a');
  yield new Promise(res => setTimeout(() => res('b'), 50));
  yield 'c';
}

(async () => {
  for await (const v of asyncGen()) {
    console.log(v);
  }
  // Output order:
  // a
  // b
  // c
})();
```

- Any async iterable (object with [Symbol.asyncIterator]) can be consumed with for await...of.

---

### Streams (Node.js) — readable as async iterables

- Node streams are useful for handling large/continuous data. Readable streams implement async iteration (Node 10+), so you can use for await...of to consume chunks.

```javascript
// Example: Readable.from (creates a Readable that's async-iterable)
const { Readable } = require('stream');

(async () => {
  const r = Readable.from(['line1\n', 'line2\n', 'line3\n']);
  for await (const chunk of r) {
    process.stdout.write(chunk); // writes line1\n line2\n line3\n
  }
})();
```

- Consuming a file stream with async iteration:

```javascript
// Node.js example (run in Node environment)
const fs = require('fs');

(async () => {
  const rs = fs.createReadStream('./large.txt', { encoding: 'utf8' });
  for await (const chunk of rs) {
    // chunk is a string (or Buffer) piece
    console.log('chunk length:', chunk.length);
  }
})();
```

- When to use:
  - Use streams for large files, network data, or continuous feeds to avoid loading everything into memory.
  - Combine with pipeline (stream/promises) and backpressure-aware APIs for robust apps.

Notes:

- Generators = sync lazy flows; async generators = lazy async flows.
- for await...of simplifies consuming async iterables and readable streams.
- Streams provide efficient, backpressure-aware data flow for large/continuous data.

## Event-Driven Architecture

Brief: components communicate via events (decoupled). Useful for UI, microservices, and async flows.

### Pub/Sub (Publish / Subscribe) Pattern

- Publisher emits events (messages) to topics.
- Subscribers register handlers for topics and receive messages.
- Decouples producers from consumers.

Example — simple Pub/Sub implementation:

```javascript
// Simple PubSub
class PubSub {
  constructor(){ this.topics = new Map(); }
  subscribe(topic, handler){
    if(!this.topics.has(topic)) this.topics.set(topic, new Set());
    this.topics.get(topic).add(handler);
    return () => this.topics.get(topic).delete(handler); // unsubscribe
  }
  publish(topic, data){
    const subs = this.topics.get(topic);
    if(!subs) return;
    for(const h of subs) h(data);
  }
}

// Usage
const bus = new PubSub();
const unsub = bus.subscribe('order.created', o => console.log('Order handler:', o.id));
bus.publish('order.created', { id: 42 });
// Console: Order handler: 42
unsub(); // stop receiving
bus.publish('order.created', { id: 43 }); // no output
```

When to use: event buses, decoupled modules, cross-component notifications.

### Custom Events (Browser)

- Use CustomEvent to carry data through DOM events.

Example — dispatch/listen on document:

```javascript
// Create and listen
document.addEventListener('user:login', e => console.log('User logged in', e.detail.user));

const ev = new CustomEvent('user:login', { detail: { user: 'alice' } });
document.dispatchEvent(ev);
// Console: User logged in { user: "alice" }
```

Notes: can configure bubbles/cancelable; good for component-to-component communication in browser.

### Node.js — EventEmitter

- Built-in pattern for event-driven code on server.

```javascript
const { EventEmitter } = require('events');
const em = new EventEmitter();

em.on('data', d => console.log('got', d));
em.emit('data', 123); // Console: got 123
```

### Tips

- Prefer topic namespacing (e.g., user:created) to avoid collisions.
- Unsubscribe listeners to avoid memory leaks.
- For large systems consider persistent message brokers (Kafka, RabbitMQ) for durability/scaling.

## Security in JS

Concise, practical notes and tiny examples.

### XSS (Cross-Site Scripting) — prevention

- What: attacker injects JS/HTML that runs in victim's browser (steals cookies, performs actions).
- Prevent:
  - Escape/encode user data when inserting into HTML.
  - Prefer textContent / value over innerHTML.
  - Use well-tested sanitizers (DOMPurify) when inserting trusted HTML is required.
  - Apply Content Security Policy (CSP) to limit allowed sources.

Unsafe example (vulnerable):

```javascript
const userInput = '<img src=x onerror="alert(1)">';
document.getElementById('out').innerHTML = userInput; // XSS risk
```

Safe alternatives:

```javascript
// 1) Treat as plain text
document.getElementById('out').textContent = userInput;

// 2) Sanitize if you must allow some HTML (use DOMPurify)
import DOMPurify from 'dompurify';
document.getElementById('out').innerHTML = DOMPurify.sanitize(userInput);
```

CSP header example (server-side header):
Content-Security-Policy: default-src 'self'; script-src 'self' <https://trusted.cdn.com>; object-src 'none';

Notes:

- Escape values used in attributes or URL contexts.
- Avoid eval(), new Function(), and unsafe inline handlers.

### CSRF (Cross-Site Request Forgery) — basics

- What: attacker tricks an authenticated user into making requests (e.g., form submission) to your site.
- Mitigations:
  - CSRF tokens: unique token per session/form validated server-side.
  - SameSite cookies: set cookies with SameSite=Lax or Strict to prevent cross-site sending.
  - Require custom request headers (e.g., X-Requested-With or X-CSRF-Token) for state-changing endpoints — browsers don't send custom headers cross-origin without CORS.
  - Use Same-Origin policy and CORS correctly.

Client example: include token in fetch header

```javascript
// assume server provided token in a meta tag or initial JSON
const csrf = document.querySelector('meta[name="csrf-token"]').content;
fetch('/api/transfer', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', 'X-CSRF-Token': csrf },
  body: JSON.stringify({ amount: 10 })
});
```

Server-side: validate token per session/form and set cookie flags: HttpOnly, Secure, SameSite.

### Sanitization (input & output)

- Principle: validate early (server-side) and escape late (when rendering).
- Input validation: enforce types/length/format server-side (never trust client).
- Output encoding: encode for the context (HTML, attribute, JS, URL).
- Libraries:
  - DOMPurify for HTML sanitization in browser.
  - Server-side sanitization libraries for your stack (e.g., validator.js, OWASP ESAPI, built-in frameworks).
- Example with DOMPurify:

```javascript
import DOMPurify from 'dompurify';
const safe = DOMPurify.sanitize(userHtml);
document.getElementById('content').innerHTML = safe;
```

Quick checklist

- Never insert raw user content via innerHTML.
- Encode/escape for the specific output context.
- Use CSP as defense-in-depth.
- Enforce SameSite and Secure on cookies; validate CSRF tokens server-side.
- Log and monitor suspicious input patterns.

## Testing

Concise notes, examples and quick commands for unit, integration, and mocking.

### Unit testing (Jest, Mocha)

- Purpose: verify small units (functions/classes) in isolation.
- Run via npm scripts (e.g., "test": "jest" or "mocha").

Jest example (common, fast, includes mocks):

```javascript
// add.js
function add(a, b) { return a + b; }
module.exports = add;

// add.test.js (Jest)
const add = require('./add');
test('adds 2+3 to equal 5', () => {
  expect(add(2, 3)).toBe(5);
});
```

Command: npm install --save-dev jest && npm test
Typical output: PASS ./add.test.js (1 test)

Mocha + Chai example:

```javascript
// add.test.js (Mocha + Chai)
const { expect } = require('chai');
const add = require('./add');
describe('add', () => {
  it('adds numbers', () => expect(add(1,2)).to.equal(3));
});
```

Command: npm install --save-dev mocha chai && npx mocha
Typical output: 1 passing (XXms)

---

### Integration testing

- Purpose: test multiple units together (APIs, DB, filesystem). Use test doubles for external services or dedicated test DB.
- Tools: supertest (HTTP), jest/mocha, testcontainers for DB.

Express + supertest example:

```javascript
// app.js (Express minimum)
const express = require('express');
const app = express();
app.get('/', (req, res) => res.json({ ok: true }));
module.exports = app;

// app.test.js (Jest + supertest)
const request = require('supertest');
const app = require('./app');
test('GET / returns ok', async () => {
  const res = await request(app).get('/');
  expect(res.status).toBe(200);
  expect(res.body).toEqual({ ok: true });
});
```

Command: npm install --save-dev supertest && run tests.
Typical output: PASS (integration tests)

Tips:

- Use a separate test DB or in-memory DB (sqlite, mongodb-memory-server).
- Seed/teardown fixtures in before/after hooks.

---

### Mocking & stubs

- Purpose: replace dependencies to isolate tests and control behavior.
- Mocks: verify interactions (calls/args). Stubs: control return values. Spies: observe calls.

Jest mocking (built-in):

```javascript
// api.js
module.exports.get = async () => ({ id: 1, name: 'real' });

// fetchData.js
const api = require('./api');
module.exports = async () => { return (await api.get()).id; };

// fetchData.test.js
jest.mock('./api', () => ({ get: jest.fn(() => Promise.resolve({ id: 42 })) }));
const fetchData = require('./fetchData');
test('uses mocked api.get', async () => {
  const id = await fetchData();
  expect(id).toBe(42);
});
```

Sinon example (Mocha):

```javascript
const sinon = require('sinon');
const api = require('./api');
describe('stub example', () => {
  afterEach(() => sinon.restore());
  it('stubs api.get', async () => {
    sinon.stub(api, 'get').resolves({ id: 7 });
    const fetchData = require('./fetchData');
    const id = await fetchData();
    // assertions...
  });
});
```

Guidelines:

- Mock external services (HTTP, payment gateways) to avoid network in unit tests.
- Prefer stubs for replacing behavior, spies for verifying calls, mocks for verifying interactions.
- Keep tests deterministic; reset mocks/stubs between tests.

---

### Quick checklist

- Run tests with CI (GitHub Actions / GitLab CI).
- Keep unit tests fast; move slower end-to-end or integration tests to separate stage.
- Use coverage tools (jest --coverage, nyc) to monitor gaps.
- Clean up resources (DB connections, timers, listeners) in after/afterEach hooks.

## Deep Dive Topics (Concise)

### JavaScript engines (V8 internals — high level)

- Main stages: parsing → AST → bytecode → execution & optimization.
- V8 pipeline (simplified):
  - Parser → produces AST (Abstract Syntax Tree).
  - Ignition → interpreter that runs bytecode quickly (baseline).
  - Turbofan → optimizing JIT compiler that produces highly optimized machine code for hot functions.
- Runtime optimizations:
  - Hidden classes / shapes: V8 creates internal shapes for object layouts to speed property access.
  - Inline Caching (IC): caches call/property lookup results to avoid repeated lookups.
- Garbage Collection:
  - Young generation (scavenger) for short-lived objects; Old generation uses mark-sweep/mark-compact.
  - Generational & incremental GC reduce pause times.

Quick note: write predictable, monomorphic code and avoid frequently changing object shapes to help engines optimize.

### JIT compilation (practical)

- JIT goal: compile hot code to native machine code for speed.
- Typical flow: run on interpreter → collect type feedback → compile optimized code → if assumptions break → deoptimize/back to interpreter.
- Optimization triggers: repeated calls, stable types, monomorphic call sites.
- Deoptimization (bailouts): expensive — avoid patterns that constantly change types or shapes.

Example (helps optimization):

```javascript
function sumArray(nums) {
  let s = 0;
  for (let i = 0; i < nums.length; i++) s += nums[i]; // monomorphic numeric loop
  return s;
}
```

Avoid mixing number/string types in hot loops to prevent deopt.

### Abstract Syntax Trees (AST)

- AST: structured tree representation of source code nodes (Program, FunctionDeclaration, Identifier, Literal, etc.).
- Tools: Esprima, Acorn, Babel parser (@babel/parser), Recast.
- Uses: linters, transpilers, code transforms, static analysis, minifiers.

Example: parse with acorn (Node):

```javascript
// npm i acorn
const acorn = require('acorn');
const ast = acorn.parse('const x = 1 + 2;', { ecmaVersion: 2020 });
console.log(ast.type); // "Program"
```

- Traversal/transforms: use estraverse or @babel/traverse to read/modify AST, then generate code with @babel/generator or recast.

Cheat: AST = structured, language-agnostic representation you can inspect/transform programmatically.

### Polyfills & Transpilers (Babel)

- Transpiler (Babel): converts modern JS syntax into compatible older syntax (syntax transform).
- Polyfill (core-js, regenerator-runtime): add missing runtime APIs (Promise, Array.from, Symbol, etc.).
- preset-env: compile only the features needed for target environments (browsers/node versions).
- Source maps: keep original-source mapping for debugging after transpilation/bundling.

Minimal example (npm + .babelrc):

```bash
npm install --save-dev @babel/core @babel/cli @babel/preset-env core-js
```

.babelrc

```json
{
  "presets": [
    ["@babel/preset-env", {
      "useBuiltIns": "usage",
      "corejs": 3,
      "targets": "> 0.25%, not dead"
    }]
  ]
}
```

- "useBuiltIns": "usage" injects polyfills automatically where needed (via core-js).
- For async/await you may need regenerator or core-js polyfills depending on targets.

Tips:

- Prefer transpiling syntax (Babel) and polyfilling behavior (core-js) separately.
- Test on target environments and use feature-detection where possible.
- Keep an eye on bundle size when adding polyfills (use targeted builds).

---

Summary cheat-sheet:

- Engines: parse → AST → bytecode (Ignition) → optimized native code (Turbofan).
- JIT: fast for hot/stable code; avoid frequent type/shape changes.
- AST: programmatic code representation — used by Babel, linters, code mods.
- Transpilers vs polyfills: transpilers rewrite syntax; polyfills add missing APIs. Use preset-env + core-js for targeted compatibility.

## Scalability & Architecture

Concise notes, patterns, and tiny examples to remember.

### Monorepos

- Single repository hosting multiple packages/apps (shared utils, consistent tooling).
- Tools: pnpm workspaces, Yarn workspaces, Lerna, Nx, Turborepo.
- Benefits: easy code sharing, atomic refactors, consolidated CI, consistent deps.
- Trade-offs: larger repo size, more complex CI, dependency hoisting issues.

Quick pnpm workspace example (package.json):

```json
// workspace root package.json
{
  "private": true,
  "workspaces": ["apps/*", "packages/*"]
}
```

Tips: keep clear boundaries, use incremental builds (Nx/Turborepo), enforce linting/formatting centrally.

### Micro-frontends

> Break a large frontend into independently deployable pieces (per team).
> Integration approaches: build-time composition, run-time composition (module federation), web components, import maps or single-spa.
> Use cases: independent releases, team autonomy.
> Minimal runtime dynamic import pattern:

```javascript
// host app loads remote module URL (simplified)
async function loadRemote(url) {
  const m = await import(/* webpackIgnore: true */ url);
  m.mount(document.getElementById('slot')); // remote exposes mount()
}
loadRemote('https://cdn.example.com/cart/remote.js');
```

Tips: share runtime libs carefully (avoid duplicate React), establish contracts (props, lifecycle), isolate styles.

---
