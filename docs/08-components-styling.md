# Component Architecture & Styling

> **A superior component architecture with VSCode theme integration**

## Table of Contents
- [Component Composition](#component-composition)
- [Styling Strategy](#styling-strategy)
- [Component Patterns](#component-patterns)
- [Performance](#performance)

---

## Component Composition

### Atomic Design Hierarchy

```
Atoms → Molecules → Organisms → Templates → Pages
```

**Example Structure:**

```typescript
// Atoms: Basic building blocks
export function Button({ children, onClick, variant = 'primary' }: ButtonProps) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {children}
    </button>
  );
}

export function Input({ value, onChange, label }: InputProps) {
  const id = useId();
  return (
    <div className="input-wrapper">
      <label htmlFor={id}>{label}</label>
      <input id={id} value={value} onChange={onChange} />
    </div>
  );
}

// Molecules: Combinations of atoms
export function FormField({ label, error, children }: FormFieldProps) {
  return (
    <div className="form-field">
      <label>{label}</label>
      {children}
      {error && <span className="error">{error}</span>}
    </div>
  );
}

export function SearchBar({ value, onChange, onSearch }: SearchBarProps) {
  return (
    <div className="search-bar">
      <Input value={value} onChange={onChange} label="Search" />
      <Button onClick={onSearch}>Search</Button>
    </div>
  );
}

// Organisms: Complex components
export function PathEditor({ path, onSave }: PathEditorProps) {
  return (
    <div className="path-editor">
      <FormField label="Path">
        <Input value={path.path} onChange={handlePathChange} />
      </FormField>
      <FormField label="Summary">
        <Input value={path.summary} onChange={handleSummaryChange} />
      </FormField>
      <Button onClick={onSave}>Save</Button>
    </div>
  );
}
```

### Compound Components Pattern

```typescript
// src/components/tabs.tsx

interface TabsContextValue {
  activeTab: string;
  setActiveTab: (id: string) => void;
}

const TabsContext = React.createContext<TabsContextValue | null>(null);

function useTabs() {
  const context = React.useContext(TabsContext);
  if (!context) throw new Error('useTabs must be used within Tabs');
  return context;
}

// Root component
export function Tabs({ children, defaultTab }: TabsProps) {
  const [activeTab, setActiveTab] = React.useState(defaultTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

// Sub-components
Tabs.List = function TabsList({ children }: { children: React.ReactNode }) {
  return <div className="tabs-list" role="tablist">{children}</div>;
};

Tabs.Tab = function Tab({ id, children }: TabProps) {
  const { activeTab, setActiveTab } = useTabs();
  const isActive = activeTab === id;

  return (
    <button
      role="tab"
      aria-selected={isActive}
      onClick={() => setActiveTab(id)}
      className={isActive ? 'tab active' : 'tab'}
    >
      {children}
    </button>
  );
};

Tabs.Panel = function TabPanel({ id, children }: TabPanelProps) {
  const { activeTab } = useTabs();
  if (activeTab !== id) return null;

  return (
    <div role="tabpanel" className="tab-panel">
      {children}
    </div>
  );
};

// Usage
function Editor() {
  return (
    <Tabs defaultTab="info">
      <Tabs.List>
        <Tabs.Tab id="info">Info</Tabs.Tab>
        <Tabs.Tab id="paths">Paths</Tabs.Tab>
        <Tabs.Tab id="schemas">Schemas</Tabs.Tab>
      </Tabs.List>

      <Tabs.Panel id="info">
        <InfoEditor />
      </Tabs.Panel>
      <Tabs.Panel id="paths">
        <PathsEditor />
      </Tabs.Panel>
      <Tabs.Panel id="schemas">
        <SchemasEditor />
      </Tabs.Panel>
    </Tabs>
  );
}
```

### Render Props Pattern

```typescript
// src/components/data-loader.tsx

interface DataLoaderProps<T> {
  load: () => Promise<T>;
  children: (state: {
    data: T | null;
    loading: boolean;
    error: Error | null;
    refetch: () => void;
  }) => React.ReactNode;
}

export function DataLoader<T>({ load, children }: DataLoaderProps<T>) {
  const [data, setData] = React.useState<T | null>(null);
  const [loading, setLoading] = React.useState(true);
  const [error, setError] = React.useState<Error | null>(null);

  const fetchData = React.useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const result = await load();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  }, [load]);

  React.useEffect(() => {
    fetchData();
  }, [fetchData]);

  return <>{children({ data, loading, error, refetch: fetchData })}</>;
}

// Usage
function SchemaList() {
  return (
    <DataLoader load={() => fetchSchemas()}>
      {({ data, loading, error, refetch }) => {
        if (loading) return <Spinner />;
        if (error) return <ErrorMessage error={error} onRetry={refetch} />;
        if (!data) return null;

        return (
          <ul>
            {data.map((schema) => (
              <li key={schema.id}>{schema.name}</li>
            ))}
          </ul>
        );
      }}
    </DataLoader>
  );
}
```

---

## Styling Strategy

### CSS Modules with VSCode Variables

```typescript
// src/components/button.module.css
.button {
  padding: 6px 14px;
  border: none;
  border-radius: 2px;
  font-family: var(--vscode-font-family);
  font-size: var(--vscode-font-size);
  cursor: pointer;
  transition: background-color 0.1s;
}

.primary {
  background-color: var(--vscode-button-background);
  color: var(--vscode-button-foreground);
}

.primary:hover {
  background-color: var(--vscode-button-hoverBackground);
}

.secondary {
  background-color: var(--vscode-button-secondaryBackground);
  color: var(--vscode-button-secondaryForeground);
}

.secondary:hover {
  background-color: var(--vscode-button-secondaryHoverBackground);
}

// src/components/button.tsx
import styles from './button.module.css';

export function Button({ variant = 'primary', children, ...props }: ButtonProps) {
  return (
    <button
      className={`${styles.button} ${styles[variant]}`}
      {...props}
    >
      {children}
    </button>
  );
}
```

### Styled Components Alternative

```typescript
// src/components/styled-button.tsx
import styled from 'styled-components';

const StyledButton = styled.button<{ variant: 'primary' | 'secondary' }>`
  padding: 6px 14px;
  border: none;
  border-radius: 2px;
  font-family: var(--vscode-font-family);
  cursor: pointer;

  ${(props) =>
    props.variant === 'primary'
      ? `
    background-color: var(--vscode-button-background);
    color: var(--vscode-button-foreground);
    
    &:hover {
      background-color: var(--vscode-button-hoverBackground);
    }
  `
      : `
    background-color: var(--vscode-button-secondaryBackground);
    color: var(--vscode-button-secondaryForeground);
    
    &:hover {
      background-color: var(--vscode-button-secondaryHoverBackground);
    }
  `}
`;

export function Button({ variant = 'primary', children, ...props }: ButtonProps) {
  return (
    <StyledButton variant={variant} {...props}>
      {children}
    </StyledButton>
  );
}
```

### Utility Classes Approach

```css
/* src/styles/utilities.css */

/* Spacing */
.mt-1 { margin-top: 4px; }
.mt-2 { margin-top: 8px; }
.mt-3 { margin-top: 12px; }
.mt-4 { margin-top: 16px; }

.p-2 { padding: 8px; }
.p-4 { padding: 16px; }

/* Layout */
.flex { display: flex; }
.flex-col { flex-direction: column; }
.items-center { align-items: center; }
.justify-between { justify-content: space-between; }
.gap-2 { gap: 8px; }

/* Colors (use VSCode variables) */
.text-foreground { color: var(--vscode-foreground); }
.text-error { color: var(--vscode-errorForeground); }
.bg-editor { background-color: var(--vscode-editor-background); }

/* Typography */
.font-bold { font-weight: 600; }
.text-sm { font-size: 12px; }
.text-base { font-size: var(--vscode-font-size); }
```

```tsx
// Usage
function Header() {
  return (
    <div className="flex items-center justify-between p-4 bg-editor">
      <h1 className="font-bold text-base">API Editor</h1>
      <Button>Save</Button>
    </div>
  );
}
```

---

## Component Patterns

### Controlled vs Uncontrolled

```typescript
// Controlled: Parent manages state
function ControlledInput() {
  const [value, setValue] = React.useState('');

  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}

// Uncontrolled: Component manages own state with ref
function UncontrolledInput() {
  const inputRef = React.useRef<HTMLInputElement>(null);

  const getValue = () => inputRef.current?.value;

  return <input ref={inputRef} />;
}

// Flexible: Support both patterns
function FlexibleInput({
  value,
  defaultValue,
  onChange,
}: {
  value?: string;
  defaultValue?: string;
  onChange?: (value: string) => void;
}) {
  const [internalValue, setInternalValue] = React.useState(defaultValue ?? '');
  
  const isControlled = value !== undefined;
  const currentValue = isControlled ? value : internalValue;

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value;
    if (!isControlled) {
      setInternalValue(newValue);
    }
    onChange?.(newValue);
  };

  return <input value={currentValue} onChange={handleChange} />;
}
```

### Polymorphic Components

```typescript
// src/components/text.tsx
type TextProps<C extends React.ElementType> = {
  as?: C;
  children: React.ReactNode;
} & React.ComponentPropsWithoutRef<C>;

export function Text<C extends React.ElementType = 'span'>({
  as,
  children,
  ...props
}: TextProps<C>) {
  const Component = as || 'span';
  return <Component {...props}>{children}</Component>;
}

// Usage
function Example() {
  return (
    <>
      <Text>Default span</Text>
      <Text as="h1">Heading</Text>
      <Text as="p">Paragraph</Text>
      <Text as="a" href="/path">Link</Text>
    </>
  );
}
```

### Container/Presenter Pattern

```typescript
// Container: Logic
function PathEditorContainer({ pathId }: { pathId: string }) {
  const path = useAppSelector((state) => 
    pathsSelectors.selectById(state, pathId)
  );
  const dispatch = useAppDispatch();

  const handleUpdate = (updates: Partial<PathItem>) => {
    dispatch(updatePath({ id: pathId, changes: updates }));
  };

  if (!path) return <div>Path not found</div>;

  return <PathEditorPresenter path={path} onUpdate={handleUpdate} />;
}

// Presenter: UI only
function PathEditorPresenter({
  path,
  onUpdate,
}: {
  path: PathItem;
  onUpdate: (updates: Partial<PathItem>) => void;
}) {
  return (
    <div className="path-editor">
      <Input
        label="Path"
        value={path.path}
        onChange={(e) => onUpdate({ path: e.target.value })}
      />
      <Input
        label="Summary"
        value={path.summary}
        onChange={(e) => onUpdate({ summary: e.target.value })}
      />
    </div>
  );
}
```

---

## Performance

### Memoization

```typescript
// Memoize expensive components
const ExpensiveList = React.memo(function ExpensiveList({ items }: { items: Item[] }) {
  return (
    <ul>
      {items.map((item) => (
        <ExpensiveItem key={item.id} item={item} />
      ))}
    </ul>
  );
}, (prev, next) => {
  // Custom comparison
  return prev.items.length === next.items.length &&
    prev.items.every((item, i) => item.id === next.items[i].id);
});

// Memoize expensive computations
function SchemaValidator({ schema }: { schema: Schema }) {
  const errors = React.useMemo(() => {
    return validateSchema(schema); // Expensive operation
  }, [schema]);

  return <ErrorList errors={errors} />;
}

// Memoize callbacks
function PathList({ paths }: { paths: PathItem[] }) {
  const dispatch = useAppDispatch();

  const handleSelect = React.useCallback((id: string) => {
    dispatch(selectPath(id));
  }, [dispatch]);

  return (
    <ul>
      {paths.map((path) => (
        <PathItem key={path.id} path={path} onSelect={handleSelect} />
      ))}
    </ul>
  );
}
```

### Code Splitting

```typescript
// Lazy load heavy components
const SchemaEditor = React.lazy(() => import('./schema-editor'));
const AdvancedSettings = React.lazy(() => import('./advanced-settings'));

function App() {
  return (
    <React.Suspense fallback={<Spinner />}>
      <Routes>
        <Route path="/schema" element={<SchemaEditor />} />
        <Route path="/settings" element={<AdvancedSettings />} />
      </Routes>
    </React.Suspense>
  );
}
```

### Virtual Scrolling

```typescript
// src/components/virtual-list.tsx
import { useVirtualizer } from '@tanstack/react-virtual';

export function VirtualList({ items }: { items: any[] }) {
  const parentRef = React.useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 35,
    overscan: 5,
  });

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          position: 'relative',
        }}
      >
        {virtualizer.getVirtualItems().map((virtualRow) => (
          <div
            key={virtualRow.index}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            <PathItem path={items[virtualRow.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Best Practices Summary

### ✅ DO

- **Use composition** over inheritance
- **Keep components small** and focused
- **Memoize expensive** operations
- **Use VSCode CSS variables** for theming
- **Implement code splitting** for large features
- **Use TypeScript** for prop types
- **Test components** in isolation

### ❌ DON'T

- **Don't prop drill** - use context or Redux
- **Don't premature optimize** - measure first
- **Don't inline styles** unless dynamic
- **Don't forget accessibility**
- **Don't make giant components**
- **Don't skip memoization** for lists

### 🎯 Key Principles

1. **Composition**: Build complex UIs from simple components
2. **Separation**: Logic in containers, UI in presenters
3. **Reusability**: DRY with compound components
4. **Performance**: Lazy load, virtualize, memoize
5. **Accessibility**: Semantic HTML, ARIA, keyboard nav
6. **Theming**: Use VSCode variables consistently

---

This component architecture provides a scalable, performant foundation that's superior to bulletproof-react for VSCode extensions.
