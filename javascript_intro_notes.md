# JavaScript Introduction: Complete Notes

## 1. What is JavaScript

JavaScript is a programming language that was created to make web pages interactive. It runs inside web browsers, and it also runs outside browsers using a program called Node.js. Because of this, JavaScript is used for both the front end of websites (what the user sees and clicks) and the back end (servers, databases, APIs).

JavaScript is:

- A high level language, meaning you do not need to manage memory by hand.
- An interpreted language, meaning the code is read and run directly, without a separate compiling step that produces a binary file (though modern engines do compile internally for speed).
- Dynamically typed, meaning a variable can hold a number today and a string tomorrow, without you declaring its type ahead of time.
- Multi paradigm, meaning you can write it in a procedural style, an object oriented style, or a functional style.

JavaScript has nothing to do with Java. The names are similar for historical marketing reasons from the 1990s, but the languages are different.

## 2. How JavaScript Runs

In a browser, JavaScript runs inside a JavaScript engine built into the browser. Examples are V8 in Chrome and Edge, SpiderMonkey in Firefox, and JavaScriptCore in Safari.

Outside the browser, Node.js uses the V8 engine to run JavaScript directly on your computer or on a server, without a browser at all. This is how JavaScript is used to build backend applications, command line tools, and scripts.

A browser also gives JavaScript access to extra tools that are not part of the language itself, such as the DOM (Document Object Model, used to read and change a web page) and the `fetch` function (used to make network requests). Node.js gives JavaScript access to different extra tools, such as the file system and networking modules.

## 3. Setting Up

To run JavaScript in a browser, you can open the developer console (usually F12 or right click and Inspect, then go to the Console tab) and type code directly.

To run JavaScript in a file in the browser, you write your code in a `.js` file and link it from an HTML file:

```html
<script src="script.js"></script>
```

To run JavaScript outside the browser, install Node.js from nodejs.org, then run a file from the terminal:

```
node script.js
```

## 4. Basic Syntax and Statements

A JavaScript program is a sequence of statements. A statement is one instruction, such as creating a variable or calling a function.

```javascript
let name = "Asha";
console.log(name);
```

Each statement is usually ended with a semicolon. JavaScript can often work without semicolons because of a feature called automatic semicolon insertion, but writing them yourself is safer and avoids rare bugs, so it is recommended.

`console.log()` prints a value to the console. It is the most common tool for checking what a piece of code is doing while you write it.

JavaScript is case sensitive. `myVariable` and `myvariable` are two different names.

## 5. Comments

Comments are notes in the code that JavaScript ignores when running. They are used to explain what the code does.

```javascript
// This is a single line comment

/*
This is a
multi line comment
*/
```

## 6. Variables: var, let, and const

A variable is a named container for a value. JavaScript has three ways to declare a variable.

```javascript
var oldWay = "avoid this in modern code";
let canChange = "use this when the value will change";
const cannotReassign = "use this when the value will not change";
```

`let` and `const` were introduced in 2015 (a version of JavaScript called ES6) to fix problems with `var`. In modern JavaScript, `var` is rarely used.

Differences:

- `var` is function scoped. It is visible anywhere inside the function it was declared in, even before the line it was declared on (this is called hoisting, explained more in the scope section).
- `let` and `const` are block scoped. They are only visible inside the nearest set of curly braces `{ }` that contains them, such as inside an if statement or a loop.
- `const` means the variable cannot be reassigned to a new value after it is set. It does not mean the value is frozen. If the value is an object or array, the contents inside it can still be changed.

```javascript
const numbers = [1, 2, 3];
numbers.push(4); // allowed, the array contents changed
numbers = [9, 9, 9]; // not allowed, this would throw an error
```

A general modern practice is to use `const` by default, and switch to `let` only when you know the variable's value needs to change later.

Variable names in JavaScript can contain letters, digits, underscores, and dollar signs, but cannot start with a digit. JavaScript is case sensitive and reserved words (like `let`, `if`, `function`) cannot be used as variable names.

## 7. Data Types

JavaScript has two categories of data types: primitive types and objects.

### Primitive Types

- **String**: text, written inside quotes. Example: `"hello"`, `'hello'`.
- **Number**: any number, whether it has a decimal point or not. JavaScript does not separate integers and decimals into different types the way some languages do. Example: `42`, `3.14`, `-7`.
- **Boolean**: only two possible values, `true` or `false`.
- **Undefined**: a variable that has been declared but has not been given a value yet. Example: `let x;` makes `x` equal to `undefined`.
- **Null**: represents the intentional absence of a value. You assign this yourself when you want to say "there is nothing here on purpose."
- **BigInt**: used for very large whole numbers beyond what the Number type can safely represent. Written with an `n` at the end, example: `12345678901234567890n`.
- **Symbol**: used to create a guaranteed unique value, mostly used in advanced situations like defining special object behavior. Less commonly seen by beginners.

### Object Type

Everything that is not one of the primitives above is an object. This includes:

- Plain objects, written with curly braces: `{ name: "Asha" }`
- Arrays, written with square brackets: `[1, 2, 3]`
- Functions
- Dates, regular expressions, and other built in object types

### Checking a Type

The `typeof` operator tells you the type of a value.

```javascript
typeof "hello";    // "string"
typeof 42;         // "number"
typeof true;       // "boolean"
typeof undefined;  // "undefined"
typeof null;       // "object" (this is a long standing quirk in JavaScript, null is technically reported as an object)
typeof {};         // "object"
typeof [];         // "object" (arrays are a kind of object)
typeof function(){}; // "function"
```

## 8. Type Conversion and Coercion

Type conversion is when you intentionally change a value from one type to another. Type coercion is when JavaScript does this automatically, often during comparisons or operations.

```javascript
String(42);       // "42", intentional conversion to string
Number("42");     // 42, intentional conversion to number
Boolean(0);       // false, intentional conversion to boolean
Boolean("");      // false
Boolean("hello"); // true

"5" + 3;   // "53", coercion: the number 3 becomes a string because of the + with a string
"5" - 3;   // 2, coercion: the string "5" becomes a number because - only works with numbers
```

Values that are treated as `false` when converted to boolean are called "falsy" values. The falsy values in JavaScript are: `false`, `0`, `""` (empty string), `null`, `undefined`, and `NaN`. Every other value is "truthy," including `"0"` (a string containing zero) and empty objects or arrays.

## 9. Operators

### Arithmetic Operators

```javascript
5 + 3;  // 8, addition
5 - 3;  // 2, subtraction
5 * 3;  // 15, multiplication
5 / 3;  // 1.666..., division
5 % 3;  // 2, remainder (modulus)
5 ** 2; // 25, exponent (5 to the power of 2)
```

### Assignment Operators

```javascript
let x = 5;
x += 3;  // same as x = x + 3
x -= 3;  // same as x = x - 3
x *= 3;  // same as x = x * 3
x /= 3;  // same as x = x / 3
```

### Comparison Operators

```javascript
5 == "5";   // true, loose equality, allows type coercion before comparing
5 === "5";  // false, strict equality, checks value and type together
5 != "5";   // false, loose inequality
5 !== "5";  // true, strict inequality
5 > 3;      // true
5 < 3;      // false
5 >= 5;     // true
5 <= 4;     // false
```

It is strongly recommended to use `===` and `!==` instead of `==` and `!=` in almost all situations, because loose equality can produce confusing and unexpected results due to type coercion.

### Logical Operators

```javascript
true && false; // false, AND, true only if both sides are true
true || false; // true, OR, true if at least one side is true
!true;         // false, NOT, flips the boolean
```

### Other Useful Operators

```javascript
let result = condition ? "yes" : "no"; // ternary operator, a short if/else

let a = null;
let b = a ?? "default"; // nullish coalescing, returns "default" only if a is null or undefined

let c = a?.someProperty; // optional chaining, returns undefined instead of throwing an error if a is null or undefined
```

## 10. Control Flow: Conditionals

```javascript
let age = 20;

if (age >= 18) {
  console.log("adult");
} else if (age >= 13) {
  console.log("teen");
} else {
  console.log("child");
}
```

A `switch` statement compares one value against several possible matches.

```javascript
let day = "Monday";

switch (day) {
  case "Saturday":
  case "Sunday":
    console.log("weekend");
    break;
  default:
    console.log("weekday");
}
```

`break` stops the switch from checking further cases. Without it, execution would continue (fall through) into the next case.

## 11. Control Flow: Loops

### for loop

```javascript
for (let i = 0; i < 5; i++) {
  console.log(i);
}
```

### while loop

```javascript
let i = 0;
while (i < 5) {
  console.log(i);
  i++;
}
```

### do while loop

Runs the body at least once, then checks the condition.

```javascript
let i = 0;
do {
  console.log(i);
  i++;
} while (i < 5);
```

### for...of loop

Loops over the values in an array or other iterable object.

```javascript
const fruits = ["apple", "banana", "cherry"];
for (const fruit of fruits) {
  console.log(fruit);
}
```

### for...in loop

Loops over the keys of an object.

```javascript
const person = { name: "Asha", age: 30 };
for (const key in person) {
  console.log(key, person[key]);
}
```

### break and continue

`break` stops a loop entirely. `continue` skips the current pass and moves to the next one.

```javascript
for (let i = 0; i < 10; i++) {
  if (i === 5) break;
  if (i % 2 === 0) continue;
  console.log(i);
}
```

## 12. Functions

A function is a reusable block of code that performs a task.

### Function Declaration

```javascript
function greet(name) {
  return "Hello, " + name;
}

greet("Asha"); // "Hello, Asha"
```

### Function Expression

A function stored in a variable.

```javascript
const greet = function (name) {
  return "Hello, " + name;
};
```

### Arrow Functions

A shorter way to write functions, introduced in ES6. Arrow functions also handle the `this` keyword differently, explained in the section on `this`.

```javascript
const greet = (name) => {
  return "Hello, " + name;
};

// If the function body is a single expression that is returned, it can be shortened:
const greet = (name) => "Hello, " + name;

// If there is exactly one parameter, the parentheses are optional:
const square = x => x * x;
```

### Default Parameters

```javascript
function greet(name = "stranger") {
  return "Hello, " + name;
}

greet(); // "Hello, stranger"
```

### Rest Parameters

Collects any number of arguments into an array.

```javascript
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3, 4); // 10
```

### Function Declarations vs Function Expressions

A key practical difference: function declarations are hoisted, meaning you can call them before the line where they appear in the code. Function expressions (including arrow functions) are not hoisted in the same way, the variable holding them must be assigned before you can call it.

```javascript
sayHi(); // works, this is allowed

function sayHi() {
  console.log("hi");
}
```

```javascript
sayHi(); // error, cannot call before it is assigned

const sayHi = () => {
  console.log("hi");
};
```

## 13. Scope

Scope determines where a variable can be accessed in your code.

- **Global scope**: variables declared outside any function or block. Accessible everywhere in the file.
- **Function scope**: variables declared with `var` inside a function are accessible anywhere inside that function.
- **Block scope**: variables declared with `let` or `const` inside a block (anything inside `{ }`, such as an if statement or loop) are only accessible inside that block.

```javascript
function example() {
  if (true) {
    var functionScoped = "I am visible outside this if block";
    let blockScoped = "I am only visible inside this if block";
  }
  console.log(functionScoped); // works
  console.log(blockScoped); // error, not defined here
}
```

### Hoisting

JavaScript moves function and variable declarations to the top of their scope before running the code. With `var`, the variable name is hoisted but its value is not, so it exists as `undefined` until the line where it is actually assigned. With `let` and `const`, the variable is hoisted but cannot be used before its declaration line, attempting to do so causes an error. This unusable period is called the "temporal dead zone."

## 14. Closures

A closure happens when a function remembers and can still access variables from the scope it was created in, even after that outer scope has finished running.

```javascript
function makeCounter() {
  let count = 0;
  return function () {
    count = count + 1;
    return count;
  };
}

const counter = makeCounter();
counter(); // 1
counter(); // 2
counter(); // 3
```

In this example, the inner function keeps access to `count`, even though `makeCounter` already finished running. Each call to `makeCounter()` creates a separate, independent `count` variable. Closures are commonly used to create private state and to build functions that remember settings or configuration.

## 15. Arrays

An array is an ordered list of values.

```javascript
const fruits = ["apple", "banana", "cherry"];

fruits[0];        // "apple", indexing starts at 0
fruits.length;     // 3
fruits[fruits.length - 1]; // "cherry", last item
```

Arrays can hold mixed types, including other arrays and objects, although it is common practice to keep an array's contents consistent in type.

```javascript
const mixed = [1, "two", true, { name: "three" }, [4, 5]];
```

## 16. Array Methods

These are some of the most commonly used built in array methods.

```javascript
const numbers = [1, 2, 3, 4, 5];

numbers.push(6);       // adds to the end, returns new length
numbers.pop();         // removes from the end, returns the removed item
numbers.unshift(0);    // adds to the start
numbers.shift();       // removes from the start

numbers.indexOf(3);    // 2, position of the first match, -1 if not found
numbers.includes(3);   // true, whether the value exists

numbers.slice(1, 3);   // returns a new array with items from index 1 up to (not including) index 3, original array unchanged
numbers.splice(1, 2);  // removes 2 items starting at index 1, changes the original array

numbers.join(", ");    // turns the array into a string, "1, 2, 3, 4, 5"

numbers.reverse();     // reverses the array in place
numbers.sort();        // sorts in place, default sort works alphabetically on strings, this can give odd results on numbers without a custom function
```

### Higher Order Array Methods

These take a function as an argument and are extremely common in modern JavaScript.

```javascript
const numbers = [1, 2, 3, 4, 5];

numbers.forEach(n => console.log(n)); // runs a function once per item, returns nothing

const doubled = numbers.map(n => n * 2); // [2, 4, 6, 8, 10], new array with the result of a function applied to each item

const evens = numbers.filter(n => n % 2 === 0); // [2, 4], new array with only the items that pass a test

const total = numbers.reduce((sum, n) => sum + n, 0); // 15, combines all items into a single value, the second argument is the starting value

const found = numbers.find(n => n > 3); // 4, the first item that passes a test

const exists = numbers.some(n => n > 4); // true, whether at least one item passes a test

const allPass = numbers.every(n => n > 0); // true, whether every item passes a test
```

`map`, `filter`, and `reduce` are some of the most important methods to understand well, since they appear constantly in real JavaScript code.

## 17. Objects

An object is a collection of key and value pairs.

```javascript
const person = {
  name: "Asha",
  age: 30,
  isStudent: false,
};

person.name;        // "Asha", dot notation
person["age"];       // 30, bracket notation, required if the key is stored in a variable or has spaces

person.email = "asha@example.com"; // adding a new property
delete person.isStudent;            // removing a property
```

Bracket notation is needed when the key name is dynamic:

```javascript
const key = "name";
person[key]; // "Asha"
```

### Nested Objects

```javascript
const user = {
  name: "Asha",
  address: {
    city: "Nairobi",
    country: "Kenya",
  },
};

user.address.city; // "Nairobi"
```

### Object Methods (Built In)

```javascript
Object.keys(person);    // array of all keys
Object.values(person);  // array of all values
Object.entries(person); // array of [key, value] pairs

Object.assign({}, person); // copies properties into a new object
```

## 18. Object Methods and the this Keyword

A function stored as a property of an object is called a method.

```javascript
const person = {
  name: "Asha",
  greet() {
    return "Hello, my name is " + this.name;
  },
};

person.greet(); // "Hello, my name is Asha"
```

Inside a regular method, `this` refers to the object the method was called on. This is a frequent source of confusion in JavaScript, because the value of `this` depends on how a function is called, not where it is written.

```javascript
function showThis() {
  console.log(this);
}

const obj = { showThis };
obj.showThis(); // this refers to obj
showThis();     // this refers to the global object (or undefined in strict mode/modules)
```

Arrow functions do not have their own `this`. Instead, they use the `this` value from the place where they were written, which is often what people actually want.

```javascript
const person = {
  name: "Asha",
  greetLater() {
    setTimeout(() => {
      console.log("Hello, " + this.name); // this still refers to person, because arrow functions inherit this from greetLater
    }, 1000);
  },
};
```

If a regular function had been used instead of an arrow function inside `setTimeout`, `this` would not refer to `person` anymore, which is a very common bug for beginners.

## 19. Strings and String Methods

Strings can be written with single quotes, double quotes, or backticks. Backticks allow template literals, covered in the next section.

```javascript
const message = "Hello, World!";

message.length;             // 13
message.toUpperCase();      // "HELLO, WORLD!"
message.toLowerCase();      // "hello, world!"
message.includes("World");  // true
message.indexOf("World");   // 7
message.slice(0, 5);        // "Hello"
message.replace("World", "Kenya"); // "Hello, Kenya!"
message.trim();              // removes whitespace from both ends
message.split(", ");         // ["Hello", "World!"], turns a string into an array
```

Strings in JavaScript are immutable, meaning none of these methods change the original string. They all return a new string.

## 20. Template Literals

Template literals use backticks and allow you to embed variables and expressions directly inside a string using `${}`.

```javascript
const name = "Asha";
const age = 30;

const message = `My name is ${name} and I am ${age} years old.`;
```

Template literals also support multiple lines without special characters:

```javascript
const message = `Line one
Line two`;
```

## 21. Destructuring

Destructuring lets you pull values out of arrays or objects into separate variables in one step.

### Array Destructuring

```javascript
const numbers = [1, 2, 3];
const [first, second, third] = numbers;

const [a, , c] = numbers; // skip the second item using a blank space
```

### Object Destructuring

```javascript
const person = { name: "Asha", age: 30 };
const { name, age } = person;

const { name: fullName } = person; // rename while destructuring, fullName now holds "Asha"

const { country = "Kenya" } = person; // default value, used because person has no "country" key
```

Destructuring is very commonly used in function parameters, especially when a function receives an object:

```javascript
function printUser({ name, age }) {
  console.log(name, age);
}

printUser({ name: "Asha", age: 30 });
```

## 22. Spread and Rest

The spread operator `...` expands an array or object into individual elements. It looks identical to the rest parameter syntax shown earlier with functions, but it does the opposite job, spread expands, rest collects.

```javascript
const numbers = [1, 2, 3];
const moreNumbers = [...numbers, 4, 5]; // [1, 2, 3, 4, 5]

const person = { name: "Asha", age: 30 };
const updatedPerson = { ...person, age: 31 }; // copies person, then overwrites age

function sum(a, b, c) {
  return a + b + c;
}
const args = [1, 2, 3];
sum(...args); // spreads the array into separate arguments, same as sum(1, 2, 3)
```

Spread is commonly used to copy arrays and objects without changing the original, which fits well with how `const` is normally used in modern JavaScript.

## 23. Classes

Classes are a way to create a blueprint for objects, especially useful when you need many objects with the same shape and behavior.

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    return "Hello, my name is " + this.name;
  }
}

const asha = new Person("Asha", 30);
asha.greet(); // "Hello, my name is Asha"
```

The `constructor` method runs automatically when a new object is created with `new`. `this` inside the constructor and methods refers to the specific object being created or used.

### Inheritance

```javascript
class Student extends Person {
  constructor(name, age, school) {
    super(name, age); // calls the parent class constructor
    this.school = school;
  }

  greet() {
    return super.greet() + " and I study at " + this.school;
  }
}

const ben = new Student("Ben", 20, "University of Nairobi");
ben.greet(); // "Hello, my name is Ben and I study at University of Nairobi"
```

`extends` makes one class inherit from another. `super` is used to call the parent class's constructor or methods.

### Note on Prototypes

Under the surface, JavaScript classes are built on something called prototypal inheritance, where objects can inherit properties and methods from other objects through a hidden link called the prototype chain. Classes are mostly a cleaner, more familiar way to write this same underlying behavior, and as a beginner you can work productively with classes without studying prototypes directly right away.

## 24. Error Handling

```javascript
try {
  let result = 10 / 0; // this does not throw an error in JavaScript, it produces Infinity
  let broken = JSON.parse("not valid json"); // this does throw an error
  console.log(result);
} catch (error) {
  console.log("Something went wrong: " + error.message);
} finally {
  console.log("This always runs, whether there was an error or not");
}
```

You can also create and throw your own errors:

```javascript
function checkAge(age) {
  if (age < 0) {
    throw new Error("Age cannot be negative");
  }
  return age;
}

try {
  checkAge(-5);
} catch (error) {
  console.log(error.message); // "Age cannot be negative"
}
```

## 25. Asynchronous JavaScript: Callbacks

JavaScript normally runs one line at a time, in order, and waits for each line to finish before moving to the next. This is called synchronous execution. But some tasks, such as waiting for a server response or waiting for a timer, take time, and it would be wasteful to freeze the whole program while waiting. JavaScript handles this with asynchronous code.

The oldest method for this is the callback, a function passed into another function to be run later, once a task finishes.

```javascript
console.log("Start");

setTimeout(() => {
  console.log("This runs after 2 seconds");
}, 2000);

console.log("End");

// Output order: "Start", "End", then after 2 seconds, "This runs after 2 seconds"
```

This shows that `setTimeout` does not pause the program. The rest of the code keeps running, and the callback runs later when the timer finishes.

Heavy use of nested callbacks can become difficult to read, sometimes called "callback hell." This problem led to the introduction of promises.

## 26. Promises

A promise is an object representing a value that may not be available yet, but will be at some point, either successfully (resolved) or unsuccessfully (rejected).

```javascript
const promise = new Promise((resolve, reject) => {
  let success = true;

  if (success) {
    resolve("It worked");
  } else {
    reject("It failed");
  }
});

promise
  .then(result => console.log(result))
  .catch(error => console.log(error));
```

Promises can be chained, since `.then()` itself returns a new promise:

```javascript
fetch("https://example.com/data")
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.log("Error: " + error));
```

`fetch` is a built in function for making network requests, and it returns a promise.

## 27. Async and Await

`async` and `await` are a more modern way to write asynchronous code that looks like ordinary synchronous code, while still being non blocking underneath. They are built directly on top of promises.

```javascript
async function getData() {
  try {
    const response = await fetch("https://example.com/data");
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.log("Error: " + error);
  }
}

getData();
```

`async` is placed before a function to mark that it will use `await` and will always return a promise. `await` pauses execution inside that function until the promise it is waiting on settles, without blocking the rest of the program outside that function. `try` and `catch` are used around `await` to handle errors, similar to how `.catch()` works with promise chains.

## 28. The DOM (Document Object Model)

When a browser loads an HTML page, it builds a tree like structure in memory representing the page, called the DOM. JavaScript can read and change this structure, which is how web pages become interactive.

```javascript
document.getElementById("title");        // selects a single element by its id attribute
document.querySelector(".item");          // selects the first element matching a CSS selector
document.querySelectorAll(".item");       // selects all elements matching a CSS selector, returns a list

const heading = document.querySelector("h1");
heading.textContent = "New Title";        // changes the visible text
heading.style.color = "blue";              // changes a CSS style directly
heading.classList.add("highlight");        // adds a CSS class

const newParagraph = document.createElement("p");
newParagraph.textContent = "This is new";
document.body.appendChild(newParagraph);   // adds the new element to the page
```

## 29. Events

Events are actions that happen in the browser, such as a click, a key press, or a page load. JavaScript can listen for these events and respond.

```javascript
const button = document.querySelector("button");

button.addEventListener("click", function () {
  console.log("Button was clicked");
});
```

The function passed to `addEventListener` receives an event object with details about what happened.

```javascript
button.addEventListener("click", function (event) {
  console.log(event.target); // the exact element that was clicked
});
```

Common events include `click`, `submit`, `keydown`, `keyup`, `mouseover`, `mouseout`, `load`, and `change`.

## 30. JSON

JSON (JavaScript Object Notation) is a text format for representing structured data. It looks similar to JavaScript object and array syntax, but it is a separate text format used widely for sending data between a browser and a server, or for saving data to a file.

```javascript
const person = { name: "Asha", age: 30 };

const jsonString = JSON.stringify(person); // '{"name":"Asha","age":30}', converts an object to a JSON formatted string

const parsedBack = JSON.parse(jsonString); // { name: "Asha", age: 30 }, converts a JSON formatted string back into a JavaScript object
```

JSON only supports a limited set of types: strings, numbers, booleans, null, arrays, and plain objects. It does not support functions, undefined values, or special object types like Date directly.

## 31. Modules

Modules let you split JavaScript code across multiple files, and share specific pieces between them using `export` and `import`.

```javascript
// file: math.js
export function add(a, b) {
  return a + b;
}

export const PI = 3.14159;

export default function multiply(a, b) {
  return a * b;
}
```

```javascript
// file: main.js
import multiply, { add, PI } from "./math.js";

console.log(add(2, 3));      // 5
console.log(multiply(2, 3)); // 6
console.log(PI);              // 3.14159
```

A `default` export is the main thing a file exports, and it can be imported under any name. Named exports (like `add` and `PI`) must be imported using their exact names, optionally renamed with `as`.

```javascript
import { add as addNumbers } from "./math.js";
```

In the browser, a script using modules must be loaded with `type="module"`:

```html
<script type="module" src="main.js"></script>
```

In Node.js, module support depends on the project's configuration, either using this same `import`/`export` syntax, or an older style using `require()` and `module.exports`.

## 32. Common Pitfalls and Best Practices

- Prefer `const` and `let` over `var`, and prefer `const` over `let` unless reassignment is actually needed.
- Prefer `===` and `!==` over `==` and `!=` to avoid confusing type coercion.
- Remember that arrays and objects assigned to `const` can still have their contents changed, only reassignment of the variable itself is blocked.
- Remember that arrow functions do not have their own `this`, which is useful inside callbacks but can be a problem if you actually need the calling object's own `this`.
- Be careful with floating point numbers. Because of how computers store decimals, something like `0.1 + 0.2` does not exactly equal `0.3` in JavaScript, due to small rounding differences. For precise decimal math, such as money, special handling or libraries are often used.
- Remember that array and string methods like `map`, `filter`, `slice`, and `replace` return new values rather than changing the original, while methods like `push`, `pop`, `splice`, and `sort` do change the original.
- Always handle errors in asynchronous code, either with `.catch()` on promises or `try`/`catch` around `await`.
- Indent code consistently and use clear variable names, since JavaScript does not enforce a particular style itself, but consistent style avoids many mistakes.

## 33. Suggested Next Steps

A reasonable order to deepen your knowledge after this introduction:

1. Practice writing small functions and using array methods like `map`, `filter`, and `reduce` until they feel natural.
2. Build a few small projects that touch the DOM and events, such as a simple to do list or a calculator, to connect JavaScript to something visible.
3. Learn `fetch` and practice working with an example public API, to get comfortable with promises and `async`/`await`.
4. Learn classes in more depth, then look briefly at prototypes to understand what classes are built on.
5. Learn how modules and a build tool (such as Vite) are used in a real, multi file JavaScript project.
6. Once comfortable, look into a framework such as React, Vue, or Svelte, which are built on top of the JavaScript fundamentals covered in these notes.
