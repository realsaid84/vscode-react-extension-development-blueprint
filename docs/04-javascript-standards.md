# JavaScript Core Standards

> **Essential JavaScript patterns aligned with Airbnb style guide**

## Table of Contents
- [References](#references)
- [Objects](#objects)
- [Arrays](#arrays)
- [Destructuring](#destructuring)
- [Strings](#strings)
- [Functions](#functions)
- [Arrow Functions](#arrow-functions)
- [Modules](#modules)
- [Comparison Operators](#comparison-operators)
- [Comments](#comments)

---

## References

### const and let (Airbnb Standard)

```javascript
// ✅ Good: Use const for immutable references
const a = 1;
const b = 2;

// ✅ Good: Use let for mutable references
let count = 1;
if (true) {
  count += 1;
}

// ❌ Bad: Never use var
var a = 1;
var b = 2;
```

### One Declaration Per Variable

```javascript
// ✅ Good: One const/let per variable
const items = getItems();
const goSportsTeam = true;
const dragonball = 'z';

// ❌ Bad: Multiple declarations
const items = getItems(),
  goSportsTeam = true,
  dragonball = 'z';
```

---

## Objects

### Object Creation (Airbnb Standard)

```javascript
// ✅ Good: Use literal syntax
const item = {};

// ❌ Bad: Use Object constructor
const item = new Object();
```

### Computed Property Names (Airbnb Standard)

```javascript
// ✅ Good: Use computed property names
const getKey = (k) => `a key named ${k}`;

const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
};

// ❌ Bad: Create object then add property
const obj = {
  id: 5,
  name: 'San Francisco',
};
obj[getKey('enabled')] = true;
```

### Object Method Shorthand (Airbnb Standard)

```javascript
// ✅ Good: Use method shorthand
const atom = {
  value: 1,
  
  addValue(value) {
    return atom.value + value;
  },
};

// ❌ Bad: Use function keyword
const atom = {
  value: 1,
  
  addValue: function(value) {
    return atom.value + value;
  },
};
```

### Property Value Shorthand (Airbnb Standard)

```javascript
// ✅ Good: Use property shorthand
const lukeSkywalker = 'Luke Skywalker';

const obj = {
  lukeSkywalker,
};

// ❌ Bad: Redundant property assignment
const obj = {
  lukeSkywalker: lukeSkywalker,
};
```

### Grouped Shorthand Properties

```javascript
// ✅ Good: Group shorthand properties at the beginning
const anakinSkywalker = 'Anakin Skywalker';
const lukeSkywalker = 'Luke Skywalker';

const obj = {
  lukeSkywalker,
  anakinSkywalker,
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  episodeThree: 3,
  mayTheFourth: 4,
};
```

### Object Spread

```javascript
// ✅ Good: Use object spread for shallow copy
const original = { a: 1, b: 2 };
const copy = { ...original, c: 3 }; // { a: 1, b: 2, c: 3 }

// ✅ Good: Use spread to merge objects
const obj1 = { a: 1 };
const obj2 = { b: 2 };
const merged = { ...obj1, ...obj2 }; // { a: 1, b: 2 }

// ❌ Bad: Use Object.assign with mutation
const original = { a: 1, b: 2 };
const copy = Object.assign(original, { c: 3 }); // Mutates original!
```

---

## Arrays

### Array Creation (Airbnb Standard)

```javascript
// ✅ Good: Use literal syntax
const items = [];

// ❌ Bad: Use Array constructor
const items = new Array();
```

### Array.push vs Direct Assignment

```javascript
// ✅ Good: Use push to add items
const someStack = [];
someStack.push('abracadabra');

// ❌ Bad: Use direct assignment
const someStack = [];
someStack[someStack.length] = 'abracadabra';
```

### Array Spreads (Airbnb Standard)

```javascript
// ✅ Good: Use spreads to copy arrays
const items = [1, 2, 3];
const itemsCopy = [...items];

// ❌ Bad: Use loops to copy
const itemsCopy = [];
for (let i = 0; i < items.length; i += 1) {
  itemsCopy[i] = items[i];
}
```

### Array.from for Array-like Objects

```javascript
// ✅ Good: Use Array.from for array-like objects
const foo = document.querySelectorAll('.foo');
const nodes = Array.from(foo);

// ✅ Good: Use Array.from with mapping
const arr = Array.from([1, 2, 3], (x) => x * x); // [1, 4, 9]

// ❌ Bad: Use spread on NodeList (not array-like in old browsers)
const foo = document.querySelectorAll('.foo');
const nodes = [...foo];
```

### Return Statements in Array Methods

```javascript
// ✅ Good: Use return statement
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});

// ✅ Good: Implicit return for single expression
[1, 2, 3].map((x) => x + 1);

// ❌ Bad: Implicit return with side effects
let sum = 0;
[1, 2, 3].map((x) => sum += x); // Don't use map for side effects
```

---

## Destructuring

### Object Destructuring (Airbnb Standard)

```javascript
// ✅ Good: Use object destructuring
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}

// ❌ Bad: Access properties individually
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;
  return `${firstName} ${lastName}`;
}
```

### Array Destructuring (Airbnb Standard)

```javascript
// ✅ Good: Use array destructuring
const arr = [1, 2, 3, 4];
const [first, second] = arr;

// ❌ Bad: Access by index
const first = arr[0];
const second = arr[1];
```

### Multiple Return Values

```javascript
// ✅ Good: Use object destructuring for multiple return values
function processInput(input) {
  return { left, right, top, bottom };
}

const { left, top } = processInput(input);

// ❌ Bad: Use array destructuring (order-dependent)
function processInput(input) {
  return [left, right, top, bottom];
}

const [left, , top] = processInput(input); // Must skip values
```

---

## Strings

### Quote Style (Airbnb Standard)

```javascript
// ✅ Good: Use single quotes for strings
const name = 'Capt. Janeway';

// ❌ Bad: Use double quotes (except in JSX)
const name = "Capt. Janeway";
```

### Template Literals (Airbnb Standard)

```javascript
// ✅ Good: Use template literals for interpolation
function sayHi(name) {
  return `How are you, ${name}?`;
}

// ✅ Good: Use template literals for multiline strings
const html = `
  <div>
    <span>Hello</span>
  </div>
`;

// ❌ Bad: Use string concatenation
function sayHi(name) {
  return 'How are you, ' + name + '?';
}

// ❌ Bad: Use concatenation for multiline
const html = '<div>\n' +
  '<span>Hello</span>\n' +
  '</div>';
```

### String Building

```javascript
// ✅ Good: Use template literals
let errorMessage = `This is a super long error that was thrown because 
of Batman. When you stop to think about how Batman had anything to do 
with this, you would get nowhere fast.`;

// ❌ Bad: Use concatenation
let errorMessage = 'This is a super long error that was thrown because ' +
  'of Batman. When you stop to think about how Batman had anything to do ' +
  'with this, you would get nowhere fast.';
```

---

## Functions

### Function Declarations vs Expressions (Airbnb Standard)

```javascript
// ✅ Good: Use named function expressions
const short = function longUniqueMoreDescriptiveLexicalFoo() {
  // ...
};

// ❌ Bad: Use function declarations (hoisting issues)
function foo() {
  // ...
}
```

### IIFE (Immediately Invoked Function Expressions)

```javascript
// ✅ Good: Wrap IIFE in parentheses
(function() {
  console.log('Welcome to the Internet.');
}());
```

### Never Declare Functions in Non-Function Blocks

```javascript
// ❌ Bad: Function declaration in block
if (currentUser) {
  function test() {
    console.log('Nope.');
  }
}

// ✅ Good: Use function expression
let test;
if (currentUser) {
  test = function() {
    console.log('Yup.');
  };
}
```

### Default Parameters

```javascript
// ✅ Good: Use default parameters
function handleThings(name, opts = {}) {
  // ...
}

// ❌ Bad: Mutate function arguments
function handleThings(name, opts) {
  opts = opts || {};
  // ...
}
```

### Parameter Mutation

```javascript
// ❌ Bad: Don't mutate parameters
function f1(obj) {
  obj.key = 1;
}

// ✅ Good: Return new object
function f2(obj) {
  return {
    ...obj,
    key: 1,
  };
}
```

### Rest Parameters

```javascript
// ✅ Good: Use rest parameters
function concatenateAll(...args) {
  return args.join('');
}

// ❌ Bad: Use arguments object
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('');
}
```

---

## Arrow Functions

### When to Use Arrow Functions (Airbnb Standard)

```javascript
// ✅ Good: Use arrow functions for inline callbacks
[1, 2, 3].map((x) => x * x);

// ✅ Good: Use arrow functions for complex callbacks
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});

// ❌ Bad: Use function keyword for callbacks
[1, 2, 3].map(function(x) {
  return x * x;
});
```

### Implicit Returns

```javascript
// ✅ Good: Implicit return for single expression
[1, 2, 3].map((x) => x * x);

// ✅ Good: Implicit return with object literal (use parentheses)
[1, 2, 3].map((x) => ({ value: x }));

// ❌ Bad: Forgot parentheses (returns undefined)
[1, 2, 3].map((x) => { value: x }); // Thinks { value: x } is a block
```

### Always Use Parentheses (Airbnb Standard)

```javascript
// ✅ Good: Always use parentheses
[1, 2, 3].map((x) => x * x);

// ❌ Bad: Omit parentheses
[1, 2, 3].map(x => x * x);
```

---

## Modules

### Import/Export (Airbnb Standard)

```javascript
// ✅ Good: Use import/export
import { es6 } from './AirbnbStyleGuide';
export default es6;

// ❌ Bad: Use require/module.exports
const AirbnbStyleGuide = require('./AirbnbStyleGuide');
module.exports = AirbnbStyleGuide.es6;
```

### Import Everything at Top

```javascript
// ✅ Good: Import at top of file
import foo from 'foo';
import bar from 'bar';

foo.init();

// ❌ Bad: Import in middle of file
import foo from 'foo';
foo.init();

import bar from 'bar'; // Import after code
```

### Multiline Imports

```javascript
// ✅ Good: Multiline imports
import {
  longNameA,
  longNameB,
  longNameC,
  longNameD,
  longNameE,
} from 'path';

// ❌ Bad: Single line when long
import { longNameA, longNameB, longNameC, longNameD, longNameE } from 'path';
```

### No Wildcard Imports

```javascript
// ✅ Good: Import specific exports
import { specificFunction } from './module';

// ❌ Bad: Import everything
import * as Module from './module';
```

### Export from Import

```javascript
// ✅ Good: Separate import and export
import { foo } from './foo';
export { foo };

// ❌ Bad: Export directly from import
export { foo } from './foo';
```

---

## Comparison Operators

### Use === and !== (Airbnb Standard)

```javascript
// ✅ Good: Use === and !==
if (name === 'test') {
  // ...
}

// ❌ Bad: Use == and !=
if (name == 'test') {
  // ...
}
```

### Shortcuts for Booleans

```javascript
// ✅ Good: Shortcuts for booleans
if (isValid) {
  // ...
}

if (!isValid) {
  // ...
}

// ❌ Bad: Explicit comparison
if (isValid === true) {
  // ...
}
```

### Explicit Comparisons for Strings and Numbers

```javascript
// ✅ Good: Explicit comparison for strings
if (name !== '') {
  // ...
}

// ✅ Good: Explicit comparison for numbers
if (collection.length > 0) {
  // ...
}

// ❌ Bad: Implicit comparison
if (name) {
  // Falsy for empty string, but also for many other values
}
```

### Ternary Operators

```javascript
// ✅ Good: Simple ternaries
const foo = (a === b) ? 1 : 2;

// ✅ Good: Ternary on multiple lines when long
const value = (condition)
  ? resultWhenTrue
  : resultWhenFalse;

// ❌ Bad: Nested ternaries
const foo = (a === b)
  ? 1
  : (a === c)
    ? 2
    : 3;
```

---

## Comments

### Multiline Comments (Airbnb Standard)

```javascript
// ✅ Good: Use /** */ for multiline comments
/**
 * make() returns a new element
 * based on the passed-in tag name
 */
function make(tag) {
  // ...
  return element;
}

// ❌ Bad: Use multiple single-line comments
// make() returns a new element
// based on the passed-in tag name
function make(tag) {
  // ...
  return element;
}
```

### Single Line Comments (Airbnb Standard)

```javascript
// ✅ Good: Single line comment above code
// is current tab
const active = true;

// ❌ Bad: Inline comment
const active = true; // is current tab
```

### FIXME and TODO

```javascript
// ✅ Good: Use FIXME for problems
class Calculator extends Abacus {
  constructor() {
    super();
    
    // FIXME: shouldn't use a global here
    total = 0;
  }
}

// ✅ Good: Use TODO for solutions
class Calculator extends Abacus {
  constructor() {
    super();
    
    // TODO: total should be configurable by an options param
    this.total = 0;
  }
}
```

---

## Best Practices Summary

### ✅ DO

- **Use const** for immutable references
- **Use let** for mutable references
- **Use literal syntax** for objects and arrays
- **Use template literals** for string interpolation
- **Use destructuring** for objects and arrays
- **Use arrow functions** for callbacks
- **Use === and !==** for comparisons
- **Use default parameters** instead of mutation
- **Use import/export** instead of require

### ❌ DON'T

- **Don't use var** - use const/let
- **Don't use == or !=** - use === and !==
- **Don't mutate parameters**
- **Don't use function declarations** - use expressions
- **Don't use string concatenation** - use templates
- **Don't use wildcard imports**
- **Don't nest ternaries**
- **Don't use inline comments**

### 🎯 Quick Checklist

- [ ] All var replaced with const/let
- [ ] Using === instead of ==
- [ ] Template literals for strings
- [ ] Arrow functions for callbacks
- [ ] Object/array destructuring
- [ ] Proper import/export usage
- [ ] No parameter mutation
- [ ] Comments use /** */ or //

---

These JavaScript standards ensure clean, modern, and maintainable code throughout the DAPA extension, fully aligned with Airbnb guidelines.