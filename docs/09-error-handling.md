# Error Handling & Graceful Degradation

> **Robust error handling patterns for VSCode extensions with React, ensuring the application never crashes**

## Table of Contents
- [Overview](#overview)
- [Error Boundary Pattern](#error-boundary-pattern)
- [VSCode Problems API Integration](#vscode-problems-api-integration)
- [Error Types & Classification](#error-types--classification)
- [Validation Error Reporting](#validation-error-reporting)
- [Graceful Degradation Strategies](#graceful-degradation-strategies)
- [Error Logging & Monitoring](#error-logging--monitoring)
- [Recovery Mechanisms](#recovery-mechanisms)

---

## Overview

### The Three Layers of Error Handling

```
┌─────────────────────────────────────────────────────────────┐
│           LAYER 1: React Error Boundaries                    │
│  Catch rendering errors, prevent app crash                  │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│           LAYER 2: Try/Catch + Error States                  │
│  Handle async operations, API failures                      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│           LAYER 3: VSCode Problems API                       │
│  Report errors to VSCode UI, lint-style feedback           │
└─────────────────────────────────────────────────────────────┘
```

---

## Error Boundary Pattern

### Root Error Boundary

```typescript
// src/components/error-boundaries/root-error-boundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';
import { ErrorFallback } from './error-fallback';

interface Props {
  children: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
  errorInfo: ErrorInfo | null;
}

/**
 * Root error boundary that catches all unhandled errors
 * Prevents the entire application from crashing
 */
export class RootErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
    };
  }

  static getDerivedStateFromError(error: Error): Partial<State> {
    return {
      hasError: true,
      error,
    };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    // Log error to console
    console.error('Root Error Boundary caught an error:', error, errorInfo);
    
    // Update state with error info
    this.setState({
      errorInfo,
    });
    
    // Call optional error handler
    this.props.onError?.(error, errorInfo);
    
    // Report to VSCode
    this.reportToVSCode(error, errorInfo);
  }

  private reportToVSCode(error: Error, errorInfo: ErrorInfo): void {
    if (typeof acquireVsCodeApi === 'function') {
      const vscode = acquireVsCodeApi();
      vscode.postMessage({
        command: 'error',
        error: {
          message: error.message,
          stack: error.stack,
          componentStack: errorInfo.componentStack,
        },
      });
    }
  }

  private handleReset = (): void => {
    this.setState({
      hasError: false,
      error: null,
      errorInfo: null,
    });
  };

  render(): ReactNode {
    if (this.state.hasError) {
      return (
        <ErrorFallback
          error={this.state.error!}
          errorInfo={this.state.errorInfo!}
          onReset={this.handleReset}
        />
      );
    }

    return this.props.children;
  }
}
```

### Feature-Level Error Boundaries

```typescript
// src/components/error-boundaries/feature-error-boundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  featureName: string;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

/**
 * Feature-level error boundary
 * Allows rest of app to continue if one feature fails
 */
export class FeatureErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): Partial<State> {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    console.error(`Error in ${this.props.featureName}:`, error, errorInfo);
  }

  render(): ReactNode {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <div className="feature-error">
          <h3>⚠️ {this.props.featureName} encountered an error</h3>
          <p>This feature is temporarily unavailable.</p>
          <details>
            <summary>Error details</summary>
            <pre>{this.state.error?.message}</pre>
          </details>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### Component-Level Error Boundaries

```typescript
// src/components/error-boundaries/component-error-boundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: (error: Error) => ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

/**
 * Granular error boundary for individual components
 * Use for complex components that might fail independently
 */
export class ComponentErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): Partial<State> {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    this.props.onError?.(error, errorInfo);
  }

  render(): ReactNode {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback(this.state.error!);
      }

      return (
        <div className="component-error">
          <span>⚠️ Component failed to render</span>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### Error Fallback UI

```typescript
// src/components/error-boundaries/error-fallback.tsx
import React from 'react';
import type { ErrorInfo } from 'react';

interface ErrorFallbackProps {
  error: Error;
  errorInfo: ErrorInfo;
  onReset: () => void;
}

export function ErrorFallback({ error, errorInfo, onReset }: ErrorFallbackProps) {
  const [showDetails, setShowDetails] = React.useState(false);

  return (
    <div className="error-fallback">
      <div className="error-header">
        <h1>⚠️ Something went wrong</h1>
        <p>The application encountered an unexpected error.</p>
      </div>

      <div className="error-actions">
        <button onClick={onReset} className="primary">
          Try Again
        </button>
        <button onClick={() => setShowDetails(!showDetails)}>
          {showDetails ? 'Hide' : 'Show'} Details
        </button>
      </div>

      {showDetails && (
        <div className="error-details">
          <h3>Error Details</h3>
          <pre className="error-message">{error.message}</pre>
          
          <h3>Stack Trace</h3>
          <pre className="error-stack">{error.stack}</pre>
          
          <h3>Component Stack</h3>
          <pre className="error-component-stack">
            {errorInfo.componentStack}
          </pre>
        </div>
      )}
    </div>
  );
}
```

---

## VSCode Problems API Integration

### Diagnostic Collection Manager

```typescript
// src/vscode/diagnostics-manager.ts
import * as vscode from 'vscode';

/**
 * Manages VSCode diagnostics (Problems panel)
 * Reports validation errors as lint-style problems
 */
export class DiagnosticsManager {
  private diagnostics: vscode.DiagnosticCollection;

  constructor(context: vscode.ExtensionContext) {
    this.diagnostics = vscode.languages.createDiagnosticCollection('dapa');
    context.subscriptions.push(this.diagnostics);
  }

  /**
   * Report validation errors to Problems panel
   */
  reportValidationErrors(
    uri: vscode.Uri,
    errors: ValidationError[]
  ): void {
    const diagnostics: vscode.Diagnostic[] = errors.map((error) =>
      this.createDiagnostic(error)
    );

    this.diagnostics.set(uri, diagnostics);
  }

  /**
   * Create a VSCode diagnostic from validation error
   */
  private createDiagnostic(error: ValidationError): vscode.Diagnostic {
    const range = this.errorToRange(error);
    const diagnostic = new vscode.Diagnostic(
      range,
      error.message,
      this.severityFromError(error)
    );

    diagnostic.code = error.code;
    diagnostic.source = 'DAPA';

    if (error.suggestions) {
      diagnostic.relatedInformation = error.suggestions.map((suggestion) => 
        new vscode.DiagnosticRelatedInformation(
          new vscode.Location(error.uri, range),
          suggestion
        )
      );
    }

    return diagnostic;
  }

  /**
   * Convert error position to VSCode range
   */
  private errorToRange(error: ValidationError): vscode.Range {
    const line = error.line ?? 0;
    const column = error.column ?? 0;
    
    return new vscode.Range(
      new vscode.Position(line, column),
      new vscode.Position(line, column + (error.length ?? 1))
    );
  }

  /**
   * Map error severity to VSCode severity
   */
  private severityFromError(error: ValidationError): vscode.DiagnosticSeverity {
    switch (error.severity) {
      case 'error':
        return vscode.DiagnosticSeverity.Error;
      case 'warning':
        return vscode.DiagnosticSeverity.Warning;
      case 'info':
        return vscode.DiagnosticSeverity.Information;
      default:
        return vscode.DiagnosticSeverity.Hint;
    }
  }

  /**
   * Clear all diagnostics for a URI
   */
  clear(uri?: vscode.Uri): void {
    if (uri) {
      this.diagnostics.delete(uri);
    } else {
      this.diagnostics.clear();
    }
  }

  /**
   * Get all diagnostics for a URI
   */
  get(uri: vscode.Uri): readonly vscode.Diagnostic[] | undefined {
    return this.diagnostics.get(uri);
  }
}

interface ValidationError {
  message: string;
  code?: string;
  severity: 'error' | 'warning' | 'info' | 'hint';
  line?: number;
  column?: number;
  length?: number;
  uri: vscode.Uri;
  suggestions?: string[];
}
```

### Webview to Extension Communication

```typescript
// Extension side: src/extension/webview-provider.ts
import * as vscode from 'vscode';
import { DiagnosticsManager } from './diagnostics-manager';

export class DapaWebviewProvider implements vscode.WebviewViewProvider {
  private diagnosticsManager: DiagnosticsManager;

  constructor(
    private context: vscode.ExtensionContext,
    diagnosticsManager: DiagnosticsManager
  ) {
    this.diagnosticsManager = diagnosticsManager;
  }

  resolveWebviewView(webviewView: vscode.WebviewView): void {
    // Setup message handling
    webviewView.webview.onDidReceiveMessage(
      async (message) => {
        switch (message.command) {
          case 'validationErrors':
            await this.handleValidationErrors(message.errors);
            break;

          case 'error':
            await this.handleRuntimeError(message.error);
            break;
        }
      },
      undefined,
      this.context.subscriptions
    );
  }

  private async handleValidationErrors(errors: any[]): Promise<void> {
    const document = vscode.window.activeTextEditor?.document;
    if (!document) return;

    const validationErrors: ValidationError[] = errors.map((error) => ({
      message: error.message,
      code: error.code,
      severity: error.severity || 'error',
      line: error.path ? this.pathToLine(error.path, document) : 0,
      column: 0,
      uri: document.uri,
      suggestions: error.suggestions,
    }));

    this.diagnosticsManager.reportValidationErrors(
      document.uri,
      validationErrors
    );
  }

  private async handleRuntimeError(error: any): Promise<void> {
    vscode.window.showErrorMessage(
      `DAPA Error: ${error.message}`,
      'Show Details'
    ).then((selection) => {
      if (selection === 'Show Details') {
        // Show error in output channel
        const outputChannel = vscode.window.createOutputChannel('DAPA Errors');
        outputChannel.appendLine('=== Error Details ===');
        outputChannel.appendLine(`Message: ${error.message}`);
        outputChannel.appendLine(`Stack: ${error.stack}`);
        outputChannel.show();
      }
    });
  }

  private pathToLine(path: string, document: vscode.TextDocument): number {
    // Convert JSON path (e.g., "paths./users.get") to line number
    const text = document.getText();
    const lines = text.split('\n');
    
    const searchTerm = path.split('.').pop() || '';
    
    for (let i = 0; i < lines.length; i++) {
      if (lines[i].includes(searchTerm)) {
        return i;
      }
    }
    
    return 0;
  }
}
```

### Webview Side: Error Reporter Hook

```typescript
// src/hooks/use-error-reporter.ts
import { useEffect } from 'react';
import { useAppSelector } from '../store';

/**
 * Hook to report errors to VSCode Problems API
 */
export function useErrorReporter() {
  const validationErrors = useAppSelector(
    (state) => state.openapi.validationErrors
  );

  useEffect(() => {
    if (typeof acquireVsCodeApi === 'function') {
      const vscode = acquireVsCodeApi();
      
      // Report validation errors to VSCode
      vscode.postMessage({
        command: 'validationErrors',
        errors: validationErrors.map((error) => ({
          message: error.message,
          code: error.code,
          severity: error.severity,
          path: error.path,
          suggestions: error.suggestions,
        })),
      });
    }
  }, [validationErrors]);
}

// Usage in root component
function App() {
  useErrorReporter();
  
  return (
    <RootErrorBoundary>
      {/* App content */}
    </RootErrorBoundary>
  );
}
```

---

## Error Types & Classification

```typescript
// src/types/errors.ts

/**
 * Base error class for all application errors
 */
export abstract class DapaError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly severity: 'error' | 'warning' | 'info' = 'error'
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

/**
 * Validation errors from schema validation
 */
export class ValidationError extends DapaError {
  constructor(
    message: string,
    public readonly path: string,
    public readonly suggestions?: string[]
  ) {
    super(message, 'VALIDATION_ERROR', 'error');
  }
}

/**
 * Network/API errors
 */
export class NetworkError extends DapaError {
  constructor(
    message: string,
    public readonly statusCode?: number,
    public readonly endpoint?: string
  ) {
    super(message, 'NETWORK_ERROR', 'error');
  }
}

/**
 * File I/O errors
 */
export class FileError extends DapaError {
  constructor(
    message: string,
    public readonly filepath: string,
    public readonly operation: 'read' | 'write' | 'delete'
  ) {
    super(message, 'FILE_ERROR', 'error');
  }
}

/**
 * Parse errors from ANTLR or JSON parsing
 */
export class ParseError extends DapaError {
  constructor(
    message: string,
    public readonly line?: number,
    public readonly column?: number
  ) {
    super(message, 'PARSE_ERROR', 'error');
  }
}

/**
 * Configuration errors
 */
export class ConfigurationError extends DapaError {
  constructor(
    message: string,
    public readonly configKey: string
  ) {
    super(message, 'CONFIG_ERROR', 'warning');
  }
}
```

---

## Validation Error Reporting

```typescript
// src/validation/error-reporter.ts
import Ajv from 'ajv';
import type { ErrorObject } from 'ajv';
import { ValidationError } from '../types/errors';

/**
 * Convert AJV errors to application ValidationErrors
 */
export function convertAjvErrors(
  errors: ErrorObject[] | null | undefined
): ValidationError[] {
  if (!errors) return [];

  return errors.map((error) => {
    const path = error.instancePath || '/';
    const message = createUserFriendlyMessage(error);
    const suggestions = getSuggestions(error);

    return new ValidationError(message, path, suggestions);
  });
}

/**
 * Create user-friendly error messages
 */
function createUserFriendlyMessage(error: ErrorObject): string {
  const { keyword, params, message } = error;

  switch (keyword) {
    case 'required':
      return `Missing required property: ${params.missingProperty}`;

    case 'type':
      return `Expected type ${params.type}, but got ${params.actualType}`;

    case 'enum':
      return `Value must be one of: ${params.allowedValues.join(', ')}`;

    case 'minLength':
      return `Value must be at least ${params.limit} characters`;

    case 'maxLength':
      return `Value must be at most ${params.limit} characters`;

    case 'pattern':
      return `Value must match pattern: ${params.pattern}`;

    case 'additionalProperties':
      return `Unexpected property: ${params.additionalProperty}`;

    default:
      return message || 'Validation error';
  }
}

/**
 * Get suggestions for fixing errors
 */
function getSuggestions(error: ErrorObject): string[] | undefined {
  const { keyword, params } = error;

  switch (keyword) {
    case 'enum':
      return params.allowedValues.map(
        (value: string) => `Try using "${value}"`
      );

    case 'required':
      return [`Add the "${params.missingProperty}" property`];

    case 'additionalProperties':
      return [
        `Remove the "${params.additionalProperty}" property`,
        'Check for typos in property name',
      ];

    default:
      return undefined;
  }
}
```

---

## Graceful Degradation Strategies

### Strategy 1: Feature Flags

```typescript
// src/hooks/use-feature-safely.ts
import React from 'react';
import { FeatureErrorBoundary } from '../components/error-boundaries';

/**
 * Wrap features with error boundaries and fallbacks
 */
export function useFeatureSafely<T>(
  featureName: string,
  featureHook: () => T,
  fallbackValue: T
): T {
  const [hasError, setHasError] = React.useState(false);

  React.useEffect(() => {
    // Reset error state when feature changes
    setHasError(false);
  }, [featureName]);

  if (hasError) {
    return fallbackValue;
  }

  try {
    return featureHook();
  } catch (error) {
    console.error(`Feature ${featureName} failed:`, error);
    setHasError(true);
    return fallbackValue;
  }
}

// Usage
function MyComponent() {
  const data = useFeatureSafely(
    'schema-editor',
    () => useSchemaEditor(),
    { schemas: [], isLoading: false }
  );

  return <div>{/* Use data */}</div>;
}
```

### Strategy 2: Progressive Enhancement

```typescript
// src/components/openapi-editor.tsx
import React, { Suspense } from 'react';
import { ComponentErrorBoundary } from './error-boundaries';

// Lazy load advanced features
const AdvancedSchemaEditor = React.lazy(
  () => import('./advanced-schema-editor')
);

const SimpleSchemaEditor = React.lazy(
  () => import('./simple-schema-editor')
);

export function SchemaEditor() {
  const [useAdvanced, setUseAdvanced] = React.useState(true);

  return (
    <ComponentErrorBoundary
      fallback={(error) => {
        // If advanced editor fails, fall back to simple editor
        if (useAdvanced) {
          console.warn('Advanced editor failed, using simple editor');
          setUseAdvanced(false);
          return null;
        }
        
        // If simple editor also fails, show error
        return <div>Schema editor unavailable</div>;
      }}
    >
      <Suspense fallback={<div>Loading editor...</div>}>
        {useAdvanced ? <AdvancedSchemaEditor /> : <SimpleSchemaEditor />}
      </Suspense>
    </ComponentErrorBoundary>
  );
}
```

### Strategy 3: Partial Rendering

```typescript
// src/components/openapi-document.tsx
import React from 'react';
import { ComponentErrorBoundary } from './error-boundaries';

export function OpenAPIDocument() {
  return (
    <div className="document">
      {/* Info section with error boundary */}
      <ComponentErrorBoundary
        fallback={() => <div>Info section unavailable</div>}
      >
        <InfoSection />
      </ComponentErrorBoundary>

      {/* Paths section with error boundary */}
      <ComponentErrorBoundary
        fallback={() => <div>Paths section unavailable</div>}
      >
        <PathsSection />
      </ComponentErrorBoundary>

      {/* Components section with error boundary */}
      <ComponentErrorBoundary
        fallback={() => <div>Components section unavailable</div>}
      >
        <ComponentsSection />
      </ComponentErrorBoundary>
    </div>
  );
}
```

---

## Error Logging & Monitoring

```typescript
// src/services/error-logger.ts

interface ErrorLog {
  timestamp: Date;
  error: Error;
  context?: Record<string, any>;
  userAction?: string;
}

/**
 * Centralized error logging service
 */
export class ErrorLogger {
  private logs: ErrorLog[] = [];
  private maxLogs = 100;

  /**
   * Log an error with context
   */
  log(error: Error, context?: Record<string, any>, userAction?: string): void {
    const log: ErrorLog = {
      timestamp: new Date(),
      error,
      context,
      userAction,
    };

    this.logs.push(log);

    // Keep only last N logs
    if (this.logs.length > this.maxLogs) {
      this.logs.shift();
    }

    // Log to console in development
    if (process.env.NODE_ENV === 'development') {
      console.group('🔴 Error Logged');
      console.error('Error:', error);
      console.log('Context:', context);
      console.log('User Action:', userAction);
      console.groupEnd();
    }

    // Send to VSCode extension
    this.sendToExtension(log);
  }

  /**
   * Get recent error logs
   */
  getRecentLogs(count = 10): ErrorLog[] {
    return this.logs.slice(-count);
  }

  /**
   * Export logs for debugging
   */
  exportLogs(): string {
    return JSON.stringify(this.logs, null, 2);
  }

  /**
   * Send error to VSCode extension for telemetry
   */
  private sendToExtension(log: ErrorLog): void {
    if (typeof acquireVsCodeApi === 'function') {
      const vscode = acquireVsCodeApi();
      vscode.postMessage({
        command: 'logError',
        log: {
          timestamp: log.timestamp.toISOString(),
          message: log.error.message,
          stack: log.error.stack,
          context: log.context,
          userAction: log.userAction,
        },
      });
    }
  }
}

export const errorLogger = new ErrorLogger();
```

---

## Recovery Mechanisms

### Auto-Save on Error

```typescript
// src/hooks/use-auto-save-on-error.ts
import { useEffect } from 'react';
import { useAppSelector } from '../store';

/**
 * Automatically save state when error occurs
 */
export function useAutoSaveOnError() {
  const document = useAppSelector((state) => state.openapi.document);

  useEffect(() => {
    // Save to localStorage on error
    window.addEventListener('error', () => {
      try {
        localStorage.setItem(
          'dapa-emergency-save',
          JSON.stringify({
            document,
            timestamp: new Date().toISOString(),
          })
        );
        console.log('Emergency save completed');
      } catch (error) {
        console.error('Emergency save failed:', error);
      }
    });

    // Save on unhandled promise rejection
    window.addEventListener('unhandledrejection', () => {
      try {
        localStorage.setItem(
          'dapa-emergency-save',
          JSON.stringify({
            document,
            timestamp: new Date().toISOString(),
          })
        );
      } catch (error) {
        console.error('Emergency save failed:', error);
      }
    });
  }, [document]);
}
```

### Recovery UI

```typescript
// src/components/recovery-prompt.tsx
import React from 'react';

export function RecoveryPrompt() {
  const [hasRecoveryData, setHasRecoveryData] = React.useState(false);

  React.useEffect(() => {
    const saved = localStorage.getItem('dapa-emergency-save');
    setHasRecoveryData(!!saved);
  }, []);

  const handleRecover = () => {
    const saved = localStorage.getItem('dapa-emergency-save');
    if (saved) {
      const { document, timestamp } = JSON.parse(saved);
      // Restore document to Redux
      store.dispatch(setDocument(document));
      // Clear recovery data
      localStorage.removeItem('dapa-emergency-save');
      setHasRecoveryData(false);
    }
  };

  const handleDiscard = () => {
    localStorage.removeItem('dapa-emergency-save');
    setHasRecoveryData(false);
  };

  if (!hasRecoveryData) return null;

  return (
    <div className="recovery-prompt">
      <h3>⚠️ Unsaved Changes Detected</h3>
      <p>We found unsaved changes from a previous session.</p>
      <div className="actions">
        <button onClick={handleRecover} className="primary">
          Recover Changes
        </button>
        <button onClick={handleDiscard}>Discard</button>
      </div>
    </div>
  );
}
```

---

## Best Practices Summary

### ✅ DO

- **Use error boundaries** at multiple levels (root, feature, component)
- **Report to VSCode** Problems API for validation errors
- **Provide fallbacks** for every error boundary
- **Log errors** with context for debugging
- **Auto-save** on critical errors
- **Show recovery** options to users
- **Degrade gracefully** - never crash the entire app
- **Test error scenarios** regularly

### ❌ DON'T

- **Don't let errors propagate** to the root without handling
- **Don't crash** the entire extension on single component failure
- **Don't ignore** validation errors
- **Don't lose user data** on errors
- **Don't show technical** stack traces to users
- **Don't skip logging** for "small" errors

### 🎯 Key Benefits

1. **Resilience**: App continues working even when parts fail
2. **Better UX**: Users see helpful errors, not crashes
3. **VSCode Integration**: Errors appear in Problems panel naturally
4. **Data Safety**: Auto-save prevents data loss
5. **Debuggability**: Comprehensive error logging
6. **Recovery**: Users can recover from errors gracefully

---

This comprehensive error handling approach ensures the DAPA extension is robust, user-friendly, and professionally integrated with VSCode's error reporting systems.
