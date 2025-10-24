# Code Quality & Tooling

> **Automated code quality enforcement with ESLint, Prettier, and SonarQube**

## Table of Contents
- [ESLint Configuration](#eslint-configuration)
- [Prettier Configuration](#prettier-configuration)
- [SonarQube Integration](#sonarqube-integration)
- [Pre-commit Hooks](#pre-commit-hooks)
- [CI/CD Integration](#cicd-integration)

---

## ESLint Configuration

### Complete ESLint Setup

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
    'prettier', // Must be last
  ],
  plugins: [
    'react',
    '@typescript-eslint',
    'sonarjs',
    'jsx-a11y',
    'check-file',
    'import',
  ],
  rules: {
    // React
    'react/react-in-jsx-scope': 'off',
    'react/jsx-filename-extension': ['error', { extensions: ['.tsx'] }],
    'react/prop-types': 'off',
    'react/require-default-props': 'off',
    'react/jsx-props-no-spreading': ['error', {
      exceptions: ['Component', 'input', 'button'],
    }],
    'react/function-component-definition': ['error', {
      namedComponents: 'function-declaration',
    }],

    // TypeScript
    '@typescript-eslint/no-unused-vars': ['error', {
      argsIgnorePattern: '^_',
      varsIgnorePattern: '^_',
    }],
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-explicit-any': 'error',
    '@typescript-eslint/no-non-null-assertion': 'warn',
    '@typescript-eslint/consistent-type-imports': ['error', {
      prefer: 'type-imports',
    }],

    // Import
    'import/prefer-default-export': 'off',
    'import/no-default-export': 'off',
    'import/extensions': ['error', 'ignorePackages', {
      ts: 'never',
      tsx: 'never',
    }],
    'import/order': ['error', {
      groups: [
        'builtin',
        'external',
        'internal',
        ['parent', 'sibling'],
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
    }],
    'import/no-restricted-paths': ['error', {
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
    }],

    // General
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'prefer-const': 'error',
    'no-var': 'error',
    'no-param-reassign': ['error', { props: false }],

    // SonarJS
    'sonarjs/cognitive-complexity': ['error', 15],
    'sonarjs/no-duplicate-string': ['error', 3],
    'sonarjs/no-identical-functions': 'error',

    // File naming
    'check-file/filename-naming-convention': ['error', {
      '**/*.{ts,tsx}': 'KEBAB_CASE',
    }, {
      ignoreMiddleExtensions: true,
    }],
    'check-file/folder-naming-convention': ['error', {
      'src/**/!(__tests__)': 'KEBAB_CASE',
    }],

    // Accessibility
    'jsx-a11y/anchor-is-valid': 'off', // VSCode doesn't use traditional anchors
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
  overrides: [
    {
      files: ['*.test.ts', '*.test.tsx', '*.spec.ts', '*.spec.tsx'],
      rules: {
        '@typescript-eslint/no-explicit-any': 'off',
        'sonarjs/no-duplicate-string': 'off',
      },
    },
  ],
};
```

### Package Dependencies

```json
{
  "devDependencies": {
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.50.0",
    "eslint-config-airbnb": "^19.0.4",
    "eslint-config-airbnb-typescript": "^17.1.0",
    "eslint-config-prettier": "^9.0.0",
    "eslint-plugin-check-file": "^2.6.0",
    "eslint-plugin-import": "^2.29.0",
    "eslint-plugin-jsx-a11y": "^6.8.0",
    "eslint-plugin-react": "^7.33.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-sonarjs": "^0.23.0"
  }
}
```

### VS Code Integration

```json
// .vscode/settings.json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ]
}
```

---

## Prettier Configuration

### Prettier Setup

```json
// .prettierrc
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
  "jsxBracketSameLine": false,
  "proseWrap": "preserve"
}
```

### Prettier Ignore

```
// .prettierignore
dist
build
coverage
node_modules
*.min.js
*.min.css
package-lock.json
yarn.lock
```

### Integration with ESLint

```bash
# Install prettier and eslint-config-prettier
npm install -D prettier eslint-config-prettier

# eslint-config-prettier disables ESLint rules that conflict with Prettier
# Add 'prettier' as the LAST item in extends array in .eslintrc.cjs
```

---

## SonarQube Integration

### SonarQube Scanner Setup

```javascript
// sonarqube-scanner.js
const scanner = require('sonarqube-scanner');

scanner(
  {
    serverUrl: process.env.SONAR_HOST_URL || 'http://localhost:9000',
    token: process.env.SONAR_TOKEN,
    options: {
      'sonar.projectKey': 'dapa-vscode-extension',
      'sonar.projectName': 'DAPA VSCode Extension',
      'sonar.projectVersion': '1.0.0',
      
      // Source directories
      'sonar.sources': 'src',
      'sonar.tests': 'src',
      'sonar.test.inclusions': '**/*.test.ts,**/*.test.tsx,**/*.spec.ts,**/*.spec.tsx',
      
      // TypeScript
      'sonar.typescript.tsconfigPaths': 'tsconfig.json',
      
      // Coverage
      'sonar.javascript.lcov.reportPaths': 'coverage/lcov.info',
      'sonar.typescript.lcov.reportPaths': 'coverage/lcov.info',
      'sonar.coverage.exclusions': '**/*.test.ts,**/*.test.tsx,**/*.spec.ts,**/*.spec.tsx',
      
      // Exclusions
      'sonar.exclusions': '**/node_modules/**,**/dist/**,**/build/**,**/coverage/**',
      
      // Encoding
      'sonar.sourceEncoding': 'UTF-8',
      
      // Quality gates
      'sonar.qualitygate.wait': true,
    },
  },
  () => process.exit()
);
```

### Quality Gate Configuration

```yaml
# sonar-project.properties
sonar.projectKey=dapa-vscode-extension
sonar.projectName=DAPA VSCode Extension
sonar.projectVersion=1.0.0

# Paths
sonar.sources=src
sonar.tests=src
sonar.test.inclusions=**/*.test.ts,**/*.test.tsx

# TypeScript
sonar.typescript.tsconfigPath=tsconfig.json

# Coverage
sonar.javascript.lcov.reportPaths=coverage/lcov.info

# Quality gate thresholds
sonar.coverage.minimum=85
sonar.duplicated_lines_density.maximum=3
sonar.cognitive_complexity.maximum=15
```

### Package Scripts

```json
{
  "scripts": {
    "sonar": "node sonarqube-scanner.js",
    "sonar:local": "SONAR_HOST_URL=http://localhost:9000 npm run sonar"
  }
}
```

---

## Pre-commit Hooks

### Husky Setup

```bash
# Install husky and lint-staged
npm install -D husky lint-staged

# Initialize husky
npx husky install

# Add prepare script
npm pkg set scripts.prepare="husky install"
```

### Lint-Staged Configuration

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md,yml,yaml}": [
      "prettier --write"
    ]
  }
}
```

### Pre-commit Hook

```bash
#!/bin/sh
# .husky/pre-commit
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
```

### Commit Message Hook

```bash
#!/bin/sh
# .husky/commit-msg
. "$(dirname "$0")/_/husky.sh"

npx --no -- commitlint --edit "$1"
```

### Commitlint Configuration

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',     // New feature
        'fix',      // Bug fix
        'docs',     // Documentation
        'style',    // Formatting
        'refactor', // Code restructuring
        'perf',     // Performance
        'test',     // Tests
        'chore',    // Maintenance
        'revert',   // Revert commit
      ],
    ],
    'subject-case': [2, 'always', 'sentence-case'],
    'subject-max-length': [2, 'always', 100],
  },
};
```

**Example commit messages:**

```bash
✅ Good commits:
feat: Add OpenAPI 3.1 support
fix: Resolve null pointer in path editor
docs: Update installation guide
refactor: Simplify validation logic

❌ Bad commits:
added feature
Fixed bug
WIP
asdfasdf
```

---

## CI/CD Integration

### GitHub Actions Workflow

```yaml
# .github/workflows/quality.yml
name: Code Quality

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Run Prettier check
        run: npm run format:check

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests with coverage
        run: npm run test:coverage
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  sonarqube:
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0 # Full history for better analysis
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests for coverage
        run: npm run test:coverage
      
      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      
      - name: SonarQube Quality Gate check
        uses: sonarsource/sonarqube-quality-gate-action@master
        timeout-minutes: 5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### NPM Scripts

```json
{
  "scripts": {
    "lint": "eslint 'src/**/*.{ts,tsx}'",
    "lint:fix": "eslint 'src/**/*.{ts,tsx}' --fix",
    "format": "prettier --check .",
    "format:fix": "prettier --write .",
    "format:check": "prettier --check .",
    "type-check": "tsc --noEmit",
    "quality": "npm run lint && npm run format:check && npm run type-check",
    "quality:fix": "npm run lint:fix && npm run format:fix"
  }
}
```

---

## Best Practices Summary

### ✅ DO

- **Run linting** before commits
- **Use pre-commit hooks** to enforce quality
- **Configure IDE** for auto-fix on save
- **Set up CI/CD** quality gates
- **Monitor SonarQube** metrics
- **Keep dependencies** updated
- **Use conventional** commit messages
- **Enforce coverage** thresholds

### ❌ DON'T

- **Don't disable rules** without justification
- **Don't skip pre-commit** hooks
- **Don't ignore warnings** in CI/CD
- **Don't commit unformatted** code
- **Don't lower quality gates** to pass
- **Don't use `eslint-disable`** without comments
- **Don't ignore TypeScript** errors

### 🎯 Quality Metrics

**Target Thresholds:**
- Test Coverage: ≥85%
- Duplicated Lines: ≤3%
- Cognitive Complexity: ≤15 per function
- Code Smells: 0 critical issues
- Security Vulnerabilities: 0
- ESLint Errors: 0
- TypeScript Errors: 0

### 📊 Quality Dashboard

Monitor these metrics:
- Coverage trends over time
- Technical debt ratio
- Code duplication percentage
- Cognitive complexity hotspots
- Security vulnerability count
- Reliability rating (A-E)
- Maintainability rating (A-E)

---

This code quality setup ensures the DAPA extension maintains high standards throughout development with automated enforcement and continuous monitoring.