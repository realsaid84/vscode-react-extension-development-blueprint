# API Layer & Integration

> **Efficient patterns for API integration, file I/O, and data fetching**

## Table of Contents
- [HTTP Proxy Setup](#http-proxy-setup)
- [API Client Architecture](#api-client-architecture)
- [File I/O Patterns](#file-io-patterns)
- [Request Interceptors](#request-interceptors)
- [Caching Strategy](#caching-strategy)

---

## HTTP Proxy Setup

### Development Proxy Middleware

```javascript
// setupProxy.js (Create React App)
const { createProxyMiddleware } = require('http-proxy-middleware');

module.exports = function(app) {
  // Proxy API requests
  app.use(
    '/api',
    createProxyMiddleware({
      target: 'https://api.example.com',
      changeOrigin: true,
      pathRewrite: {
        '^/api': '', // Remove /api prefix
      },
      onProxyReq: (proxyReq, req, res) => {
        // Add auth headers
        const token = process.env.API_TOKEN;
        if (token) {
          proxyReq.setHeader('Authorization', `Bearer ${token}`);
        }
      },
      onError: (err, req, res) => {
        console.error('Proxy error:', err);
        res.status(500).send('Proxy error');
      },
    })
  );

  // Proxy schema registry
  app.use(
    '/registry',
    createProxyMiddleware({
      target: 'https://schema-registry.example.com',
      changeOrigin: true,
    })
  );
};
```

### Webpack Dev Server Proxy

```javascript
// webpack.config.js
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: 'https://api.example.com',
        changeOrigin: true,
        pathRewrite: { '^/api': '' },
        secure: false,
      },
      '/registry': {
        target: 'https://schema-registry.example.com',
        changeOrigin: true,
      },
    },
  },
};
```

---

## API Client Architecture

### Base API Client

```typescript
// src/api/client.ts
import type { OpenAPIDocument } from '../types/openapi31.types';

interface RequestConfig extends RequestInit {
  params?: Record<string, string>;
}

class ApiError extends Error {
  constructor(
    message: string,
    public status: number,
    public data?: any
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

export class ApiClient {
  constructor(private baseUrl: string = '/api') {}

  private async request<T>(
    endpoint: string,
    config: RequestConfig = {}
  ): Promise<T> {
    const { params, ...fetchConfig } = config;

    // Build URL with query params
    let url = `${this.baseUrl}${endpoint}`;
    if (params) {
      const query = new URLSearchParams(params).toString();
      url += `?${query}`;
    }

    try {
      const response = await fetch(url, {
        ...fetchConfig,
        headers: {
          'Content-Type': 'application/json',
          ...fetchConfig.headers,
        },
      });

      if (!response.ok) {
        const error = await response.json().catch(() => ({}));
        throw new ApiError(
          error.message || 'Request failed',
          response.status,
          error
        );
      }

      // Handle empty responses
      const contentType = response.headers.get('content-type');
      if (!contentType?.includes('application/json')) {
        return undefined as T;
      }

      return response.json();
    } catch (error) {
      if (error instanceof ApiError) throw error;
      throw new ApiError('Network error', 0, error);
    }
  }

  async get<T>(endpoint: string, params?: Record<string, string>): Promise<T> {
    return this.request<T>(endpoint, { method: 'GET', params });
  }

  async post<T, D = unknown>(endpoint: string, data: D): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  async put<T, D = unknown>(endpoint: string, data: D): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }

  async patch<T, D = unknown>(endpoint: string, data: D): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'PATCH',
      body: JSON.stringify(data),
    });
  }

  async delete<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, { method: 'DELETE' });
  }
}

export const apiClient = new ApiClient();
```

### API Hooks

```typescript
// src/api/hooks.ts
import { useState, useEffect } from 'react';
import { apiClient } from './client';

export function useApi<T>(
  endpoint: string,
  options?: {
    skip?: boolean;
    params?: Record<string, string>;
  }
) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(!options?.skip);
  const [error, setError] = useState<Error | null>(null);

  const refetch = async () => {
    try {
      setLoading(true);
      setError(null);
      const result = await apiClient.get<T>(endpoint, options?.params);
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    if (options?.skip) return;
    refetch();
  }, [endpoint, options?.skip, JSON.stringify(options?.params)]);

  return { data, loading, error, refetch };
}

// Usage
function SchemaList() {
  const { data: schemas, loading, error, refetch } = useApi<Schema[]>('/schemas');

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage error={error} onRetry={refetch} />;

  return (
    <ul>
      {schemas?.map((schema) => (
        <li key={schema.id}>{schema.name}</li>
      ))}
    </ul>
  );
}
```

---

## File I/O Patterns

### VSCode File Operations

```typescript
// src/extension/file-operations.ts
import * as vscode from 'vscode';
import * as path from 'path';

export class FileOperations {
  /**
   * Read file as text
   */
  async readFile(uri: vscode.Uri): Promise<string> {
    try {
      const bytes = await vscode.workspace.fs.readFile(uri);
      return Buffer.from(bytes).toString('utf8');
    } catch (error) {
      throw new Error(`Failed to read ${uri.fsPath}: ${error}`);
    }
  }

  /**
   * Write file with backup
   */
  async writeFile(uri: vscode.Uri, content: string): Promise<void> {
    try {
      // Create backup if file exists
      const exists = await this.fileExists(uri);
      if (exists) {
        await this.createBackup(uri);
      }

      // Write file
      await vscode.workspace.fs.writeFile(
        uri,
        Buffer.from(content, 'utf8')
      );
    } catch (error) {
      throw new Error(`Failed to write ${uri.fsPath}: ${error}`);
    }
  }

  /**
   * Read JSON file
   */
  async readJSON<T>(uri: vscode.Uri): Promise<T> {
    const content = await this.readFile(uri);
    try {
      return JSON.parse(content);
    } catch (error) {
      throw new Error(`Invalid JSON in ${uri.fsPath}`);
    }
  }

  /**
   * Write JSON file
   */
  async writeJSON(uri: vscode.Uri, data: any): Promise<void> {
    const content = JSON.stringify(data, null, 2);
    await this.writeFile(uri, content);
  }

  /**
   * Check if file exists
   */
  async fileExists(uri: vscode.Uri): Promise<boolean> {
    try {
      await vscode.workspace.fs.stat(uri);
      return true;
    } catch {
      return false;
    }
  }

  /**
   * Create backup file
   */
  private async createBackup(uri: vscode.Uri): Promise<void> {
    const backupUri = uri.with({
      path: `${uri.path}.backup`,
    });
    await vscode.workspace.fs.copy(uri, backupUri, { overwrite: true });
  }

  /**
   * List files in directory
   */
  async listFiles(
    dirUri: vscode.Uri,
    pattern?: string
  ): Promise<vscode.Uri[]> {
    try {
      const entries = await vscode.workspace.fs.readDirectory(dirUri);
      
      const files = entries
        .filter(([name, type]) => type === vscode.FileType.File)
        .map(([name]) => vscode.Uri.joinPath(dirUri, name));

      if (pattern) {
        const regex = new RegExp(pattern);
        return files.filter((uri) => regex.test(path.basename(uri.fsPath)));
      }

      return files;
    } catch (error) {
      throw new Error(`Failed to list files in ${dirUri.fsPath}: ${error}`);
    }
  }

  /**
   * Watch directory for changes
   */
  watchDirectory(
    dirUri: vscode.Uri,
    callback: (uri: vscode.Uri) => void
  ): vscode.Disposable {
    const pattern = new vscode.RelativePattern(dirUri, '**/*');
    const watcher = vscode.workspace.createFileSystemWatcher(pattern);

    watcher.onDidCreate(callback);
    watcher.onDidChange(callback);
    watcher.onDidDelete(callback);

    return watcher;
  }
}

export const fileOps = new FileOperations();
```

### File Upload Handling

```typescript
// src/webview/components/file-uploader.tsx
import React from 'react';

interface FileUploaderProps {
  accept?: string;
  onUpload: (content: string, filename: string) => void;
}

export function FileUploader({ accept, onUpload }: FileUploaderProps) {
  const [isDragging, setIsDragging] = React.useState(false);
  const inputRef = React.useRef<HTMLInputElement>(null);

  const handleFile = async (file: File) => {
    try {
      const content = await file.text();
      onUpload(content, file.name);
    } catch (error) {
      console.error('Failed to read file:', error);
    }
  };

  const handleDrop = (e: React.DragEvent) => {
    e.preventDefault();
    setIsDragging(false);

    const file = e.dataTransfer.files[0];
    if (file) handleFile(file);
  };

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (file) handleFile(file);
  };

  return (
    <div
      className={`file-uploader ${isDragging ? 'dragging' : ''}`}
      onDragOver={(e) => {
        e.preventDefault();
        setIsDragging(true);
      }}
      onDragLeave={() => setIsDragging(false)}
      onDrop={handleDrop}
      onClick={() => inputRef.current?.click()}
    >
      <input
        ref={inputRef}
        type="file"
        accept={accept}
        onChange={handleChange}
        style={{ display: 'none' }}
      />
      <p>Drop file here or click to browse</p>
    </div>
  );
}
```

---

## Request Interceptors

### Authentication Interceptor

```typescript
// src/api/interceptors.ts
type Interceptor = (config: RequestInit) => RequestInit | Promise<RequestInit>;

class InterceptorManager {
  private interceptors: Interceptor[] = [];

  use(interceptor: Interceptor): () => void {
    this.interceptors.push(interceptor);
    return () => {
      const index = this.interceptors.indexOf(interceptor);
      if (index > -1) {
        this.interceptors.splice(index, 1);
      }
    };
  }

  async execute(config: RequestInit): Promise<RequestInit> {
    let result = config;
    for (const interceptor of this.interceptors) {
      result = await interceptor(result);
    }
    return result;
  }
}

export const requestInterceptors = new InterceptorManager();

// Auth interceptor
requestInterceptors.use(async (config) => {
  const token = await getAuthToken();
  
  return {
    ...config,
    headers: {
      ...config.headers,
      Authorization: `Bearer ${token}`,
    },
  };
});

// Logging interceptor
requestInterceptors.use((config) => {
  console.log('Request:', config);
  return config;
});

// Apply interceptors in client
class ApiClient {
  private async request<T>(endpoint: string, config: RequestConfig): Promise<T> {
    const interceptedConfig = await requestInterceptors.execute(config);
    // ... rest of request logic
  }
}
```

### Retry Logic

```typescript
// src/api/retry.ts
interface RetryOptions {
  maxRetries?: number;
  delay?: number;
  backoff?: boolean;
}

export async function withRetry<T>(
  fn: () => Promise<T>,
  options: RetryOptions = {}
): Promise<T> {
  const { maxRetries = 3, delay = 1000, backoff = true } = options;

  let lastError: Error;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error instanceof Error ? error : new Error('Unknown error');

      if (attempt < maxRetries) {
        const waitTime = backoff ? delay * Math.pow(2, attempt) : delay;
        await new Promise((resolve) => setTimeout(resolve, waitTime));
      }
    }
  }

  throw lastError!;
}

// Usage
const data = await withRetry(
  () => apiClient.get('/schemas'),
  { maxRetries: 3, backoff: true }
);
```

---

## Caching Strategy

### In-Memory Cache

```typescript
// src/api/cache.ts
interface CacheEntry<T> {
  data: T;
  timestamp: number;
}

export class Cache<T> {
  private cache = new Map<string, CacheEntry<T>>();

  constructor(private ttl: number = 5 * 60 * 1000) {} // 5 minutes default

  set(key: string, data: T): void {
    this.cache.set(key, {
      data,
      timestamp: Date.now(),
    });
  }

  get(key: string): T | null {
    const entry = this.cache.get(key);
    
    if (!entry) return null;

    // Check if expired
    if (Date.now() - entry.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }

    return entry.data;
  }

  clear(): void {
    this.cache.clear();
  }

  invalidate(keyPattern: RegExp): void {
    for (const key of this.cache.keys()) {
      if (keyPattern.test(key)) {
        this.cache.delete(key);
      }
    }
  }
}

// Usage
const schemaCache = new Cache<Schema[]>(10 * 60 * 1000); // 10 minutes

async function getSchemas(): Promise<Schema[]> {
  const cached = schemaCache.get('schemas');
  if (cached) return cached;

  const data = await apiClient.get<Schema[]>('/schemas');
  schemaCache.set('schemas', data);
  return data;
}
```

### LocalStorage Persistence

```typescript
// src/api/persistent-cache.ts
export class PersistentCache<T> {
  constructor(
    private key: string,
    private ttl: number = 24 * 60 * 60 * 1000 // 24 hours
  ) {}

  set(data: T): void {
    const entry = {
      data,
      timestamp: Date.now(),
    };
    localStorage.setItem(this.key, JSON.stringify(entry));
  }

  get(): T | null {
    const item = localStorage.getItem(this.key);
    if (!item) return null;

    try {
      const entry = JSON.parse(item);
      
      if (Date.now() - entry.timestamp > this.ttl) {
        localStorage.removeItem(this.key);
        return null;
      }

      return entry.data;
    } catch {
      localStorage.removeItem(this.key);
      return null;
    }
  }

  clear(): void {
    localStorage.removeItem(this.key);
  }
}
```

---

## Best Practices Summary

### ✅ DO

- **Use http-proxy** in development
- **Implement retry logic** for failed requests
- **Cache responses** appropriately
- **Handle file I/O** errors gracefully
- **Use interceptors** for cross-cutting concerns
- **Validate responses** before using
- **Create backups** before writing files

### ❌ DON'T

- **Don't hardcode** API URLs
- **Don't ignore** network errors
- **Don't cache** forever
- **Don't trust** file contents
- **Don't block** on I/O operations
- **Don't forget** to handle large files
- **Don't expose** sensitive data

### 🎯 Key Principles

1. **Resilience**: Retry failed requests, handle errors
2. **Performance**: Cache responses, batch requests
3. **Security**: Validate inputs, sanitize outputs
4. **Reliability**: Create backups, verify writes
5. **Observability**: Log requests, track errors

---

This API layer provides efficient, reliable data fetching and file handling patterns for the DAPA extension.
