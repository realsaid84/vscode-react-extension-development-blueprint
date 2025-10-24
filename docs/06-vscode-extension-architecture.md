# VSCode Extension Architecture

> **Practical patterns for building React-based VSCode extensions**

## Table of Contents
- [Extension Lifecycle](#extension-lifecycle)
- [Webview Integration](#webview-integration)
- [Message Passing](#message-passing)
- [File System Access](#file-system-access)
- [Extension API Patterns](#extension-api-patterns)

---

## Extension Lifecycle

### Activation

```typescript
// src/extension.ts
import * as vscode from 'vscode';
import { DapaWebviewProvider } from './webview-provider';
import { DiagnosticsManager } from './diagnostics-manager';

/**
 * Extension activation entry point
 */
export function activate(context: vscode.ExtensionContext) {
  console.log('DAPA extension activating...');

  // Initialize services
  const diagnostics = new DiagnosticsManager(context);
  const webviewProvider = new DapaWebviewProvider(context, diagnostics);

  // Register webview provider
  context.subscriptions.push(
    vscode.window.registerWebviewViewProvider(
      'dapa.editorView',
      webviewProvider,
      { webviewOptions: { retainContextWhenHidden: true } }
    )
  );

  // Register commands
  context.subscriptions.push(
    vscode.commands.registerCommand('dapa.openEditor', () => {
      vscode.commands.executeCommand('workbench.view.extension.dapa');
    })
  );

  context.subscriptions.push(
    vscode.commands.registerCommand('dapa.newDocument', async () => {
      const doc = await createNewDocument();
      webviewProvider.updateDocument(doc);
    })
  );

  console.log('DAPA extension activated');
}

export function deactivate() {
  console.log('DAPA extension deactivated');
}
```

### Package.json Configuration

```json
{
  "name": "dapa-vscode-extension",
  "displayName": "DAPA API Authoring",
  "description": "No-code API specification authoring",
  "version": "1.0.0",
  "engines": {
    "vscode": "^1.80.0"
  },
  "categories": ["Other"],
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
        "title": "DAPA: Open Editor"
      },
      {
        "command": "dapa.newDocument",
        "title": "DAPA: New Document"
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
  }
}
```

---

## Webview Integration

### Webview Provider

```typescript
// src/webview-provider.ts
import * as vscode from 'vscode';
import * as path from 'path';

export class DapaWebviewProvider implements vscode.WebviewViewProvider {
  private view?: vscode.WebviewView;

  constructor(
    private context: vscode.ExtensionContext,
    private diagnostics: DiagnosticsManager
  ) {}

  resolveWebviewView(webviewView: vscode.WebviewView) {
    this.view = webviewView;

    // Configure webview
    webviewView.webview.options = {
      enableScripts: true,
      localResourceRoots: [
        vscode.Uri.joinPath(this.context.extensionUri, 'dist'),
      ],
    };

    // Set HTML content
    webviewView.webview.html = this.getHtmlContent(webviewView.webview);

    // Handle messages from webview
    webviewView.webview.onDidReceiveMessage(
      (message) => this.handleMessage(message),
      undefined,
      this.context.subscriptions
    );

    // Send initial state
    this.sendMessage({ command: 'initialize', theme: this.getTheme() });
  }

  private getHtmlContent(webview: vscode.Webview): string {
    const scriptUri = webview.asWebviewUri(
      vscode.Uri.joinPath(this.context.extensionUri, 'dist', 'webview.js')
    );

    const styleUri = webview.asWebviewUri(
      vscode.Uri.joinPath(this.context.extensionUri, 'dist', 'webview.css')
    );

    const nonce = this.getNonce();

    return `<!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta http-equiv="Content-Security-Policy" 
            content="default-src 'none'; 
                     style-src ${webview.cspSource} 'unsafe-inline'; 
                     script-src 'nonce-${nonce}';">
      <link href="${styleUri}" rel="stylesheet">
      <title>DAPA Editor</title>
    </head>
    <body>
      <div id="root"></div>
      <script nonce="${nonce}" src="${scriptUri}"></script>
    </body>
    </html>`;
  }

  private handleMessage(message: any) {
    switch (message.command) {
      case 'save':
        this.saveDocument(message.document);
        break;
      case 'validationErrors':
        this.diagnostics.reportValidationErrors(
          vscode.window.activeTextEditor!.document.uri,
          message.errors
        );
        break;
    }
  }

  private sendMessage(message: any) {
    this.view?.webview.postMessage(message);
  }

  private getNonce(): string {
    let text = '';
    const possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    for (let i = 0; i < 32; i++) {
      text += possible.charAt(Math.floor(Math.random() * possible.length));
    }
    return text;
  }

  private getTheme(): string {
    const config = vscode.workspace.getConfiguration('dapa');
    const theme = config.get<string>('theme', 'auto');
    
    if (theme === 'auto') {
      return vscode.window.activeColorTheme.kind === vscode.ColorThemeKind.Dark
        ? 'dark'
        : 'light';
    }
    
    return theme;
  }

  private async saveDocument(document: any) {
    const uri = vscode.window.activeTextEditor?.document.uri;
    if (!uri) return;

    const content = JSON.stringify(document, null, 2);
    await vscode.workspace.fs.writeFile(uri, Buffer.from(content, 'utf8'));
    
    vscode.window.showInformationMessage('Document saved successfully');
  }
}
```

### React Entry Point

```typescript
// src/webview/index.tsx
import React from 'react';
import { createRoot } from 'react-dom/client';
import { Provider } from 'react-redux';
import { store } from './store';
import { App } from './App';

// VSCode API reference
declare function acquireVsCodeApi(): any;
const vscode = acquireVsCodeApi();

// Make vscode API available globally
(window as any).vscode = vscode;

// Render app
const container = document.getElementById('root')!;
const root = createRoot(container);

root.render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

---

## Message Passing

### Type-Safe Message Protocol

```typescript
// src/types/messages.ts

// Webview → Extension messages
export type WebviewMessage =
  | { command: 'save'; document: OpenAPIDocument }
  | { command: 'validationErrors'; errors: ValidationError[] }
  | { command: 'ready' }
  | { command: 'log'; level: 'info' | 'warn' | 'error'; message: string };

// Extension → Webview messages
export type ExtensionMessage =
  | { command: 'initialize'; theme: string; document?: OpenAPIDocument }
  | { command: 'loadDocument'; document: OpenAPIDocument }
  | { command: 'themeChanged'; theme: string }
  | { command: 'save' };
```

### Message Handler Hook

```typescript
// src/webview/hooks/use-vscode-messages.ts
import { useEffect } from 'react';
import type { ExtensionMessage } from '../../types/messages';
import { useAppDispatch } from '../store';
import { setDocument, setTheme } from '../store/slices/ui.slice';

export function useVSCodeMessages() {
  const dispatch = useAppDispatch();

  useEffect(() => {
    const handleMessage = (event: MessageEvent<ExtensionMessage>) => {
      const message = event.data;

      switch (message.command) {
        case 'initialize':
          dispatch(setTheme(message.theme));
          if (message.document) {
            dispatch(setDocument(message.document));
          }
          break;

        case 'loadDocument':
          dispatch(setDocument(message.document));
          break;

        case 'themeChanged':
          dispatch(setTheme(message.theme));
          break;

        case 'save':
          handleSave();
          break;
      }
    };

    window.addEventListener('message', handleMessage);
    
    // Signal ready
    window.vscode?.postMessage({ command: 'ready' });

    return () => window.removeEventListener('message', handleMessage);
  }, [dispatch]);

  const handleSave = () => {
    const state = store.getState();
    window.vscode?.postMessage({
      command: 'save',
      document: state.openapi.document,
    });
  };
}
```

---

## File System Access

### Reading Files

```typescript
// src/extension/file-manager.ts
import * as vscode from 'vscode';

export class FileManager {
  /**
   * Read document from workspace
   */
  async readDocument(uri: vscode.Uri): Promise<string> {
    try {
      const bytes = await vscode.workspace.fs.readFile(uri);
      return Buffer.from(bytes).toString('utf8');
    } catch (error) {
      throw new Error(`Failed to read file: ${error}`);
    }
  }

  /**
   * Write document to workspace
   */
  async writeDocument(uri: vscode.Uri, content: string): Promise<void> {
    try {
      await vscode.workspace.fs.writeFile(
        uri,
        Buffer.from(content, 'utf8')
      );
    } catch (error) {
      throw new Error(`Failed to write file: ${error}`);
    }
  }

  /**
   * Watch file for changes
   */
  watchFile(
    uri: vscode.Uri,
    callback: (content: string) => void
  ): vscode.Disposable {
    const watcher = vscode.workspace.createFileSystemWatcher(
      new vscode.RelativePattern(uri, '*')
    );

    watcher.onDidChange(async () => {
      const content = await this.readDocument(uri);
      callback(content);
    });

    return watcher;
  }

  /**
   * Get workspace folder
   */
  getWorkspaceFolder(): vscode.Uri | undefined {
    return vscode.workspace.workspaceFolders?.[0]?.uri;
  }
}
```

---

## Extension API Patterns

### Configuration Management

```typescript
// src/extension/config-manager.ts
import * as vscode from 'vscode';

export class ConfigManager {
  private config: vscode.WorkspaceConfiguration;

  constructor() {
    this.config = vscode.workspace.getConfiguration('dapa');
  }

  get<T>(key: string, defaultValue: T): T {
    return this.config.get(key, defaultValue);
  }

  async set(key: string, value: any): Promise<void> {
    await this.config.update(key, value, vscode.ConfigurationTarget.Global);
  }

  onChange(callback: () => void): vscode.Disposable {
    return vscode.workspace.onDidChangeConfiguration((e) => {
      if (e.affectsConfiguration('dapa')) {
        callback();
      }
    });
  }
}
```

### Command Registration

```typescript
// src/extension/commands.ts
import * as vscode from 'vscode';

export function registerCommands(
  context: vscode.ExtensionContext,
  webviewProvider: DapaWebviewProvider
): void {
  // Open editor
  context.subscriptions.push(
    vscode.commands.registerCommand('dapa.openEditor', () => {
      vscode.commands.executeCommand('workbench.view.extension.dapa');
    })
  );

  // New document
  context.subscriptions.push(
    vscode.commands.registerCommand('dapa.newDocument', async () => {
      const docType = await vscode.window.showQuickPick(
        ['OpenAPI 3.0', 'OpenAPI 3.1', 'AsyncAPI 2.6'],
        { placeHolder: 'Select document type' }
      );

      if (docType) {
        const template = getTemplate(docType);
        webviewProvider.loadDocument(template);
      }
    })
  );

  // Export document
  context.subscriptions.push(
    vscode.commands.registerCommand('dapa.export', async () => {
      const format = await vscode.window.showQuickPick(
        ['JSON', 'YAML'],
        { placeHolder: 'Select format' }
      );

      if (format) {
        await exportDocument(format);
      }
    })
  );
}

function getTemplate(type: string): any {
  switch (type) {
    case 'OpenAPI 3.1':
      return {
        openapi: '3.1.0',
        info: { title: 'New API', version: '1.0.0' },
        paths: {},
      };
    default:
      return {};
  }
}
```

### Status Bar Integration

```typescript
// src/extension/status-bar.ts
import * as vscode from 'vscode';

export class StatusBarManager {
  private item: vscode.StatusBarItem;

  constructor(context: vscode.ExtensionContext) {
    this.item = vscode.window.createStatusBarItem(
      vscode.StatusBarAlignment.Right,
      100
    );
    
    this.item.command = 'dapa.openEditor';
    context.subscriptions.push(this.item);
    
    this.updateStatus('Ready');
  }

  updateStatus(status: string) {
    this.item.text = `$(file-code) DAPA: ${status}`;
    this.item.show();
  }

  showProgress(message: string) {
    this.item.text = `$(sync~spin) ${message}`;
  }

  hide() {
    this.item.hide();
  }
}
```

---

## Best Practices Summary

### ✅ DO

- **Use webview providers** for persistent UI
- **Enable `retainContextWhenHidden`** to preserve state
- **Implement type-safe** message passing
- **Use CSP** (Content Security Policy) properly
- **Handle file I/O** with try-catch
- **Register disposables** in context.subscriptions
- **Validate messages** from webview

### ❌ DON'T

- **Don't trust webview messages** - always validate
- **Don't block extension host** with long operations
- **Don't forget nonce** for inline scripts
- **Don't skip error handling** in file operations
- **Don't store sensitive data** in webview
- **Don't use `eval()`** or unsafe HTML

### 🎯 Key Principles

1. **Separation of Concerns**: Extension host handles VSCode API, webview handles UI
2. **Type Safety**: Use TypeScript for message protocols
3. **Error Boundaries**: Isolate failures
4. **Performance**: Keep extension host responsive
5. **Security**: Use CSP and validate inputs

---

This architecture provides a solid, secure foundation for building React-based VSCode extensions without over-engineering.
