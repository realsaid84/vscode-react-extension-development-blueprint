
# 13. Testing Strategy

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