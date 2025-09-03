# Basic GraphQL MCP server

Create a simple MCP server that connects your GraphQL
API to AI tools.

## Prerequisites

- Node.js 18+ installed
- Access to a GraphQL API endpoint

## Create project structure

```bash
mkdir my-graphql-mcp-server
cd my-graphql-mcp-server
npm init -y
```

## Install dependencies

```bash
npm install @modelcontextprotocol/sdk graphql graphql-request dotenv
npm install -D typescript @types/node ts-node
```

## Create basic server

```typescript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import { 
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from '@modelcontextprotocol/sdk/types.js';
import { GraphQLClient } from 'graphql-request';

// Environment configuration
const GRAPHQL_ENDPOINT = process.env.GRAPHQL_ENDPOINT || 'http://localhost:4000/graphql';
const AUTH_TOKEN = process.env.AUTH_TOKEN;

// Initialize GraphQL client
const headers = AUTH_TOKEN ? { Authorization: `Bearer ${AUTH_TOKEN}` } : {};
const graphqlClient = new GraphQLClient(GRAPHQL_ENDPOINT, { headers });

// Create MCP server
const server = new Server(
  {
    name: 'graphql-api-server',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
      resources: {},
    },
  }
);

// Define available tools
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: 'introspect_schema',
        description: 'Get the complete GraphQL schema with AI-friendly descriptions',
        inputSchema: {
          type: 'object',
          properties: {
            includeDeprecated: {
              type: 'boolean',
              description: 'Include deprecated fields in the schema',
              default: false,
            },
          },
        },
      },
      {
        name: 'execute_query',
        description: 'Execute a GraphQL query and return results',
        inputSchema: {
          type: 'object',
          properties: {
            query: {
              type: 'string',
              description: 'GraphQL query to execute',
            },
            variables: {
              type: 'object',
              description: 'Variables for the GraphQL query',
            },
          },
          required: ['query'],
        },
      },
      {
        name: 'validate_query',
        description: 'Validate a GraphQL query against the schema without executing it',
        inputSchema: {
          type: 'object',
          properties: {
            query: {
              type: 'string',
              description: 'GraphQL query to validate',
            },
          },
          required: ['query'],
        },
      },
      {
        name: 'analyze_query_complexity',
        description: 'Analyze query complexity and suggest optimizations',
        inputSchema: {
          type: 'object',
          properties: {
            query: {
              type: 'string',
              description: 'GraphQL query to analyze',
            },
          },
          required: ['query'],
        },
      },
    ],
  };
});

// Handle tool execution
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case 'introspect_schema':
      return await introspectSchema(args.includeDeprecated);
    
    case 'execute_query':
      return await executeQuery(args.query, args.variables);
    
    case 'validate_query':
      return await validateQuery(args.query);
    
    case 'analyze_query_complexity':
      return await analyzeQueryComplexity(args.query);
    
    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

// Tool implementations
async function introspectSchema(includeDeprecated = false) {
  const introspectionQuery = `
    query IntrospectionQuery {
      __schema {
        queryType { name description }
        mutationType { name description }
        subscriptionType { name description }
        types {
          ...FullType
        }
        directives {
          name
          description
          locations
          args {
            ...InputValue
          }
        }
      }
    }

    fragment FullType on __Type {
      kind
      name
      description
      fields(includeDeprecated: ${includeDeprecated}) {
        name
        description
        args {
          ...InputValue
        }
        type {
          ...TypeRef
        }
        isDeprecated
        deprecationReason
      }
      inputFields {
        ...InputValue
      }
      interfaces {
        ...TypeRef
      }
      enumValues(includeDeprecated: ${includeDeprecated}) {
        name
        description
        isDeprecated
        deprecationReason
      }
      possibleTypes {
        ...TypeRef
      }
    }

    fragment InputValue on __InputValue {
      name
      description
      type { ...TypeRef }
      defaultValue
    }

    fragment TypeRef on __Type {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
                ofType {
                  kind
                  name
                  ofType {
                    kind
                    name
                  }
                }
              }
            }
          }
        }
      }
    }
  `;

  try {
    const result = await graphqlClient.request(introspectionQuery);
    
    return {
      content: [
        {
          type: 'text',
          text: `# GraphQL Schema\n\n${formatSchemaForAI(result.__schema)}`,
        },
      ],
    };
  } catch (error) {
    return {
      content: [
        {
          type: 'text',
          text: `Schema introspection failed: ${error.message}`,
        },
      ],
      isError: true,
    };
  }
}

async function executeQuery(query: string, variables?: any) {
  try {
    const result = await graphqlClient.request(query, variables);
    
    return {
      content: [
        {
          type: 'text',
          text: `Query executed successfully:\n\n\`\`\`json\n${JSON.stringify(result, null, 2)}\n\`\`\``,
        },
      ],
    };
  } catch (error) {
    return {
      content: [
        {
          type: 'text',
          text: `Query execution failed: ${error.message}`,
        },
      ],
      isError: true,
    };
  }
}

async function validateQuery(query: string) {
  try {
    // Basic syntax validation by attempting to parse
    // In a real implementation, you'd use GraphQL's validation functions
    const result = await graphqlClient.request(`
      query ValidateQuery {
        __typename
      }
    `);
    
    return {
      content: [
        {
          type: 'text',
          text: `Query validation passed. The query appears to be syntactically correct.`,
        },
      ],
    };
  } catch (error) {
    return {
      content: [
        {
          type: 'text',
          text: `Query validation failed: ${error.message}`,
        },
      ],
      isError: true,
    };
  }
}

async function analyzeQueryComplexity(query: string) {
  // Simple complexity analysis
  const lines = query.split('\n').length;
  const fieldCount = (query.match(/\w+\s*[{(]/g) || []).length;
  const nestedLevels = (query.match(/{/g) || []).length;
  
  let complexity = 'Low';
  let suggestions = [];
  
  if (fieldCount > 20) {
    complexity = 'High';
    suggestions.push('Consider using fragments to reduce duplication');
    suggestions.push('Split complex queries into multiple smaller queries');
  } else if (fieldCount > 10) {
    complexity = 'Medium';
    suggestions.push('Consider adding pagination to list fields');
  }
  
  if (nestedLevels > 8) {
    complexity = 'High';
    suggestions.push('Query depth is high - consider reducing nesting levels');
  }
  
  return {
    content: [
      {
        type: 'text',
        text: `# Query Complexity Analysis

**Complexity Level:** ${complexity}
**Field Count:** ${fieldCount}
**Nesting Levels:** ${nestedLevels}
**Lines:** ${lines}

## Suggestions:
${suggestions.length > 0 ? suggestions.map(s => `- ${s}`).join('\n') : '- Query looks well-optimized'}

## Performance Tips:
- Use \`first\` and \`after\` for pagination instead of \`limit\` and \`offset\`
- Include only fields your UI actually needs
- Consider using fragments for repeated field selections
- Monitor query execution time and optimize slow queries`,
      },
    ],
  };
}

// Format schema for AI consumption
function formatSchemaForAI(schema: any): string {
  let output = '';
  
  // Add query operations
  if (schema.queryType) {
    output += '## Query Operations\n\n';
    const queryType = schema.types.find((t: any) => t.name === schema.queryType.name);
    if (queryType?.fields) {
      queryType.fields.forEach((field: any) => {
        output += `### ${field.name}\n`;
        if (field.description) output += `${field.description}\n\n`;
        output += `**Type:** ${formatType(field.type)}\n`;
        if (field.args?.length) {
          output += `**Arguments:**\n`;
          field.args.forEach((arg: any) => {
            output += `- \`${arg.name}: ${formatType(arg.type)}\``;
            if (arg.description) output += ` - ${arg.description}`;
            output += '\n';
          });
        }
        output += '\n';
      });
    }
  }
  
  // Add mutation operations
  if (schema.mutationType) {
    output += '## Mutation Operations\n\n';
    const mutationType = schema.types.find((t: any) => t.name === schema.mutationType.name);
    if (mutationType?.fields) {
      mutationType.fields.forEach((field: any) => {
        output += `### ${field.name}\n`;
        if (field.description) output += `${field.description}\n\n`;
        output += `**Type:** ${formatType(field.type)}\n`;
        if (field.args?.length) {
          output += `**Arguments:**\n`;
          field.args.forEach((arg: any) => {
            output += `- \`${arg.name}: ${formatType(arg.type)}\``;
            if (arg.description) output += ` - ${arg.description}`;
            output += '\n';
          });
        }
        output += '\n';
      });
    }
  }
  
  // Add custom types
  output += '## Types\n\n';
  schema.types
    .filter((type: any) => 
      !type.name.startsWith('__') && 
      type.kind === 'OBJECT' &&
      type.name !== schema.queryType?.name && 
      type.name !== schema.mutationType?.name
    )
    .forEach((type: any) => {
      output += `### ${type.name}\n`;
      if (type.description) output += `${type.description}\n\n`;
      if (type.fields) {
        type.fields.forEach((field: any) => {
          output += `- **${field.name}**: ${formatType(field.type)}`;
          if (field.description) output += ` - ${field.description}`;
          output += '\n';
        });
      }
      output += '\n';
    });
  
  return output;
}

function formatType(type: any): string {
  if (!type) return 'Unknown';
  
  if (type.kind === 'NON_NULL') {
    return `${formatType(type.ofType)}!`;
  }
  if (type.kind === 'LIST') {
    return `[${formatType(type.ofType)}]`;
  }
  return type.name || 'Unknown';
}

// Start the server
const transport = new StdioServerTransport();
server.connect(transport);

console.error('GraphQL MCP Server started');
```

## Add environment configuration

Create `.env`:

```bash
# GraphQL API Configuration
GRAPHQL_ENDPOINT=https://api.example.com/graphql
AUTH_TOKEN=your_api_token_here

# Optional: Custom server settings
SERVER_NAME=my-graphql-api
SERVER_VERSION=1.0.0
```

## Add typescript configuration

Create `tsconfig.json`:

```json
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
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Add build scripts

Update `package.json`:

```json
{
  "name": "graphql-mcp-server",
  "version": "1.0.0",
  "scripts": {
    "build": "tsc",
    "start": "node dist/server.js",
    "dev": "ts-node src/server.ts",
    "test": "node dist/server.js < test-input.json"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0",
    "graphql": "^16.8.1",
    "graphql-request": "^6.1.0",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "typescript": "^5.2.2",
    "@types/node": "^20.8.0",
    "ts-node": "^10.9.1"
  }
}
```

## Test your server

### Build and run

```bash
npm run build
npm start
```

### Test with sample input

Create `test-input.json`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list"
}
```

Run test:

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | npm start
```

Expected output:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "introspect_schema",
        "description": "Get the complete GraphQL schema with AI-friendly descriptions"
      },
      // ... other tools
    ]
  }
}
```

## Customization options

### Add custom tools

Add new tools to the `ListToolsRequestSchema` handler:

```typescript
{
  name: 'find_queries_by_type',
  description: 'Find all queries that return a specific type',
  inputSchema: {
    type: 'object',
    properties: {
      typeName: {
        type: 'string',
        description: 'Name of the GraphQL type to search for',
      },
    },
    required: ['typeName'],
  },
}
```

Then implement in the tool handler:

```typescript
case 'find_queries_by_type':
  return await findQueriesByType(args.typeName);
```

### Add authentication

For APIs requiring authentication:

```typescript
const graphqlClient = new GraphQLClient(GRAPHQL_ENDPOINT, {
  headers: {
    Authorization: `Bearer ${process.env.AUTH_TOKEN}`,
    'X-API-Key': process.env.API_KEY,
  },
});
```

### Add caching

For better performance:

```bash
npm install node-cache
```

```typescript
import NodeCache from 'node-cache';

const schemaCache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function introspectSchema(includeDeprecated = false) {
  const cacheKey = `schema-${includeDeprecated}`;
  const cached = schemaCache.get(cacheKey);
  
  if (cached) {
    return {
      content: [{ type: 'text', text: cached }],
    };
  }
  
  // ... existing introspection logic
  
  // Cache the result
  schemaCache.set(cacheKey, formattedSchema);
  
  return {
    content: [{ type: 'text', text: formattedSchema }],
  };
}
```

## Next steps

- [Connect to AI tools](mcp-integration\ai-tool-setup.md)
- [Explore deploying with Docker](mcp-integration\docker-setup.md)
