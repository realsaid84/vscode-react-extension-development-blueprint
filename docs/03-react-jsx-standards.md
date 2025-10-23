# React & JSX Standards

> **Component architecture and JSX patterns aligned with Airbnb style guide**

## Table of Contents
- [Component Architecture](#component-architecture)
- [Hooks Patterns](#hooks-patterns)
- [Performance Optimization](#performance-optimization)
- [Atomic Design System](#atomic-design-system)
- [JSX Formatting](#jsx-formatting)

---

## Component Architecture

### Component Declaration (Airbnb Standard)

```tsx
// ✅ Good: Functional component with explicit typing
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

export { Button };

// ❌ Bad: Class component for simple UI
class BadButton extends React.Component<ButtonProps> {
  render() {
    return <button onClick={this.props.onClick}>{this.props.children}</button>;
  }
}
```

### File Extensions (Airbnb Standard)

```tsx
// ✅ Good: Use .tsx for React components
// File: ReservationCard.tsx
interface ReservationCardProps {
  title: string;
}

export function ReservationCard({ title }: ReservationCardProps) {
  return <div>{title}</div>;
}

// ✅ Good: Use .ts for non-React code
// File: validation.ts
export function validateEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

### Component Naming (Airbnb Standard)

```tsx
// ✅ Good: PascalCase for components, camelCase for instances
import ReservationCard from './ReservationCard';

const reservationItem = <ReservationCard title="Room 101" />;

// ❌ Bad: Incorrect casing
import reservationCard from './ReservationCard';
const ReservationItem = <ReservationCard title="Room 101" />;
```

### Props Naming (Airbnb Standard)

```tsx
// ✅ Good: camelCase for props, omit boolean true
<Foo
  userName="hello"
  phoneNumber={12345678}
  Component={SomeComponent}
  hidden
/>

// ❌ Bad: Incorrect naming and unnecessary explicit true
<Foo
  UserName="hello"
  phone_number={12345678}
  hidden={true}
/>
```

### Default Props

```tsx
// ✅ Good: Use default parameters
interface ButtonProps {
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  children: React.ReactNode;
}

function Button({
  variant = 'primary',
  disabled = false,
  children,
}: ButtonProps) {
  return (
    <button className={`btn btn-${variant}`} disabled={disabled}>
      {children}
    </button>
  );
}

// ❌ Bad: Old defaultProps pattern (deprecated)
Button.defaultProps = {
  variant: 'primary',
  disabled: false,
};
```

---

## Hooks Patterns

### useState

```tsx
// ✅ Good: Descriptive state names
function UserForm() {
  const [email, setEmail] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  return (/* ... */);
}

// ❌ Bad: Generic state names
function UserForm() {
  const [value, setValue] = useState('');
  const [flag, setFlag] = useState(false);
  const [data, setData] = useState(null);
}
```

### useEffect

```tsx
// ✅ Good: Single responsibility, cleanup
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      const data = await apiClient.get<User>(`/users/${userId}`);
      if (!cancelled) {
        setUser(data);
      }
    }

    fetchUser();

    return () => {
      cancelled = true;
    };
  }, [userId]);

  return <div>{user?.name}</div>;
}

// ❌ Bad: No cleanup, missing dependencies
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/users/${userId}`)
      .then((res) => res.json())
      .then(setUser);
  }); // Missing dependency array!
}
```

### Custom Hooks

```tsx
// ✅ Good: Reusable logic in custom hooks
function useApi<T>(endpoint: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchData() {
      try {
        setLoading(true);
        const result = await apiClient.get<T>(endpoint);
        if (!cancelled) {
          setData(result);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err as Error);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }

    fetchData();

    return () => {
      cancelled = true;
    };
  }, [endpoint]);

  return { data, loading, error };
}

// Usage
function UserList() {
  const { data: users, loading, error } = useApi<User[]>('/users');

  if (loading) return <Spinner />;
  if (error) return <Error error={error} />;

  return (
    <ul>
      {users?.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### useCallback and useMemo

```tsx
// ✅ Good: Memoize callbacks passed to children
function ParentComponent() {
  const [count, setCount] = useState(0);

  const handleIncrement = useCallback(() => {
    setCount((c) => c + 1);
  }, []);

  return <ChildComponent onIncrement={handleIncrement} />;
}

// ✅ Good: Memoize expensive computations
function DataTable({ data }: { data: Item[] }) {
  const sortedData = useMemo(() => {
    return [...data].sort((a, b) => a.name.localeCompare(b.name));
  }, [data]);

  return (
    <table>
      {sortedData.map((item) => (
        <tr key={item.id}>
          <td>{item.name}</td>
        </tr>
      ))}
    </table>
  );
}
```

### useRef (Airbnb Standard)

```tsx
// ✅ Good: Use useRef in functional components
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} />;
}

// ❌ Bad: String refs (deprecated)
<Foo ref="myRef" />
```

---

## Performance Optimization

### React.memo

```tsx
// ✅ Good: Memoize expensive components
interface ItemProps {
  id: string;
  name: string;
  onSelect: (id: string) => void;
}

export const Item = React.memo(function Item({ id, name, onSelect }: ItemProps) {
  console.log('Item rendered:', name);
  
  return (
    <div onClick={() => onSelect(id)}>
      {name}
    </div>
  );
});

// ✅ Good: Custom comparison function
export const ItemWithCustomComparison = React.memo(
  Item,
  (prevProps, nextProps) => {
    return prevProps.id === nextProps.id && prevProps.name === nextProps.name;
  }
);
```

### Lazy Loading (Airbnb Standard)

```tsx
// ✅ Good: Lazy load components
const LazyEditor = React.lazy(() => import('./Editor'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <LazyEditor />
    </Suspense>
  );
}
```

### Avoiding Inline Functions (Airbnb Standard)

```tsx
// ✅ Good: Stable callback reference
function TodoList({ todos }: { todos: Todo[] }) {
  const handleToggle = useCallback((id: string) => {
    // Toggle logic
  }, []);

  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} onToggle={handleToggle} />
      ))}
    </ul>
  );
}

// ❌ Bad: Creates new function on every render
function TodoList({ todos }: { todos: Todo[] }) {
  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} onToggle={() => toggle(todo.id)} />
      ))}
    </ul>
  );
}
```

---

## Atomic Design System

### Atoms (Basic Elements)

```tsx
// Button.tsx
interface ButtonProps {
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  children: React.ReactNode;
}

export function Button({ onClick, variant = 'primary', children }: ButtonProps) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {children}
    </button>
  );
}

// Input.tsx
interface InputProps {
  value: string;
  onChange: (value: string) => void;
  label: string;
  type?: 'text' | 'email' | 'password';
}

export function Input({ value, onChange, label, type = 'text' }: InputProps) {
  const id = useId();
  
  return (
    <div className="input-wrapper">
      <label htmlFor={id}>{label}</label>
      <input
        id={id}
        type={type}
        value={value}
        onChange={(e) => onChange(e.target.value)}
      />
    </div>
  );
}
```

### Molecules (Combinations)

```tsx
// FormField.tsx
interface FormFieldProps {
  label: string;
  error?: string;
  children: React.ReactNode;
}

export function FormField({ label, error, children }: FormFieldProps) {
  return (
    <div className="form-field">
      <label>{label}</label>
      {children}
      {error && <span className="error">{error}</span>}
    </div>
  );
}

// SearchBar.tsx
interface SearchBarProps {
  value: string;
  onChange: (value: string) => void;
  onSearch: () => void;
}

export function SearchBar({ value, onChange, onSearch }: SearchBarProps) {
  return (
    <div className="search-bar">
      <Input label="Search" value={value} onChange={onChange} />
      <Button onClick={onSearch}>Search</Button>
    </div>
  );
}
```

### Organisms (Complex Components)

```tsx
// UserForm.tsx
interface UserFormProps {
  initialData?: Partial<User>;
  onSubmit: (user: User) => void;
  onCancel: () => void;
}

export function UserForm({ initialData, onSubmit, onCancel }: UserFormProps) {
  const [name, setName] = useState(initialData?.name ?? '');
  const [email, setEmail] = useState(initialData?.email ?? '');
  const [errors, setErrors] = useState<Record<string, string>>({});

  const handleSubmit = () => {
    const newErrors: Record<string, string> = {};
    
    if (!name) newErrors.name = 'Name is required';
    if (!email) newErrors.email = 'Email is required';
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }
    
    onSubmit({ name, email } as User);
  };

  return (
    <form>
      <FormField label="Name" error={errors.name}>
        <Input value={name} onChange={setName} label="" />
      </FormField>
      
      <FormField label="Email" error={errors.email}>
        <Input value={email} onChange={setEmail} label="" type="email" />
      </FormField>
      
      <div className="actions">
        <Button onClick={handleSubmit}>Submit</Button>
        <Button onClick={onCancel} variant="secondary">Cancel</Button>
      </div>
    </form>
  );
}
```

---

## JSX Formatting

### Alignment (Airbnb Standard)

```tsx
// ✅ Good: Multiline props with closing bracket on new line
<Foo
  superLongParam="bar"
  anotherSuperLongParam="baz"
>
  <Quux />
</Foo>

// ✅ Good: Single line when props fit
<Foo bar="bar" />

// ❌ Bad: Inconsistent alignment
<Foo superLongParam="bar"
     anotherSuperLongParam="baz" />
```

### Quotes (Airbnb Standard)

```tsx
// ✅ Good: Double quotes for JSX, single for JS
<Foo bar="bar" />
<Foo style={{ left: '20px' }} />

// ❌ Bad: Single quotes in JSX
<Foo bar='bar' />
```

### Spacing (Airbnb Standard)

```tsx
// ✅ Good: No padding in JSX curly braces
<Foo bar={baz} />

// ❌ Bad: Padding in curly braces
<Foo bar={ baz } />
```

### Keys (Airbnb Standard)

```tsx
// ✅ Good: Stable ID as key
{todos.map((todo) => (
  <Todo key={todo.id} {...todo} />
))}

// ❌ Bad: Index as key (unstable)
{todos.map((todo, index) => (
  <Todo key={index} {...todo} />
))}
```

### Parentheses (Airbnb Standard)

```tsx
// ✅ Good: Parentheses for multiline JSX
function render() {
  return (
    <div>
      <Component />
    </div>
  );
}

// ✅ Good: No parentheses needed for single line
function render() {
  return <div>Simple</div>;
}

// ✅ Good: Parentheses for conditional rendering
{showButton && (
  <Button />
)}

// ❌ Bad: No parentheses for multiline
{showButton &&
  <Button />
}
```

### Self-Closing Tags (Airbnb Standard)

```tsx
// ✅ Good: Self-closing when no children
<Foo />

// ❌ Bad: Unnecessary closing tag
<Foo></Foo>
```

### Props Spreading (Airbnb Standard)

```tsx
// ✅ Good: Spread props explicitly
function Button({ className, ...rest }: ButtonProps) {
  return <button className={`btn ${className}`} {...rest} />;
}

// ⚠️ Acceptable: Spread in HOCs or wrappers
<Component {...props} />

// ❌ Bad: Indiscriminate spreading
<div {...props} /> // What props are these?
```

---

## Best Practices Summary

### ✅ DO

- **Use functional components** with hooks
- **Use `.tsx` extension** for React files
- **Name components** in PascalCase
- **Use camelCase** for prop names
- **Omit boolean `true`** values
- **Use keys** with stable IDs
- **Memoize callbacks** passed to children
- **Lazy load** heavy components
- **Follow atomic design** hierarchy
- **Extract custom hooks** for reusable logic

### ❌ DON'T

- **Don't use class components** for new code
- **Don't use string refs** (deprecated)
- **Don't use index as key** in lists
- **Don't create inline functions** in render
- **Don't forget cleanup** in useEffect
- **Don't mutate state** directly
- **Don't use defaultProps** (use default parameters)
- **Don't nest ternaries** in JSX

### 🎯 Quick Checklist

- [ ] Functional components only
- [ ] Explicit TypeScript types
- [ ] Proper useEffect cleanup
- [ ] Stable keys in lists
- [ ] Memoization where needed
- [ ] Custom hooks extracted
- [ ] Atomic design followed
- [ ] Airbnb formatting rules

---

This React architecture provides a modern, performant, and maintainable foundation for the DAPA extension, fully aligned with Airbnb standards.
