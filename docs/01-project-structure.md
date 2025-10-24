# Project Structure & Standards

> **Scalable project organization for VSCode extensions with React and TypeScript**

## Table of Contents
- [Modular Monorepo Pattern](#modular-monorepo-pattern)
- [Feature-Based Organization](#feature-based-organization)
- [File Naming Conventions](#file-naming-conventions)
- [Absolute Imports Configuration](#absolute-imports-configuration)
- [Directory Structure Standards](#directory-structure-standards)

---

## Modular Monorepo Pattern

### Root Structure

```
dapa/
├── apps/
│   ├── vscode-extension/     # Extension host + React webview UI
│   ├── web-dashboard/        # Optional standalone web UI
│   └── cli-tool/             # Supporting CLI utilities
│
├── packages/
│   ├── ui/                   # Reusable React components
│   ├── api-client/           # REST/GraphQL client abstraction
│   ├── config/               # Shared configuration
│   ├── utils/                # Pure utility functions
│   ├── types/                # Shared TypeScript interfaces
│   └── eslint-config/        # Shared linting rules
│
├── docs/                     # Documentation
├── e2e/                      # End-to-end tests
├── .github/                  # GitHub workflows
├── package.json              # Root package.json
├── tsconfig.json             # Root TypeScript config
└── turbo.json                # Turborepo config (optional)
```

### Benefits

✅ **Code Reuse**: Share components across extension, web, and CLI  
✅ **Independent Deployment**: Deploy apps independently  
✅ **Focused Dependencies**: Each package has only what it needs  
✅ **Easy Testing**: Test packages in isolation  
✅ **Team Scalability**: Teams can own specific packages

### Package.json Setup

```json
{
  "name": "dapa",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "scripts": {
    "dev": "npm run dev --workspace=apps/vscode-extension",
    "build": "npm run build --workspaces",
    "test": "npm run test --workspaces",
    "lint": "npm run lint --workspaces"
  }
}
```

---

## Feature-Based Organization

### Feature Module Structure

```
src/features/openapi/
├── api/                      # API calls for this feature
│   ├── openapi-api.ts
│   └── hooks.ts
├── components/               # Feature-specific components
│   ├── info-editor.tsx
│   ├── path-editor.tsx
│   └── schema-editor.tsx
├── hooks/                    # Feature-specific hooks
│   ├── use-openapi-document.ts
│   └── use-validation.ts
├── store/                    # Feature Redux slice
│   └── openapi-slice.ts
├── types/                    # Feature types
│   └── openapi.types.ts
├── utils/                    # Feature utilities
│   └── validators.ts
└── index.ts                  # Public API exports
```

### Feature Index Pattern

```typescript
// src/features/openapi/index.ts

// Export only the public API
export { InfoEditor, PathEditor, SchemaEditor } from './components';
export { useOpenAPIDocument, useValidation } from './hooks';
export { openapiSlice } from './store/openapi-slice';
export type { OpenAPIDocument, PathItem, Schema } from './types/openapi.types';

// Internal exports (not re-exported)
// - api/
// - utils/
```

### Shared Code Organization

```
src/
├── components/               # Shared components
│   ├── button.tsx
│   ├── input.tsx
│   └── modal.tsx
├── hooks/                    # Shared hooks
│   ├── use-vscode-theme.ts
│   └── use-keyboard-shortcuts.ts
├── utils/                    # Shared utilities
│   ├── date.ts
│   └── string.ts
├── types/                    # Shared types
│   └── common.types.ts
├── features/                 # Feature modules
│   ├── openapi/
│   ├── asyncapi/
│   └── taxilang/
└── app.tsx                   # Root component
```

### Enforcing Feature Boundaries

```javascript
// .eslintrc.js
module.exports = {
  rules: {
    'import/no-restricted-paths': [
      'error',
      {
        zones: [
          // Prevent cross-feature imports
          {
            target: './src/features/openapi',
            from: './src/features',
            except: ['./openapi'],
          },
          {
            target: './src/features/asyncapi',
            from: './src/features',
            except: ['./asyncapi'],
          },
          // Prevent features importing from app
          {
            target: './src/features',
            from: './src/app',
          },
        ],
      },
    ],
  },
};
```

---

## File Naming Conventions

### Components (PascalCase)

```
✅ Good
components/
├── Button.tsx
├── InfoEditor.tsx
└── PathItemEditor.tsx

❌ Bad
components/
├── button.tsx              # Should be PascalCase
├── infoEditor.tsx          # Should be PascalCase
└── path_item_editor.tsx    # Should use hyphens, not underscores
```

### Hooks (camelCase with 'use' prefix)

```
✅ Good
hooks/
├── useOpenAPIDocument.ts
├── useValidation.ts
└── useVSCodeTheme.ts

❌ Bad
hooks/
├── OpenAPIDocument.ts      # Missing 'use' prefix
├── UseValidation.ts        # Should be camelCase
└── use-vscode-theme.ts     # Should be camelCase, not kebab-case
```

### Utilities (kebab-case)

```
✅ Good
utils/
├── format-date.ts
├── validate-schema.ts
└── parse-openapi.ts

❌ Bad
utils/
├── formatDate.ts           # Should be kebab-case
├── ValidateSchema.ts       # Should be kebab-case
└── parse_openapi.ts        # Should use hyphens, not underscores
```

### Types (PascalCase with .types.ts suffix)

```
✅ Good
types/
├── OpenAPI.types.ts
├── User.types.ts
└── Validation.types.ts

❌ Bad
types/
├── openapi.ts              # Should have .types.ts suffix
├── userTypes.ts            # Should be PascalCase
└── validation.d.ts         # Prefer .types.ts over .d.ts
```

### Constants (UPPER_SNAKE_CASE)

```typescript
✅ Good
// constants/api-endpoints.ts
export const API_BASE_URL = 'https://api.example.com';
export const DEFAULT_TIMEOUT = 5000;
export const MAX_RETRIES = 3;

❌ Bad
export const apiBaseUrl = 'https://api.example.com';     // Should be UPPER_SNAKE_CASE
export const defaultTimeout = 5000;                       // Should be UPPER_SNAKE_CASE
```

### Test Files (*.test.ts or *.spec.ts)

```
✅ Good
components/
├── Button.tsx
├── Button.test.tsx
└── Button.stories.tsx

utils/
├── format-date.ts
└── format-date.test.ts

❌ Bad
components/
├── Button.tsx
└── ButtonTest.tsx          # Should use .test.tsx suffix

utils/
├── format-date.ts
└── test-format-date.ts     # Should use .test.ts suffix
```

### ESLint File Naming Enforcement

```javascript
// .eslintrc.js
module.exports = {
  plugins: ['check-file'],
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
  },
};
```

---

## Absolute Imports Configuration

### TypeScript Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@features/*": ["./src/features/*"],
      "@hooks/*": ["./src/hooks/*"],
      "@utils/*": ["./src/utils/*"],
      "@types/*": ["./src/types/*"],
      "@store/*": ["./src/store/*"],
      "@api/*": ["./src/api/*"]
    }
  }
}
```

### Webpack Configuration

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@features': path.resolve(__dirname, 'src/features'),
      '@hooks': path.resolve(__dirname, 'src/hooks'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@types': path.resolve(__dirname, 'src/types'),
      '@store': path.resolve(__dirname, 'src/store'),
      '@api': path.resolve(__dirname, 'src/api'),
    },
  },
};
```

### Usage Examples

```typescript
// ❌ Bad: Relative imports get messy
import { Button } from '../../../components/button';
import { useOpenAPIDocument } from '../../features/openapi/hooks';
import { formatDate } from '../../../utils/format-date';

// ✅ Good: Absolute imports are clean
import { Button } from '@components/button';
import { useOpenAPIDocument } from '@features/openapi/hooks';
import { formatDate } from '@utils/format-date';
```

### ESLint Import Order

```javascript
// .eslintrc.js
module.exports = {
  rules: {
    'import/order': [
      'error',
      {
        groups: [
          'builtin',          // Node.js built-in modules
          'external',         // External packages
          'internal',         // Absolute imports
          ['parent', 'sibling'], // Relative imports
          'index',
        ],
        pathGroups: [
          {
            pattern: '@/**',
            group: 'internal',
            position: 'before',
          },
        ],
        'newlines-between': 'always',
        alphabetize: {
          order: 'asc',
          caseInsensitive: true,
        },
      },
    ],
  },
};
```

**Result:**

```typescript
// Properly ordered imports
import { useState, useEffect } from 'react';           // builtin
import { useDispatch } from 'react-redux';             // external

import { Button } from '@components/button';           // internal (@/...)
import { useOpenAPIDocument } from '@features/openapi'; // internal (@/...)
import { formatDate } from '@utils/format-date';       // internal (@/...)

import { localHelper } from './helpers';               // sibling
```

---

## Directory Structure Standards

### App Structure (VSCode Extension)

```
apps/vscode-extension/
├── src/
│   ├── extension/            # Extension host code
│   │   ├── extension.ts      # Entry point
│   │   ├── webview-provider.ts
│   │   ├── commands.ts
│   │   └── diagnostics-manager.ts
│   │
│   ├── webview/              # React webview code
│   │   ├── index.tsx         # React entry
│   │   ├── app.tsx           # Root component
│   │   ├── components/       # Shared components
│   │   ├── features/         # Feature modules
│   │   ├── hooks/            # Shared hooks
│   │   ├── store/            # Redux store
│   │   └── styles/           # Global styles
│   │
│   └── types/                # Shared types
│       ├── messages.ts       # Message protocol
│       └── vscode.d.ts       # VSCode API augmentation
│
├── media/                    # Static assets
│   ├── icon.svg
│   └── styles.css
│
├── dist/                     # Build output
├── package.json
├── tsconfig.json
└── webpack.config.js
```

### Package Structure (Shared UI)

```
packages/ui/
├── src/
│   ├── components/           # Component library
│   │   ├── atoms/
│   │   ├── molecules/
│   │   └── organisms/
│   │
│   ├── hooks/                # Reusable hooks
│   ├── utils/                # UI utilities
│   ├── styles/               # Design tokens
│   └── index.ts              # Public exports
│
├── stories/                  # Storybook stories
├── dist/                     # Build output
├── package.json
└── tsconfig.json
```

### Avoiding Common Pitfalls

```
❌ Bad: Flat structure
src/
├── component1.tsx
├── component2.tsx
├── hook1.ts
├── hook2.ts
├── util1.ts
└── util2.ts

✅ Good: Organized structure
src/
├── components/
│   ├── component1.tsx
│   └── component2.tsx
├── hooks/
│   ├── hook1.ts
│   └── hook2.ts
└── utils/
    ├── util1.ts
    └── util2.ts
```

```
❌ Bad: Deep nesting
src/components/forms/inputs/text/advanced/TextField.tsx

✅ Good: Max 3 levels deep
src/components/text-field.tsx
or
src/components/forms/text-field.tsx
```

---

## Best Practices Summary

### ✅ DO

- **Use monorepo** for code sharing
- **Organize by feature** not file type
- **Enforce naming conventions** with ESLint
- **Use absolute imports** for cleaner code
- **Keep directory depth** to 3 levels max
- **Export public API** from feature index
- **Prevent cross-feature** imports

### ❌ DON'T

- **Don't create giant** folders with 50+ files
- **Don't nest too deeply** (max 3 levels)
- **Don't use mixed** naming conventions
- **Don't allow circular** dependencies
- **Don't skip the** index.ts exports
- **Don't import from** sibling features

### 🎯 Quick Checklist

- [ ] Monorepo structure in place
- [ ] Features are isolated modules
- [ ] File naming follows conventions
- [ ] Absolute imports configured
- [ ] ESLint enforces structure
- [ ] Max 3 directory levels
- [ ] Index files export public APIs
- [ ] Cross-feature imports blocked

---

This structure provides a scalable, maintainable foundation for the DAPA VSCode extension while supporting code reuse across multiple applications.