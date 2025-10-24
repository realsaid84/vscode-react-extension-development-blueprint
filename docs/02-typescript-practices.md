# TypeScript Best Practices

> **Type safety patterns for production-ready VSCode extensions**

## Table of Contents
- [Type Safety Patterns](#type-safety-patterns)
- [Interface Design](#interface-design)
- [Generic Usage](#generic-usage)
- [Utility Types](#utility-types)
- [Advanced Patterns](#advanced-patterns)

---

## Type Safety Patterns

### Always Explicit Types

```typescript
// ✅ Good: Explicit return type
function getUser(id: string): Promise<User> {
  return apiClient.get<User>(`/users/${id}`);
}

// ❌ Bad: Inferred type (less safe for APIs)
function getUser(id: string) {
  return apiClient.get(`/users/${id}`);
}
```

### Prefer `unknown` Over `any`

```typescript
// ✅ Good: Force type checking
function processInput(input: unknown): string {
  if (typeof input === 'string') {
    return input.toUpperCase();
  }
  if (typeof input === 'number') {
    return input.toString();
  }
  throw new Error('Invalid input type');
}

// ❌ Bad: No type safety
function processInput(input: any): string {
  return input.toUpperCase(); // Runtime error if input is not a string
}
```

### Strict Null Checks

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "strictNullChecks": true
  }
}

// ✅ Good: Handle null explicitly
function getUserName(user: User | null): string {
  if (user === null) {
    return 'Anonymous';
  }
  return user.name;
}

// Or use optional chaining
function getUserName(user: User | null): string {
  return user?.name ?? 'Anonymous';
}

// ❌ Bad: Assumes user is not null
function getUserName(user: User): string {
  return user.name; // Crashes if user is null
}
```

### Type Guards

```typescript
// ✅ Good: Type guard functions
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value
  );
}

function processValue(value: unknown) {
  if (isUser(value)) {
    // TypeScript knows value is User here
    console.log(value.name);
  }
}

// ✅ Good: Discriminated unions
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'rectangle'; width: number; height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'rectangle':
      return shape.width * shape.height;
  }
}
```

---

## Interface Design

### Interfaces vs Types

```typescript
// ✅ Good: Use interfaces for object shapes
interface User {
  id: string;
  name: string;
  email: string;
}

interface Admin extends User {
  permissions: string[];
}

// ✅ Good: Use types for unions, intersections, primitives
type Status = 'active' | 'inactive' | 'pending';
type ID = string | number;
type UserOrAdmin = User | Admin;
type UserWithTimestamp = User & { createdAt: Date };
```

### Interface Segregation

```typescript
// ✅ Good: Small, focused interfaces
interface Identifiable {
  id: string;
}

interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}

interface Auditable {
  createdBy: string;
  updatedBy: string;
}

interface User extends Identifiable, Timestamped, Auditable {
  name: string;
  email: string;
}

// ❌ Bad: Kitchen sink interface
interface User {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
  updatedBy: string;
  // ... 20 more properties
}
```

### Readonly Properties

```typescript
// ✅ Good: Immutable by default
interface Config {
  readonly apiUrl: string;
  readonly timeout: number;
}

const config: Config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
};

// config.apiUrl = 'new url'; // Error: Cannot assign to 'apiUrl'

// ✅ Good: Deep readonly
interface State {
  readonly user: {
    readonly id: string;
    readonly name: string;
  };
}

// Better: Use Readonly utility
interface State {
  user: Readonly<{
    id: string;
    name: string;
  }>;
}
```

### Optional vs Undefined

```typescript
// ✅ Good: Use optional for properties that may not exist
interface User {
  id: string;
  name: string;
  email?: string;        // May not be provided
  phone?: string;        // May not be provided
}

// ✅ Good: Use undefined for properties that can be explicitly undefined
interface Settings {
  theme: 'light' | 'dark' | undefined;  // Explicitly set to undefined
  language: string | undefined;
}

// ❌ Bad: Mixing optional and undefined unnecessarily
interface BadUser {
  id?: string | undefined;  // Redundant
  name?: string | undefined;
}
```

### Index Signatures

```typescript
// ✅ Good: Type-safe index signature
interface StringMap {
  [key: string]: string;
}

interface NumberMap {
  [key: string]: number;
}

// ✅ Good: Mixed known and unknown properties
interface OpenAPIDocument {
  openapi: string;
  info: Info;
  [key: string]: unknown;  // Allow extension properties
}

// ✅ Better: Use Record type
type StringMap = Record<string, string>;
type NumberMap = Record<string, number>;
```

---

## Generic Usage

### Generic Functions

```typescript
// ✅ Good: Generic API client
async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);
  return response.json() as T;
}

// Usage with type inference
const user = await fetchData<User>('/api/users/1');
const users = await fetchData<User[]>('/api/users');

// ✅ Good: Generic with constraints
interface Identifiable {
  id: string;
}

function findById<T extends Identifiable>(
  items: T[],
  id: string
): T | undefined {
  return items.find((item) => item.id === id);
}
```

### Generic Components

```typescript
// ✅ Good: Generic React component
interface DataListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
}

function DataList<T>({ items, renderItem, keyExtractor }: DataListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// Usage
<DataList
  items={users}
  renderItem={(user) => <span>{user.name}</span>}
  keyExtractor={(user) => user.id}
/>
```

### Generic Constraints

```typescript
// ✅ Good: Constrain generic to specific types
function logLength<T extends { length: number }>(item: T): void {
  console.log(item.length);
}

logLength('hello');        // OK
logLength([1, 2, 3]);      // OK
logLength({ length: 5 });  // OK
// logLength(123);         // Error: number doesn't have length

// ✅ Good: Multiple constraints
interface HasId {
  id: string;
}

interface HasName {
  name: string;
}

function displayItem<T extends HasId & HasName>(item: T): string {
  return `${item.id}: ${item.name}`;
}
```

### Generic Defaults

```typescript
// ✅ Good: Default generic type
interface ApiResponse<T = unknown> {
  data: T;
  status: number;
  message: string;
}

// Can use without specifying type
const response: ApiResponse = await fetch('/api/data');

// Or specify type
const userResponse: ApiResponse<User> = await fetch('/api/user');
```

---

## Utility Types

### Built-in Utility Types

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

// Partial<T> - All properties optional
type PartialUser = Partial<User>;
// { id?: string; name?: string; email?: string; age?: number; }

// Required<T> - All properties required
type RequiredUser = Required<Partial<User>>;

// Readonly<T> - All properties readonly
type ReadonlyUser = Readonly<User>;

// Pick<T, K> - Select specific properties
type UserBasic = Pick<User, 'id' | 'name'>;
// { id: string; name: string; }

// Omit<T, K> - Exclude specific properties
type UserWithoutAge = Omit<User, 'age'>;
// { id: string; name: string; email: string; }

// Record<K, T> - Object type with specific keys
type UserMap = Record<string, User>;
// { [key: string]: User }

// Extract<T, U> - Extract types from union
type Status = 'active' | 'inactive' | 'pending';
type ActiveStatus = Extract<Status, 'active' | 'inactive'>;
// 'active' | 'inactive'

// Exclude<T, U> - Exclude types from union
type NonPendingStatus = Exclude<Status, 'pending'>;
// 'active' | 'inactive'

// NonNullable<T> - Exclude null and undefined
type NonNullableString = NonNullable<string | null | undefined>;
// string

// ReturnType<T> - Extract function return type
function getUser(): User { /* ... */ }
type UserReturnType = ReturnType<typeof getUser>;
// User

// Parameters<T> - Extract function parameters
function createUser(name: string, email: string): User { /* ... */ }
type CreateUserParams = Parameters<typeof createUser>;
// [name: string, email: string]
```

### Custom Utility Types

```typescript
// ✅ Good: Make specific properties optional
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

interface User {
  id: string;
  name: string;
  email: string;
}

type UserWithOptionalEmail = PartialBy<User, 'email'>;
// { id: string; name: string; email?: string }

// ✅ Good: Make specific properties required
type RequiredBy<T, K extends keyof T> = T & Required<Pick<T, K>>;

interface PartialUser {
  id?: string;
  name?: string;
  email?: string;
}

type UserWithRequiredId = RequiredBy<PartialUser, 'id'>;
// { id: string; name?: string; email?: string }

// ✅ Good: Deep partial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

interface Config {
  api: {
    url: string;
    timeout: number;
  };
  theme: string;
}

type PartialConfig = DeepPartial<Config>;
// { api?: { url?: string; timeout?: number; }; theme?: string; }

// ✅ Good: Deep readonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};
```

---

## Advanced Patterns

### Template Literal Types

```typescript
// ✅ Good: Type-safe string patterns
type HTTPMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
type Route = '/users' | '/posts' | '/comments';
type Endpoint = `${HTTPMethod} ${Route}`;
// 'GET /users' | 'GET /posts' | ... | 'DELETE /comments'

// ✅ Good: CSS property types
type CSSProperty = 'color' | 'background' | 'border';
type CSSValue = `${CSSProperty}: ${string}`;
// 'color: red' | 'background: blue' | etc.
```

### Conditional Types

```typescript
// ✅ Good: Conditional type based on another type
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// ✅ Good: Extract promise type
type Awaited<T> = T extends Promise<infer U> ? U : T;

type UserPromise = Promise<User>;
type UserType = Awaited<UserPromise>;  // User

// ✅ Good: Function argument types
type ArgumentsType<T> = T extends (...args: infer A) => any ? A : never;

function example(a: string, b: number): void {}
type ExampleArgs = ArgumentsType<typeof example>;  // [string, number]
```

### Mapped Types

```typescript
// ✅ Good: Make all properties optional and nullable
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

type NullableUser = Nullable<User>;
// { id: string | null; name: string | null; ... }

// ✅ Good: Create getters for all properties
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

interface State {
  name: string;
  age: number;
}

type StateGetters = Getters<State>;
// { getName: () => string; getAge: () => number; }
```

### Branded Types

```typescript
// ✅ Good: Prevent mixing similar types
type UserId = string & { readonly __brand: 'UserId' };
type PostId = string & { readonly __brand: 'PostId' };

function createUserId(id: string): UserId {
  return id as UserId;
}

function createPostId(id: string): PostId {
  return id as PostId;
}

function getUser(userId: UserId): User { /* ... */ }

const userId = createUserId('123');
const postId = createPostId('456');

getUser(userId);  // OK
// getUser(postId);  // Error: PostId is not assignable to UserId
```

### Const Assertions

```typescript
// ✅ Good: Preserve literal types
const routes = {
  home: '/',
  about: '/about',
  contact: '/contact',
} as const;

type Route = typeof routes[keyof typeof routes];
// '/' | '/about' | '/contact'

// ✅ Good: Readonly arrays
const colors = ['red', 'green', 'blue'] as const;
type Color = typeof colors[number];
// 'red' | 'green' | 'blue'

// Without as const:
const badColors = ['red', 'green', 'blue'];
type BadColor = typeof badColors[number];
// string (not specific literals)
```

---

## Best Practices Summary

### ✅ DO

- **Use strict mode** in tsconfig.json
- **Prefer interfaces** for object shapes
- **Use types** for unions and primitives
- **Make properties readonly** by default
- **Use unknown** instead of any
- **Add type guards** for runtime checks
- **Leverage utility types** for transformations
- **Use generics** for reusable code
- **Create branded types** for domain concepts

### ❌ DON'T

- **Don't use `any`** - use `unknown` instead
- **Don't skip return types** on functions
- **Don't use `as` casting** without validation
- **Don't create overly complex** generic types
- **Don't ignore strictNullChecks**
- **Don't use `@ts-ignore`** - fix the issue
- **Don't create giant interfaces** - segregate them

### 🎯 Quick Checklist

- [ ] Strict mode enabled
- [ ] No `any` types
- [ ] Explicit function return types
- [ ] Readonly where appropriate
- [ ] Type guards for runtime checks
- [ ] Utility types for transformations
- [ ] Generics with constraints
- [ ] Discriminated unions for variants

---

This TypeScript approach ensures type safety throughout the DAPA extension, catching errors at compile time rather than runtime.
