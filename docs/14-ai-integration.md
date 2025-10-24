# AI Integration with VSCode Extensibility

> **Integrating Copilot AI into DAPA for intelligent API design and data modeling assistance**

## Table of Contents
- [AI Integration Overview](#ai-integration-overview)
- [Chat Participant for API Design](#chat-participant-for-api-design)
- [Language Model API Integration](#language-model-api-integration)
- [Language Model Tools for Agent Mode](#language-model-tools-for-agent-mode)
- [Prompt Engineering with TSX](#prompt-engineering-with-tsx)
- [AI-Powered Smart Actions](#ai-powered-smart-actions)

---

## AI Integration Overview

### Why Integrate AI in DAPA?

DAPA can leverage VSCode's AI extensibility to provide:

**For API Design:**
- Generate OpenAPI specifications from natural language descriptions
- Suggest API endpoints based on domain models
- Validate API designs and suggest improvements
- Auto-complete API documentation
- Generate example requests/responses

**For Data Modeling:**
- Convert natural language to TaxiLang/data models
- Suggest relationships between entities
- Validate model constraints
- Generate test data fixtures
- Explain complex model structures

### Integration Options

VSCode provides four AI integration approaches:

| Approach | Use Case | DAPA Application |
|----------|----------|------------------|
| **Chat Participant** | Domain-specific assistant | `@dapa` expert for API/model design |
| **Language Model Tool** | Agent mode capabilities | Auto-invoked tools for schema validation |
| **Language Model API** | Direct AI integration | Smart code actions, hover providers |
| **MCP Tool** | External service integration | Connect to API registries, databases |

**For DAPA**, we'll implement:
1. **Chat Participant** (`@dapa`) - Main AI interface
2. **Language Model Tools** - Schema validation, API generation
3. **Language Model API** - Smart actions throughout the UI

---

## Chat Participant for API Design

### 1. Register the Chat Participant

```json
// package.json
{
  "contributes": {
    "chatParticipants": [
      {
        "id": "dapa.api-designer",
        "name": "dapa",
        "fullName": "DAPA API Designer",
        "description": "Your AI assistant for API specifications and data modeling",
        "isSticky": true,
        "commands": [
          {
            "name": "generate",
            "description": "Generate an API specification from description"
          },
          {
            "name": "validate",
            "description": "Validate and suggest improvements for your API"
          },
          {
            "name": "model",
            "description": "Create or modify data models"
          },
          {
            "name": "examples",
            "description": "Generate example requests and responses"
          }
        ],
        "disambiguation": [
          {
            "category": "api-design",
            "description": "Questions about designing, validating, or documenting REST APIs, OpenAPI specifications, or API best practices",
            "examples": [
              "How do I design a user authentication API?",
              "Generate an OpenAPI spec for a blog API",
              "What's wrong with my API design?",
              "Add pagination to my API endpoints"
            ]
          },
          {
            "category": "data-modeling",
            "description": "Questions about data models, entity relationships, schema design, or domain modeling",
            "examples": [
              "Create a data model for an e-commerce system",
              "How should I model user permissions?",
              "Convert this JSON to a TaxiLang model"
            ]
          }
        ]
      }
    ]
  }
}
```

### 2. Implement the Request Handler

```typescript
// src/chat/dapa-participant.ts
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  // Create the @dapa chat participant
  const dapa = vscode.chat.createChatParticipant(
    'dapa.api-designer',
    async (
      request: vscode.ChatRequest,
      context: vscode.ChatContext,
      stream: vscode.ChatResponseStream,
      token: vscode.CancellationToken
    ) => {
      // Handle the request based on command
      if (request.command === 'generate') {
        return handleGenerate(request, context, stream, token);
      } else if (request.command === 'validate') {
        return handleValidate(request, context, stream, token);
      } else if (request.command === 'model') {
        return handleModel(request, context, stream, token);
      } else if (request.command === 'examples') {
        return handleExamples(request, context, stream, token);
      } else {
        // General Q&A about API design
        return handleGeneral(request, context, stream, token);
      }
    }
  );

  // Set participant properties
  dapa.iconPath = vscode.Uri.joinPath(context.extensionUri, 'resources', 'dapa-icon.png');

  // Add follow-up provider
  dapa.followupProvider = {
    provideFollowups(
      result: DapaResult,
      context: vscode.ChatContext,
      token: vscode.CancellationToken
    ) {
      if (result.command === 'generate') {
        return [
          {
            prompt: '@dapa /validate my generated API',
            label: vscode.l10n.t('Validate the API design')
          },
          {
            prompt: '@dapa /examples',
            label: vscode.l10n.t('Generate example requests')
          }
        ];
      }
      return [];
    }
  };

  context.subscriptions.push(dapa);
}
```

### 3. Implement Command Handlers

```typescript
// src/chat/handlers/generate-handler.ts
async function handleGenerate(
  request: vscode.ChatRequest,
  context: vscode.ChatContext,
  stream: vscode.ChatResponseStream,
  token: vscode.CancellationToken
): Promise<DapaResult> {
  stream.progress('Analyzing your API requirements...');

  try {
    // Select language model
    const [model] = await vscode.lm.selectChatModels({
      vendor: 'copilot',
      family: 'gpt-4o'
    });

    // Build the prompt
    const messages = [
      vscode.LanguageModelChatMessage.User(
        `You are an expert API designer. Generate a complete OpenAPI 3.1 specification based on the user's description.

Rules:
1. Follow OpenAPI 3.1 specification exactly
2. Include comprehensive descriptions
3. Add realistic examples
4. Use proper HTTP methods and status codes
5. Include common headers and error responses
6. Output ONLY valid JSON (no markdown, no explanations)

User description: ${request.prompt}`
      )
    ];

    // Send request to language model
    const response = await model.sendRequest(messages, {}, token);

    // Stream the response
    let fullText = '';
    for await (const fragment of response.text) {
      fullText += fragment;
      stream.markdown(fragment);
    }

    // Try to parse as OpenAPI spec
    try {
      const spec = JSON.parse(fullText);
      
      // Validate it's OpenAPI
      if (spec.openapi && spec.info && spec.paths) {
        // Offer to create a new file
        stream.button({
          command: 'dapa.saveSpec',
          title: vscode.l10n.t('Save as OpenAPI File'),
          arguments: [spec]
        });
      }
    } catch (e) {
      // Response wasn't valid JSON, but that's okay
    }

    return { command: 'generate', success: true };
  } catch (err) {
    if (err instanceof vscode.LanguageModelError) {
      stream.markdown(`⚠️ ${err.message}`);
    }
    return { command: 'generate', success: false };
  }
}
```

```typescript
// src/chat/handlers/validate-handler.ts
async function handleValidate(
  request: vscode.ChatRequest,
  context: vscode.ChatContext,
  stream: vscode.ChatResponseStream,
  token: vscode.CancellationToken
): Promise<DapaResult> {
  stream.progress('Analyzing your API specification...');

  // Get current OpenAPI document from editor
  const editor = vscode.window.activeTextEditor;
  if (!editor) {
    stream.markdown('Please open an OpenAPI specification file first.');
    return { command: 'validate', success: false };
  }

  const document = editor.document.getText();

  try {
    const [model] = await vscode.lm.selectChatModels({
      vendor: 'copilot',
      family: 'gpt-4o'
    });

    const messages = [
      vscode.LanguageModelChatMessage.User(
        `You are an API design expert. Review this OpenAPI specification and provide constructive feedback.

Focus on:
1. RESTful best practices
2. Security considerations
3. API design patterns
4. Documentation quality
5. Error handling
6. Versioning strategy

Format your response with:
- ✅ What's good
- ⚠️ What needs improvement
- 💡 Specific recommendations

OpenAPI Specification:
\`\`\`json
${document}
\`\`\``
      )
    ];

    const response = await model.sendRequest(messages, {}, token);

    // Stream the validation feedback
    for await (const fragment of response.text) {
      stream.markdown(fragment);
    }

    // Add references to relevant API design resources
    stream.reference(
      vscode.Uri.parse('https://restfulapi.net/rest-api-design-tutorial-with-example/')
    );

    return { command: 'validate', success: true };
  } catch (err) {
    if (err instanceof vscode.LanguageModelError) {
      stream.markdown(`⚠️ ${err.message}`);
    }
    return { command: 'validate', success: false };
  }
}
```

### 4. Implement Model Generation

```typescript
// src/chat/handlers/model-handler.ts
async function handleModel(
  request: vscode.ChatRequest,
  context: vscode.ChatContext,
  stream: vscode.ChatResponseStream,
  token: vscode.CancellationToken
): Promise<DapaResult> {
  stream.progress('Creating your data model...');

  try {
    const [model] = await vscode.lm.selectChatModels({
      vendor: 'copilot',
      family: 'gpt-4o'
    });

    const messages = [
      vscode.LanguageModelChatMessage.User(
        `You are a data modeling expert. Create a TaxiLang data model based on the user's description.

TaxiLang syntax rules:
- Use 'type' for simple types, 'model' for complex entities
- Fields: fieldName : FieldType
- Optional fields: fieldName : FieldType?
- Relationships: use type references
- Add descriptions with /** */

Example:
\`\`\`taxi
model User {
  /** Unique identifier */
  id : UserId inherits String
  
  /** User's email address */
  email : Email inherits String
  
  /** User's full name */
  name : String
  
  /** Optional profile picture URL */
  avatarUrl : Url?
}
\`\`\`

User description: ${request.prompt}

Output ONLY valid TaxiLang code (no markdown, no explanations).`
      )
    ];

    const response = await model.sendRequest(messages, {}, token);

    // Collect the full response
    let fullText = '';
    stream.markdown('```taxi\n');
    for await (const fragment of response.text) {
      fullText += fragment;
      stream.markdown(fragment);
    }
    stream.markdown('\n```\n');

    // Offer to insert into editor
    stream.button({
      command: 'dapa.insertModel',
      title: vscode.l10n.t('Insert into Editor'),
      arguments: [fullText]
    });

    return { command: 'model', success: true };
  } catch (err) {
    if (err instanceof vscode.LanguageModelError) {
      stream.markdown(`⚠️ ${err.message}`);
    }
    return { command: 'model', success: false };
  }
}
```

---

## Language Model API Integration

### Smart Code Actions

Use the Language Model API to power smart code actions in the editor:

```typescript
// src/features/openapi/code-actions/suggest-description.ts
import * as vscode from 'vscode';

export class SuggestDescriptionCodeAction implements vscode.CodeActionProvider {
  async provideCodeActions(
    document: vscode.TextDocument,
    range: vscode.Range,
    context: vscode.CodeActionContext,
    token: vscode.CancellationToken
  ): Promise<vscode.CodeAction[]> {
    const actions: vscode.CodeAction[] = [];

    // Check if cursor is on an API path without description
    const line = document.lineAt(range.start.line);
    if (line.text.includes('"/') && !this.hasDescription(document, range)) {
      const action = new vscode.CodeAction(
        '✨ Generate description with AI',
        vscode.CodeActionKind.RefactorRewrite
      );
      
      action.command = {
        command: 'dapa.generateDescription',
        title: 'Generate Description',
        arguments: [document, range]
      };
      
      actions.push(action);
    }

    return actions;
  }

  private hasDescription(document: vscode.TextDocument, range: vscode.Range): boolean {
    // Check nearby lines for "description" field
    const startLine = Math.max(0, range.start.line - 3);
    const endLine = Math.min(document.lineCount - 1, range.start.line + 3);
    
    for (let i = startLine; i <= endLine; i++) {
      if (document.lineAt(i).text.includes('"description"')) {
        return true;
      }
    }
    return false;
  }
}
```

```typescript
// src/features/openapi/commands/generate-description.ts
import * as vscode from 'vscode';

export async function generateDescription(
  document: vscode.TextDocument,
  range: vscode.Range
): Promise<void> {
  const editor = vscode.window.activeTextEditor;
  if (!editor) return;

  try {
    // Show progress
    await vscode.window.withProgress(
      {
        location: vscode.ProgressLocation.Notification,
        title: 'Generating description...',
        cancellable: true
      },
      async (progress, token) => {
        // Get context around the cursor
        const startLine = Math.max(0, range.start.line - 10);
        const endLine = Math.min(document.lineCount - 1, range.start.line + 10);
        const context = document.getText(
          new vscode.Range(startLine, 0, endLine, 0)
        );

        // Select language model (prefer fast model for quick actions)
        const [model] = await vscode.lm.selectChatModels({
          vendor: 'copilot',
          family: 'gpt-4o-mini' // Faster model for quick actions
        });

        const messages = [
          vscode.LanguageModelChatMessage.User(
            `Generate a concise, professional description for this API endpoint.

Context:
\`\`\`json
${context}
\`\`\`

Rules:
- One sentence, 50-100 characters
- Start with a verb (e.g., "Retrieves", "Creates", "Updates")
- Be specific about what the endpoint does
- Output ONLY the description text, no quotes or formatting`
          )
        ];

        const response = await model.sendRequest(messages, {}, token);

        // Collect the response
        let description = '';
        for await (const fragment of response.text) {
          description += fragment;
        }

        // Clean up the description
        description = description.trim().replace(/^["']|["']$/g, '');

        // Insert the description
        const line = document.lineAt(range.start.line);
        const indent = line.text.match(/^\s*/)?.[0] || '';
        const descriptionLine = `${indent}  "description": "${description}",\n`;

        await editor.edit((editBuilder) => {
          editBuilder.insert(
            new vscode.Position(range.start.line + 1, 0),
            descriptionLine
          );
        });
      }
    );
  } catch (err) {
    if (err instanceof vscode.LanguageModelError) {
      vscode.window.showErrorMessage(`AI Error: ${err.message}`);
    } else {
      vscode.window.showErrorMessage('Failed to generate description');
    }
  }
}
```

### Hover Provider with AI

```typescript
// src/features/openapi/hover/explain-schema.ts
import * as vscode from 'vscode';

export class SchemaExplainHoverProvider implements vscode.HoverProvider {
  async provideHover(
    document: vscode.TextDocument,
    position: vscode.Position,
    token: vscode.CancellationToken
  ): Promise<vscode.Hover | undefined> {
    const range = document.getWordRangeAtPosition(position);
    if (!range) return;

    const word = document.getText(range);

    // Only show AI hover for complex schema definitions
    if (!this.isSchemaKeyword(word)) return;

    try {
      const [model] = await vscode.lm.selectChatModels({
        vendor: 'copilot',
        family: 'gpt-4o-mini'
      });

      // Get context around the hover position
      const context = this.getContext(document, position);

      const messages = [
        vscode.LanguageModelChatMessage.User(
          `Explain this OpenAPI schema element in 2-3 sentences.

Element: ${word}
Context:
\`\`\`json
${context}
\`\`\`

Be concise and practical. Focus on what the developer needs to know.`
        )
      ];

      const response = await model.sendRequest(messages, {}, token);

      let explanation = '';
      for await (const fragment of response.text) {
        explanation += fragment;
      }

      const markdown = new vscode.MarkdownString();
      markdown.appendMarkdown(`**AI Explanation**\n\n${explanation}`);
      markdown.appendMarkdown('\n\n---\n\n');
      markdown.appendMarkdown('[OpenAPI Docs](https://spec.openapis.org/)');

      return new vscode.Hover(markdown, range);
    } catch (err) {
      // Silently fail for hover providers
      return undefined;
    }
  }

  private isSchemaKeyword(word: string): boolean {
    const keywords = ['schema', 'properties', 'allOf', 'oneOf', 'anyOf', 'discriminator'];
    return keywords.includes(word);
  }

  private getContext(document: vscode.TextDocument, position: vscode.Position): string {
    const startLine = Math.max(0, position.line - 5);
    const endLine = Math.min(document.lineCount - 1, position.line + 5);
    return document.getText(new vscode.Range(startLine, 0, endLine, 0));
  }
}
```

---

## Language Model Tools for Agent Mode

Language model tools enable agent mode to automatically use DAPA capabilities:

### 1. Define OpenAPI Validation Tool

```typescript
// src/tools/validate-openapi-tool.ts
import * as vscode from 'vscode';

export function registerValidateOpenAPITool(context: vscode.ExtensionContext) {
  const tool = vscode.lm.registerTool('dapa_validate_openapi', {
    displayName: 'Validate OpenAPI Specification',
    description: 'Validates an OpenAPI specification against the official schema and best practices',
    tags: ['api-design', 'validation'],
    
    inputSchema: {
      type: 'object',
      properties: {
        specification: {
          type: 'string',
          description: 'The OpenAPI specification as JSON string'
        },
        version: {
          type: 'string',
          enum: ['3.0', '3.1'],
          description: 'OpenAPI version to validate against'
        }
      },
      required: ['specification']
    },

    invoke: async (input, token) => {
      try {
        const spec = JSON.parse(input.specification);
        const version = input.version || '3.1';

        // Validate against official schema
        const validation = await validateAgainstSchema(spec, version);

        if (validation.valid) {
          return {
            content: [
              {
                type: 'text',
                text: '✅ OpenAPI specification is valid!'
              }
            ]
          };
        } else {
          const errors = validation.errors.map(e => `- ${e.path}: ${e.message}`).join('\n');
          return {
            content: [
              {
                type: 'text',
                text: `❌ Validation errors:\n${errors}`
              }
            ]
          };
        }
      } catch (err) {
        return {
          content: [
            {
              type: 'text',
              text: `Error validating specification: ${(err as Error).message}`
            }
          ]
        };
      }
    }
  });

  context.subscriptions.push(tool);
}
```

### 2. Define Schema Generation Tool

```typescript
// src/tools/generate-schema-tool.ts
import * as vscode from 'vscode';

export function registerGenerateSchemaToolfunction(context: vscode.ExtensionContext) {
  const tool = vscode.lm.registerTool('dapa_generate_schema', {
    displayName: 'Generate API Schema',
    description: 'Generates OpenAPI or JSON Schema from a description or example data',
    tags: ['api-design', 'schema-generation'],
    
    inputSchema: {
      type: 'object',
      properties: {
        description: {
          type: 'string',
          description: 'Natural language description of the API or data structure'
        },
        example: {
          type: 'string',
          description: 'Optional: Example JSON data to base the schema on'
        },
        type: {
          type: 'string',
          enum: ['openapi', 'json-schema', 'taxi'],
          description: 'Type of schema to generate'
        }
      },
      required: ['description', 'type']
    },

    invoke: async (input, token) => {
      try {
        // Use language model to generate schema
        const [model] = await vscode.lm.selectChatModels({
          vendor: 'copilot',
          family: 'gpt-4o'
        });

        let prompt = '';
        if (input.type === 'openapi') {
          prompt = `Generate a complete OpenAPI 3.1 specification for: ${input.description}`;
        } else if (input.type === 'json-schema') {
          prompt = `Generate a JSON Schema (draft-07) for: ${input.description}`;
        } else if (input.type === 'taxi') {
          prompt = `Generate TaxiLang model definitions for: ${input.description}`;
        }

        if (input.example) {
          prompt += `\n\nExample data:\n${input.example}`;
        }

        const messages = [
          vscode.LanguageModelChatMessage.User(
            `${prompt}\n\nOutput ONLY valid ${input.type} (no markdown, no explanations).`
          )
        ];

        const response = await model.sendRequest(messages, {}, token);

        let schema = '';
        for await (const fragment of response.text) {
          schema += fragment;
        }

        return {
          content: [
            {
              type: 'text',
              text: schema
            }
          ]
        };
      } catch (err) {
        return {
          content: [
            {
              type: 'text',
              text: `Error generating schema: ${(err as Error).message}`
            }
          ]
        };
      }
    }
  });

  context.subscriptions.push(tool);
}
```

### 3. Define Data Model Conversion Tool

```typescript
// src/tools/convert-model-tool.ts
export function registerConvertModelTool(context: vscode.ExtensionContext) {
  const tool = vscode.lm.registerTool('dapa_convert_model', {
    displayName: 'Convert Data Model',
    description: 'Converts between different data modeling formats (JSON Schema, TaxiLang, OpenAPI, TypeScript)',
    tags: ['data-modeling', 'conversion'],
    
    inputSchema: {
      type: 'object',
      properties: {
        source: {
          type: 'string',
          description: 'Source model definition'
        },
        fromFormat: {
          type: 'string',
          enum: ['json-schema', 'openapi', 'taxi', 'typescript'],
          description: 'Source format'
        },
        toFormat: {
          type: 'string',
          enum: ['json-schema', 'openapi', 'taxi', 'typescript'],
          description: 'Target format'
        }
      },
      required: ['source', 'fromFormat', 'toFormat']
    },

    invoke: async (input, token) => {
      try {
        const [model] = await vscode.lm.selectChatModels({
          vendor: 'copilot',
          family: 'gpt-4o'
        });

        const messages = [
          vscode.LanguageModelChatMessage.User(
            `Convert this ${input.fromFormat} model to ${input.toFormat}.

Source:
\`\`\`
${input.source}
\`\`\`

Requirements:
- Preserve all field names and types
- Maintain relationships and constraints
- Add appropriate descriptions
- Output ONLY valid ${input.toFormat} (no markdown, no explanations)`
          )
        ];

        const response = await model.sendRequest(messages, {}, token);

        let converted = '';
        for await (const fragment of response.text) {
          converted += fragment;
        }

        return {
          content: [
            {
              type: 'text',
              text: converted
            }
          ]
        };
      } catch (err) {
        return {
          content: [
            {
              type: 'text',
              text: `Error converting model: ${(err as Error).message}`
            }
          ]
        };
      }
    }
  });

  context.subscriptions.push(tool);
}
```

---

## Prompt Engineering with TSX

Use `@vscode/prompt-tsx` for complex, token-budget-aware prompts:

### 1. Install Dependencies

```bash
npm install @vscode/prompt-tsx
```

### 2. Define Prompt Components

```typescript
// src/prompts/components/base-instructions.tsx
import { PromptElement, UserMessage, BasePromptElementProps } from '@vscode/prompt-tsx';

interface BaseInstructionsProps extends BasePromptElementProps {
  domain: 'api-design' | 'data-modeling';
}

export class BaseInstructions extends PromptElement<BaseInstructionsProps> {
  render() {
    const instructions = this.props.domain === 'api-design'
      ? this.apiDesignInstructions()
      : this.dataModelingInstructions();

    return (
      <UserMessage priority={100}>
        {instructions}
      </UserMessage>
    );
  }

  private apiDesignInstructions(): string {
    return `You are an expert API designer and OpenAPI specification expert.

Your responsibilities:
- Design RESTful APIs following industry best practices
- Generate valid OpenAPI 3.1 specifications
- Validate API designs for security, performance, and maintainability
- Suggest improvements based on common patterns

Follow these principles:
- Use proper HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Include comprehensive error responses (400, 401, 403, 404, 500)
- Add detailed descriptions for all endpoints
- Use consistent naming conventions
- Include examples for request/response bodies
- Consider pagination, filtering, and sorting
- Think about versioning strategy`;
  }

  private dataModelingInstructions(): string {
    return `You are an expert data modeler and domain modeling specialist.

Your responsibilities:
- Design clear, maintainable data models
- Generate TaxiLang, JSON Schema, or OpenAPI schemas
- Identify entity relationships and constraints
- Validate models for consistency and completeness

Follow these principles:
- Use descriptive, domain-specific names
- Document all fields with clear descriptions
- Identify required vs. optional fields
- Define proper types and constraints
- Consider normalization and relationships
- Think about data integrity and validation`;
  }
}
```

```typescript
// src/prompts/components/history.tsx
import {
  PromptElement,
  UserMessage,
  AssistantMessage,
  BasePromptElementProps,
  PrioritizedList,
  PromptPiece
} from '@vscode/prompt-tsx';
import { ChatContext, ChatRequestTurn, ChatResponseTurn } from 'vscode';

interface HistoryProps extends BasePromptElementProps {
  history: ChatContext['history'];
  newer: number;
  older: number;
  passPriority: true;
}

export class History extends PromptElement<HistoryProps> {
  render(): PromptPiece {
    const history: (UserMessage | AssistantMessage)[] = [];

    for (const turn of this.props.history) {
      if (turn instanceof ChatRequestTurn) {
        history.push(<UserMessage>{turn.prompt}</UserMessage>);
      } else if (turn instanceof ChatResponseTurn) {
        const content = this.extractContent(turn);
        history.push(
          <AssistantMessage name={turn.participant}>
            {content}
          </AssistantMessage>
        );
      }
    }

    // Split into older and newer messages
    const olderMessages = history.slice(0, -2);
    const newerMessages = history.slice(-2);

    return (
      <>
        {olderMessages.length > 0 && (
          <PrioritizedList priority={this.props.older} descending={false}>
            {olderMessages}
          </PrioritizedList>
        )}
        {newerMessages.length > 0 && (
          <PrioritizedList priority={this.props.newer} descending={false}>
            {newerMessages}
          </PrioritizedList>
        )}
      </>
    );
  }

  private extractContent(turn: ChatResponseTurn): string {
    // Extract text content from response
    return turn.response
      .map(part => {
        if ('value' in part && typeof part.value === 'string') {
          return part.value;
        }
        return '';
      })
      .join('');
  }
}
```

```typescript
// src/prompts/components/context.tsx
import {
  PromptElement,
  UserMessage,
  BasePromptElementProps,
  PromptSizing
} from '@vscode/prompt-tsx';
import * as vscode from 'vscode';

interface ContextProps extends BasePromptElementProps {
  files?: vscode.TextDocument[];
  workspace?: vscode.WorkspaceFolder;
}

export class Context extends PromptElement<ContextProps> {
  async render(state: void, sizing: PromptSizing) {
    const contextParts: string[] = [];

    // Include workspace context
    if (this.props.workspace) {
      contextParts.push(`Working in workspace: ${this.props.workspace.name}`);
    }

    // Include relevant files (with token budget awareness)
    if (this.props.files && this.props.files.length > 0) {
      const availableTokens = sizing.tokenBudget - sizing.tokensUsed;
      const filesContext = await this.getFilesContext(
        this.props.files,
        availableTokens
      );
      contextParts.push(filesContext);
    }

    return (
      <UserMessage priority={70}>
        {contextParts.join('\n\n')}
      </UserMessage>
    );
  }

  private async getFilesContext(
    files: vscode.TextDocument[],
    maxTokens: number
  ): Promise<string> {
    const parts: string[] = ['**Current files:**'];
    let usedTokens = 20; // Rough estimate for header

    for (const file of files) {
      const fileName = path.basename(file.uri.fsPath);
      const content = file.getText();
      
      // Rough token estimation (4 chars ≈ 1 token)
      const estimatedTokens = content.length / 4;

      if (usedTokens + estimatedTokens > maxTokens) {
        // Truncate content to fit
        const availableChars = (maxTokens - usedTokens) * 4;
        const truncated = content.substring(0, availableChars);
        parts.push(`\n\`${fileName}\` (truncated):\n\`\`\`\n${truncated}\n...\n\`\`\``);
        break;
      }

      parts.push(`\n\`${fileName}\`:\n\`\`\`\n${content}\n\`\`\``);
      usedTokens += estimatedTokens;
    }

    return parts.join('\n');
  }
}
```

### 3. Compose the Final Prompt

```typescript
// src/prompts/api-design-prompt.tsx
import {
  PromptElement,
  UserMessage,
  BasePromptElementProps,
  renderPrompt
} from '@vscode/prompt-tsx';
import { ChatContext } from 'vscode';
import { BaseInstructions } from './components/base-instructions';
import { History } from './components/history';
import { Context } from './components/context';

interface ApiDesignPromptProps extends BasePromptElementProps {
  history: ChatContext['history'];
  userQuery: string;
  files?: vscode.TextDocument[];
  workspace?: vscode.WorkspaceFolder;
}

export class ApiDesignPrompt extends PromptElement<ApiDesignPromptProps> {
  render() {
    return (
      <>
        {/* Base instructions - highest priority */}
        <BaseInstructions domain="api-design" priority={100} />

        {/* Recent history - high priority */}
        <History
          history={this.props.history}
          passPriority
          older={0}
          newer={80}
          flexGrow={2}
          flexReserve="/5"
        />

        {/* Current user query - very high priority */}
        <UserMessage priority={90}>
          {this.props.userQuery}
        </UserMessage>

        {/* Context from workspace - medium priority */}
        <Context
          files={this.props.files}
          workspace={this.props.workspace}
          priority={70}
          flexGrow={1}
        />
      </>
    );
  }
}

// Helper function to render the prompt
export async function renderApiDesignPrompt(
  props: ApiDesignPromptProps,
  model: vscode.LanguageModelChat,
  token: vscode.CancellationToken
): Promise<vscode.LanguageModelChatMessage[]> {
  const { messages } = await renderPrompt(
    ApiDesignPrompt,
    props,
    { modelMaxPromptTokens: model.maxInputTokens },
    model,
    token
  );

  return messages;
}
```

### 4. Use the Prompt in Chat Handler

```typescript
// src/chat/handlers/generate-with-tsx.ts
import { renderApiDesignPrompt } from '../../prompts/api-design-prompt';

async function handleGenerateWithTsx(
  request: vscode.ChatRequest,
  context: vscode.ChatContext,
  stream: vscode.ChatResponseStream,
  token: vscode.CancellationToken
): Promise<DapaResult> {
  try {
    const [model] = await vscode.lm.selectChatModels({
      vendor: 'copilot',
      family: 'gpt-4o'
    });

    // Get current workspace and files
    const workspace = vscode.workspace.workspaceFolders?.[0];
    const activeEditor = vscode.window.activeTextEditor;
    const files = activeEditor ? [activeEditor.document] : [];

    // Render the prompt using TSX components
    const messages = await renderApiDesignPrompt(
      {
        history: context.history,
        userQuery: request.prompt,
        files,
        workspace
      },
      model,
      token
    );

    // Send request with the rendered prompt
    const response = await model.sendRequest(messages, {}, token);

    // Stream the response
    for await (const fragment of response.text) {
      stream.markdown(fragment);
    }

    return { command: 'generate', success: true };
  } catch (err) {
    if (err instanceof vscode.LanguageModelError) {
      stream.markdown(`⚠️ ${err.message}`);
    }
    return { command: 'generate', success: false };
  }
}
```

---

## AI-Powered Smart Actions

### 1. Completion Provider for API Paths

```typescript
// src/features/openapi/completions/path-completion.ts
import * as vscode from 'vscode';

export class APIPathCompletionProvider implements vscode.CompletionItemProvider {
  async provideCompletionItems(
    document: vscode.TextDocument,
    position: vscode.Position,
    token: vscode.CancellationToken
  ): Promise<vscode.CompletionItem[]> {
    const line = document.lineAt(position.line).text;
    
    // Only provide AI completions when typing API paths
    if (!line.includes('"paths"') && !line.match(/^\s*"\/\w*/)) {
      return [];
    }

    try {
      const [model] = await vscode.lm.selectChatModels({
        vendor: 'copilot',
        family: 'gpt-4o-mini'
      });

      // Get context from document
      const context = document.getText(new vscode.Range(0, 0, position.line, 0));

      const messages = [
        vscode.LanguageModelChatMessage.User(
          `Suggest 3 RESTful API path completions based on this OpenAPI specification context.

Context:
\`\`\`json
${context}
\`\`\`

Rules:
- Follow REST conventions
- Match existing path patterns
- Include common HTTP methods
- Output format: path|method|description (one per line)
- Example: /users/{id}/orders|GET|Get user's orders`
        )
      ];

      const response = await model.sendRequest(messages, {}, token);

      let suggestions = '';
      for await (const fragment of response.text) {
        suggestions += fragment;
      }

      // Parse suggestions and create completion items
      const items: vscode.CompletionItem[] = [];
      const lines = suggestions.split('\n').filter(l => l.trim());

      for (const line of lines) {
        const parts = line.split('|');
        if (parts.length >= 3) {
          const [path, method, description] = parts;
          
          const item = new vscode.CompletionItem(
            path.trim(),
            vscode.CompletionItemKind.Value
          );
          
          item.detail = `${method.trim()} - AI suggested`;
          item.documentation = new vscode.MarkdownString(description.trim());
          item.insertText = new vscode.SnippetString(
            `"${path.trim()}": {\n  "${method.trim().toLowerCase()}": {\n    "summary": "$1",\n    "responses": {\n      "200": {\n        "description": "$2"\n      }\n    }\n  }\n}`
          );
          
          items.push(item);
        }
      }

      return items;
    } catch (err) {
      // Silently fail for completions
      return [];
    }
  }
}
```

### 2. Diagnostic Provider with AI Suggestions

```typescript
// src/features/openapi/diagnostics/ai-diagnostic.ts
import * as vscode from 'vscode';

export class AIEnhancedDiagnosticProvider {
  private diagnosticCollection: vscode.DiagnosticCollection;

  constructor() {
    this.diagnosticCollection = vscode.languages.createDiagnosticCollection('dapa-ai');
  }

  async updateDiagnostics(document: vscode.TextDocument): Promise<void> {
    if (!this.isOpenAPIDocument(document)) {
      return;
    }

    try {
      const [model] = await vscode.lm.selectChatModels({
        vendor: 'copilot',
        family: 'gpt-4o'
      });

      const messages = [
        vscode.LanguageModelChatMessage.User(
          `Analyze this OpenAPI specification for potential issues.

Focus on:
- Missing required fields
- Inconsistent naming
- Security concerns
- Missing descriptions
- Improper HTTP methods

Specification:
\`\`\`json
${document.getText()}
\`\`\`

Output format: line|severity|message (one per line)
Example: 15|warning|Missing description for endpoint
Severity values: error, warning, info`
        )
      ];

      const response = await model.sendRequest(messages, {}, new vscode.CancellationTokenSource().token);

      let analysis = '';
      for await (const fragment of response.text) {
        analysis += fragment;
      }

      // Parse and create diagnostics
      const diagnostics: vscode.Diagnostic[] = [];
      const lines = analysis.split('\n').filter(l => l.trim());

      for (const line of lines) {
        const parts = line.split('|');
        if (parts.length >= 3) {
          const [lineNum, severity, message] = parts;
          
          const lineNumber = parseInt(lineNum.trim(), 10) - 1;
          if (lineNumber >= 0 && lineNumber < document.lineCount) {
            const range = document.lineAt(lineNumber).range;
            
            let diagSeverity = vscode.DiagnosticSeverity.Information;
            if (severity.trim() === 'error') {
              diagSeverity = vscode.DiagnosticSeverity.Error;
            } else if (severity.trim() === 'warning') {
              diagSeverity = vscode.DiagnosticSeverity.Warning;
            }

            const diagnostic = new vscode.Diagnostic(
              range,
              message.trim(),
              diagSeverity
            );
            diagnostic.source = 'DAPA AI';
            
            diagnostics.push(diagnostic);
          }
        }
      }

      this.diagnosticCollection.set(document.uri, diagnostics);
    } catch (err) {
      // Clear diagnostics on error
      this.diagnosticCollection.clear();
    }
  }

  private isOpenAPIDocument(document: vscode.TextDocument): boolean {
    const text = document.getText();
    return text.includes('"openapi"') && text.includes('"paths"');
  }

  dispose() {
    this.diagnosticCollection.dispose();
  }
}
```

---

## Best Practices Summary

### ✅ DO

**Chat Participant:**
- ✅ Use clear, specific commands (`/generate`, `/validate`)
- ✅ Provide domain context in system prompts
- ✅ Stream responses for smooth UX
- ✅ Add buttons for common actions
- ✅ Implement follow-up suggestions
- ✅ Handle errors gracefully
- ✅ Use participant detection for natural language

**Language Model API:**
- ✅ Use `gpt-4o-mini` for quick interactions (code actions, hover)
- ✅ Use `gpt-4o` for complex tasks (generation, validation)
- ✅ Check model availability with `selectChatModels`
- ✅ Handle `LanguageModelError` appropriately
- ✅ Respect user consent (call during user actions)
- ✅ Show progress for long operations

**Tools:**
- ✅ Define clear, specific tool purposes
- ✅ Use descriptive names and descriptions
- ✅ Provide complete JSON schemas for inputs
- ✅ Add appropriate tags for discoverability
- ✅ Handle errors and edge cases
- ✅ Return structured results

**Prompt Engineering:**
- ✅ Use priority-based pruning with prompt-tsx
- ✅ Split history (recent vs. older)
- ✅ Reserve token budget for context
- ✅ Use `flexGrow` for cooperative sizing
- ✅ Provide clear instructions in prompts
- ✅ Include relevant examples
- ✅ Request specific output formats

### ❌ DON'T

- ❌ Don't call Language Model API in tests (rate limits)
- ❌ Don't expect deterministic responses
- ❌ Don't show AI errors to users without context
- ❌ Don't make requests without user action (consent)
- ❌ Don't forget to handle offline scenarios
- ❌ Don't rely on specific model versions forever
- ❌ Don't use AI for everything (overkill for simple tasks)
- ❌ Don't forget to add telemetry for success measurement

### 🎯 Success Metrics

Monitor these metrics:
- **Chat engagement**: Request count, command usage
- **User satisfaction**: Feedback (helpful/unhelpful)
- **Response quality**: Acceptance rate of suggestions
- **Error rate**: Failed requests, timeouts
- **Feature usage**: Which AI features are used most
- **Performance**: Response time, token usage

### 📊 Example Telemetry

```typescript
// src/telemetry/ai-telemetry.ts
import * as vscode from 'vscode';

export class AITelemetry {
  private logger: vscode.TelemetryLogger;

  constructor(context: vscode.ExtensionContext) {
    this.logger = vscode.env.createTelemetryLogger({
      // Telemetry implementation
    });
  }

  trackChatRequest(command: string, success: boolean, duration: number) {
    this.logger.logUsage('chat.request', {
      command,
      success: success ? 'true' : 'false',
      duration: duration.toString()
    });
  }

  trackFeedback(command: string, kind: vscode.ChatResultFeedbackKind) {
    this.logger.logUsage('chat.feedback', {
      command,
      kind: kind === vscode.ChatResultFeedbackKind.Unhelpful ? 'unhelpful' : 'helpful'
    });
  }

  trackToolInvocation(toolId: string, success: boolean) {
    this.logger.logUsage('tool.invocation', {
      toolId,
      success: success ? 'true' : 'false'
    });
  }
}
```

---

This AI integration makes DAPA a powerful, intelligent assistant for API design and data modeling, seamlessly integrating with VSCode's Copilot experience while maintaining practical, focused functionality.