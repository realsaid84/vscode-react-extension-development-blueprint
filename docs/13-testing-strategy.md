# Testing Strategy

> **Comprehensive testing approach with Vitest, Testing Library, and Playwright**

## Table of Contents
- [Testing Pyramid](#testing-pyramid)
- [Unit Testing](#unit-testing)
- [Component Testing](#component-testing)
- [Integration Testing](#integration-testing)
- [E2E Testing](#e2e-testing)
- [Coverage Requirements](#coverage-requirements)

---

## Testing Pyramid

```
        /\
       /  \
      / E2E \           Few, slow, expensive
     /______\           Test critical user flows
    /        \
   /Integration\        Test feature interactions
  /____________\
 /              \
/  Unit Tests    \      Many, fast, cheap
/__________________\    Test individual functions
```

### Test Distribution

- **Unit Tests**: 70% - Fast, isolated, pure functions
- **Integration Tests**: 20% - Feature interactions, API calls
- **E2E Tests**: 10% - Critical user journeys

---

## Unit Testing

### Vitest Setup

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/dist/**',
        '**/*.test.{ts,tsx}',
        '**/*.spec.{ts,tsx}',
      ],
      lines: 85,
      functions: 85,
      branches: 85,
      statements: 85,
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@features': path.resolve(__dirname, './src/features'),
      '@utils': path.resolve(__dirname, './src/utils'),
    },
  },
});
```

### Test Setup File

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
import { expect, afterEach, vi } from 'vitest';
import { cleanup } from '@testing-library/react';

// Cleanup after each test
afterEach(() => {
  cleanup();
});

// Mock VSCode API
global.acquireVsCodeApi = vi.fn(() => ({
  postMessage: vi.fn(),
  setState: vi.fn(),
  getState: vi.fn(),
}));

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation((query) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});
```

### Testing Pure Functions

```typescript
// src/utils/format-date.test.ts
import { describe, it, expect } from 'vitest';
import { formatDate, parseDate, isValidDate } from './format-date';

describe('formatDate', () => {
  it('should format date as YYYY-MM-DD', () => {
    const date = new Date('2025-01-15');
    expect(formatDate(date)).toBe('2025-01-15');
  });

  it('should handle invalid dates', () => {
    expect(formatDate(new Date('invalid'))).toBe('Invalid Date');
  });

  it('should format with custom pattern', () => {
    const date = new Date('2025-01-15');
    expect(formatDate(date, 'MM/DD/YYYY')).toBe('01/15/2025');
  });
});

describe('parseDate', () => {
  it('should parse YYYY-MM-DD format', () => {
    const result = parseDate('2025-01-15');
    expect(result).toBeInstanceOf(Date);
    expect(result?.getFullYear()).toBe(2025);
  });

  it('should return null for invalid dates', () => {
    expect(parseDate('invalid')).toBeNull();
  });
});

describe('isValidDate', () => {
  it('should return true for valid dates', () => {
    expect(isValidDate('2025-01-15')).toBe(true);
  });

  it('should return false for invalid dates', () => {
    expect(isValidDate('not-a-date')).toBe(false);
  });
});
```

### Testing Hooks

```typescript
// src/hooks/use-counter.test.ts
import { describe, it, expect } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './use-counter';

describe('useCounter', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('should initialize with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it('should increment counter', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  it('should decrement counter', () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(4);
  });

  it('should reset counter', () => {
    const { result } = renderHook(() => useCounter(10));

    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });

    expect(result.current.count).toBe(10);
  });
});
```

---

## Component Testing

### Testing Library Setup

```typescript
// src/test/test-utils.tsx
import { ReactElement } from 'react';
import { render, RenderOptions } from '@testing-library/react';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import type { RootState } from '@/store';

interface ExtendedRenderOptions extends Omit<RenderOptions, 'queries'> {
  preloadedState?: Partial<RootState>;
}

export function renderWithProviders(
  ui: ReactElement,
  {
    preloadedState = {},
    ...renderOptions
  }: ExtendedRenderOptions = {}
) {
  const store = configureStore({
    reducer: {
      // Add your reducers here
    },
    preloadedState,
  });

  function Wrapper({ children }: { children: React.ReactNode }) {
    return <Provider store={store}>{children}</Provider>;
  }

  return {
    store,
    ...render(ui, { wrapper: Wrapper, ...renderOptions }),
  };
}

export * from '@testing-library/react';
export { renderWithProviders as render };
```

### Testing Simple Components

```typescript
// src/components/button.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
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

  it('should apply variant class', () => {
    render(<Button variant="secondary">Click me</Button>);
    const button = screen.getByText('Click me');
    expect(button).toHaveClass('btn-secondary');
  });

  it('should not call onClick when disabled', () => {
    const handleClick = vi.fn();
    render(
      <Button disabled onClick={handleClick}>
        Click me
      </Button>
    );

    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

### Testing Forms

```typescript
// src/components/user-form.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, waitFor } from '@/test/test-utils';
import userEvent from '@testing-library/user-event';
import { UserForm } from './user-form';

describe('UserForm', () => {
  it('should render form fields', () => {
    render(<UserForm onSubmit={vi.fn()} />);

    expect(screen.getByLabelText(/name/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
  });

  it('should display validation errors', async () => {
    const user = userEvent.setup();
    render(<UserForm onSubmit={vi.fn()} />);

    const submitButton = screen.getByRole('button', { name: /submit/i });
    await user.click(submitButton);

    await waitFor(() => {
      expect(screen.getByText(/name is required/i)).toBeInTheDocument();
      expect(screen.getByText(/email is required/i)).toBeInTheDocument();
    });
  });

  it('should submit valid form', async () => {
    const user = userEvent.setup();
    const handleSubmit = vi.fn();
    render(<UserForm onSubmit={handleSubmit} />);

    await user.type(screen.getByLabelText(/name/i), 'John Doe');
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    await user.click(screen.getByRole('button', { name: /submit/i }));

    await waitFor(() => {
      expect(handleSubmit).toHaveBeenCalledWith({
        name: 'John Doe',
        email: 'john@example.com',
      });
    });
  });

  it('should populate form with initial data', () => {
    render(
      <UserForm
        initialData={{ name: 'Jane Doe', email: 'jane@example.com' }}
        onSubmit={vi.fn()}
      />
    );

    expect(screen.getByLabelText(/name/i)).toHaveValue('Jane Doe');
    expect(screen.getByLabelText(/email/i)).toHaveValue('jane@example.com');
  });
});
```

### Testing Components with Redux

```typescript
// src/features/openapi/components/info-editor.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@/test/test-utils';
import userEvent from '@testing-library/user-event';
import { InfoEditor } from './info-editor';

describe('InfoEditor', () => {
  const mockState = {
    openapi: {
      document: {
        openapi: '3.1.0',
        info: {
          title: 'My API',
          version: '1.0.0',
          description: 'Test description',
        },
        paths: {},
      },
    },
  };

  it('should display current info', () => {
    render(<InfoEditor />, { preloadedState: mockState });

    expect(screen.getByDisplayValue('My API')).toBeInTheDocument();
    expect(screen.getByDisplayValue('1.0.0')).toBeInTheDocument();
    expect(screen.getByDisplayValue('Test description')).toBeInTheDocument();
  });

  it('should update title', async () => {
    const user = userEvent.setup();
    const { store } = render(<InfoEditor />, { preloadedState: mockState });

    const titleInput = screen.getByLabelText(/title/i);
    await user.clear(titleInput);
    await user.type(titleInput, 'Updated API');

    const state = store.getState();
    expect(state.openapi.document?.info.title).toBe('Updated API');
  });
});
```

---

## Integration Testing

### Testing API Integration

```typescript
// src/api/openapi-client.test.ts
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';
import { OpenAPIClient } from './openapi-client';

const server = setupServer();

beforeEach(() => server.listen());
afterEach(() => server.resetHandlers());
afterEach(() => server.close());

describe('OpenAPIClient', () => {
  const client = new OpenAPIClient('http://api.test');

  it('should fetch document', async () => {
    const mockDocument = {
      openapi: '3.1.0',
      info: { title: 'Test API', version: '1.0.0' },
      paths: {},
    };

    server.use(
      http.get('http://api.test/documents/123', () => {
        return HttpResponse.json(mockDocument);
      })
    );

    const result = await client.getDocument('123');
    expect(result).toEqual(mockDocument);
  });

  it('should handle errors', async () => {
    server.use(
      http.get('http://api.test/documents/123', () => {
        return HttpResponse.json(
          { message: 'Not found' },
          { status: 404 }
        );
      })
    );

    await expect(client.getDocument('123')).rejects.toThrow('Not found');
  });

  it('should retry on failure', async () => {
    let attempts = 0;

    server.use(
      http.get('http://api.test/documents/123', () => {
        attempts++;
        if (attempts < 3) {
          return HttpResponse.json(
            { message: 'Server error' },
            { status: 500 }
          );
        }
        return HttpResponse.json({ openapi: '3.1.0' });
      })
    );

    const result = await client.getDocument('123', { maxRetries: 3 });
    expect(attempts).toBe(3);
    expect(result).toHaveProperty('openapi', '3.1.0');
  });
});
```

### Testing Redux Slices

```typescript
// src/store/openapi-slice.test.ts
import { describe, it, expect } from 'vitest';
import { configureStore } from '@reduxjs/toolkit';
import openapiReducer, {
  setDocument,
  updateInfo,
  addPath,
} from './openapi-slice';

describe('openapi slice', () => {
  const createStore = () =>
    configureStore({
      reducer: {
        openapi: openapiReducer,
      },
    });

  it('should handle setDocument', () => {
    const store = createStore();
    const document = {
      openapi: '3.1.0',
      info: { title: 'Test', version: '1.0.0' },
      paths: {},
    };

    store.dispatch(setDocument(document));

    expect(store.getState().openapi.document).toEqual(document);
  });

  it('should handle updateInfo', () => {
    const store = createStore();
    const document = {
      openapi: '3.1.0',
      info: { title: 'Test', version: '1.0.0' },
      paths: {},
    };

    store.dispatch(setDocument(document));
    store.dispatch(updateInfo({ title: 'Updated' }));

    expect(store.getState().openapi.document?.info.title).toBe('Updated');
  });

  it('should handle addPath', () => {
    const store = createStore();
    const document = {
      openapi: '3.1.0',
      info: { title: 'Test', version: '1.0.0' },
      paths: {},
    };

    store.dispatch(setDocument(document));
    store.dispatch(
      addPath({
        path: '/users',
        item: { get: { summary: 'Get users' } },
      })
    );

    expect(store.getState().openapi.document?.paths).toHaveProperty('/users');
  });
});
```

---

## E2E Testing

### Playwright Setup

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### E2E Test Example

```typescript
// e2e/openapi-editor.spec.ts
import { test, expect } from '@playwright/test';

test.describe('OpenAPI Editor', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('should create new document', async ({ page }) => {
    // Click new document button
    await page.click('button:has-text("New Document")');

    // Select OpenAPI 3.1
    await page.click('text=OpenAPI 3.1');

    // Fill in info
    await page.fill('input[name="title"]', 'My API');
    await page.fill('input[name="version"]', '1.0.0');

    // Save
    await page.click('button:has-text("Save")');

    // Verify success message
    await expect(page.locator('text=Document saved')).toBeVisible();
  });

  test('should add path', async ({ page }) => {
    // Open existing document
    await page.click('text=/users/123');

    // Add new path
    await page.click('button:has-text("Add Path")');
    await page.fill('input[name="path"]', '/posts');
    await page.click('button:has-text("Add GET")');

    // Fill operation details
    await page.fill('input[name="summary"]', 'Get all posts');
    await page.click('button:has-text("Save")');

    // Verify path appears in list
    await expect(page.locator('text=/posts')).toBeVisible();
  });

  test('should validate document', async ({ page }) => {
    // Open document
    await page.click('text=/users/123');

    // Clear required field
    await page.fill('input[name="title"]', '');

    // Trigger validation
    await page.click('button:has-text("Validate")');

    // Verify error message
    await expect(page.locator('text=Title is required')).toBeVisible();
  });
});
```

---

## Coverage Requirements

### Coverage Configuration

```typescript
// vitest.config.ts coverage section
coverage: {
  provider: 'v8',
  reporter: ['text', 'json', 'html', 'lcov'],
  
  // Thresholds
  lines: 85,
  functions: 85,
  branches: 85,
  statements: 85,
  
  // Exclusions
  exclude: [
    'node_modules/',
    'src/test/',
    '**/*.d.ts',
    '**/*.config.*',
    '**/dist/**',
    '**/*.test.{ts,tsx}',
    '**/*.spec.{ts,tsx}',
  ],
}
```

### NPM Scripts

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:debug": "playwright test --debug",
    "test:all": "npm run test:coverage && npm run test:e2e"
  }
}
```

---

## Best Practices Summary

### ✅ DO

- **Write tests first** (TDD when possible)
- **Test behavior** not implementation
- **Use descriptive** test names
- **Mock external** dependencies
- **Test edge cases** and errors
- **Keep tests isolated** and independent
- **Use proper assertions** (toBeInTheDocument, toHaveValue)
- **Follow AAA pattern** (Arrange, Act, Assert)

### ❌ DON'T

- **Don't test implementation** details
- **Don't share state** between tests
- **Don't use real APIs** in unit tests
- **Don't skip cleanup**
- **Don't test third-party** libraries
- **Don't ignore coverage** warnings
- **Don't write brittle** tests

### 🎯 Quick Checklist

- [ ] Unit tests for utilities
- [ ] Component tests for UI
- [ ] Integration tests for features
- [ ] E2E tests for critical flows
- [ ] 85%+ code coverage
- [ ] All tests passing
- [ ] No flaky tests
- [ ] CI/CD integration

---

This testing strategy ensures the DAPA extension is reliable, maintainable, and regression-proof through comprehensive automated testing at all levels.