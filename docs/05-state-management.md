# Advanced State Management

> **A superior approach to state management in React applications, specifically optimized for VSCode extensions**

## Table of Contents
- [Overview](#overview)
- [State Architecture](#state-architecture)
- [Redux Toolkit Patterns](#redux-toolkit-patterns)
- [Local State with Zustand](#local-state-with-zustand)
- [State Normalization](#state-normalization)
- [RTK Query for Data Fetching](#rtk-query-for-data-fetching)
- [VSCode Extension State Bridge](#vscode-extension-state-bridge)
- [Performance Optimization](#performance-optimization)

---

## Overview

### Why This Approach is Superior to Bulletproof React

| Aspect | Bulletproof React | Our Approach | Why Better |
|--------|-------------------|--------------|------------|
| **Global State** | Context + Zustand mix | Redux Toolkit + RTK Query | Better DevTools, time-travel debugging, persistence |
| **Server State** | React Query | RTK Query | Integrated caching, type inference, less boilerplate |
| **Local State** | useState hooks | Zustand stores | Better performance, less re-renders, easier testing |
| **Normalization** | Manual | @reduxjs/toolkit entities | Automatic CRUD, optimistic updates, relationships |
| **DevTools** | Limited | Redux DevTools + Extension API | Full state inspection, action replay, VSCode integration |
| **Persistence** | Manual | redux-persist + VSCode globalState | Automatic, versioned, cross-window sync |

### State Classification

```typescript
/**
 * State is categorized into 4 types for optimal management
 */

// 1. SPECIFICATION STATE - Persisted document being authored
interface SpecificationState {
  openapi: OpenAPIDocument | null;    // Current OpenAPI doc
  asyncapi: AsyncAPIDocument | null;  // Current AsyncAPI doc
  taxilang: TaxiDocument | null;      // Current DSL doc
}

// 2. UI STATE - Ephemeral UI interactions
interface UIState {
  selectedPath: string | null;
  expandedSections: string[];
  activeTab: string;
  sidebarCollapsed: boolean;
}

// 3. SERVER STATE - Data from external APIs
interface ServerState {
  schemas: Schema[];        // Fetched from registry
  templates: Template[];    // Fetched from repository
}

// 4. DERIVED STATE - Computed from other state
interface DerivedState {
  validationErrors: ValidationError[];  // Computed from spec
  isDirty: boolean;                     // Computed from history
  canUndo: boolean;                     // Computed from past states
}
```

---

## State Architecture

### The Three-Layer Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                    LAYER 1: Global State                     │
│                   (Redux Toolkit Store)                      │
├─────────────────────────────────────────────────────────────┤
│  • Specification documents (OpenAPI, AsyncAPI, etc.)        │
│  • Normalized entities (schemas, paths, operations)         │
│  • Undo/redo history                                        │
│  • Global UI preferences                                    │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                LAYER 2: Feature-Local State                  │
│                    (Zustand Stores)                          │
├─────────────────────────────────────────────────────────────┤
│  • Component-specific UI state                              │
│  • Form state (uncontrolled)                                │
│  • Modal/drawer open states                                 │
│  • Temporary selections                                     │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  LAYER 3: Server State                       │
│                     (RTK Query)                              │
├─────────────────────────────────────────────────────────────┤
│  • API data fetching                                        │
│  • Automatic caching                                        │
│  • Background refetching                                    │
│  • Optimistic updates                                       │
└─────────────────────────────────────────────────────────────┘
```

### Store Configuration

```typescript
// src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { setupListeners } from '@reduxjs/toolkit/query';
import {
  persistStore,
  persistReducer,
  FLUSH,
  REHYDRATE,
  PAUSE,
  PERSIST,
  PURGE,
  REGISTER,
} from 'redux-persist';
import storage from 'redux-persist/lib/storage';

// Slices
import openapiReducer from './slices/openapi.slice';
import asyncapiReducer from './slices/asyncapi.slice';
import uiReducer from './slices/ui.slice';

// RTK Query APIs
import { schemaApi } from './api/schema.api';
import { templateApi } from './api/template.api';

/**
 * Redux persist configuration
 * Only persist specification state, not UI state
 */
const persistConfig = {
  key: 'dapa-root',
  version: 1,
  storage,
  whitelist: ['openapi', 'asyncapi', 'taxilang'], // Only persist these
  blacklist: ['ui', schemaApi.reducerPath, templateApi.reducerPath],
};

const rootReducer = {
  openapi: openapiReducer,
  asyncapi: asyncapiReducer,
  ui: uiReducer,
  [schemaApi.reducerPath]: schemaApi.reducer,
  [templateApi.reducerPath]: templateApi.reducer,
};

const persistedReducer = persistReducer(persistConfig, rootReducer);

/**
 * Configure store with middleware
 */
export const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: [FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER],
      },
    })
      .concat(schemaApi.middleware)
      .concat(templateApi.middleware),
  devTools: process.env.NODE_ENV !== 'production',
});

// Enable RTK Query features like refetchOnFocus
setupListeners(store.dispatch);

export const persistor = persistStore(store);

// Types
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// Typed hooks
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

---

## Redux Toolkit Patterns

### Normalized Entity Pattern

```typescript
// src/store/slices/openapi.slice.ts
import { createSlice, createEntityAdapter, PayloadAction } from '@reduxjs/toolkit';
import type { PathItem, Schema } from '../../types/openapi31.types';

/**
 * Entity adapter for normalized path storage
 * Automatically provides CRUD reducers
 */
const pathsAdapter = createEntityAdapter<PathItem & { id: string }>({
  selectId: (path) => path.id,
  sortComparer: (a, b) => a.id.localeCompare(b.id),
});

const schemasAdapter = createEntityAdapter<Schema & { id: string }>({
  selectId: (schema) => schema.id,
});

interface OpenAPIState {
  document: {
    openapi: string;
    info: Info;
  } | null;
  paths: ReturnType<typeof pathsAdapter.getInitialState>;
  schemas: ReturnType<typeof schemasAdapter.getInitialState>;
  selectedPath: string | null;
  history: {
    past: any[];
    future: any[];
  };
}

const initialState: OpenAPIState = {
  document: null,
  paths: pathsAdapter.getInitialState(),
  schemas: schemasAdapter.getInitialState(),
  selectedPath: null,
  history: {
    past: [],
    future: [],
  },
};

export const openapiSlice = createSlice({
  name: 'openapi',
  initialState,
  reducers: {
    // Document operations
    setDocument(state, action: PayloadAction<OpenAPIDocument>) {
      state.document = {
        openapi: action.payload.openapi,
        info: action.payload.info,
      };
      
      // Normalize paths into entity adapter
      const pathsWithIds = Object.entries(action.payload.paths || {}).map(
        ([path, item]) => ({ id: path, ...item })
      );
      pathsAdapter.setAll(state.paths, pathsWithIds);
      
      // Normalize schemas
      const schemasWithIds = Object.entries(
        action.payload.components?.schemas || {}
      ).map(([name, schema]) => ({ id: name, ...schema }));
      schemasAdapter.setAll(state.schemas, schemasWithIds);
    },
    
    // Path operations using adapter methods
    addPath: pathsAdapter.addOne,
    updatePath: pathsAdapter.updateOne,
    removePath: pathsAdapter.removeOne,
    
    // Schema operations
    addSchema: schemasAdapter.addOne,
    updateSchema: schemasAdapter.updateOne,
    removeSchema: schemasAdapter.removeOne,
    
    // Undo/Redo with history
    undo(state) {
      if (state.history.past.length === 0) return;
      
      const previous = state.history.past[state.history.past.length - 1];
      const newPast = state.history.past.slice(0, -1);
      
      state.history.future = [
        { paths: state.paths, schemas: state.schemas },
        ...state.history.future,
      ];
      state.history.past = newPast;
      
      state.paths = previous.paths;
      state.schemas = previous.schemas;
    },
    
    redo(state) {
      if (state.history.future.length === 0) return;
      
      const next = state.history.future[0];
      const newFuture = state.history.future.slice(1);
      
      state.history.past = [
        ...state.history.past,
        { paths: state.paths, schemas: state.schemas },
      ];
      state.history.future = newFuture;
      
      state.paths = next.paths;
      state.schemas = next.schemas;
    },
  },
});

// Export selectors using adapter selectors
export const pathsSelectors = pathsAdapter.getSelectors<RootState>(
  (state) => state.openapi.paths
);

export const schemasSelectors = schemasAdapter.getSelectors<RootState>(
  (state) => state.openapi.schemas
);

// Memoized selectors with reselect
import { createSelector } from '@reduxjs/toolkit';

export const selectAllPaths = createSelector(
  [pathsSelectors.selectAll],
  (paths) => paths
);

export const selectPathsByMethod = createSelector(
  [selectAllPaths, (state: RootState, method: string) => method],
  (paths, method) => paths.filter((path) => path[method] !== undefined)
);

export const selectCanUndo = (state: RootState) =>
  state.openapi.history.past.length > 0;

export const selectCanRedo = (state: RootState) =>
  state.openapi.history.future.length > 0;

export const { setDocument, addPath, updatePath, removePath, undo, redo } =
  openapiSlice.actions;

export default openapiSlice.reducer;
```

---

## Local State with Zustand

For component-specific UI state that doesn't need to be in Redux:

```typescript
// src/stores/editor-ui.store.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface EditorUIState {
  // Sidebar state
  sidebarWidth: number;
  sidebarCollapsed: boolean;
  setSidebarWidth: (width: number) => void;
  toggleSidebar: () => void;
  
  // Panel state
  activePanelId: string | null;
  expandedPanels: Set<string>;
  setActivePanel: (id: string | null) => void;
  togglePanel: (id: string) => void;
  
  // Modal state
  openModals: Set<string>;
  openModal: (id: string) => void;
  closeModal: (id: string) => void;
}

/**
 * Zustand store for local UI state
 * Faster than Redux for frequently changing UI state
 */
export const useEditorUIStore = create<EditorUIState>()(
  devtools(
    persist(
      (set) => ({
        // Initial state
        sidebarWidth: 300,
        sidebarCollapsed: false,
        activePanelId: null,
        expandedPanels: new Set<string>(),
        openModals: new Set<string>(),
        
        // Actions
        setSidebarWidth: (width) => set({ sidebarWidth: width }),
        
        toggleSidebar: () =>
          set((state) => ({ sidebarCollapsed: !state.sidebarCollapsed })),
        
        setActivePanel: (id) => set({ activePanelId: id }),
        
        togglePanel: (id) =>
          set((state) => {
            const newExpanded = new Set(state.expandedPanels);
            if (newExpanded.has(id)) {
              newExpanded.delete(id);
            } else {
              newExpanded.add(id);
            }
            return { expandedPanels: newExpanded };
          }),
        
        openModal: (id) =>
          set((state) => ({
            openModals: new Set(state.openModals).add(id),
          })),
        
        closeModal: (id) =>
          set((state) => {
            const newModals = new Set(state.openModals);
            newModals.delete(id);
            return { openModals: newModals };
          }),
      }),
      {
        name: 'editor-ui-storage',
        partialize: (state) => ({
          sidebarWidth: state.sidebarWidth,
          sidebarCollapsed: state.sidebarCollapsed,
        }),
      }
    ),
    { name: 'EditorUI' }
  )
);

// Usage in components
function Sidebar() {
  const { sidebarWidth, sidebarCollapsed, setSidebarWidth, toggleSidebar } =
    useEditorUIStore();
  
  return (
    <div style={{ width: sidebarCollapsed ? 0 : sidebarWidth }}>
      {/* Sidebar content */}
    </div>
  );
}
```

---

## State Normalization

### Why Normalize?

```typescript
// ❌ Bad: Nested, denormalized state
interface BadState {
  openapi: {
    paths: {
      '/users': {
        get: {
          responses: {
            '200': {
              content: {
                'application/json': {
                  schema: { $ref: '#/components/schemas/User' }
                }
              }
            }
          }
        }
      }
    },
    components: {
      schemas: {
        User: { type: 'object', properties: { /* ... */ } }
      }
    }
  }
}

// ✅ Good: Normalized with relationships
interface GoodState {
  paths: {
    ids: string[];
    entities: Record<string, PathItem>;
  };
  operations: {
    ids: string[];
    entities: Record<string, Operation>;
  };
  schemas: {
    ids: string[];
    entities: Record<string, Schema>;
  };
  // Relationships
  pathToOperations: Record<string, string[]>;
  operationToSchemas: Record<string, string[]>;
}
```

### Normalization Helper

```typescript
// src/utils/normalization.ts
import { normalize, schema } from 'normalizr';

// Define entity schemas
const schemaEntity = new schema.Entity('schemas');
const operationEntity = new schema.Entity('operations', {
  requestBody: { schema: schemaEntity },
  responses: {
    schema: new schema.Values({
      content: {
        schema: new schema.Values({ schema: schemaEntity }),
      },
    }),
  },
});
const pathEntity = new schema.Entity('paths', {
  operations: [operationEntity],
});

// Normalize OpenAPI document
export function normalizeOpenAPIDocument(document: OpenAPIDocument) {
  // Transform document into normalizable structure
  const normalizable = {
    paths: Object.entries(document.paths || {}).map(([path, item]) => ({
      id: path,
      ...item,
      operations: Object.entries(item)
        .filter(([method]) => ['get', 'post', 'put', 'delete', 'patch'].includes(method))
        .map(([method, op]) => ({ id: `${path}:${method}`, method, ...op })),
    })),
  };
  
  return normalize(normalizable, { paths: [pathEntity] });
}
```

---

## RTK Query for Data Fetching

```typescript
// src/store/api/schema.api.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import type { Schema } from '../../types/openapi31.types';

/**
 * RTK Query API for schema registry
 * Automatically handles caching, refetching, and state updates
 */
export const schemaApi = createApi({
  reducerPath: 'schemaApi',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Schema'],
  endpoints: (builder) => ({
    // Query: Fetch all schemas
    getSchemas: builder.query<Schema[], void>({
      query: () => '/schemas',
      providesTags: (result) =>
        result
          ? [
              ...result.map(({ id }) => ({ type: 'Schema' as const, id })),
              { type: 'Schema', id: 'LIST' },
            ]
          : [{ type: 'Schema', id: 'LIST' }],
    }),
    
    // Query: Fetch single schema
    getSchema: builder.query<Schema, string>({
      query: (id) => `/schemas/${id}`,
      providesTags: (result, error, id) => [{ type: 'Schema', id }],
    }),
    
    // Mutation: Create schema
    createSchema: builder.mutation<Schema, Partial<Schema>>({
      query: (body) => ({
        url: '/schemas',
        method: 'POST',
        body,
      }),
      invalidatesTags: [{ type: 'Schema', id: 'LIST' }],
    }),
    
    // Mutation: Update schema
    updateSchema: builder.mutation<Schema, { id: string; updates: Partial<Schema> }>({
      query: ({ id, updates }) => ({
        url: `/schemas/${id}`,
        method: 'PATCH',
        body: updates,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'Schema', id }],
      // Optimistic update
      async onQueryStarted({ id, updates }, { dispatch, queryFulfilled }) {
        const patchResult = dispatch(
          schemaApi.util.updateQueryData('getSchema', id, (draft) => {
            Object.assign(draft, updates);
          })
        );
        
        try {
          await queryFulfilled;
        } catch {
          patchResult.undo();
        }
      },
    }),
    
    // Mutation: Delete schema
    deleteSchema: builder.mutation<void, string>({
      query: (id) => ({
        url: `/schemas/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: (result, error, id) => [
        { type: 'Schema', id },
        { type: 'Schema', id: 'LIST' },
      ],
    }),
  }),
});

// Export hooks for use in components
export const {
  useGetSchemasQuery,
  useGetSchemaQuery,
  useCreateSchemaMutation,
  useUpdateSchemaMutation,
  useDeleteSchemaMutation,
} = schemaApi;
```

**Usage in Components:**

```typescript
function SchemaList() {
  const { data: schemas, isLoading, error } = useGetSchemasQuery();
  const [deleteSchema] = useDeleteSchemaMutation();
  
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  
  return (
    <ul>
      {schemas?.map((schema) => (
        <li key={schema.id}>
          {schema.name}
          <button onClick={() => deleteSchema(schema.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

---

## VSCode Extension State Bridge

Sync state between webview and extension host:

```typescript
// src/bridge/state-bridge.ts
import { store } from '../store';
import type { OpenAPIDocument } from '../types/openapi31.types';

/**
 * Bridge between React state and VSCode extension API
 */
export class StateBridge {
  private vscode: any;
  
  constructor() {
    this.vscode = acquireVsCodeApi();
    this.initializeListeners();
  }
  
  /**
   * Listen for state changes and sync to VSCode
   */
  private initializeListeners() {
    // Subscribe to Redux store changes
    let previousState = store.getState();
    
    store.subscribe(() => {
      const currentState = store.getState();
      
      // Only sync if openapi document changed
      if (currentState.openapi.document !== previousState.openapi.document) {
        this.syncToVSCode('documentChanged', currentState.openapi.document);
      }
      
      previousState = currentState;
    });
    
    // Listen for messages from VSCode
    window.addEventListener('message', (event) => {
      const message = event.data;
      
      switch (message.command) {
        case 'loadDocument':
          store.dispatch(setDocument(message.document));
          break;
        
        case 'save':
          this.handleSave();
          break;
        
        case 'themeChanged':
          store.dispatch(setTheme(message.theme));
          break;
      }
    });
  }
  
  /**
   * Send state to VSCode extension host
   */
  private syncToVSCode(command: string, data: any) {
    this.vscode.postMessage({ command, data });
  }
  
  /**
   * Save document to VSCode workspace
   */
  private async handleSave() {
    const state = store.getState();
    const document = this.denormalizeDocument(state.openapi);
    
    this.vscode.postMessage({
      command: 'saveDocument',
      document,
    });
  }
  
  /**
   * Convert normalized state back to OpenAPI document
   */
  private denormalizeDocument(state: any): OpenAPIDocument {
    const paths = pathsSelectors.selectAll(store.getState());
    const schemas = schemasSelectors.selectAll(store.getState());
    
    return {
      ...state.document,
      paths: Object.fromEntries(paths.map((p) => [p.id, p])),
      components: {
        schemas: Object.fromEntries(schemas.map((s) => [s.id, s])),
      },
    };
  }
}

// Initialize bridge
export const stateBridge = new StateBridge();
```

---

## Performance Optimization

### Selector Optimization with Reselect

```typescript
// src/store/selectors/openapi.selectors.ts
import { createSelector } from '@reduxjs/toolkit';
import type { RootState } from '../index';

/**
 * Memoized selectors for derived state
 * Only recompute when dependencies change
 */

// Base selectors
const selectPaths = (state: RootState) => state.openapi.paths.entities;
const selectSchemas = (state: RootState) => state.openapi.schemas.entities;

// Derived selectors
export const selectPathCount = createSelector(
  [selectPaths],
  (paths) => Object.keys(paths).length
);

export const selectSchemaCount = createSelector(
  [selectSchemas],
  (schemas) => Object.keys(schemas).length
);

// Complex derived state
export const selectValidationSummary = createSelector(
  [selectPaths, selectSchemas],
  (paths, schemas) => {
    const errors: string[] = [];
    
    // Validate paths
    Object.entries(paths).forEach(([path, pathItem]) => {
      if (!pathItem.summary) {
        errors.push(`Path ${path} missing summary`);
      }
    });
    
    // Validate schemas
    Object.entries(schemas).forEach(([name, schema]) => {
      if (!schema.type) {
        errors.push(`Schema ${name} missing type`);
      }
    });
    
    return {
      errorCount: errors.length,
      errors,
      isValid: errors.length === 0,
    };
  }
);

// Parametric selectors
export const makeSelectPathById = () =>
  createSelector(
    [selectPaths, (state: RootState, pathId: string) => pathId],
    (paths, pathId) => paths[pathId]
  );
```

### Preventing Unnecessary Re-renders

```typescript
// ❌ Bad: Component re-renders on any state change
function BadComponent() {
  const state = useAppSelector((state) => state.openapi);
  return <div>{state.document?.info.title}</div>;
}

// ✅ Good: Only re-renders when title changes
function GoodComponent() {
  const title = useAppSelector((state) => state.openapi.document?.info.title);
  return <div>{title}</div>;
}

// ✅ Even Better: Use memoized selector
const selectTitle = createSelector(
  [(state: RootState) => state.openapi.document],
  (document) => document?.info.title
);

function BestComponent() {
  const title = useAppSelector(selectTitle);
  return <div>{title}</div>;
}
```

---

## Best Practices Summary

### ✅ DO

- **Use Redux Toolkit** for global specification state
- **Use Zustand** for local UI state
- **Use RTK Query** for server data fetching
- **Normalize nested data** with entity adapters
- **Create memoized selectors** for derived state
- **Implement undo/redo** with history pattern
- **Bridge state** to VSCode extension API
- **Use TypeScript strictly** - no `any` in state

### ❌ DON'T

- **Don't store derived state** - compute it with selectors
- **Don't store UI state in Redux** if it's component-specific
- **Don't mutate state directly** - use immer (built into RTK)
- **Don't normalize everything** - only normalize when beneficial
- **Don't skip memoization** for expensive computations
- **Don't forget to clean up** subscriptions and listeners

### 🎯 Key Benefits

1. **Better Performance**: Normalized state + memoized selectors = fewer re-renders
2. **Better DevTools**: Full Redux DevTools + time-travel debugging
3. **Better Testing**: Pure reducers + isolated selectors = easy testing
4. **Better Persistence**: Automatic state persistence with versioning
5. **Better DX**: Type-safe throughout with TypeScript inference

---

This state management approach provides a robust, scalable, and performant foundation for the DAPA VSCode extension, significantly improving upon the patterns described in bulletproof-react.
