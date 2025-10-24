# VSCode UX & Accessibility

> **Creating native-feeling, accessible extensions that follow VSCode design principles**

## Table of Contents
- [VSCode Design Principles](#vscode-design-principles)
- [Theme Integration](#theme-integration)
- [Keyboard Navigation](#keyboard-navigation)
- [Accessibility Standards](#accessibility-standards)
- [Focus Management](#focus-management)

---

## VSCode Design Principles

### Native Look & Feel

```typescript
// Use VSCode's Codicons for consistent iconography
import { Codicon } from '@vscode/codicons';

function PathItem() {
  return (
    <div className="path-item">
      <i className={Codicon.symbolMethod} /> GET /users
    </div>
  );
}
```

### CSS Variables for Theming

```css
/* src/webview/styles/theme.css */

/* Use VSCode CSS variables */
.editor-container {
  background-color: var(--vscode-editor-background);
  color: var(--vscode-editor-foreground);
  font-family: var(--vscode-font-family);
  font-size: var(--vscode-font-size);
}

.input {
  background-color: var(--vscode-input-background);
  color: var(--vscode-input-foreground);
  border: 1px solid var(--vscode-input-border);
}

.input:focus {
  outline: 1px solid var(--vscode-focusBorder);
  outline-offset: -1px;
}

.button {
  background-color: var(--vscode-button-background);
  color: var(--vscode-button-foreground);
  border: none;
  padding: 4px 14px;
}

.button:hover {
  background-color: var(--vscode-button-hoverBackground);
}

.error {
  color: var(--vscode-errorForeground);
  background-color: var(--vscode-inputValidation-errorBackground);
  border: 1px solid var(--vscode-inputValidation-errorBorder);
}

.warning {
  color: var(--vscode-editorWarning-foreground);
  background-color: var(--vscode-inputValidation-warningBackground);
}
```

---

## Theme Integration

### React Theme Hook

```typescript
// src/webview/hooks/use-vscode-theme.ts
import { useState, useEffect } from 'react';

type Theme = 'light' | 'dark' | 'high-contrast';

export function useVSCodeTheme(): Theme {
  const [theme, setTheme] = useState<Theme>('dark');

  useEffect(() => {
    // Get initial theme from body class
    const getTheme = (): Theme => {
      if (document.body.classList.contains('vscode-light')) return 'light';
      if (document.body.classList.contains('vscode-high-contrast')) return 'high-contrast';
      return 'dark';
    };

    setTheme(getTheme());

    // Listen for theme changes
    const observer = new MutationObserver(() => {
      setTheme(getTheme());
    });

    observer.observe(document.body, {
      attributes: true,
      attributeFilter: ['class'],
    });

    return () => observer.disconnect();
  }, []);

  return theme;
}

// Usage
function MyComponent() {
  const theme = useVSCodeTheme();
  
  return (
    <div className={`component theme-${theme}`}>
      Content adapts to theme
    </div>
  );
}
```

### Dynamic Icon Colors

```typescript
// src/webview/components/icon.tsx
import React from 'react';

interface IconProps {
  name: string;
  color?: 'foreground' | 'error' | 'warning' | 'info';
}

export function Icon({ name, color = 'foreground' }: IconProps) {
  const colorVar = `var(--vscode-${
    color === 'foreground' ? 'foreground' : `${color}Foreground`
  })`;

  return (
    <i
      className={`codicon codicon-${name}`}
      style={{ color: colorVar }}
      aria-hidden="true"
    />
  );
}
```

---

## Keyboard Navigation

### Focus Trap for Modals

```typescript
// src/webview/components/modal.tsx
import React, { useEffect, useRef } from 'react';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  children: React.ReactNode;
}

export function Modal({ isOpen, onClose, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);
  const firstFocusableRef = useRef<HTMLElement>(null);
  const lastFocusableRef = useRef<HTMLElement>(null);

  useEffect(() => {
    if (!isOpen) return;

    // Focus first element
    firstFocusableRef.current?.focus();

    // Handle Escape key
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Escape') {
        onClose();
      }

      // Tab trapping
      if (e.key === 'Tab') {
        const focusableElements = modalRef.current?.querySelectorAll(
          'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );

        if (!focusableElements || focusableElements.length === 0) return;

        const firstElement = focusableElements[0] as HTMLElement;
        const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;

        if (e.shiftKey && document.activeElement === firstElement) {
          e.preventDefault();
          lastElement.focus();
        } else if (!e.shiftKey && document.activeElement === lastElement) {
          e.preventDefault();
          firstElement.focus();
        }
      }
    };

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div
        ref={modalRef}
        className="modal"
        onClick={(e) => e.stopPropagation()}
        role="dialog"
        aria-modal="true"
      >
        {children}
      </div>
    </div>
  );
}
```

### Keyboard Shortcuts

```typescript
// src/webview/hooks/use-keyboard-shortcuts.ts
import { useEffect } from 'react';

interface Shortcut {
  key: string;
  ctrl?: boolean;
  shift?: boolean;
  alt?: boolean;
  action: () => void;
  description: string;
}

export function useKeyboardShortcuts(shortcuts: Shortcut[]) {
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      for (const shortcut of shortcuts) {
        const ctrlMatch = shortcut.ctrl === undefined || shortcut.ctrl === e.ctrlKey;
        const shiftMatch = shortcut.shift === undefined || shortcut.shift === e.shiftKey;
        const altMatch = shortcut.alt === undefined || shortcut.alt === e.altKey;
        const keyMatch = shortcut.key.toLowerCase() === e.key.toLowerCase();

        if (ctrlMatch && shiftMatch && altMatch && keyMatch) {
          e.preventDefault();
          shortcut.action();
          break;
        }
      }
    };

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [shortcuts]);
}

// Usage
function Editor() {
  useKeyboardShortcuts([
    {
      key: 's',
      ctrl: true,
      action: () => saveDocument(),
      description: 'Save document',
    },
    {
      key: 'z',
      ctrl: true,
      action: () => undo(),
      description: 'Undo',
    },
    {
      key: 'z',
      ctrl: true,
      shift: true,
      action: () => redo(),
      description: 'Redo',
    },
  ]);

  return <div>Editor content</div>;
}
```

---

## Accessibility Standards

### Semantic HTML

```tsx
// ✅ Good: Semantic, accessible
function PathList({ paths }: { paths: PathItem[] }) {
  return (
    <nav aria-label="API Paths">
      <ul role="list">
        {paths.map((path) => (
          <li key={path.id}>
            <button
              onClick={() => selectPath(path.id)}
              aria-label={`${path.method} ${path.path}`}
            >
              <span className="method">{path.method}</span>
              <span className="path">{path.path}</span>
            </button>
          </li>
        ))}
      </ul>
    </nav>
  );
}

// ❌ Bad: Not semantic, not accessible
function BadPathList({ paths }: { paths: PathItem[] }) {
  return (
    <div>
      {paths.map((path) => (
        <div key={path.id} onClick={() => selectPath(path.id)}>
          <span>{path.method}</span>
          <span>{path.path}</span>
        </div>
      ))}
    </div>
  );
}
```

### ARIA Labels & Descriptions

```tsx
function SchemaEditor({ schema }: { schema: Schema }) {
  return (
    <section aria-labelledby="schema-title">
      <h2 id="schema-title">{schema.name}</h2>
      
      <form aria-describedby="schema-desc">
        <p id="schema-desc">
          Define the structure and validation rules for this schema
        </p>
        
        <label htmlFor="schema-type">Type</label>
        <select
          id="schema-type"
          value={schema.type}
          onChange={handleTypeChange}
          aria-required="true"
        >
          <option value="">Select type</option>
          <option value="object">Object</option>
          <option value="array">Array</option>
          <option value="string">String</option>
        </select>
        
        <button type="submit" aria-label="Save schema changes">
          Save
        </button>
      </form>
    </section>
  );
}
```

### Screen Reader Announcements

```typescript
// src/webview/utils/announcer.ts

/**
 * Announce messages to screen readers
 */
export class ScreenReaderAnnouncer {
  private liveRegion: HTMLElement;

  constructor() {
    // Create live region for announcements
    this.liveRegion = document.createElement('div');
    this.liveRegion.setAttribute('role', 'status');
    this.liveRegion.setAttribute('aria-live', 'polite');
    this.liveRegion.setAttribute('aria-atomic', 'true');
    this.liveRegion.className = 'sr-only';
    document.body.appendChild(this.liveRegion);
  }

  announce(message: string, priority: 'polite' | 'assertive' = 'polite') {
    this.liveRegion.setAttribute('aria-live', priority);
    this.liveRegion.textContent = message;

    // Clear after announcement
    setTimeout(() => {
      this.liveRegion.textContent = '';
    }, 1000);
  }
}

export const announcer = new ScreenReaderAnnouncer();

// Usage
function SaveButton() {
  const handleSave = async () => {
    await saveDocument();
    announcer.announce('Document saved successfully');
  };

  return <button onClick={handleSave}>Save</button>;
}
```

### CSS for Screen Readers

```css
/* Screen reader only content */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

/* Skip to content link */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: var(--vscode-button-background);
  color: var(--vscode-button-foreground);
  padding: 8px;
  text-decoration: none;
}

.skip-link:focus {
  top: 0;
}
```

---

## Focus Management

### Custom Focus Hook

```typescript
// src/webview/hooks/use-focus-management.ts
import { useEffect, useRef } from 'react';

export function useFocusManagement<T extends HTMLElement>() {
  const ref = useRef<T>(null);

  const focusElement = () => {
    ref.current?.focus();
  };

  const isFocused = () => {
    return document.activeElement === ref.current;
  };

  return { ref, focusElement, isFocused };
}

// Usage
function Input() {
  const { ref, focusElement } = useFocusManagement<HTMLInputElement>();

  useEffect(() => {
    // Auto-focus on mount
    focusElement();
  }, []);

  return <input ref={ref} />;
}
```

### Focus Indicator Styling

```css
/* Ensure focus is always visible */
*:focus {
  outline: 1px solid var(--vscode-focusBorder);
  outline-offset: -1px;
}

/* Custom focus for specific elements */
.button:focus {
  outline: 2px solid var(--vscode-focusBorder);
  outline-offset: 2px;
}

.tree-item:focus {
  background-color: var(--vscode-list-focusBackground);
  color: var(--vscode-list-focusForeground);
}

/* Don't remove focus outline */
/* ❌ Never do this: */
/* *:focus { outline: none; } */
```

### Roving Tab Index

```typescript
// src/webview/components/tree-view.tsx
import React, { useState, useRef, useEffect } from 'react';

interface TreeViewProps {
  items: TreeItem[];
}

export function TreeView({ items }: TreeViewProps) {
  const [focusedIndex, setFocusedIndex] = useState(0);
  const itemRefs = useRef<(HTMLDivElement | null)[]>([]);

  useEffect(() => {
    itemRefs.current[focusedIndex]?.focus();
  }, [focusedIndex]);

  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex((prev) => Math.min(prev + 1, items.length - 1));
        break;
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex((prev) => Math.max(prev - 1, 0));
        break;
      case 'Home':
        e.preventDefault();
        setFocusedIndex(0);
        break;
      case 'End':
        e.preventDefault();
        setFocusedIndex(items.length - 1);
        break;
    }
  };

  return (
    <div role="tree" onKeyDown={handleKeyDown}>
      {items.map((item, index) => (
        <div
          key={item.id}
          ref={(el) => (itemRefs.current[index] = el)}
          role="treeitem"
          tabIndex={index === focusedIndex ? 0 : -1}
          onClick={() => setFocusedIndex(index)}
          aria-selected={index === focusedIndex}
        >
          {item.label}
        </div>
      ))}
    </div>
  );
}
```

---

## Best Practices Summary

### ✅ DO

- **Use VSCode CSS variables** for theming
- **Follow VSCode design patterns** for native feel
- **Support keyboard navigation** everywhere
- **Provide ARIA labels** for screen readers
- **Test with screen readers** (NVDA, JAWS, VoiceOver)
- **Make focus indicators** clearly visible
- **Announce state changes** to screen readers
- **Use semantic HTML** elements

### ❌ DON'T

- **Don't remove focus outlines**
- **Don't rely only on color** for information
- **Don't use div/span** when semantic elements exist
- **Don't forget alt text** for images
- **Don't break keyboard navigation**
- **Don't ignore WCAG guidelines**
- **Don't forget mobile accessibility** (for web versions)

### 🎯 WCAG 2.1 Level AA Compliance

1. **Perceivable**: Provide text alternatives, captions, adaptable content
2. **Operable**: Keyboard accessible, enough time, no seizures, navigable
3. **Understandable**: Readable text, predictable behavior, input assistance
4. **Robust**: Compatible with assistive technologies

---

This guide ensures the DAPA extension provides an excellent, accessible experience for all users, following VSCode's design principles and WCAG standards.
