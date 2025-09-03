# AI tool setup guide for GraphQL MCP

Connect your GraphQL MCP server to popular AI clients 
and development tools to enable intelligent interactions with your GraphQL API.

## Prerequisites

- Completed GraphQL MCP server from the [basic server guide](basic-server.md)
- Built and tested server: `npm run build` and verified it starts without errors
- AI client software

## Overview

Your GraphQL MCP server can connect to various AI clients, each offering different integration patterns:

- Claude Desktop: Standalone AI assistant with MCP support
- VS Code: Code editor with AI agent capabilities  
- Cursor: AI-powered code editor
- Continue.dev: AI coding assistant extension
- Custom integrations: Build your own MCP client connections

## Claude Desktop

Claude Desktop provides an easy way to test your GraphQL MCP server with a full-featured AI assistant.

### Install Claude Desktop

1. Download Claude Desktop from [claude.ai/download](https://claude.ai/download).
2. Install and sign in with your Anthropic account.
3. Verify installation. Claude should start without errors.

### Configure MCP connection

1. Locate Claude's configuration directory:
   - macOS: `~/Library/Application Support/Claude/`
   - Windows: `%APPDATA%\Claude\`
   - Linux: `~/.config/Claude/`

2. Create or edit `claude_desktop_config.json`:

   ```json
   {
     "mcpServers": {
       "my-graphql-api": {
         "command": "node",
         "args": ["/path/to/your/graphql-mcp-server/dist/server.js"],
         "env": {
           "GRAPHQL_ENDPOINT": "https://api.yourdomain.com/graphql",
           "AUTH_TOKEN": "your_api_token_here"
         }
       }
     }
   }
   ```

3. Update the path. Replace `/path/to/your/graphql-mcp-server/` with your actual project path.

4. Set environment variables. Update `GRAPHQL_ENDPOINT` and `AUTH_TOKEN` for your API.

### Test Claude integration

1. Restart Claude Desktop to load the new configuration.
2. Look for MCP indicator. Claude should show your server is connected.
3. Test your GraphQL tools:

    - Ask Claude to introspect your API: `Can you show me the GraphQL schema for my API? Use the introspect_schema tool?`
    - Execute a GraphQL query: `Execute this GraphQL query for me...`
    - Analyze query complexity: `Analyze the complexity of this GraphQL query and suggest optimizations...`

## VS Code with agent mode

VS Code's AI agent can use your GraphQL MCP server for intelligent code assistance and API interactions.

### Setup VS Code integration

1. Install VS Code (latest version) if not already installed.
2. Enable MCP support (check VS Code's AI features documentation for current setup).
3. Create MCP configuration in your project. Create `.vscode/mcp.json`:

```json
{
  "mcp": {
    "servers": {
      "graphql-api": {
        "command": "node",
        "args": ["../dist/server.js"],
        "type": "stdio",
        "env": {
          "GRAPHQL_ENDPOINT": "https://api.yourdomain.com/graphql",
          "AUTH_TOKEN": "your_api_token_here"
        }
      }
    }
  }
}
```

4. Add to `.gitignore`:

   ```
   .vscode/mcp.json
   ```

### Use VS Code agent mode

1. Open VS Code in a project that uses your GraphQL API.
2. Start a chat and select **Agent mode**.
3. Test your GraphQL tools:

    - Ask Claude to introspect your API: `Can you show me the GraphQL schema for my API? Use the introspect_schema tool?`
    - Execute a GraphQL query: `Execute this GraphQL query for me...`
    - Analyze query complexity: `Analyze the complexity of this GraphQL query and suggest optimizations...`

## Cursor editor

Cursor provides AI-powered code editing with MCP server integration.

### Configure Cursor

1. Install Cursor from [cursor.so](https://cursor.so).
2. Configure MCP settings in Cursor's AI settings.
3. Add your GraphQL MCP server. In Cursor's settings, add MCP server configuration:

```json
{
  "mcp": {
    "servers": [
      {
        "name": "graphql-api", 
        "command": "node",
        "args": ["/path/to/your/dist/server.js"],
        "env": {
          "GRAPHQL_ENDPOINT": "https://api.yourdomain.com/graphql",
          "AUTH_TOKEN": "your_token"
        }
      }
    ]
  }
}
```

4. Test your GraphQL tools:

    - Ask Claude to introspect your API: `Can you show me the GraphQL schema for my API? Use the introspect_schema tool?`
    - Execute a GraphQL query: `Execute this GraphQL query for me...`
    - Analyze query complexity: `Analyze the complexity of this GraphQL query and suggest optimizations...`

## Continue.dev extension

Continue.dev provides AI coding assistance that can leverage your GraphQL MCP server.

### Install Continue.dev

1. Install in your IDE.
2. Configure MCP connection. Edit Continue's configuration to include your MCP server:

```json
{
  "models": [
    // your existing models
  ],
  "mcpServers": [
    {
      "name": "graphql-api",
      "command": "node",
      "args": ["/absolute/path/to/dist/server.js"],
      "env": {
        "GRAPHQL_ENDPOINT": "https://api.yourdomain.com/graphql", 
        "AUTH_TOKEN": "your_token"
      }
    }
  ]
}
```

3. Test your GraphQL tools:

    - Ask Claude to introspect your API: `Can you show me the GraphQL schema for my API? Use the introspect_schema tool?`
    - Execute a GraphQL query: `Execute this GraphQL query for me...`
    - Analyze query complexity: `Analyze the complexity of this GraphQL query and suggest optimizations...`

## Custom MCP client integration

For advanced use cases, you can build custom integrations with your GraphQL MCP server.

### Basic MCP client example

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';
import { spawn } from 'child_process';

// Start your GraphQL MCP server as a subprocess
const serverProcess = spawn('node', ['dist/server.js'], {
  stdio: ['pipe', 'pipe', 'pipe'],
  env: {
    ...process.env,
    GRAPHQL_ENDPOINT: 'https://api.yourdomain.com/graphql',
    AUTH_TOKEN: 'your_token'
  }
});

// Connect to the server
const transport = new StdioClientTransport({
  reader: serverProcess.stdout,
  writer: serverProcess.stdin
});

const client = new Client(
  { name: 'my-graphql-client', version: '1.0.0' },
  { capabilities: {} }
);

await client.connect(transport);

// List available tools
const tools = await client.listTools();
console.log('Available GraphQL tools:', tools.tools);

// Execute GraphQL introspection
const schemaResult = await client.callTool('introspect_schema', {
  includeDeprecated: false
});
console.log('GraphQL Schema:', schemaResult.content[0].text);

// Execute a GraphQL query
const queryResult = await client.callTool('execute_query', {
  query: 'query { users(first: 5) { id name email } }',
  variables: {}
});
console.log('Query Result:', queryResult.content[0].text);

// Clean up
serverProcess.kill();
```

### Build web interface

```html
<!DOCTYPE html>
<html>
<head>
    <title>GraphQL MCP Web Client</title>
    <script src="https://unpkg.com/graphql-ws/umd/graphql-ws.min.js"></script>
</head>
<body>
    <div id="app">
        <h1>GraphQL API Explorer</h1>
        
        <button onclick="loadSchema()">Load Schema</button>
        <div id="schema"></div>
        
        <textarea id="queryInput" placeholder="Enter GraphQL query..."></textarea>
        <button onclick="executeQuery()">Execute Query</button>
        <div id="queryResult"></div>
    </div>

    <script>
        // Connect to your MCP server via WebSocket or HTTP proxy
        async function loadSchema() {
            const response = await fetch('/mcp/introspect_schema', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ includeDeprecated: false })
            });
            const result = await response.json();
            document.getElementById('schema').innerHTML = `<pre>${result.content[0].text}</pre>`;
        }
        
        async function executeQuery() {
            const query = document.getElementById('queryInput').value;
            const response = await fetch('/mcp/execute_query', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ query })
            });
            const result = await response.json();
            document.getElementById('queryResult').innerHTML = `<pre>${JSON.stringify(result, null, 2)}</pre>`;
        }
    </script>
</body>
</html>
```

## Testing Your AI tool integration

### Functional tests

Schema introspection:

- Ask the AI to show your GraphQL schema
- Verify all types, queries, and mutations are displayed correctly
- Check that descriptions and deprecation info appear properly

Query execution:

- Test simple queries with no variables
- Test complex queries with variables
- Verify error handling for invalid queries

Query analysis:

- Ask for complexity analysis of various queries
- Test validation of syntactically incorrect queries
- Verify performance suggestions are relevant

Error scenarios:

- Test with invalid GraphQL endpoint
- Test with expired authentication tokens  
- Test with malformed GraphQL queries
- Verify AI handles errors gracefully

### Performance tests

Response times:
- Measure schema introspection time
- Test query execution performance
- Monitor memory usage during long conversations

Concurrent usage:
- Test multiple AI clients using the same MCP server
- Verify no conflicts or race conditions
- Check resource usage under load
