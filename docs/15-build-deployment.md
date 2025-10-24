# Build & Deployment

> **Production-ready build pipeline and deployment strategies**

## Table of Contents
- [Webpack Configuration](#webpack-configuration)
- [Extension Packaging](#extension-packaging)
- [CI/CD Pipeline](#cicd-pipeline)
- [Release Process](#release-process)
- [Monitoring & Analytics](#monitoring--analytics)

---

## Webpack Configuration

### Extension Build Config

```javascript
// webpack.config.js
const path = require('path');
const webpack = require('webpack');

module.exports = (env, argv) => {
  const isProduction = argv.mode === 'production';

  return [
    // Extension host configuration
    {
      name: 'extension',
      target: 'node',
      mode: isProduction ? 'production' : 'development',
      entry: './src/extension/extension.ts',
      output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'extension.js',
        libraryTarget: 'commonjs2',
      },
      externals: {
        vscode: 'commonjs vscode',
      },
      resolve: {
        extensions: ['.ts', '.js'],
        alias: {
          '@': path.resolve(__dirname, 'src'),
        },
      },
      module: {
        rules: [
          {
            test: /\.ts$/,
            exclude: /node_modules/,
            use: 'ts-loader',
          },
        ],
      },
      devtool: isProduction ? 'source-map' : 'inline-source-map',
    },

    // Webview configuration
    {
      name: 'webview',
      target: 'web',
      mode: isProduction ? 'production' : 'development',
      entry: './src/webview/index.tsx',
      output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'webview.js',
      },
      resolve: {
        extensions: ['.ts', '.tsx', '.js', '.jsx'],
        alias: {
          '@': path.resolve(__dirname, 'src'),
          '@components': path.resolve(__dirname, 'src/components'),
          '@features': path.resolve(__dirname, 'src/features'),
        },
      },
      module: {
        rules: [
          {
            test: /\.tsx?$/,
            exclude: /node_modules/,
            use: {
              loader: 'ts-loader',
              options: {
                configFile: 'tsconfig.webview.json',
              },
            },
          },
          {
            test: /\.css$/,
            use: ['style-loader', 'css-loader'],
          },
        ],
      },
      plugins: [
        new webpack.DefinePlugin({
          'process.env.NODE_ENV': JSON.stringify(
            isProduction ? 'production' : 'development'
          ),
        }),
      ],
      devtool: isProduction ? 'source-map' : 'inline-source-map',
      optimization: {
        minimize: isProduction,
      },
    },
  ];
};
```

### TypeScript Configurations

```json
// tsconfig.json (extension)
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/extension/**/*", "src/types/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

```json
// tsconfig.webview.json (React)
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@features/*": ["./src/features/*"]
    }
  },
  "include": ["src/webview/**/*", "src/types/**/*"]
}
```

### Build Scripts

```json
{
  "scripts": {
    "clean": "rimraf dist",
    "compile": "webpack --mode development",
    "watch": "webpack --mode development --watch",
    "build": "webpack --mode production",
    "prebuild": "npm run clean && npm run lint && npm run test",
    "package": "vsce package",
    "publish": "vsce publish"
  }
}
```

---

## Extension Packaging

### package.json Configuration

```json
{
  "name": "dapa-vscode-extension",
  "displayName": "DAPA API Authoring",
  "description": "No-code API specification authoring for OpenAPI, AsyncAPI, and more",
  "version": "1.0.0",
  "publisher": "your-publisher-name",
  "repository": {
    "type": "git",
    "url": "https://github.com/your-org/dapa-vscode-extension"
  },
  "engines": {
    "vscode": "^1.80.0"
  },
  "categories": [
    "Other",
    "Programming Languages"
  ],
  "keywords": [
    "openapi",
    "asyncapi",
    "api",
    "specification",
    "no-code"
  ],
  "activationEvents": [
    "onView:dapa.editorView",
    "onCommand:dapa.openEditor"
  ],
  "main": "./dist/extension.js",
  "contributes": {
    "viewsContainers": {
      "activitybar": [
        {
          "id": "dapa",
          "title": "DAPA",
          "icon": "resources/icon.svg"
        }
      ]
    },
    "views": {
      "dapa": [
        {
          "type": "webview",
          "id": "dapa.editorView",
          "name": "API Editor"
        }
      ]
    },
    "commands": [
      {
        "command": "dapa.openEditor",
        "title": "DAPA: Open Editor",
        "icon": "$(file-code)"
      }
    ],
    "configuration": {
      "title": "DAPA",
      "properties": {
        "dapa.theme": {
          "type": "string",
          "enum": ["auto", "light", "dark"],
          "default": "auto",
          "description": "Theme preference"
        }
      }
    }
  },
  "icon": "resources/icon.png",
  "galleryBanner": {
    "color": "#1e1e1e",
    "theme": "dark"
  }
}
```

### .vscodeignore

```
# Build artifacts
.vscode/**
.vscode-test/**
src/**
node_modules/**
*.vsix

# Development files
.eslintrc.cjs
.prettierrc
tsconfig*.json
webpack.config.js
vitest.config.ts
playwright.config.ts

# Documentation
docs/**
CONTRIBUTING.md
CHANGELOG.md

# Tests
**/*.test.ts
**/*.test.tsx
**/*.spec.ts
**/*.spec.tsx
e2e/**
coverage/**

# Git
.git
.gitignore
.gitattributes

# CI/CD
.github/**
.gitlab-ci.yml

# Misc
.DS_Store
*.log
*.map
```

### Creating VSIX Package

```bash
# Install vsce (VSCode Extension CLI)
npm install -g @vscode/vsce

# Package extension
vsce package

# This creates: dapa-vscode-extension-1.0.0.vsix
```

---

## CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
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
      
      - name: Run quality checks
        run: |
          npm run lint
          npm run type-check
          npm run test:coverage
      
      - name: Build extension
        run: npm run build
      
      - name: Package extension
        run: npx vsce package
      
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: vsix
          path: '*.vsix'

  publish:
    needs: build
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/')
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: vsix
      
      - name: Publish to VS Marketplace
        run: npx vsce publish -p ${{ secrets.VSCE_PAT }}
        env:
          VSCE_PAT: ${{ secrets.VSCE_PAT }}
      
      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          files: '*.vsix'
          generate_release_notes: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Version Bumping

```bash
# Patch version (1.0.0 -> 1.0.1)
npm version patch

# Minor version (1.0.0 -> 1.1.0)
npm version minor

# Major version (1.0.0 -> 2.0.0)
npm version major

# This automatically:
# 1. Updates package.json version
# 2. Creates git commit
# 3. Creates git tag
```

---

## Release Process

### Semantic Versioning

Follow [SemVer](https://semver.org/):

- **MAJOR** (1.0.0 → 2.0.0): Breaking changes
- **MINOR** (1.0.0 → 1.1.0): New features, backward compatible
- **PATCH** (1.0.0 → 1.0.1): Bug fixes, backward compatible

### CHANGELOG.md

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- New feature X

### Changed
- Updated Y

### Fixed
- Bug fix Z

## [1.0.0] - 2025-01-15

### Added
- Initial release
- OpenAPI 3.0/3.1 support
- AsyncAPI 2.6 support
- Visual editor for API specifications
- Validation against official schemas

### Security
- Implemented input sanitization
```

### Release Checklist

```markdown
## Pre-Release Checklist

- [ ] All tests passing
- [ ] Code coverage ≥85%
- [ ] Linting passes
- [ ] Type checking passes
- [ ] CHANGELOG.md updated
- [ ] Version bumped in package.json
- [ ] Documentation updated
- [ ] README.md screenshots updated
- [ ] Breaking changes documented

## Release Steps

1. [ ] Create release branch
2. [ ] Update CHANGELOG.md
3. [ ] Bump version (`npm version`)
4. [ ] Create PR to main
5. [ ] Merge PR
6. [ ] Push tag to trigger release
7. [ ] Verify marketplace listing
8. [ ] Announce release
```

### Publishing to VS Marketplace

```bash
# First time: Login
vsce login your-publisher-name

# Package
vsce package

# Publish (auto-increments version)
vsce publish

# Publish specific version
vsce publish 1.0.1

# Publish without incrementing
vsce publish --no-update-package-json
```

---

## Monitoring & Analytics

### Telemetry Integration

```typescript
// src/extension/telemetry.ts
import * as vscode from 'vscode';
import TelemetryReporter from '@vscode/extension-telemetry';

const TELEMETRY_KEY = process.env.TELEMETRY_KEY || '';

export class Telemetry {
  private reporter: TelemetryReporter;

  constructor(context: vscode.ExtensionContext) {
    this.reporter = new TelemetryReporter(TELEMETRY_KEY);
    context.subscriptions.push(this.reporter);
  }

  trackEvent(name: string, properties?: Record<string, string>) {
    if (!vscode.workspace.getConfiguration('telemetry').get('enableTelemetry')) {
      return;
    }

    this.reporter.sendTelemetryEvent(name, properties);
  }

  trackError(error: Error, properties?: Record<string, string>) {
    this.reporter.sendTelemetryErrorEvent('error', properties, {
      error: error.message,
      stack: error.stack || '',
    });
  }
}

// Usage
const telemetry = new Telemetry(context);

telemetry.trackEvent('documentCreated', {
  type: 'openapi',
  version: '3.1',
});

telemetry.trackError(new Error('Validation failed'), {
  document: 'openapi',
});
```

### Performance Monitoring

```typescript
// src/extension/performance.ts
export class PerformanceMonitor {
  private timings: Map<string, number> = new Map();

  start(label: string): void {
    this.timings.set(label, Date.now());
  }

  end(label: string): number {
    const start = this.timings.get(label);
    if (!start) return 0;

    const duration = Date.now() - start;
    this.timings.delete(label);

    console.log(`[Performance] ${label}: ${duration}ms`);
    return duration;
  }

  async measure<T>(label: string, fn: () => Promise<T>): Promise<T> {
    this.start(label);
    try {
      return await fn();
    } finally {
      this.end(label);
    }
  }
}

// Usage
const perf = new PerformanceMonitor();

perf.start('documentLoad');
const document = await loadDocument();
perf.end('documentLoad');

// Or with measure
const document = await perf.measure('documentLoad', () => loadDocument());
```

### Error Reporting (Sentry)

```typescript
// src/extension/error-reporting.ts
import * as Sentry from '@sentry/node';

export function initErrorReporting() {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    release: `dapa-vscode@${process.env.npm_package_version}`,
  });
}

export function reportError(error: Error, context?: Record<string, any>) {
  Sentry.captureException(error, {
    extra: context,
  });
}

// Usage
try {
  await validateDocument(doc);
} catch (error) {
  reportError(error as Error, {
    documentType: 'openapi',
    documentVersion: '3.1',
  });
}
```

---

## Best Practices Summary

### ✅ DO

- **Use semantic versioning** consistently
- **Automate builds** with CI/CD
- **Test before release** (all quality gates)
- **Update CHANGELOG** for every release
- **Monitor performance** in production
- **Track usage** with telemetry (with user consent)
- **Report errors** to monitoring service
- **Keep dependencies** updated

### ❌ DON'T

- **Don't skip testing** before release
- **Don't publish** without updating docs
- **Don't ignore** bundle size
- **Don't expose** secrets in code
- **Don't break** semantic versioning
- **Don't release** without changelog
- **Don't ignore** user feedback

### 🎯 Quick Checklist

- [ ] Webpack configured
- [ ] Build scripts working
- [ ] CI/CD pipeline set up
- [ ] Version bumping automated
- [ ] CHANGELOG maintained
- [ ] Telemetry integrated
- [ ] Error reporting configured
- [ ] Marketplace listing complete

### 📊 Key Metrics to Monitor

- **Installation count**
- **Active users** (daily/monthly)
- **Error rate**
- **Performance** (load times)
- **User ratings**
- **Feature usage**
- **Crash reports**

---

This build and deployment process ensures the DAPA extension is production-ready, maintainable, and properly monitored in the wild.