# 12. Code Quality Tools

### 12.1 ESLint Configuration

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
