# DAPA No-Code Authoring VSCode Extension
### Comprehensive Developer Style & Best Practices Guide (2025 Edition)

This guide is a structured, stepwise roadmap for developing the **DAPA Low-Code/No-Code API Authoring VSCode Extension** using **React 19**, **TypeScript 5.x**, **webpack**, and the **VSCode Extension API**. It incorporates industry-leading patterns from MAANG companies and the Airbnb style guides, emphasizing **scalable architecture**, **code quality**, **component reuse beyond the VSCode extension**, **dynamic configuration**, **multi-REST API integration**, and **open-source readiness**.

---

## Table of Contents

1. [Project Structure](#1-project-structure)
2. [TypeScript Best Practices](#2-typescript-best-practices)
3. [React & JSX Practices](#3-react--jsx-practices)
4. [JavaScript Core Standards](#4-javascript-core-standards)
5. [Performance Best Practices](#5-performance-best-practices)
6. [REST API Integration](#6-rest-api-integration)
7. [Dynamic Configuration](#7-dynamic-configuration)
8. [Code Quality Tools](#8-code-quality-tools)
9. [Testing Strategy](#9-testing-strategy)
10. [VSCode API Integration](#10-vscode-api-integration)
11. [File Naming & Organization](#11-file-naming--organization)
12. [Project Standards Enforcement](#12-project-standards-enforcement)
13. [Governance & Onboarding](#13-governance--onboarding)

---

## 1. Project Structure — Modular Monorepo Pattern

```
dapa/
├── apps/
│   ├── vscode-extension/     # Extension host + React webview UI
│   ├── web-dashboard/        # Optional external/config UI
│   └── cli-tool/             # Supporting CLI or automation
│
├── packages/
│   ├── ui/                   # Reusable React components, design tokens
│   ├── api-client/           # REST/gRPC client abstraction
│   ├── config/               # Dynamic, typed config loader
│   ├── utils/                # Hooks, helpers (pure functions)
│   ├── types/                # Shared TypeScript interfaces
│   └── eslint-config/        # Central lint/format rules and presets
│
└── e2e/                      # Integration and end-to-end tests
```

### Feature-Based Module Organization

```sh
src/features/awesome-feature/
├── api/         # exported API request declarations and api hooks
├── assets/      # static files for this feature
├── components/  # components scoped to this feature
├── hooks/       # hooks scoped to this feature
├── stores/      # state stores for this feature
├── types/       # TypeScript types for this feature
└── utils/       # utility functions for this feature
```

**Key Principles:**
- Decouple features by domain, not just file type
- Apps consume shared packages, enabling UI reuse across VSCode, web, and CLI
- Each feature folder contains code specific to that feature
- Avoid cross-feature imports; compose features at the application level

**Anti-pattern:** Flat structures with loose file grouping create dependency mess and inhibit scalability

---

## 2. TypeScript Best Practices

### 2.1 Always Use Explicit Types

```ts
// ✅ Recommended
interface User {
  id: string;
  name: string;
  email: string;
}

const getUser = async (id: string): Promise<User> => {
  return api.fetch<User>(`/users/${id}`);
};

// ❌ Anti-pattern
const getUserData = async (id) => api.fetch(`/users/${id}`);
```

### 2.2 Prefer `unknown` Over `any` for Safety

```ts
// ✅ Good
let payload: unknown;
if (typeof payload === 'object' && payload !== null) {
  // safe to use as object here
}

// ❌ Bad
let payload: any;
```

### 2.3 Use Immutable Data with `readonly`

```ts
// ✅ Good
interface Config {
  readonly baseUrl: string;
  readonly timeout: number;
}

const config: Config = {
  baseUrl: 'https://api.dapa.io',
  timeout: 5000,
};

// ❌ Bad - allows mutation
interface BadConfig {
  baseUrl: string;
  timeout: number;
}
```

### 2.4 Use Const Assertions for Literal Types

```ts
// ✅ Good - preserves literal types
const routes = {
  home: '/home',
  about: '/about',
} as const;

// ❌ Bad - loses literal types
const routes = {
  home: '/home',
  about: '/about',
};
```

### 2.5 Prefer Interfaces for Object Shapes

```ts
// ✅ Good - interfaces are more extensible
interface UserProps {
  name: string;
  age: number;
}

// ⚠️ Use type for unions, intersections, and primitives
type Status = 'active' | 'inactive' | 'pending';
type ID = string | number;
```

---

## 3. React & JSX Practices

### 3.1 Component Declaration (Airbnb Standard)

```tsx
// ✅ Good - Functional components with explicit typing
interface ButtonProps {
  onClick: () => void;
  disabled?: boolean;
  children: React.ReactNode;
}

function Button({ onClick, disabled = false, children }: ButtonProps) {
  return (
    <button disabled={disabled} onClick={onClick}>
      {children}
    </button>
  );
}

// ❌ Bad - Class component for simple UI
class BadButton extends React.Component<ButtonProps> {
  render() {
    return (
      <button onClick={this.props.onClick}>
        {this.props.children}
      </button>
    );
  }
}
```

### 3.2 File Extensions and Naming (Airbnb Standard)

```tsx
// ✅ Good - Use .tsx extension for React components
// File: ReservationCard.tsx
interface ReservationCardProps {
  title: string;
}

export function ReservationCard({ title }: ReservationCardProps) {
  return <div>{title}</div>;
}

// ✅ Good - PascalCase for component names, camelCase for instances
import ReservationCard from './ReservationCard';
const reservationItem = <ReservationCard title="Room 101" />;

// ❌ Bad
import reservationCard from './ReservationCard';
const ReservationItem = <ReservationCard title="Room 101" />;
```

### 3.3 JSX Formatting and Alignment (Airbnb Standard)

```tsx
// ✅ Good - multiline props with closing bracket on new line
<Foo
  superLongParam="bar"
  anotherSuperLongParam="baz"
>
  <Quux />
</Foo>

// ✅ Good - single line when props fit
<Foo bar="bar" />

// ✅ Good - conditional rendering with parentheses
{showButton && (
  <Button />
)}

// ❌ Bad
<Foo superLongParam="bar"
     anotherSuperLongParam="baz" />

// ❌ Bad
{showButton &&
  <Button />
}
```

### 3.4 Props and Naming (Airbnb Standard)

```tsx
// ✅ Good - camelCase for props, omit boolean true
<Foo
  userName="hello"
  phoneNumber={12345678}
  Component={SomeComponent}
  hidden
/>

// ❌ Bad
<Foo
  UserName="hello"
  phone_number={12345678}
  hidden={true}
/>
```

### 3.5 Quotes in JSX (Airbnb Standard)

```tsx
// ✅ Good - double quotes for JSX attributes, single for JS
<Foo bar="bar" />
<Foo style={{ left: '20px' }} />

// ❌ Bad
<Foo bar='bar' />
<Foo style={{ left: "20px" }} />
```

### 3.6 Hook Extraction for Async Logic

```ts
// ✅ Good - custom hook for reusable async logic
function useApi<T>(endpoint: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);

    fetch(endpoint)
      .then((res) => res.json())
      .then((json) => {
        if (!cancelled) {
          setData(json);
          setLoading(false);
        }
      })
      .catch((err) => {
        if (!cancelled) {
          setError(err);
          setLoading(false);
        }
      });

    return () => {
      cancelled = true;
    };
  }, [endpoint]);

  return { data, loading, error };
}
```

### 3.7 Refs (Airbnb Standard)

```tsx
// ✅ Good - use useRef hook in functional components
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} />;
}

// ❌ Bad - string refs (deprecated)
<Foo ref="myRef" />
```

### 3.8 Keys in Lists (Airbnb Standard)

```tsx
// ✅ Good - stable ID as key
{todos.map((todo) => (
  <Todo {...todo} key={todo.id} />
))}

// ❌ Bad - array index as key
{todos.map((todo, index) => (
  <Todo {...todo} key={index} />
))}
```

### 3.9 Atomic Design System for UI Reuse

| Level | Description | Example |
|-------|-------------|---------|
| Atoms | Basic UI elements | `<Button>`, `<Input>`, `<Icon>` |
| Molecules | Combinations of atoms | `<SearchBar>`, `<FormField>` |
| Organisms | Complex UI sections | `<ConfigPanel>`, `<DataTable>` |
| Templates | Page layouts | `<AuthoringWorkspace>` |
| Pages | Complete pages | `<DashboardPage>` |

**Recommendation:** Use Storybook to document components; publish `packages/ui` as a separate library for reuse across projects.

---

## 4. JavaScript Core Standards

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

---

## 5. Performance Best Practices

### 5.1 Lazy Loading & Suspense

```tsx
// ✅ Good - lazy load components
const LazyPanel = React.lazy(() => import('./Panel'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <LazyPanel />
    </Suspense>
  );
}
```

### 5.2 Memoization

```tsx
// ✅ Good - memo for expensive renders
const MemoList = React.memo(({ items }: { items: string[] }) => (
  <ul>
    {items.map((i) => (
      <li key={i}>{i}</li>
    ))}
  </ul>
));

// ✅ Good - useMemo for expensive computations
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);

// ✅ Good - useCallback for stable function references
const handleClick = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

### 5.3 Avoid Inline Functions in Render (Airbnb Standard)

```tsx
// ✅ Good - stable callback reference
const handleClick = useCallback(() => {
  /* logic */
}, []);

<button onClick={handleClick}>Click</button>;

// ❌ Bad - creates new function on every render
<button onClick={() => alert('clicked')}>Click</button>;
```

### 5.4 Prefer Functional Updates

```tsx
// ✅ Good - functional update
setCount((prevCount) => prevCount + 1);

// ⚠️ Acceptable but not recommended
setCount(count + 1);
```

---

## 6. REST API Integration

### 6.1 Dev-time Proxy Setup

```javascript
// setupProxy.js
const { createProxyMiddleware } = require('http-proxy-middleware');

module.exports = function(app) {
  app.use(
    '/api/users',
    createProxyMiddleware({
      target: 'https://users.api',
      changeOrigin: true,
    })
  );

  app.use(
    '/api/config',
    createProxyMiddleware({
      target: 'https://config.api',
      changeOrigin: true,
    })
  );
};
```

### 6.2 Typed API Client

```ts
// ✅ Good - typed API client with error handling
class ApiClient {
  constructor(private baseUrl: string) {}

  async get<T>(path: string): Promise<T> {
    const response = await fetch(`${this.baseUrl}${path}`);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return response.json() as Promise<T>;
  }

  async post<T, D>(path: string, data: D): Promise<T> {
    const response = await fetch(`${this.baseUrl}${path}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return response.json() as Promise<T>;
  }
}

// Usage
const api = new ApiClient('https://api.dapa.io');
const user = await api.get<User>('/users/123');
```

### 6.3 React Query Integration

```tsx
// ✅ Good - use React Query for data fetching
import { useQuery, useMutation } from '@tanstack/react-query';

function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => api.get<User>(`/users/${userId}`),
  });
}

function useUpdateUser() {
  return useMutation({
    mutationFn: (data: UpdateUserData) => api.post('/users', data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}
```

---

## 7. Dynamic Configuration

### 7.1 Runtime Configuration Loading

```ts
// ✅ Good - load configuration at runtime
import * as vscode from 'vscode';

function getConfig() {
  const config = vscode.workspace.getConfiguration('dapa');

  return {
    apiUrl: config.get<string>('apiBaseUrl') ?? 'https://default.api',
    theme: config.get<string>('theme') ?? 'light',
    timeout: config.get<number>('timeout') ?? 5000,
  };
}
```

### 7.2 User-Configurable Settings in package.json

```json
{
  "contributes": {
    "configuration": {
      "title": "DAPA",
      "properties": {
        "dapa.apiBaseUrl": {
          "type": "string",
          "default": "https://api.dapa.io",
          "description": "API base URL for DAPA services"
        },
        "dapa.theme": {
          "type": "string",
          "enum": ["light", "dark", "auto"],
          "default": "auto",
          "description": "UI theme preference"
        },
        "dapa.timeout": {
          "type": "number",
          "default": 5000,
          "description": "API request timeout in milliseconds"
        }
      }
    }
  }
}
```

---

## 8. Code Quality Tools

### 8.1 ESLint Configuration

```javascript
// .eslintrc.cjs
module.exports = {
  root: true,
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2023,
    sourceType: 'module',
    ecmaFeatures: {
      jsx: true,
    },
    project: './tsconfig.json',
  },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:@typescript-eslint/recommended-requiring-type-checking',
    'plugin:react/recommended',
    'plugin:react/jsx-runtime',
    'plugin:react-hooks/recommended',
    'plugin:jsx-a11y/recommended',
    'plugin:sonarjs/recommended',
    'airbnb',
    'airbnb-typescript',
    'airbnb/hooks',
    'prettier',
  ],
  plugins: [
    'react',
    '@typescript-eslint',
    'sonarjs',
    'jsx-a11y',
    'check-file',
  ],
  rules: {
    // React
    'react/react-in-jsx-scope': 'off',
    'react/jsx-filename-extension': ['error', { extensions: ['.tsx'] }],
    'react/prop-types': 'off',
    'react/require-default-props': 'off',
    'react/jsx-props-no-spreading': ['error', {
      exceptions: ['Component'],
    }],

    // TypeScript
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-explicit-any': 'error',

    // Import
    'import/prefer-default-export': 'off',
    'import/no-default-export': 'off',
    'import/extensions': ['error', 'ignorePackages', {
      ts: 'never',
      tsx: 'never',
    }],

    // General
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'prefer-const': 'error',
    'no-var': 'error',

    // SonarJS
    'sonarjs/cognitive-complexity': ['error', 15],
    'sonarjs/no-duplicate-string': ['error', 3],

    // Airbnb overrides for TypeScript
    'no-use-before-define': 'off',
    '@typescript-eslint/no-use-before-define': ['error'],
  },
  settings: {
    react: {
      version: 'detect',
    },
    'import/resolver': {
      typescript: {
        alwaysTryTypes: true,
        project: './tsconfig.json',
      },
    },
  },
};
```

### 8.2 Prettier Configuration

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "arrowParens": "always",
  "endOfLine": "lf",
  "bracketSpacing": true,
  "jsxSingleQuote": false,
  "jsxBracketSameLine": false
}
```

### 8.3 File Naming Conventions

```javascript
// Add to ESLint config
rules: {
  'check-file/filename-naming-convention': [
    'error',
    {
      '**/*.{ts,tsx}': 'KEBAB_CASE',
    },
    {
      ignoreMiddleExtensions: true,
    },
  ],
  'check-file/folder-naming-convention': [
    'error',
    {
      'src/**/!(__tests__)': 'KEBAB_CASE',
    },
  ],
}
```

### 8.4 SonarQube Integration

```javascript
// sonarqube-scanner.js
const scanner = require('sonarqube-scanner');

scanner(
  {
    serverUrl: process.env.SONAR_HOST_URL,
    token: process.env.SONAR_TOKEN,
    options: {
      'sonar.projectKey': 'dapa-vscode-extension',
      'sonar.projectName': 'DAPA VSCode Extension',
      'sonar.sources': 'packages/',
      'sonar.tests': 'e2e/',
      'sonar.typescript.tsconfigPaths': 'tsconfig.json',
      'sonar.javascript.lcov.reportPaths': 'coverage/lcov.info',
      'sonar.sourceEncoding': 'UTF-8',
      'sonar.exclusions': '**/node_modules/**,**/dist/**',
    },
  },
  () => process.exit()
);
```

### 8.5 NPM Scripts

```json
{
  "scripts": {
    "lint": "eslint 'packages/**/*.{ts,tsx}'",
    "lint:fix": "eslint 'packages/**/*.{ts,tsx}' --fix",
    "format": "prettier --check .",
    "format:fix": "prettier --write .",
    "type-check": "tsc --noEmit",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "sonar": "node sonarqube-scanner.js"
  }
}
```

---

## 9. Testing Strategy

### 9.1 Unit Tests with Vitest

```ts
// ✅ Good - unit test example
import { describe, it, expect } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './use-counter';

describe('useCounter', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('should increment counter', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });
});
```

### 9.2 Component Tests with Testing Library

```tsx
// ✅ Good - component test example
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './button';

describe('Button', () => {
  it('should render children', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('should call onClick when clicked', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('should be disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByText('Click me')).toBeDisabled();
  });
});
```

### 9.3 Integration Tests with Playwright

```ts
// ✅ Good - E2E test example
import { test, expect } from '@playwright/test';

test.describe('DAPA Extension', () => {
  test('should open extension panel', async ({ page }) => {
    await page.goto('http://localhost:3000');

    // Open command palette
    await page.keyboard.press('Control+Shift+P');

    // Type command
    await page.keyboard.type('DAPA: Open Panel');
    await page.keyboard.press('Enter');

    // Verify panel is visible
    await expect(page.locator('[data-testid="dapa-panel"]')).toBeVisible();
  });
});
```

### 9.4 Coverage Requirements

```javascript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      lines: 85,
      functions: 85,
      branches: 85,
      statements: 85,
    },
  },
});
```

---

## 10. VSCode API Integration

### 10.1 Extension Activation

```ts
// ✅ Good - extension activation
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  console.log('DAPA extension is now active');

  const disposable = vscode.commands.registerCommand('dapa.start', () => {
    vscode.window.showInformationMessage('DAPA Started!');
  });

  context.subscriptions.push(disposable);
}

export function deactivate() {
  console.log('DAPA extension is now deactivated');
}
```

### 10.2 Webview Setup for React

```ts
// ✅ Good - webview with React
import * as vscode from 'vscode';
import * as path from 'path';

export class DapaPanel {
  public static currentPanel: DapaPanel | undefined;
  private readonly _panel: vscode.WebviewPanel;
  private _disposables: vscode.Disposable[] = [];

  private constructor(panel: vscode.WebviewPanel, extensionUri: vscode.Uri) {
    this._panel = panel;

    this._panel.webview.html = this._getHtmlForWebview(
      this._panel.webview,
      extensionUri
    );

    this._panel.onDidDispose(() => this.dispose(), null, this._disposables);
  }

  public static createOrShow(extensionUri: vscode.Uri) {
    const column = vscode.window.activeTextEditor
      ? vscode.window.activeTextEditor.viewColumn
      : undefined;

    if (DapaPanel.currentPanel) {
      DapaPanel.currentPanel._panel.reveal(column);
      return;
    }

    const panel = vscode.window.createWebviewPanel(
      'dapa',
      'DAPA Panel',
      column || vscode.ViewColumn.One,
      {
        enableScripts: true,
        localResourceRoots: [
          vscode.Uri.joinPath(extensionUri, 'media'),
        ],
      }
    );

    DapaPanel.currentPanel = new DapaPanel(panel, extensionUri);
  }

  private _getHtmlForWebview(
    webview: vscode.Webview,
    extensionUri: vscode.Uri
  ) {
    const scriptUri = webview.asWebviewUri(
      vscode.Uri.joinPath(extensionUri, 'media', 'main.js')
    );

    const styleUri = webview.asWebviewUri(
      vscode.Uri.joinPath(extensionUri, 'media', 'main.css')
    );

    const nonce = getNonce();

    return `<!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta http-equiv="Content-Security-Policy" content="default-src 'none'; style-src ${webview.cspSource}; script-src 'nonce-${nonce}';">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link href="${styleUri}" rel="stylesheet">
        <title>DAPA Panel</title>
      </head>
      <body>
        <div id="root"></div>
        <script nonce="${nonce}" src="${scriptUri}"></script>
      </body>
      </html>`;
  }

  public dispose() {
    DapaPanel.currentPanel = undefined;

    this._panel.dispose();

    while (this._disposables.length) {
      const disposable = this._disposables.pop();
      if (disposable) {
        disposable.dispose();
      }
    }
  }
}

function getNonce() {
  let text = '';
  const possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  for (let i = 0; i < 32; i++) {
    text += possible.charAt(Math.floor(Math.random() * possible.length));
  }
  return text;
}
```

---

## 11. File Naming & Organization

### 11.1 Absolute Imports Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@features/*": ["./src/features/*"],
      "@utils/*": ["./src/utils/*"]
    }
  }
}
```

### 11.2 File Naming Standards

- **React Components**: `PascalCase` (e.g., `UserProfile.tsx`)
- **Hooks**: `camelCase` with `use` prefix (e.g., `useAuth.ts`)
- **Utilities**: `kebab-case` (e.g., `format-date.ts`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `API_ENDPOINTS.ts`)
- **Types/Interfaces**: `PascalCase` (e.g., `UserTypes.ts`)

### 11.3 Prevent Cross-Feature Imports

```javascript
// Add to ESLint config
rules: {
  'import/no-restricted-paths': [
    'error',
    {
      zones: [
        // Disallow cross-feature imports
        {
          target: './src/features/auth',
          from: './src/features',
          except: ['./auth'],
        },
        {
          target: './src/features/dashboard',
          from: './src/features',
          except: ['./dashboard'],
        },
        // Enforce unidirectional codebase architecture
        {
          target: './src/features',
          from: './src/app',
        },
        {
          target: [
            './src/components',
            './src/hooks',
            './src/utils',
          ],
          from: ['./src/features', './src/app'],
        },
      ],
    },
  ],
}
```

---

## 12. Project Standards Enforcement

### 12.1 Pre-commit Hooks with Husky

```json
// package.json
{
  "scripts": {
    "prepare": "husky install"
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

```bash
# .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
```

### 12.2 Conventional Commits

```bash
# .husky/commit-msg
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx --no -- commitlint --edit "$1"
```

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'style',
        'refactor',
        'perf',
        'test',
        'chore',
        'revert',
      ],
    ],
  },
};
```

---

## 13. Governance & Onboarding

### 13.1 Pull Request Requirements

- ✅ All tests pass
- ✅ Code coverage meets threshold (85%+)
- ✅ ESLint and Prettier checks pass
- ✅ SonarQube quality gate passes
- ✅ At least 2 approving reviews
- ✅ Conventional commit message format
- ✅ Updated documentation if needed

### 13.2 Developer Environment Setup

```bash
# Clone repository
git clone https://github.com/your-org/dapa.git
cd dapa

# Install dependencies
npm install

# Setup pre-commit hooks
npm run prepare

# Run type checking
npm run type-check

# Run tests
npm test

# Run linting
npm run lint

# Start development server
npm run dev
```

### 13.3 Documentation Standards

- README.md in each package with usage examples
- JSDoc comments for public APIs
- Storybook for component documentation
- ADRs (Architecture Decision Records) for major decisions

---

## 14. Schema-Driven Component Generation

### 14.1 Overview: Contract-First Development

DAPA follows a **contract-first, schema-driven architecture** where UI components, state management, and validation logic are **forward-engineered** from authoritative schema sources rather than hand-coded. This ensures:

- **Single Source of Truth**: Schemas define both the contract and the implementation
- **Type Safety**: Generated TypeScript interfaces match the exact schema structure
- **Consistency**: UI components automatically align with API specifications
- **Maintainability**: Schema updates cascade through the system automatically
- **Standards Compliance**: Direct use of official specifications (OpenAPI, ANTLR)

### 14.2 Architecture Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                   Schema Sources                             │
│  ┌──────────────────┐  ┌──────────────────┐                │
│  │ OpenAPI 3.0/3.1  │  │  ANTLR Grammars  │                │
│  │   JSON Schema    │  │   (.g4 files)    │                │
│  └────────┬─────────┘  └────────┬─────────┘                │
└───────────┼─────────────────────┼──────────────────────────┘
            │                     │
            ▼                     ▼
┌─────────────────────────────────────────────────────────────┐
│              Code Generation Layer                           │
│  ┌──────────────────┐  ┌──────────────────┐                │
│  │ Schema Parser    │  │ Grammar Parser   │                │
│  │ (JSON Schema)    │  │ (ANTLR4)         │                │
│  └────────┬─────────┘  └────────┬─────────┘                │
│           │                     │                            │
│           └──────────┬──────────┘                            │
│                      ▼                                       │
│           ┌──────────────────────┐                          │
│           │  Generator Engine    │                          │
│           │  - TS Interfaces     │                          │
│           │  - React Components  │                          │
│           │  - Redux Actions     │                          │
│           │  - Validators        │                          │
│           └──────────┬───────────┘                          │
└──────────────────────┼──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              Generated Artifacts                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ packages/generated/                                   │  │
│  │ ├── openapi/                                         │  │
│  │ │   ├── types/          # TS interfaces             │  │
│  │ │   ├── components/     # React form components     │  │
│  │ │   ├── validators/     # Zod/Yup schemas          │  │
│  │ │   └── redux/          # Redux slices              │  │
│  │ └── dsl/                                             │  │
│  │     ├── types/          # AST node types            │  │
│  │     ├── components/     # Editor components         │  │
│  │     ├── parsers/        # ANTLR-generated parsers   │  │
│  │     └── redux/          # State management          │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│           DAPA No-Code Authoring UI                          │
│  ┌──────────────────┐  ┌──────────────────┐                │
│  │  API Editor      │  │  Data Model      │                │
│  │  (OpenAPI)       │  │  Editor (DSL)    │                │
│  └──────────────────┘  └──────────────────┘                │
└─────────────────────────────────────────────────────────────┘
```

### 14.3 OpenAPI Schema Integration for API Editors

#### 14.3.1 Schema Sources

DAPA supports both OpenAPI 3.0 and 3.1:

```typescript
// packages/config/src/openapi-schemas.ts
export const OPENAPI_SCHEMAS = {
  '3.0': 'https://spec.openapis.org/oas/3.0/schema/2024-10-18.html',
  '3.1': 'https://spec.openapis.org/oas/3.1/schema/2025-09-15.html',
} as const;

export type OpenAPIVersion = keyof typeof OPENAPI_SCHEMAS;
```

#### 14.3.2 Schema-to-TypeScript Generation

```typescript
// packages/generators/src/openapi/schema-to-types.ts
import { JSONSchema7 } from 'json-schema';

/**
 * ✅ Good - Generate TypeScript interfaces from OpenAPI schema
 */
export interface GeneratorOptions {
  schemaVersion: OpenAPIVersion;
  outputDir: string;
  includeValidation: boolean;
  includeComponents: boolean;
}

export async function generateTypesFromSchema(
  schemaUrl: string,
  options: GeneratorOptions
): Promise<void> {
  // 1. Fetch and parse the JSON Schema
  const schema = await fetchSchema(schemaUrl);
  
  // 2. Generate TypeScript interfaces
  await generateInterfaces(schema, options);
  
  // 3. Generate Zod validators (optional)
  if (options.includeValidation) {
    await generateValidators(schema, options);
  }
  
  // 4. Generate React components (optional)
  if (options.includeComponents) {
    await generateComponents(schema, options);
  }
}

/**
 * Example: Generate interface from OpenAPI Info Object schema
 */
function generateInfoInterface(schemaDef: JSONSchema7): string {
  return `
export interface OpenAPIInfo {
  /** ${schemaDef.properties?.title.description || ''} */
  title: string;
  
  /** ${schemaDef.properties?.summary.description || ''} */
  summary?: string;
  
  /** ${schemaDef.properties?.description.description || ''} */
  description?: string;
  
  /** ${schemaDef.properties?.termsOfService.description || ''} */
  termsOfService?: string;
  
  contact?: OpenAPIContact;
  license?: OpenAPILicense;
  
  /** ${schemaDef.properties?.version.description || ''} */
  version: string;
}
`;
}
```

#### 14.3.3 Component Generation from Schema

```typescript
// packages/generators/src/openapi/schema-to-components.tsx
import { JSONSchema7 } from 'json-schema';

/**
 * ✅ Good - Generate React form components from schema definitions
 */
export function generateFormComponent(
  objectName: string,
  schemaDef: JSONSchema7
): string {
  const properties = schemaDef.properties || {};
  const required = schemaDef.required || [];

  return `
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { ${objectName}Schema, type ${objectName} } from '../types';

export interface ${objectName}FormProps {
  initialData?: Partial<${objectName}>;
  onSubmit: (data: ${objectName}) => void;
  onCancel?: () => void;
}

export function ${objectName}Form({
  initialData,
  onSubmit,
  onCancel,
}: ${objectName}FormProps) {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<${objectName}>({
    resolver: zodResolver(${objectName}Schema),
    defaultValues: initialData,
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      ${generateFormFields(properties, required)}
      
      <div className="form-actions">
        <button type="submit">Save</button>
        {onCancel && (
          <button type="button" onClick={onCancel}>
            Cancel
          </button>
        )}
      </div>
    </form>
  );
}
`;
}

function generateFormFields(
  properties: Record<string, JSONSchema7>,
  required: string[]
): string {
  return Object.entries(properties)
    .map(([name, prop]) => {
      const isRequired = required.includes(name);
      const type = prop.type as string;
      
      return `
      <div className="form-field">
        <label htmlFor="${name}">
          ${formatLabel(name)}
          ${isRequired ? '<span className="required">*</span>' : ''}
        </label>
        ${generateInputElement(name, type, prop)}
        {errors.${name} && (
          <span className="error">{errors.${name}?.message}</span>
        )}
      </div>
      `;
    })
    .join('\n');
}
```

#### 14.3.4 Redux State Management from Schema

```typescript
// packages/generators/src/openapi/schema-to-redux.ts
import { JSONSchema7 } from 'json-schema';

/**
 * ✅ Good - Generate Redux slice from OpenAPI schema
 */
export function generateReduxSlice(
  objectName: string,
  schemaDef: JSONSchema7
): string {
  return `
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import { ${objectName} } from '../types';

interface ${objectName}State {
  items: ${objectName}[];
  current: ${objectName} | null;
  loading: boolean;
  error: string | null;
}

const initialState: ${objectName}State = {
  items: [],
  current: null,
  loading: false,
  error: null,
};

export const ${objectName.toLowerCase()}Slice = createSlice({
  name: '${objectName.toLowerCase()}',
  initialState,
  reducers: {
    set${objectName}: (state, action: PayloadAction<${objectName}>) => {
      state.current = action.payload;
    },
    
    update${objectName}Field: (
      state,
      action: PayloadAction<{ field: keyof ${objectName}; value: unknown }>
    ) => {
      if (state.current) {
        state.current[action.payload.field] = action.payload.value as never;
      }
    },
    
    add${objectName}: (state, action: PayloadAction<${objectName}>) => {
      state.items.push(action.payload);
    },
    
    remove${objectName}: (state, action: PayloadAction<string>) => {
      state.items = state.items.filter((item) => item.id !== action.payload);
    },
    
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.loading = action.payload;
    },
    
    setError: (state, action: PayloadAction<string | null>) => {
      state.error = action.payload;
    },
  },
});

export const {
  set${objectName},
  update${objectName}Field,
  add${objectName},
  remove${objectName},
  setLoading,
  setError,
} = ${objectName.toLowerCase()}Slice.actions;

export default ${objectName.toLowerCase()}Slice.reducer;
`;
}
```

#### 14.3.5 Validation Schema Generation

```typescript
// packages/generators/src/openapi/schema-to-validators.ts
import { JSONSchema7 } from 'json-schema';

/**
 * ✅ Good - Generate Zod schemas for runtime validation
 */
export function generateZodSchema(
  objectName: string,
  schemaDef: JSONSchema7
): string {
  const properties = schemaDef.properties || {};
  const required = schemaDef.required || [];

  const zodFields = Object.entries(properties)
    .map(([name, prop]) => {
      const isRequired = required.includes(name);
      return generateZodField(name, prop, isRequired);
    })
    .join(',\n  ');

  return `
import { z } from 'zod';

export const ${objectName}Schema = z.object({
  ${zodFields}
});

export type ${objectName} = z.infer<typeof ${objectName}Schema>;
`;
}

function generateZodField(
  name: string,
  prop: JSONSchema7,
  isRequired: boolean
): string {
  let zodType = 'z.unknown()';

  switch (prop.type) {
    case 'string':
      zodType = 'z.string()';
      if (prop.format === 'email') zodType += '.email()';
      if (prop.format === 'uri') zodType += '.url()';
      if (prop.minLength) zodType += `.min(${prop.minLength})`;
      if (prop.maxLength) zodType += `.max(${prop.maxLength})`;
      if (prop.pattern) zodType += `.regex(/${prop.pattern}/)`;
      break;
    case 'number':
    case 'integer':
      zodType = 'z.number()';
      if (prop.minimum) zodType += `.min(${prop.minimum})`;
      if (prop.maximum) zodType += `.max(${prop.maximum})`;
      break;
    case 'boolean':
      zodType = 'z.boolean()';
      break;
    case 'array':
      zodType = `z.array(${generateZodField('item', prop.items as JSONSchema7, true)})`;
      break;
    case 'object':
      zodType = 'z.object({})';
      break;
  }

  if (!isRequired) {
    zodType += '.optional()';
  }

  if (prop.description) {
    zodType += `.describe('${prop.description}')`;
  }

  return `${name}: ${zodType}`;
}
```

### 14.4 ANTLR Grammar Integration for DSL Editors

#### 14.4.1 Grammar-Driven Component Generation

```typescript
// packages/generators/src/antlr/grammar-to-types.ts

/**
 * ✅ Good - Parse ANTLR grammar and generate TypeScript AST types
 */
export interface AntlrGrammarConfig {
  grammarFile: string;
  grammarName: string;
  outputDir: string;
  language: 'TypeScript' | 'JavaScript';
}

export async function generateFromGrammar(
  config: AntlrGrammarConfig
): Promise<void> {
  // 1. Parse ANTLR grammar file
  const grammar = await parseGrammarFile(config.grammarFile);
  
  // 2. Generate lexer and parser using ANTLR4
  await generateAntlrParser(grammar, config);
  
  // 3. Generate TypeScript visitor interfaces
  await generateVisitorInterfaces(grammar, config);
  
  // 4. Generate AST node types
  await generateASTTypes(grammar, config);
  
  // 5. Generate editor components
  await generateEditorComponents(grammar, config);
}

/**
 * Example: Generate AST types from TaxiLang grammar
 */
export function generateASTTypes(grammar: Grammar): string {
  return `
// Generated from ${grammar.name}.g4
export interface ASTNode {
  type: string;
  start: number;
  end: number;
  children?: ASTNode[];
}

// Type Declaration Node (from 'typeDeclaration' rule)
export interface TypeDeclarationNode extends ASTNode {
  type: 'TypeDeclaration';
  modifiers: TypeModifier[];
  kind: 'Type' | 'Model';
  identifier: string;
  typeArguments?: TypeArgumentNode[];
  inherits?: TypeReference[];
  body?: TypeBodyNode;
}

// Type Body Node (from 'typeBody' rule)
export interface TypeBodyNode extends ASTNode {
  type: 'TypeBody';
  members: TypeMemberNode[];
  spreadOperator?: SpreadOperatorNode;
}

// Field Declaration Node (from 'fieldDeclaration' rule)
export interface FieldDeclarationNode extends ASTNode {
  type: 'FieldDeclaration';
  modifier?: 'closed';
  identifier: string;
  fieldType?: FieldTypeDeclarationNode | AnonymousTypeNode;
  typeProjection?: TypeProjectionNode;
  annotations: AnnotationNode[];
}

// Service Declaration Node (from 'serviceDeclaration' rule)
export interface ServiceDeclarationNode extends ASTNode {
  type: 'ServiceDeclaration';
  identifier: string;
  operations: OperationNode[];
  tables: TableNode[];
  streams: StreamNode[];
  annotations: AnnotationNode[];
}

// Operation Node (from 'serviceOperationDeclaration' rule)
export interface OperationNode extends ASTNode {
  type: 'Operation';
  scope?: 'read' | 'write';
  identifier: string;
  parameters: OperationParameterNode[];
  returnType?: TypeReference;
  annotations: AnnotationNode[];
}
`;
}
```

#### 14.4.2 Editor Component Generation from Grammar

```typescript
// packages/generators/src/antlr/grammar-to-components.tsx

/**
 * ✅ Good - Generate React editor components from ANTLR rules
 */
export function generateEditorComponent(
  ruleName: string,
  rule: GrammarRule
): string {
  return `
import { useDispatch, useSelector } from 'react-redux';
import { update${ruleName}Field } from '../redux/${ruleName.toLowerCase()}-slice';
import { RootState } from '../redux/store';

export interface ${ruleName}EditorProps {
  nodeId: string;
  onChange?: (node: ${ruleName}Node) => void;
}

export function ${ruleName}Editor({ nodeId, onChange }: ${ruleName}EditorProps) {
  const dispatch = useDispatch();
  const node = useSelector((state: RootState) => 
    state.ast.nodes[nodeId] as ${ruleName}Node
  );

  const handleFieldChange = (field: keyof ${ruleName}Node, value: unknown) => {
    dispatch(update${ruleName}Field({ nodeId, field, value }));
    if (onChange) {
      onChange({ ...node, [field]: value });
    }
  };

  return (
    <div className="${ruleName.toLowerCase()}-editor">
      <h3>${formatLabel(ruleName)}</h3>
      ${generateEditorFields(rule)}
    </div>
  );
}
`;
}

function generateEditorFields(rule: GrammarRule): string {
  return rule.elements
    .map((element) => {
      if (element.type === 'terminal') {
        return generateTerminalEditor(element);
      } else if (element.type === 'nonterminal') {
        return generateNonterminalEditor(element);
      } else if (element.type === 'choice') {
        return generateChoiceEditor(element);
      }
      return '';
    })
    .join('\n');
}
```

#### 14.4.3 Redux State for AST Management

```typescript
// packages/generators/src/antlr/grammar-to-redux.ts

/**
 * ✅ Good - Generate Redux slice for AST state management
 */
export function generateASTReduxSlice(grammarName: string): string {
  return `
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import { ASTNode } from '../types/ast';

interface ASTState {
  nodes: Record<string, ASTNode>;
  root: string | null;
  selectedNode: string | null;
  errors: ValidationError[];
}

const initialState: ASTState = {
  nodes: {},
  root: null,
  selectedNode: null,
  errors: [],
};

export const astSlice = createSlice({
  name: '${grammarName.toLowerCase()}-ast',
  initialState,
  reducers: {
    setAST: (state, action: PayloadAction<ASTNode>) => {
      const nodes: Record<string, ASTNode> = {};
      const flattenNodes = (node: ASTNode) => {
        const id = generateNodeId(node);
        nodes[id] = node;
        if (node.children) {
          node.children.forEach(flattenNodes);
        }
      };
      flattenNodes(action.payload);
      state.nodes = nodes;
      state.root = generateNodeId(action.payload);
    },
    
    updateNode: (
      state,
      action: PayloadAction<{ nodeId: string; updates: Partial<ASTNode> }>
    ) => {
      const { nodeId, updates } = action.payload;
      if (state.nodes[nodeId]) {
        state.nodes[nodeId] = { ...state.nodes[nodeId], ...updates };
      }
    },
    
    addChildNode: (
      state,
      action: PayloadAction<{ parentId: string; child: ASTNode }>
    ) => {
      const { parentId, child } = action.payload;
      const parent = state.nodes[parentId];
      if (parent) {
        const childId = generateNodeId(child);
        state.nodes[childId] = child;
        parent.children = [...(parent.children || []), child];
      }
    },
    
    removeNode: (state, action: PayloadAction<string>) => {
      const nodeId = action.payload;
      const removeRecursive = (id: string) => {
        const node = state.nodes[id];
        if (node?.children) {
          node.children.forEach((child) => removeRecursive(generateNodeId(child)));
        }
        delete state.nodes[id];
      };
      removeRecursive(nodeId);
    },
    
    selectNode: (state, action: PayloadAction<string | null>) => {
      state.selectedNode = action.payload;
    },
    
    setErrors: (state, action: PayloadAction<ValidationError[]>) => {
      state.errors = action.payload;
    },
  },
});

export const {
  setAST,
  updateNode,
  addChildNode,
  removeNode,
  selectNode,
  setErrors,
} = astSlice.actions;

export default astSlice.reducer;

function generateNodeId(node: ASTNode): string {
  return \`\${node.type}-\${node.start}-\${node.end}\`;
}
`;
}
```

### 14.5 Build Pipeline Integration

#### 14.5.1 NPM Scripts for Code Generation

```json
{
  "scripts": {
    "generate": "npm run generate:openapi && npm run generate:antlr",
    "generate:openapi": "ts-node scripts/generate-openapi.ts",
    "generate:openapi:3.0": "ts-node scripts/generate-openapi.ts --version 3.0",
    "generate:openapi:3.1": "ts-node scripts/generate-openapi.ts --version 3.1",
    "generate:antlr": "ts-node scripts/generate-antlr.ts",
    "generate:antlr:taxi": "antlr4ts -visitor grammars/Taxi.g4 -o packages/generated/dsl/taxi",
    "prebuild": "npm run generate",
    "watch:generate": "nodemon --watch schemas --watch grammars --exec npm run generate"
  }
}
```

#### 14.5.2 Generation Script Example

```typescript
// scripts/generate-openapi.ts
import { program } from 'commander';
import { generateTypesFromSchema } from '../packages/generators/src/openapi';

program
  .option('-v, --version <version>', 'OpenAPI version (3.0 or 3.1)', '3.1')
  .option('-o, --output <dir>', 'Output directory', 'packages/generated/openapi')
  .option('--validation', 'Generate validation schemas', true)
  .option('--components', 'Generate React components', true)
  .parse();

const options = program.opts();

async function main() {
  console.log(`Generating OpenAPI ${options.version} artifacts...`);
  
  const schemaUrl = OPENAPI_SCHEMAS[options.version as OpenAPIVersion];
  
  await generateTypesFromSchema(schemaUrl, {
    schemaVersion: options.version as OpenAPIVersion,
    outputDir: options.output,
    includeValidation: options.validation,
    includeComponents: options.components,
  });
  
  console.log('✅ Generation complete!');
}

main().catch((error) => {
  console.error('❌ Generation failed:', error);
  process.exit(1);
});
```

### 14.6 Persistence Strategy

#### 14.6.1 Storing Authored Content

```typescript
// packages/persistence/src/openapi-persistence.ts

/**
 * ✅ Good - Persist OpenAPI documents using generated types
 */
export class OpenAPIDocumentStore {
  async save(document: OpenAPIDocument): Promise<void> {
    // Validate using generated Zod schema
    const validationResult = OpenAPIDocumentSchema.safeParse(document);
    
    if (!validationResult.success) {
      throw new ValidationError(validationResult.error);
    }
    
    // Store as JSON
    await fs.writeFile(
      `${this.outputDir}/${document.info.title}.json`,
      JSON.stringify(document, null, 2)
    );
  }
  
  async load(path: string): Promise<OpenAPIDocument> {
    const content = await fs.readFile(path, 'utf-8');
    const document = JSON.parse(content);
    
    // Validate loaded document
    return OpenAPIDocumentSchema.parse(document);
  }
}
```

#### 14.6.2 Version Control Integration

```typescript
// packages/persistence/src/version-control.ts

/**
 * ✅ Good - Track changes to authored documents
 */
export interface DocumentVersion {
  id: string;
  timestamp: Date;
  author: string;
  message: string;
  document: OpenAPIDocument | TaxiDocument;
}

export class VersionControlStore {
  async commit(
    document: OpenAPIDocument | TaxiDocument,
    message: string
  ): Promise<string> {
    const version: DocumentVersion = {
      id: generateVersionId(),
      timestamp: new Date(),
      author: getCurrentUser(),
      message,
      document,
    };
    
    await this.saveVersion(version);
    return version.id;
  }
  
  async diff(versionA: string, versionB: string): Promise<DocumentDiff> {
    const docA = await this.loadVersion(versionA);
    const docB = await this.loadVersion(versionB);
    
    return generateDiff(docA.document, docB.document);
  }
}
```

### 14.7 Testing Generated Code

#### 14.7.1 Schema Validation Tests

```typescript
// packages/generated/openapi/__tests__/schema-validation.test.ts
import { describe, it, expect } from 'vitest';
import { OpenAPIDocumentSchema } from '../types';

describe('OpenAPI Schema Validation', () => {
  it('should validate a minimal OpenAPI document', () => {
    const minimalDoc = {
      openapi: '3.1.0',
      info: {
        title: 'Test API',
        version: '1.0.0',
      },
      paths: {},
    };
    
    expect(() => OpenAPIDocumentSchema.parse(minimalDoc)).not.toThrow();
  });
  
  it('should reject invalid version format', () => {
    const invalidDoc = {
      openapi: '2.0.0', // Invalid for 3.1 schema
      info: {
        title: 'Test API',
        version: '1.0.0',
      },
    };
    
    expect(() => OpenAPIDocumentSchema.parse(invalidDoc)).toThrow();
  });
});
```

#### 14.7.2 Component Generation Tests

```typescript
// packages/generators/__tests__/component-generation.test.ts
import { describe, it, expect } from 'vitest';
import { generateFormComponent } from '../src/openapi/schema-to-components';

describe('Component Generation', () => {
  it('should generate form component with correct props', () => {
    const schema: JSONSchema7 = {
      type: 'object',
      properties: {
        name: { type: 'string' },
        age: { type: 'number' },
      },
      required: ['name'],
    };
    
    const component = generateFormComponent('User', schema);
    
    expect(component).toContain('interface UserFormProps');
    expect(component).toContain('onSubmit: (data: User) => void');
    expect(component).toContain('register');
  });
});
```

### 14.8 Best Practices Summary

#### Schema Management
- ✅ **Version Control Schemas**: Keep schema files in version control
- ✅ **Automated Updates**: Subscribe to schema specification updates
- ✅ **Schema Validation**: Validate schemas before generation
- ✅ **Breaking Change Detection**: Detect schema changes that break existing code

#### Code Generation
- ✅ **Idempotent Generation**: Re-running generation produces same output
- ✅ **Source Maps**: Include references back to schema sources
- ✅ **Custom Templates**: Support customizable generation templates
- ✅ **Incremental Generation**: Only regenerate changed schemas

#### Type Safety
- ✅ **Strict TypeScript**: Use strict mode for generated code
- ✅ **Runtime Validation**: Generate both compile-time and runtime validation
- ✅ **Exhaustive Checks**: Ensure all schema cases are handled
- ✅ **Type Guards**: Generate type guard functions

#### Documentation
- ✅ **JSDoc Comments**: Include schema descriptions in generated code
- ✅ **Examples**: Generate example usage from schema examples
- ✅ **Migration Guides**: Document breaking changes between schema versions
- ✅ **API Documentation**: Auto-generate API docs from schemas

#### Testing
- ✅ **Golden Files**: Test generation output against known-good examples
- ✅ **Schema Coverage**: Ensure all schema patterns are tested
- ✅ **Regression Tests**: Prevent generation bugs
- ✅ **Performance Tests**: Ensure generation completes quickly

---

## Summary: Key Principles

### TypeScript
- ✅ Always use explicit types
- ✅ Prefer `unknown` over `any`
- ✅ Use `readonly` for immutability
- ✅ Leverage const assertions

### React & JSX
- ✅ Functional components with hooks
- ✅ Use `.tsx` extension for React files
- ✅ PascalCase for components, camelCase for instances
- ✅ Double quotes in JSX, single quotes in JavaScript
- ✅ Extract custom hooks for reusable logic
- ✅ Use stable keys in lists (never index)

### JavaScript (Airbnb)
- ✅ Use `const` and `let`, never `var`
- ✅ Prefer template literals over concatenation
- ✅ Use arrow functions for callbacks
- ✅ Destructure objects and arrays
- ✅ Use `===` instead of `==`
- ✅ Always use semicolons

### Architecture
- ✅ Modular monorepo structure
- ✅ Feature-based organization
- ✅ Prevent cross-feature imports
- ✅ Atomic design for UI components
- ✅ Shared packages for reusability

### Schema-Driven Development
- ✅ Use OpenAPI 3.0/3.1 JSON schemas as single source of truth for API editors
- ✅ Use ANTLR grammars for DSL parser and editor generation
- ✅ Forward-engineer TypeScript types from schemas
- ✅ Generate React components from schema definitions
- ✅ Generate Redux slices for state management from schemas
- ✅ Generate runtime validators (Zod) from schemas
- ✅ Automate code generation in build pipeline
- ✅ Version control schemas and track breaking changes
- ✅ Persist authored content as valid schema instances
- ✅ Test generated code against schema specifications

### Code Quality
- ✅ ESLint + Prettier + SonarQube
- ✅ 85%+ test coverage requirement
- ✅ Pre-commit hooks with Husky
- ✅ Conventional commits
- ✅ Quality gates in CI/CD

### Performance
- ✅ Lazy loading with React.lazy
- ✅ Memoization with useMemo/useCallback
- ✅ Code splitting
- ✅ Avoid inline functions in render

By following these best practices and patterns, the DAPA VSCode extension project will be a robust, high-quality, scalable, and open-source ready foundation, aligned to the standards of MAANG engineering teams and the Airbnb style guides. The schema-driven approach ensures consistency between the UI, state management, and persisted artifacts while leveraging official OpenAPI specifications and ANTLR grammars as authoritative sources.

---

## 14. Schema-Driven Forward Engineering Pattern

### 14.1 Overview: One-Time Generation from Official Specifications

DAPA uses **official specifications as the single source of truth** for forward engineering TypeScript artifacts. This is a **one-time generation activity per specification version** that establishes:

- ✅ **Type Safety**: TypeScript interfaces derived directly from official schemas
- ✅ **Consistency**: Same types used for UI components, Redux state, and persistence
- ✅ **Validation**: Schema-based validation using the original JSON Schema
- ✅ **Specification Compliance**: Components match the official spec exactly

**Key Principle**: Forward engineer once per spec version. The pattern is simple, repeatable, and extensible to any specification (OpenAPI, AsyncAPI, GraphQL SDL, etc.).

### 14.2 The Forward Engineering Pattern

This simple pattern applies universally to any specification:

```
┌─────────────────────────────────────────────────────┐
│  STEP 1: Obtain Official Specification             │
│  • OpenAPI JSON Schema (openapi3_1schema.json)     │
│  • AsyncAPI JSON Schema (asyncapi-2.6.0.json)      │
│  • ANTLR Grammar (.g4 file for DSLs)               │
└────────────────┬────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────┐
│  STEP 2: Generate TypeScript Interfaces            │
│  Tool: json-schema-to-typescript or antlr4ts       │
│  Output: src/types/[spec-name].types.ts            │
└────────────────┬────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────┐
│  STEP 3: Create Redux State Structure              │
│  Use generated types for state shape               │
│  Output: src/store/[spec-name].slice.ts            │
└────────────────┬────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────┐
│  STEP 4: Build React Components                    │
│  Components typed with generated interfaces         │
│  Output: src/components/[spec-name]/               │
└────────────────┬────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────┐
│  STEP 5: Implement Validation                      │
│  Use original JSON Schema with AJV                 │
│  Output: src/validation/[spec-name].validator.ts   │
└─────────────────────────────────────────────────────┘

Result: Complete No-Code Authoring capability for the specification
```

### 14.3 Example: OpenAPI 3.1 Forward Engineering

#### Step 1: Obtain the Official OpenAPI 3.1 JSON Schema

```bash
# Download the official schema (one-time activity)
curl https://spec.openapis.org/oas/3.1/schema/2025-09-15 \
  -o src/schemas/openapi3_1schema.json
```

The schema defines the complete OpenAPI document structure:

```json
{
  "$id": "https://spec.openapis.org/oas/3.1/schema/2025-09-15",
  "type": "object",
  "properties": {
    "openapi": { "type": "string", "pattern": "^3\\.1\\.\\d+(-.+)?$" },
    "info": { "$ref": "#/$defs/info" },
    "servers": { "type": "array", "items": { "$ref": "#/$defs/server" } },
    "paths": { "$ref": "#/$defs/paths" },
    "components": { "$ref": "#/$defs/components" }
  },
  "required": ["openapi", "info"]
}
```

#### Step 2: Generate TypeScript Interfaces

```bash
# One-time generation
npm install -D json-schema-to-typescript

npx json-schema-to-typescript \
  src/schemas/openapi3_1schema.json \
  -o src/types/openapi31.types.ts \
  --bannerComment "Generated from OpenAPI 3.1 JSON Schema - DO NOT EDIT"
```

**Generated Output** (`src/types/openapi31.types.ts`):

```typescript
/**
 * Generated from OpenAPI 3.1 JSON Schema - DO NOT EDIT
 */

/**
 * Root OpenAPI Document
 */
export interface OpenAPIDocument {
  openapi: string;
  info: Info;
  jsonSchemaDialect?: string;
  servers?: Server[];
  paths?: Paths;
  webhooks?: Record<string, PathItem>;
  components?: Components;
  security?: SecurityRequirement[];
  tags?: Tag[];
  externalDocs?: ExternalDocumentation;
}

/**
 * API Information
 */
export interface Info {
  title: string;
  summary?: string;
  description?: string;
  termsOfService?: string;
  contact?: Contact;
  license?: License;
  version: string;
}

/**
 * Path Item (API endpoint)
 */
export interface PathItem {
  $ref?: string;
  summary?: string;
  description?: string;
  get?: Operation;
  put?: Operation;
  post?: Operation;
  delete?: Operation;
  options?: Operation;
  head?: Operation;
  patch?: Operation;
  trace?: Operation;
  servers?: Server[];
  parameters?: (Parameter | Reference)[];
}

/**
 * Operation (HTTP method on a path)
 */
export interface Operation {
  tags?: string[];
  summary?: string;
  description?: string;
  externalDocs?: ExternalDocumentation;
  operationId?: string;
  parameters?: (Parameter | Reference)[];
  requestBody?: RequestBody | Reference;
  responses: Responses;
  callbacks?: Record<string, Callback | Reference>;
  deprecated?: boolean;
  security?: SecurityRequirement[];
  servers?: Server[];
}

// ... additional interfaces for Schema, Components, etc.
```

#### Step 3: Create Redux State Structure

Using the generated types, create the Redux slice:

```typescript
// src/store/openapi.slice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import type { OpenAPIDocument, PathItem, Operation, Info } from '../types/openapi31.types';

/**
 * Redux state for OpenAPI document authoring
 * State shape matches the OpenAPI specification exactly
 */
interface OpenAPIState {
  document: OpenAPIDocument | null;
  selectedPath: string | null;
  selectedOperation: { path: string; method: string } | null;
}

const initialState: OpenAPIState = {
  document: null,
  selectedPath: null,
  selectedOperation: null,
};

export const openAPISlice = createSlice({
  name: 'openapi',
  initialState,
  reducers: {
    // Load/Save document
    setDocument(state, action: PayloadAction<OpenAPIDocument>) {
      state.document = action.payload;
    },
    
    // Update Info section
    updateInfo(state, action: PayloadAction<Partial<Info>>) {
      if (state.document) {
        state.document.info = { ...state.document.info, ...action.payload };
      }
    },
    
    // Add/Update/Delete paths
    addPath(state, action: PayloadAction<{ path: string; item: PathItem }>) {
      if (state.document) {
        state.document.paths = state.document.paths || {};
        state.document.paths[action.payload.path] = action.payload.item;
      }
    },
    
    updatePathItem(state, action: PayloadAction<{ path: string; item: Partial<PathItem> }>) {
      if (state.document?.paths?.[action.payload.path]) {
        state.document.paths[action.payload.path] = {
          ...state.document.paths[action.payload.path],
          ...action.payload.item,
        };
      }
    },
    
    deletePath(state, action: PayloadAction<string>) {
      if (state.document?.paths) {
        delete state.document.paths[action.payload];
      }
    },
    
    // Add/Update operations
    setOperation(
      state,
      action: PayloadAction<{ path: string; method: string; operation: Operation }>
    ) {
      const { path, method, operation } = action.payload;
      if (state.document?.paths?.[path]) {
        state.document.paths[path][method as keyof PathItem] = operation;
      }
    },
    
    // Selection
    selectPath(state, action: PayloadAction<string | null>) {
      state.selectedPath = action.payload;
    },
  },
});

export const {
  setDocument,
  updateInfo,
  addPath,
  updatePathItem,
  deletePath,
  setOperation,
  selectPath,
} = openAPISlice.actions;

export default openAPISlice.reducer;
```

#### Step 4: Build React Components

Create components using the generated types:

```typescript
// src/components/openapi/info-editor.tsx
import React from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { updateInfo } from '../../store/openapi.slice';
import type { Info } from '../../types/openapi31.types';

/**
 * Editor for OpenAPI Info section
 * Uses generated Info type for full type safety
 */
export function InfoEditor() {
  const dispatch = useDispatch();
  const info = useSelector((state: RootState) => state.openapi.document?.info);

  const handleChange = (field: keyof Info, value: string) => {
    dispatch(updateInfo({ [field]: value }));
  };

  return (
    <div className="info-editor">
      <h2>API Information</h2>
      
      <div className="form-group">
        <label htmlFor="title">Title *</label>
        <input
          id="title"
          type="text"
          value={info?.title || ''}
          onChange={(e) => handleChange('title', e.target.value)}
          required
        />
      </div>

      <div className="form-group">
        <label htmlFor="version">Version *</label>
        <input
          id="version"
          type="text"
          value={info?.version || ''}
          onChange={(e) => handleChange('version', e.target.value)}
          required
        />
      </div>

      <div className="form-group">
        <label htmlFor="description">Description</label>
        <textarea
          id="description"
          value={info?.description || ''}
          onChange={(e) => handleChange('description', e.target.value)}
          rows={4}
        />
      </div>

      <div className="form-group">
        <label htmlFor="termsOfService">Terms of Service URL</label>
        <input
          id="termsOfService"
          type="url"
          value={info?.termsOfService || ''}
          onChange={(e) => handleChange('termsOfService', e.target.value)}
        />
      </div>
    </div>
  );
}
```

```typescript
// src/components/openapi/path-item-editor.tsx
import React from 'react';
import { useDispatch } from 'react-redux';
import { updatePathItem } from '../../store/openapi.slice';
import type { PathItem, Operation } from '../../types/openapi31.types';

interface PathItemEditorProps {
  path: string;
  pathItem: PathItem;
}

/**
 * Editor for a single path item
 * Handles all HTTP methods defined in OpenAPI spec
 */
export function PathItemEditor({ path, pathItem }: PathItemEditorProps) {
  const dispatch = useDispatch();
  const [selectedMethod, setSelectedMethod] = React.useState<string | null>(null);

  const httpMethods: Array<keyof PathItem> = [
    'get', 'post', 'put', 'delete', 'patch', 'options', 'head', 'trace'
  ];

  const handleAddOperation = (method: string) => {
    const newOperation: Operation = {
      responses: {},
      summary: '',
      description: '',
    };
    
    dispatch(setOperation({ path, method, operation: newOperation }));
    setSelectedMethod(method);
  };

  return (
    <div className="path-item-editor">
      <h3>{path}</h3>
      
      {pathItem.summary && <p className="path-summary">{pathItem.summary}</p>}
      
      <div className="operations">
        {httpMethods.map((method) => {
          const operation = pathItem[method] as Operation | undefined;
          
          return (
            <div key={method} className="operation">
              <button
                className={`method-badge ${method}`}
                onClick={() => operation ? setSelectedMethod(method) : handleAddOperation(method)}
              >
                {method.toUpperCase()}
              </button>
              
              {operation && (
                <span className="operation-summary">
                  {operation.summary || 'No summary'}
                </span>
              )}
            </div>
          );
        })}
      </div>
      
      {selectedMethod && pathItem[selectedMethod] && (
        <OperationEditor
          path={path}
          method={selectedMethod}
          operation={pathItem[selectedMethod] as Operation}
        />
      )}
    </div>
  );
}
```

#### Step 5: Implement Validation

Use the original JSON Schema for validation:

```typescript
// src/validation/openapi.validator.ts
import Ajv, { ValidateFunction } from 'ajv';
import addFormats from 'ajv-formats';
import openapiSchema from '../schemas/openapi3_1schema.json';
import type { OpenAPIDocument } from '../types/openapi31.types';

/**
 * AJV validator configured for OpenAPI 3.1
 */
const ajv = new Ajv({
  allErrors: true,
  strict: false,
  validateFormats: true,
});

addFormats(ajv);

/**
 * Compiled validator for OpenAPI documents
 */
export const validateOpenAPIDocument: ValidateFunction<OpenAPIDocument> = 
  ajv.compile(openapiSchema);

/**
 * Validate an OpenAPI document
 */
export function validate(document: unknown): ValidationResult {
  const valid = validateOpenAPIDocument(document);
  
  if (valid) {
    return { valid: true, errors: [] };
  }
  
  const errors = (validateOpenAPIDocument.errors || []).map((error) => ({
    path: error.instancePath || '/',
    message: error.message || 'Validation error',
    keyword: error.keyword,
    params: error.params,
  }));
  
  return { valid: false, errors };
}

interface ValidationResult {
  valid: boolean;
  errors: Array<{
    path: string;
    message: string;
    keyword: string;
    params: unknown;
  }>;
}
```

**Usage in Editor**:

```typescript
// src/components/openapi/openapi-editor.tsx
import React from 'react';
import { useSelector } from 'react-redux';
import { validate } from '../../validation/openapi.validator';
import { InfoEditor } from './info-editor';
import { PathItemEditor } from './path-item-editor';

export function OpenAPIEditor() {
  const document = useSelector((state: RootState) => state.openapi.document);
  const [validationErrors, setValidationErrors] = React.useState<ValidationError[]>([]);

  // Validate on document change
  React.useEffect(() => {
    if (document) {
      const result = validate(document);
      if (!result.valid) {
        setValidationErrors(result.errors);
      } else {
        setValidationErrors([]);
      }
    }
  }, [document]);

  return (
    <div className="openapi-editor">
      <InfoEditor />
      
      <div className="paths-section">
        <h2>Paths</h2>
        {Object.entries(document?.paths || {}).map(([path, pathItem]) => (
          <PathItemEditor key={path} path={path} pathItem={pathItem} />
        ))}
      </div>
      
      {validationErrors.length > 0 && (
        <div className="validation-errors">
          <h3>Validation Errors</h3>
          <ul>
            {validationErrors.map((error, i) => (
              <li key={i}>
                <strong>{error.path}:</strong> {error.message}
              </li>
            ))}
          </ul>
        </div>
      )}
    </div>
  );
}
```

### 14.4 Example: ANTLR Grammar Forward Engineering (TaxiLang DSL)

The same pattern applies to ANTLR grammars for DSL authoring.

#### Step 1: Obtain the ANTLR Grammar

```bash
# Download the TaxiLang grammar (one-time)
curl https://raw.githubusercontent.com/taxilang/taxilang/refs/heads/develop/compiler/src/main/antlr4/lang/taxi/Taxi.g4 \
  -o src/grammars/Taxi.g4
```

The grammar defines the DSL syntax:

```antlr
// Taxi.g4 (simplified excerpt)
grammar Taxi;

document
    : (singleNamespaceDocument | multiNamespaceDocument)
    ;

typeDeclaration
    : typeDoc? annotation* typeModifier* typeKind identifier
      typeArguments?
      (K_Inherits listOfInheritedTypes)?
      (typeBody | expressionTypeDeclaration)?
    ;

fieldDeclaration
    : fieldModifier? identifier (':' (anonymousTypeDefinition | fieldTypeDeclaration | expressionGroup))?
    ;

// ... more rules
```

#### Step 2: Generate TypeScript Parser

```bash
# One-time generation
npm install -D antlr4ts-cli antlr4ts

npx antlr4ts -visitor -Dlanguage=TypeScript \
  src/grammars/Taxi.g4 \
  -o src/parsers/taxilang
```

**Generated Output**:
- `TaxiLexer.ts` - Tokenizer
- `TaxiParser.ts` - Parser with AST context classes
- `TaxiVisitor.ts` - Visitor interface for AST traversal

#### Step 3: Create Parser Wrapper with Type Extraction

```typescript
// src/parsers/taxilang-wrapper.ts
import { CharStreams, CommonTokenStream } from 'antlr4ts';
import { TaxiLexer } from './taxilang/TaxiLexer';
import { TaxiParser, TypeDeclarationContext } from './taxilang/TaxiParser';
import { TaxiVisitor } from './taxilang/TaxiVisitor';
import { AbstractParseTreeVisitor } from 'antlr4ts/tree/AbstractParseTreeVisitor';

/**
 * Type extracted from AST
 */
export interface TaxiType {
  name: string;
  kind: 'type' | 'model';
  fields: TaxiField[];
  annotations: string[];
  inherits?: string[];
}

export interface TaxiField {
  name: string;
  type: string;
  optional: boolean;
  annotations: string[];
}

/**
 * Parse TaxiLang source code
 */
export function parseTaxiLang(source: string): TaxiType[] {
  const inputStream = CharStreams.fromString(source);
  const lexer = new TaxiLexer(inputStream);
  const tokenStream = new CommonTokenStream(lexer);
  const parser = new TaxiParser(tokenStream);
  
  // Parse document
  const tree = parser.document();
  
  // Extract types using visitor
  const visitor = new TaxiTypeExtractor();
  visitor.visit(tree);
  
  return visitor.types;
}

/**
 * Visitor to extract type declarations from AST
 */
class TaxiTypeExtractor extends AbstractParseTreeVisitor<void> implements TaxiVisitor<void> {
  types: TaxiType[] = [];
  
  defaultResult(): void {
    return undefined;
  }
  
  visitTypeDeclaration(ctx: TypeDeclarationContext): void {
    const identifier = ctx.identifier();
    const typeKind = ctx.typeKind();
    
    const type: TaxiType = {
      name: identifier.text,
      kind: typeKind.K_Model() ? 'model' : 'type',
      fields: this.extractFields(ctx.typeBody()),
      annotations: this.extractAnnotations(ctx.annotation()),
    };
    
    if (ctx.K_Inherits()) {
      type.inherits = this.extractInheritedTypes(ctx.listOfInheritedTypes());
    }
    
    this.types.push(type);
    this.visitChildren(ctx);
  }
  
  private extractFields(typeBody: TypeBodyContext | undefined): TaxiField[] {
    if (!typeBody) return [];
    
    const fields: TaxiField[] = [];
    const members = typeBody.typeMemberDeclaration();
    
    for (const member of members) {
      const fieldDecl = member.fieldDeclaration();
      if (fieldDecl) {
        fields.push({
          name: fieldDecl.identifier().text,
          type: this.extractFieldType(fieldDecl),
          optional: fieldDecl.text.includes('?'),
          annotations: this.extractAnnotations(member.annotation()),
        });
      }
    }
    
    return fields;
  }
  
  private extractAnnotations(annotations: AnnotationContext[]): string[] {
    return annotations.map((ann) => ann.qualifiedName().text);
  }
  
  private extractFieldType(fieldDecl: FieldDeclarationContext): string {
    const typeDecl = fieldDecl.fieldTypeDeclaration();
    if (typeDecl?.typeExpression()) {
      return typeDecl.typeExpression().nullableTypeReference().text;
    }
    return 'unknown';
  }
  
  private extractInheritedTypes(inheritedTypes: ListOfInheritedTypesContext | undefined): string[] {
    if (!inheritedTypes) return [];
    return inheritedTypes.typeReference().map((ref) => ref.text);
  }
}
```

#### Step 4: Build Redux State for DSL Models

```typescript
// src/store/taxilang.slice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import type { TaxiType, TaxiField } from '../parsers/taxilang-wrapper';

/**
 * State for TaxiLang model authoring
 */
interface TaxiLangState {
  types: TaxiType[];
  selectedType: string | null;
  sourceCode: string;
}

const initialState: TaxiLangState = {
  types: [],
  selectedType: null,
  sourceCode: '',
};

export const taxiLangSlice = createSlice({
  name: 'taxilang',
  initialState,
  reducers: {
    // Load types from source
    setTypes(state, action: PayloadAction<TaxiType[]>) {
      state.types = action.payload;
    },
    
    // Add new type
    addType(state, action: PayloadAction<TaxiType>) {
      state.types.push(action.payload);
    },
    
    // Update existing type
    updateType(state, action: PayloadAction<{ name: string; type: Partial<TaxiType> }>) {
      const index = state.types.findIndex((t) => t.name === action.payload.name);
      if (index !== -1) {
        state.types[index] = { ...state.types[index], ...action.payload.type };
      }
    },
    
    // Add field to type
    addField(state, action: PayloadAction<{ typeName: string; field: TaxiField }>) {
      const type = state.types.find((t) => t.name === action.payload.typeName);
      if (type) {
        type.fields.push(action.payload.field);
      }
    },
    
    // Update source code
    setSourceCode(state, action: PayloadAction<string>) {
      state.sourceCode = action.payload;
    },
    
    // Selection
    selectType(state, action: PayloadAction<string | null>) {
      state.selectedType = action.payload;
    },
  },
});

export const { setTypes, addType, updateType, addField, setSourceCode, selectType } = 
  taxiLangSlice.actions;

export default taxiLangSlice.reducer;
```

#### Step 5: Build React Components for DSL Authoring

```typescript
// src/components/taxilang/type-editor.tsx
import React from 'react';
import { useDispatch } from 'react-redux';
import { updateType, addField } from '../../store/taxilang.slice';
import type { TaxiType, TaxiField } from '../../parsers/taxilang-wrapper';

interface TypeEditorProps {
  type: TaxiType;
}

/**
 * Editor for TaxiLang type declarations
 */
export function TypeEditor({ type }: TypeEditorProps) {
  const dispatch = useDispatch();
  const [newFieldName, setNewFieldName] = React.useState('');

  const handleAddField = () => {
    if (!newFieldName) return;
    
    const newField: TaxiField = {
      name: newFieldName,
      type: 'String',
      optional: false,
      annotations: [],
    };
    
    dispatch(addField({ typeName: type.name, field: newField }));
    setNewFieldName('');
  };

  const handleUpdateName = (newName: string) => {
    dispatch(updateType({ name: type.name, type: { name: newName } }));
  };

  return (
    <div className="type-editor">
      <div className="type-header">
        <span className="type-kind">{type.kind}</span>
        <input
          type="text"
          value={type.name}
          onChange={(e) => handleUpdateName(e.target.value)}
          className="type-name-input"
        />
      </div>

      {type.inherits && type.inherits.length > 0 && (
        <div className="inherits-section">
          <span>inherits</span>
          <ul>
            {type.inherits.map((inherited) => (
              <li key={inherited}>{inherited}</li>
            ))}
          </ul>
        </div>
      )}

      <div className="fields-section">
        <h4>Fields</h4>
        {type.fields.map((field) => (
          <FieldEditor key={field.name} field={field} typeName={type.name} />
        ))}
        
        <div className="add-field">
          <input
            type="text"
            placeholder="New field name"
            value={newFieldName}
            onChange={(e) => setNewFieldName(e.target.value)}
            onKeyPress={(e) => e.key === 'Enter' && handleAddField()}
          />
          <button onClick={handleAddField}>Add Field</button>
        </div>
      </div>
    </div>
  );
}

/**
 * Editor for individual fields
 */
function FieldEditor({ field, typeName }: { field: TaxiField; typeName: string }) {
  const dispatch = useDispatch();

  return (
    <div className="field-editor">
      <input
        type="text"
        value={field.name}
        className="field-name"
        readOnly
      />
      <span>:</span>
      <input
        type="text"
        value={field.type}
        className="field-type"
        onChange={(e) => {
          // Dispatch update field action
        }}
      />
      {field.optional && <span className="optional">?</span>}
    </div>
  );
}
```

#### Step 6: Serialize Back to DSL Source

```typescript
// src/serializers/taxilang.serializer.ts
import type { TaxiType, TaxiField } from '../parsers/taxilang-wrapper';

/**
 * Serialize TaxiType back to TaxiLang source code
 */
export function serializeTaxiType(type: TaxiType): string {
  const lines: string[] = [];
  
  // Annotations
  if (type.annotations.length > 0) {
    type.annotations.forEach((ann) => {
      lines.push(`@${ann}`);
    });
  }
  
  // Type declaration
  let declaration = `${type.kind} ${type.name}`;
  
  // Inheritance
  if (type.inherits && type.inherits.length > 0) {
    declaration += ` inherits ${type.inherits.join(', ')}`;
  }
  
  lines.push(declaration + ' {');
  
  // Fields
  type.fields.forEach((field) => {
    const fieldLine = serializeField(field);
    lines.push(`  ${fieldLine}`);
  });
  
  lines.push('}');
  
  return lines.join('\n');
}

function serializeField(field: TaxiField): string {
  let line = field.name;
  
  if (field.type) {
    line += ` : ${field.type}`;
    if (field.optional) {
      line += '?';
    }
  }
  
  return line;
}

/**
 * Serialize all types to complete TaxiLang document
 */
export function serializeTaxiDocument(types: TaxiType[]): string {
  return types.map((type) => serializeTaxiType(type)).join('\n\n');
}
```

### 14.5 Extending the Pattern to Other Specifications

The same 5-step pattern works for any specification:

#### AsyncAPI 2.6

```bash
# Step 1: Get schema
curl https://www.asyncapi.com/definitions/2.6.0/asyncapi.json \
  -o src/schemas/asyncapi26.json

# Step 2: Generate types
npx json-schema-to-typescript src/schemas/asyncapi26.json \
  -o src/types/asyncapi26.types.ts

# Step 3-5: Follow same pattern as OpenAPI
```

**Key differences**: AsyncAPI focuses on event-driven architectures, so components emphasize channels, messages, and subscribe/publish operations instead of paths and HTTP methods.

#### GraphQL SDL (ANTLR Grammar)

```bash
# Step 1: Get GraphQL grammar
curl https://raw.githubusercontent.com/antlr/grammars-v4/master/graphql/GraphQL.g4 \
  -o src/grammars/GraphQL.g4

# Step 2: Generate parser
npx antlr4ts -visitor -Dlanguage=TypeScript \
  src/grammars/GraphQL.g4 \
  -o src/parsers/graphql

# Step 3-5: Follow same pattern as TaxiLang
```

**Key differences**: GraphQL focuses on type definitions, queries, mutations, and subscriptions. Components would handle `type`, `interface`, `enum`, `input`, and operation definitions.

### 14.6 Pattern Summary & Best Practices

#### The Universal 5-Step Pattern

1. **Obtain Specification** → Download official JSON Schema or ANTLR grammar
2. **Generate Types** → Use `json-schema-to-typescript` or `antlr4ts`
3. **Create State** → Redux slice matching the specification structure
4. **Build Components** → React editors using generated types
5. **Add Validation** → AJV for JSON Schema, or custom for ANTLR

#### ✅ DO

- **Keep it simple**: One-time generation per spec version
- **Use official sources**: Always use canonical schemas/grammars
- **Version independently**: Support multiple spec versions side-by-side
- **Type everything**: Full TypeScript coverage from spec to UI
- **Store as spec**: Redux state matches the specification structure exactly
- **Serialize back**: Components can save back to original format

#### ❌ DON'T

- **Don't build pipelines**: This is not a continuous generation process
- **Don't create abstractions**: State structure mirrors the spec, not an abstraction
- **Don't mix versions**: Keep OpenAPI 3.0 and 3.1 separate
- **Don't skip validation**: Always validate against the original schema

#### 🎯 Key Benefits

1. **Specification Fidelity**: Perfect alignment with official specs
2. **Type Safety**: Compiler catches mismatches
3. **Easy Extension**: Add AsyncAPI, GraphQL, etc. using the same pattern
4. **No Abstraction Layer**: Direct mapping from spec to state to UI
5. **Standard Persistence**: Save/load in the original specification format

### 14.7 Implementation Checklist

For each new specification (OpenAPI, AsyncAPI, GraphQL, etc.):

- [ ] Download official schema/grammar to `src/schemas/` or `src/grammars/`
- [ ] Generate TypeScript types to `src/types/[spec].types.ts`
- [ ] Create Redux slice in `src/store/[spec].slice.ts`
- [ ] Build React components in `src/components/[spec]/`
- [ ] Add validation in `src/validation/[spec].validator.ts`
- [ ] Test round-trip: Author → Save → Load → Validate
- [ ] Document differences from other specs (if any)

---

**Document Version:** 2025.1
**Last Updated:** 2025
**Maintained by:** DAPA Engineering Team
