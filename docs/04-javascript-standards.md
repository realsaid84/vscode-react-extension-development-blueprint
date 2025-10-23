# 4. JavaScript Core Standards

### 4.1 References (Airbnb Standard)

```javascript
// ✅ Good - use const for immutable references
const a = 1;
const b = 2;

// ✅ Good - use let for mutable references
let count = 1;
if (true) {
  count += 1;
}

// ❌ Bad - never use var
var a = 1;
var b = 2;
```

### 4.2 Objects (Airbnb Standard)

```javascript
// ✅ Good - literal syntax
const item = {};

// ✅ Good - computed property names
const getKey = (k) => `a key named ${k}`;
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
};

// ✅ Good - object method shorthand
const atom = {
  value: 1,
  addValue(value) {
    return atom.value + value;
  },
};

// ✅ Good - property value shorthand
const lukeSkywalker = 'Luke Skywalker';
const obj = { lukeSkywalker };

// ❌ Bad
const item = new Object();
const obj = {
  lukeSkywalker: lukeSkywalker,
};
```

### 4.3 Arrays (Airbnb Standard)

```javascript
// ✅ Good - literal syntax
const items = [];

// ✅ Good - use array spreads to copy
const itemsCopy = [...items];

// ✅ Good - use Array.from for array-like objects
const foo = document.querySelectorAll('.foo');
const nodes = Array.from(foo);

// ✅ Good - return statements in array methods
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});

// ❌ Bad
const items = new Array();
```

### 4.4 Destructuring (Airbnb Standard)

```javascript
// ✅ Good - object destructuring
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}

// ✅ Good - array destructuring
const arr = [1, 2, 3, 4];
const [first, second] = arr;

// ✅ Good - object destructuring for multiple return values
function processInput(input) {
  return { left, right, top, bottom };
}
const { left, top } = processInput(input);

// ❌ Bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;
  return `${firstName} ${lastName}`;
}
```

### 4.5 Strings (Airbnb Standard)

```javascript
// ✅ Good - single quotes for strings
const name = 'Capt. Janeway';

// ✅ Good - template literals for interpolation
function sayHi(name) {
  return `How are you, ${name}?`;
}

// ❌ Bad - double quotes (except in JSX)
const name = "Capt. Janeway";

// ❌ Bad - string concatenation
function sayHi(name) {
  return 'How are you, ' + name + '?';
}
```

### 4.6 Functions (Airbnb Standard)

```javascript
// ✅ Good - named function expressions
const short = function longUniqueMoreDescriptiveLexicalFoo() {
  // ...
};

// ✅ Good - arrow functions for callbacks
[1, 2, 3].map((x) => x * x);

// ✅ Good - default parameters
function handleThings(name, opts = {}) {
  // ...
}

// ❌ Bad - function declarations
function foo() {
  // ...
}

// ❌ Bad - modifying parameters
function f1(obj) {
  obj.key = 1;
}
```

### 4.7 Arrow Functions (Airbnb Standard)

```javascript
// ✅ Good - use arrow functions for inline callbacks
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});

// ✅ Good - implicit return for single expressions
[1, 2, 3].map((x) => x * x);

// ✅ Good - always use parentheses around arguments
[1, 2, 3].map((x) => x * x);

// ❌ Bad - omitting parentheses
[1, 2, 3].map(x => x * x);
```

### 4.8 Modules (Airbnb Standard)

```javascript
// ✅ Good - use import/export
import { es6 } from './AirbnbStyleGuide';
export default es6;

// ✅ Good - import everything you need at the top
import foo from 'foo';
import bar from 'bar';

// ✅ Good - multiline imports
import {
  longNameA,
  longNameB,
  longNameC,
  longNameD,
  longNameE,
} from 'path';

// ❌ Bad - require/module.exports
const AirbnbStyleGuide = require('./AirbnbStyleGuide');
module.exports = AirbnbStyleGuide.es6;

// ❌ Bad - wildcard imports
import * as AirbnbStyleGuide from './AirbnbStyleGuide';
```

### 4.9 Comparison Operators (Airbnb Standard)

```javascript
// ✅ Good - use === and !==
if (name === 'test') {
  // ...
}

// ✅ Good - shortcuts for booleans
if (isValid) {
  // ...
}

// ✅ Good - explicit comparisons for strings and numbers
if (name !== '') {
  // ...
}

if (collection.length > 0) {
  // ...
}

// ❌ Bad - use == and !=
if (name == 'test') {
  // ...
}
```

### 4.10 Comments (Airbnb Standard)

```javascript
// ✅ Good - multiline comments with /** */
/**
 * make() returns a new element
 * based on the passed-in tag name
 */
function make(tag) {
  // ...
  return element;
}

// ✅ Good - single line comments above code
// is current tab
const active = true;

// ✅ Good - use FIXME and TODO
class Calculator extends Abacus {
  constructor() {
    super();
    // FIXME: shouldn't use a global here
    total = 0;
  }
}

// ❌ Bad - inline comments
const active = true; // is current tab
```