# Schema-Driven Development

> **Forward engineering from official specifications for consistent, type-safe code generation**

## Table of Contents
- [Overview](#overview)
- [OpenAPI Schema Integration](#openapi-schema-integration)
- [ANTLR Grammar Integration](#antlr-grammar-integration)
- [Code Generation Pipeline](#code-generation-pipeline)
- [Persistence & Validation](#persistence--validation)

---

## Overview

### Contract-First Development

DAPA follows a **schema-driven architecture** where UI components, state management, and validation are **forward-engineered** from authoritative schema sources.

```
┌───────────────────────────────────────────┐
│         Official Specification            │
│  • OpenAPI 3.0/3.1 JSON Schema           │
│  • ANTLR Grammars (.g4 files)            │
└─────────────────┬─────────────────────────┘
                  │
                  ▼
┌───────────────────────────────────────────┐
│        One-Time Code Generation           │
│  • TypeScript interfaces                  │
│  • React components                       │
│  • Redux slices                           │
│  • Zod validators                         │
└─────────────────┬─────────────────────────┘
                  │
                  ▼
┌───────────────────────────────────────────┐
│      DAPA No-Code Authoring UI            │
│  • Type-safe components                   │
│  • Validated state                        │
│  • Specification-compliant output         │
└───────────────────────────────────────────┘
```

### Benefits

✅ **Single Source of Truth**: Schemas define both contract and implementation  
✅ **Type Safety**: Generated TypeScript matches schema exactly  
✅ **Consistency**: UI aligns with API specifications automatically  
✅ **Maintainability**: Schema updates cascade through system  
✅ **Standards Compliance**: Direct use of official specifications

---

## OpenAPI Schema Integration

### Step 1: Obtain Official Schema

```bash
# Download OpenAPI 3.1 JSON Schema
curl https://spec.openapis.org/oas/3.1/schema/2025-09-15 \
  -o schemas/openapi-3.1.json

# For OpenAPI 3.0
curl https://spec.openapis.org/oas/3.0/schema/2024-10-18 \
  -o schemas/openapi-3.0.json
```

### Step 2: Generate TypeScript Interfaces

```typescript
// scripts/generate-openapi-types.ts
import { compile } from 'json-schema-to-typescript';
import * as fs from 'fs/promises';

async function generateTypes() {
  const schema = await fs.readFile('schemas/openapi-3.1.json', 'utf8');
  
  const types = await compile(JSON.parse(schema), 'OpenAPIDocument', {
    bannerComment: '/* Generated from OpenAPI 3.1 Schema - DO NOT EDIT */',
    style: {
      semi: true,
      singleQuote: true,
    },
  });
  
  await fs.writeFile('src/types/openapi-3.1.types.ts', types);
  console.log('✅ Generated OpenAPI types');
}

generateTypes();
```

**Generated Output**:

```typescript
/* Generated from OpenAPI 3.1 Schema - DO NOT EDIT */

/**
 * Root OpenAPI Document
 */
export interface OpenAPIDocument {
  openapi: string;
  info: Info;
  jsonSchemaDialect?: string;
  servers?: Server[];
  paths?: Paths;
  webhooks?: Record<string, PathItem | Reference>;
  components?: Components;
  security?: SecurityRequirement[];
  tags?: Tag[];
  externalDocs?: ExternalDocumentation;
}

/**
 * API Information
 */
export interface Info {
  title: string;
  summary?: string;
  description?: string;
  termsOfService?: string;
  contact?: Contact;
  license?: License;
  version: string;
}

/**
 * Path Item Object
 */
export interface PathItem {
  $ref?: string;
  summary?: string;
  description?: string;
  get?: Operation;
  put?: Operation;
  post?: Operation;
  delete?: Operation;
  options?: Operation;
  head?: Operation;
  patch?: Operation;
  trace?: Operation;
  servers?: Server[];
  parameters?: (Parameter | Reference)[];
}

// ... more interfaces
```

### Step 3: Generate React Components

```typescript
// scripts/generate-openapi-components.ts
import { JSONSchema7 } from 'json-schema';

function generateFormComponent(
  objectName: string,
  schema: JSONSchema7
): string {
  const properties = schema.properties || {};
  const required = schema.required || [];

  return `
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { ${objectName}Schema, type ${objectName} } from '../types';

export interface ${objectName}FormProps {
  initialData?: Partial<${objectName}>;
  onSubmit: (data: ${objectName}) => void;
}

export function ${objectName}Form({
  initialData,
  onSubmit,
}: ${objectName}FormProps) {
  const { register, handleSubmit, formState: { errors } } = useForm<${objectName}>({
    resolver: zodResolver(${objectName}Schema),
    defaultValues: initialData,
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      ${generateFormFields(properties, required)}
      
      <button type="submit">Save</button>
    </form>
  );
}
`;
}

function generateFormFields(
  properties: Record<string, JSONSchema7>,
  required: string[]
): string {
  return Object.entries(properties)
    .map(([name, prop]) => {
      const isRequired = required.includes(name);
      const type = prop.type as string;
      
      return `
      <div className="form-field">
        <label htmlFor="${name}">
          ${formatLabel(name)}
          ${isRequired ? '<span className="required">*</span>' : ''}
        </label>
        ${generateInput(name, type, prop)}
        {errors.${name} && (
          <span className="error">{errors.${name}?.message}</span>
        )}
      </div>`;
    })
    .join('\n');
}

function generateInput(name: string, type: string, schema: JSONSchema7): string {
  switch (type) {
    case 'string':
      if (schema.enum) {
        return `
        <select {...register('${name}')}>
          ${schema.enum.map((v) => `<option value="${v}">${v}</option>`).join('\n')}
        </select>`;
      }
      return `<input type="text" {...register('${name}')} />`;
    
    case 'number':
    case 'integer':
      return `<input type="number" {...register('${name}')} />`;
    
    case 'boolean':
      return `<input type="checkbox" {...register('${name}')} />`;
    
    default:
      return `<input type="text" {...register('${name}')} />`;
  }
}
```

### Step 4: Generate Redux State

```typescript
// scripts/generate-openapi-redux.ts

function generateReduxSlice(objectName: string, schema: JSONSchema7): string {
  return `
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import { ${objectName} } from '../types';

interface ${objectName}State {
  items: ${objectName}[];
  current: ${objectName} | null;
  loading: boolean;
  error: string | null;
}

const initialState: ${objectName}State = {
  items: [],
  current: null,
  loading: false,
  error: null,
};

export const ${objectName.toLowerCase()}Slice = createSlice({
  name: '${objectName.toLowerCase()}',
  initialState,
  reducers: {
    set${objectName}: (state, action: PayloadAction<${objectName}>) => {
      state.current = action.payload;
    },
    
    update${objectName}Field: (
      state,
      action: PayloadAction<{ field: keyof ${objectName}; value: unknown }>
    ) => {
      if (state.current) {
        state.current[action.payload.field] = action.payload.value as never;
      }
    },
    
    add${objectName}: (state, action: PayloadAction<${objectName}>) => {
      state.items.push(action.payload);
    },
    
    remove${objectName}: (state, action: PayloadAction<string>) => {
      state.items = state.items.filter((item) => item.id !== action.payload);
    },
  },
});

export const {
  set${objectName},
  update${objectName}Field,
  add${objectName},
  remove${objectName},
} = ${objectName.toLowerCase()}Slice.actions;

export default ${objectName.toLowerCase()}Slice.reducer;
`;
}
```

### Step 5: Generate Validators

```typescript
// scripts/generate-openapi-validators.ts
import { JSONSchema7 } from 'json-schema';

function generateZodSchema(objectName: string, schema: JSONSchema7): string {
  const properties = schema.properties || {};
  const required = schema.required || [];

  const zodFields = Object.entries(properties)
    .map(([name, prop]) => {
      const isRequired = required.includes(name);
      return generateZodField(name, prop, isRequired);
    })
    .join(',\n  ');

  return `
import { z } from 'zod';

export const ${objectName}Schema = z.object({
  ${zodFields}
});

export type ${objectName} = z.infer<typeof ${objectName}Schema>;
`;
}

function generateZodField(
  name: string,
  prop: JSONSchema7,
  isRequired: boolean
): string {
  let zodType = 'z.unknown()';

  switch (prop.type) {
    case 'string':
      zodType = 'z.string()';
      if (prop.format === 'email') zodType += '.email()';
      if (prop.format === 'uri') zodType += '.url()';
      if (prop.minLength) zodType += `.min(${prop.minLength})`;
      if (prop.maxLength) zodType += `.max(${prop.maxLength})`;
      if (prop.pattern) zodType += `.regex(/${prop.pattern}/)`;
      if (prop.enum) {
        const values = prop.enum.map((v) => `'${v}'`).join(', ');
        zodType = `z.enum([${values}])`;
      }
      break;
      
    case 'number':
    case 'integer':
      zodType = 'z.number()';
      if (prop.minimum !== undefined) zodType += `.min(${prop.minimum})`;
      if (prop.maximum !== undefined) zodType += `.max(${prop.maximum})`;
      break;
      
    case 'boolean':
      zodType = 'z.boolean()';
      break;
      
    case 'array':
      const itemSchema = generateZodField('item', prop.items as JSONSchema7, true);
      zodType = `z.array(${itemSchema})`;
      break;
      
    case 'object':
      zodType = 'z.record(z.unknown())';
      break;
  }

  if (!isRequired) {
    zodType += '.optional()';
  }

  if (prop.description) {
    zodType += `.describe('${prop.description.replace(/'/g, "\\'")}')`;
  }

  return `${name}: ${zodType}`;
}
```

---

## ANTLR Grammar Integration

### Step 1: Obtain Grammar

```bash
# Download TaxiLang grammar
curl https://raw.githubusercontent.com/taxilang/taxilang/develop/compiler/src/main/antlr4/lang/taxi/Taxi.g4 \
  -o grammars/Taxi.g4
```

### Step 2: Generate Parser

```bash
# Install ANTLR4 TypeScript tools
npm install -D antlr4ts-cli antlr4ts

# Generate TypeScript parser
npx antlr4ts -visitor -Dlanguage=TypeScript \
  grammars/Taxi.g4 \
  -o src/parsers/taxilang
```

**Generated Files**:
- `TaxiLexer.ts` - Tokenizer
- `TaxiParser.ts` - Parser with AST contexts
- `TaxiVisitor.ts` - Visitor interface

### Step 3: Create AST Type Definitions

```typescript
// src/types/taxi-ast.types.ts

export interface ASTNode {
  type: string;
  start: number;
  end: number;
  children?: ASTNode[];
}

export interface TypeDeclarationNode extends ASTNode {
  type: 'TypeDeclaration';
  modifiers: string[];
  kind: 'Type' | 'Model';
  identifier: string;
  typeArguments?: string[];
  inherits?: string[];
  body?: TypeBodyNode;
}

export interface TypeBodyNode extends ASTNode {
  type: 'TypeBody';
  members: FieldDeclarationNode[];
}

export interface FieldDeclarationNode extends ASTNode {
  type: 'FieldDeclaration';
  identifier: string;
  fieldType?: string;
  optional: boolean;
  annotations: string[];
}
```

### Step 4: Create Parser Wrapper

```typescript
// src/parsers/taxi-parser.ts
import { CharStreams, CommonTokenStream } from 'antlr4ts';
import { TaxiLexer } from './taxilang/TaxiLexer';
import { TaxiParser } from './taxilang/TaxiParser';
import { TaxiVisitor } from './taxilang/TaxiVisitor';
import { AbstractParseTreeVisitor } from 'antlr4ts/tree/AbstractParseTreeVisitor';

export interface TaxiType {
  name: string;
  kind: 'type' | 'model';
  fields: TaxiField[];
  annotations: string[];
  inherits?: string[];
}

export interface TaxiField {
  name: string;
  type: string;
  optional: boolean;
}

export function parseTaxiLang(source: string): TaxiType[] {
  const inputStream = CharStreams.fromString(source);
  const lexer = new TaxiLexer(inputStream);
  const tokenStream = new CommonTokenStream(lexer);
  const parser = new TaxiParser(tokenStream);
  
  const tree = parser.document();
  
  const visitor = new TaxiTypeExtractor();
  visitor.visit(tree);
  
  return visitor.types;
}

class TaxiTypeExtractor extends AbstractParseTreeVisitor<void> implements TaxiVisitor<void> {
  types: TaxiType[] = [];
  
  defaultResult(): void {
    return undefined;
  }
  
  visitTypeDeclaration(ctx: any): void {
    const type: TaxiType = {
      name: ctx.identifier().text,
      kind: ctx.typeKind().K_Model() ? 'model' : 'type',
      fields: this.extractFields(ctx.typeBody()),
      annotations: [],
    };
    
    this.types.push(type);
    this.visitChildren(ctx);
  }
  
  private extractFields(typeBody: any): TaxiField[] {
    if (!typeBody) return [];
    
    const fields: TaxiField[] = [];
    const members = typeBody.typeMemberDeclaration();
    
    for (const member of members) {
      const fieldDecl = member.fieldDeclaration();
      if (fieldDecl) {
        fields.push({
          name: fieldDecl.identifier().text,
          type: this.extractFieldType(fieldDecl),
          optional: fieldDecl.text.includes('?'),
        });
      }
    }
    
    return fields;
  }
  
  private extractFieldType(fieldDecl: any): string {
    const typeDecl = fieldDecl.fieldTypeDeclaration();
    if (typeDecl?.typeExpression()) {
      return typeDecl.typeExpression().nullableTypeReference().text;
    }
    return 'unknown';
  }
}
```

### Step 5: Generate Components from Grammar

```typescript
// scripts/generate-taxi-components.ts

function generateTypeEditor(type: TaxiType): string {
  return `
import React from 'react';
import { useDispatch } from 'react-redux';
import { updateType, addField } from '@store/taxi-slice';

interface TypeEditorProps {
  type: TaxiType;
}

export function TypeEditor({ type }: TypeEditorProps) {
  const dispatch = useDispatch();
  const [newFieldName, setNewFieldName] = React.useState('');

  const handleAddField = () => {
    if (!newFieldName) return;
    
    dispatch(addField({
      typeName: type.name,
      field: {
        name: newFieldName,
        type: 'String',
        optional: false,
      },
    }));
    
    setNewFieldName('');
  };

  return (
    <div className="type-editor">
      <div className="type-header">
        <span className="type-kind">{type.kind}</span>
        <input
          type="text"
          value={type.name}
          onChange={(e) => dispatch(updateType({
            name: type.name,
            updates: { name: e.target.value },
          }))}
        />
      </div>

      <div className="fields-section">
        {type.fields.map((field) => (
          <FieldEditor key={field.name} field={field} typeName={type.name} />
        ))}
        
        <div className="add-field">
          <input
            placeholder="Field name"
            value={newFieldName}
            onChange={(e) => setNewFieldName(e.target.value)}
          />
          <button onClick={handleAddField}>Add</button>
        </div>
      </div>
    </div>
  );
}
`;
}
```

---

## Code Generation Pipeline

### NPM Scripts

```json
{
  "scripts": {
    "generate": "npm run generate:openapi && npm run generate:antlr",
    
    "generate:openapi": "ts-node scripts/generate-openapi.ts",
    "generate:openapi:types": "ts-node scripts/generate-openapi-types.ts",
    "generate:openapi:components": "ts-node scripts/generate-openapi-components.ts",
    "generate:openapi:redux": "ts-node scripts/generate-openapi-redux.ts",
    "generate:openapi:validators": "ts-node scripts/generate-openapi-validators.ts",
    
    "generate:antlr": "npm run generate:antlr:taxi",
    "generate:antlr:taxi": "antlr4ts -visitor grammars/Taxi.g4 -o src/parsers/taxilang",
    
    "prebuild": "npm run generate"
  }
}
```

### Generation Script

```typescript
// scripts/generate-openapi.ts
import { program } from 'commander';

program
  .option('-v, --version <version>', 'OpenAPI version', '3.1')
  .option('-o, --output <dir>', 'Output directory', 'src/generated/openapi')
  .parse();

const options = program.opts();

async function main() {
  console.log(`Generating OpenAPI ${options.version} artifacts...`);
  
  await generateTypes(options);
  await generateComponents(options);
  await generateRedux(options);
  await generateValidators(options);
  
  console.log('✅ Generation complete!');
}

main().catch((error) => {
  console.error('❌ Generation failed:', error);
  process.exit(1);
});
```

---

## Persistence & Validation

### Runtime Validation

```typescript
// src/validation/openapi-validator.ts
import Ajv from 'ajv';
import addFormats from 'ajv-formats';
import openapiSchema from '../schemas/openapi-3.1.json';

const ajv = new Ajv({
  allErrors: true,
  strict: false,
  validateFormats: true,
});

addFormats(ajv);

const validateOpenAPIDocument = ajv.compile(openapiSchema);

export function validate(document: unknown): ValidationResult {
  const valid = validateOpenAPIDocument(document);
  
  if (valid) {
    return { valid: true, errors: [] };
  }
  
  const errors = (validateOpenAPIDocument.errors || []).map((error) => ({
    path: error.instancePath || '/',
    message: error.message || 'Validation error',
    keyword: error.keyword,
  }));
  
  return { valid: false, errors };
}
```

### Save/Load Documents

```typescript
// src/persistence/document-store.ts
import * as vscode from 'vscode';
import { validate } from '../validation/openapi-validator';

export class DocumentStore {
  async save(uri: vscode.Uri, document: OpenAPIDocument): Promise<void> {
    const validation = validate(document);
    
    if (!validation.valid) {
      throw new ValidationError(validation.errors);
    }
    
    const content = JSON.stringify(document, null, 2);
    await vscode.workspace.fs.writeFile(uri, Buffer.from(content, 'utf8'));
  }
  
  async load(uri: vscode.Uri): Promise<OpenAPIDocument> {
    const bytes = await vscode.workspace.fs.readFile(uri);
    const content = Buffer.from(bytes).toString('utf8');
    const document = JSON.parse(content);
    
    const validation = validate(document);
    if (!validation.valid) {
      throw new ValidationError(validation.errors);
    }
    
    return document;
  }
}
```

---

## Best Practices Summary

### ✅ DO

- **Use official schemas** as single source of truth
- **Generate once per spec version**
- **Validate at runtime** with AJV
- **Version control schemas** separately
- **Test round-trip** (author → save → load)
- **Document generation** process clearly
- **Keep generated code** in separate directory

### ❌ DON'T

- **Don't modify generated** code manually
- **Don't skip validation** before saving
- **Don't mix spec versions** in same feature
- **Don't create abstractions** over generated types
- **Don't forget to regenerate** after schema updates

### 🎯 Quick Checklist

- [ ] Official schemas downloaded
- [ ] Generation scripts created
- [ ] Types generated correctly
- [ ] Components follow schema
- [ ] Redux state matches schema
- [ ] Validation configured
- [ ] Round-trip tested
- [ ] Documentation updated

---

This schema-driven approach ensures the DAPA extension maintains perfect alignment with official specifications while providing type safety and consistency throughout the codebase.