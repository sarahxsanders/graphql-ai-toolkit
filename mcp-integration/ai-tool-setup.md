# AI tool setup guide for GraphQL MCP

Connect your GraphQL MCP server to popular AI clients and development tools to enable intelligent interactions with your GraphQL API.

## Table of contents

- [Prerequisites](#prerequisites)
- [Overview](#overview)
- [Claude Desktop](#claude-desktop)
- [VS Code with GitHub Copilot](#vs-code-with-github-copilot)
- [Cursor editor](#cursor-editor)
- [Continue.dev extension](#continuedev-extension)
- [JetBrains IDEs](#jetbrains-ides)
- [Security considerations](#security-considerations)
- [Custom MCP client integration](#custom-mcp-client-integration)
- [Test your AI tool integration](#test-your-ai-tool-integration)
- [Resources](#resources)

## Prerequisites

- Completed GraphQL MCP server from the [basic server guide](basic-server.md)
- Built and tested server
- AI client software (Claude Desktop, VS Code, Cursor, etc.)
- Node.js v18.x.x or above

## Overview

Your GraphQL MCP server can connect to various AI clients using the Model Context Protocol (MCP), an open standard that enables secure connections between AI assistants and external tools. Each client offers different integration patterns:

- Claude Desktop: Standalone AI assistant with native MCP support
- VS Code: Code editor with GitHub Copilot and agent mode MCP integration
- Cursor: AI-powered code editor with built-in MCP support
- Continue.dev: Open-source AI coding assistant with comprehensive MCP features
- JetBrains IDEs: Built-in MCP server support for external client connections
- Custom integrations: Build your own MCP client connections

## Claude Desktop

Claude Desktop provides a mature MCP integration with a user-friendly interface for managing servers and tools.

### Install Claude Desktop

1. Download Claude Desktop from [claude.ai/download](https://claude.ai/download).
2. Install and sign in with your Anthropic account.
3. Verify installation by launching Claude Desktop.

### Configure MCP connection

1. Locate Claude's configuration directory:
   - macOS: `~/Library/Application Support/Claude/`
   - Windows: `%APPDATA%\Claude\`
   - Linux: `~/.config/Claude/` (when available)

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

3. Update configuration:
   - Replace `/path/to/your/graphql-mcp-server/` with your actual project path
   - Set `GRAPHQL_ENDPOINT` and `AUTH_TOKEN` for your API
   - Environment variables are stored securely by Claude Desktop

### Test Claude integration

1. Restart Claude Desktop to load the new configuration.
2. Look for MCP indicator in the chat input.
3. Click the **MCP indicator** to view available tools.
4. Test your GraphQL tools:
   - "Can you show me the GraphQL schema for my API?"
   - "Execute this GraphQL query: `query { users(first: 5) { id name email } }`"
   - "Analyze the complexity of this query and suggest optimizations"

## VS Code with GitHub Copilot

VS Code integrates MCP through GitHub Copilot's agent mode, providing seamless access to external tools during coding sessions.

### Prerequisites for VS Code

1. Install VS Code (version 1.102 or later).
2. Install GitHub Copilot extension.
3. Enable agent mode by setting `chat.agent.enabled: true` in settings.
4. Verify MCP support is enabled with `chat.mcp.enabled: true` (default).

### Setup VS Code integration

#### Method 1: One-click installation (for published servers)

1. Browse MCP servers:
   - Open **Extensions** view and search for `@mcp`.
   - Select **Browse MCP Servers** or run `MCP: Browse Servers` command.
   - Find your GraphQL MCP server.
   - Click **Install** to automatically configure.

#### Method 2: Manual configuration

1. Create MCP configuration in your project root:
   
   `.vscode/mcp.json`:
   ```json
   {
     "mcpServers": {
       "graphql-api": {
         "command": "node",
         "args": ["../dist/server.js"],
         "env": {
           "GRAPHQL_ENDPOINT": "https://api.yourdomain.com/graphql",
           "AUTH_TOKEN": "your_api_token_here"
         }
       }
     }
   }
   ```

2. Add to `.gitignore`:
   ```gitignore
   .vscode/mcp.json
   ```

3. Alternative: Use MCP: Add Server command:
   - Run `MCP: Add Server` from Command Palette.
   - Follow the guided setup process.

### Use VS Code agent mode

1. Open **Chat** view and select **Agent mode**.
2. Manage tools using the tools dropdown to enable/disable MCP servers.
3. Reference tools explicitly by typing `#tool-name` in your prompt.
4. Test GraphQL integration:
   - "Analyze my GraphQL schema and suggest improvements"
   - "Execute this query and help me optimize it"
   - "Show me the available GraphQL operations"

**Important**: VS Code requires user approval for each tool invocation. You can set approval preferences for current session, workspace, or all future invocations.

## Cursor editor

Cursor provides MCP support with both one-click installation and manual configuration options.

### Install and configure Cursor

1. Install Cursor from [cursor.so](https://cursor.so).
2. Choose your setup method:

#### Method 1: One-click installation (for published servers)

1. Access curated servers:
   - Go to **Settings** → **MCP**.
   - Browse available servers.
   - Install GraphQL MCP server if available.
   - Configure with OAuth if supported.

#### Method 2: Manual configuration

Option A: Project-specific setup

Create `.cursor/mcp.json` in your project root:
```json
{
  "mcpServers": {
    "graphql-api": {
      "command": "node",
      "args": ["/path/to/your/dist/server.js"],
      "env": {
        "GRAPHQL_ENDPOINT": "https://api.yourdomain.com/graphql",
        "AUTH_TOKEN": "your_token"
      }
    }
  }
}
```

Option B: Global configuration

Add to Cursor's global MCP settings through **Settings** → **MCP** → **Add Server**.

### Using Cursor with MCP

1. Verify connection: Check **Settings** → **MCP** for green status indicator.
2. Test tools:
   - "Introspect my GraphQL API and show the schema"
   - "Help me write a complex GraphQL query for user data"
   - "Analyze this GraphQL query performance"

## Continue.dev extension

Continue.dev offers comprehensive MCP support through its agent mode with YAML-based configuration.

### Install Continue.dev

1. Install Continue extension in VS Code or JetBrains.
2. Follow setup guide: [docs.continue.dev](https://docs.continue.dev).

### Configure MCP servers

#### Method 1: Workspace-specific (recommended)

1. Create MCP servers folder:
   ```bash
   mkdir -p .continue/mcpServers
   ```

2. Add GraphQL MCP server configuration:
   
   `.continue/mcpServers/graphql-mcp.yaml`:
   ```yaml
   name: GraphQL MCP Server
   version: 0.0.1
   schema: v1
   mcpServers:
     - name: GraphQL API
       command: node
       args:
         - "/path/to/your/dist/server.js"
       env:
         GRAPHQL_ENDPOINT: "https://api.yourdomain.com/graphql"
         AUTH_TOKEN: "your_token"
   ```

#### Method 2: Main configuration

Add to your main Continue configuration file:

```json
{
  "mcpServers": [
    {
      "name": "GraphQL API",
      "command": "node",
      "args": ["/path/to/your/dist/server.js"],
      "env": {
        "GRAPHQL_ENDPOINT": "https://api.yourdomain.com/graphql",
        "AUTH_TOKEN": "your_token"
      }
    }
  ]
}
```

#### Method 3: Remote servers

For remote MCP servers, use HTTP transports:

```yaml
mcpServers:
  - name: GraphQL API Remote
    type: streamable-http
    url: "https://your-mcp-server.com"
```

### Use Continue.dev with MCP

1. Enable agent mode in Continue.
2. Test GraphQL tools:
   - "Show me the GraphQL schema structure"
   - "Generate TypeScript types from my GraphQL schema"
   - "Help me build a GraphQL subscription"

## JetBrains IDEs

JetBrains IDEs (2025.2+) include built-in MCP server functionality, allowing external clients to connect and control the IDE.

### Enable MCP server in JetBrains

1. Go to **Settings** → **Tools** → **MCP Server**.
2. Enable **MCP Server**.
3. Configure client connections:
   - Auto-configure: Click "Auto-Configure" for Claude Desktop, Cursor, VS Code, etc.
   - Manual setup: Copy SSE or Stdio configurations.

### Connect external clients

Your GraphQL MCP server can run alongside JetBrains' MCP server. Configure external clients (Claude Desktop, Cursor) to connect to both:

1. Your GraphQL MCP server (for GraphQL operations).
2. JetBrains MCP server (for IDE control).

Example combined configuration for Claude Desktop:
```json
{
  "mcpServers": {
    "graphql-api": {
      "command": "node",
      "args": ["/path/to/your/graphql-server/dist/server.js"],
      "env": {
        "GRAPHQL_ENDPOINT": "https://api.yourdomain.com/graphql",
        "AUTH_TOKEN": "your_token"
      }
    },
    "jetbrains-ide": {
      "command": "sse",
      "args": ["http://localhost:34001/sse"]
    }
  }
}
```

## Security considerations

Note some important security practices when using MCP:

### User consent and approval
- Always review tool calls before approving execution.
- Enable manual approval for all tool invocations initially.
- Use session/workspace-specific approvals only after verification.
- Never auto-approve tools that modify data or execute commands.

### API security
- Use environment variables for sensitive tokens and endpoints.
- Implement proper authentication (OAuth, API keys).
- Restrict API permissions to minimum required scope.
- Use HTTPS endpoints only for production GraphQL APIs.

### Development vs production
- Test with development databases first.
- Use read-only configurations when possible.
- Implement rate limiting on your GraphQL endpoint.
- Monitor API usage for unexpected activity.

### Data protection
- Review schema exposure. Ensure no sensitive data in introspection.
- Implement field-level permissions in your GraphQL server.
- Sanitize query responses to prevent data leakage.
- Log MCP tool usage for audit purposes.

## Custom MCP client integration

For advanced use cases, build custom integrations using the MCP SDK.

### Basic MCP client example

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';
import { spawn } from 'child_process';

async function createGraphQLMCPClient() {
  // Start your GraphQL MCP server
  const serverProcess = spawn('node', ['dist/server.js'], {
    stdio: ['pipe', 'pipe', 'pipe'],
    env: {
      ...process.env,
      GRAPHQL_ENDPOINT: 'https://api.yourdomain.com/graphql',
      AUTH_TOKEN: 'your_token'
    }
  });

  // Connect via stdio transport
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

  // Cleanup
  return {
    client,
    cleanup: () => serverProcess.kill()
  };
}
```

### Remote MCP server setup

For production deployments, use streamable HTTP transport:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { SSEClientTransport } from '@modelcontextprotocol/sdk/client/sse.js';

async function createRemoteGraphQLClient() {
  const transport = new SSEClientTransport(
    'https://your-mcp-server.com/sse'
  );

  const client = new Client(
    { name: 'remote-graphql-client', version: '1.0.0' },
    { capabilities: {} }
  );

  await client.connect(transport);
  return client;
}
```

### Web interface for GraphQL MCP

```html
<!DOCTYPE html>
<html>
<head>
    <title>GraphQL MCP Web Client</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        textarea { width: 100%; height: 200px; font-family: monospace; }
        button { padding: 10px 20px; margin: 10px 0; }
        .result { background: #f5f5f5; padding: 15px; border-radius: 5px; }
        .error { background: #ffebee; color: #c62828; }
    </style>
</head>
<body>
    <div id="app">
        <h1>GraphQL MCP Explorer</h1>
        
        <div>
            <button onclick="loadSchema()">Load GraphQL Schema</button>
            <button onclick="clearResults()">Clear Results</button>
        </div>
        
        <div id="schema" class="result" style="display: none;">
            <h3>GraphQL Schema</h3>
            <pre id="schemaContent"></pre>
        </div>
        
        <div>
            <h3>Execute GraphQL Query</h3>
            <textarea id="queryInput" placeholder="Enter GraphQL query...">
query {
  users(first: 10) {
    id
    name
    email
  }
}</textarea>
            <button onclick="executeQuery()">Execute Query</button>
        </div>
        
        <div id="queryResult" class="result" style="display: none;"></div>
        <div id="error" class="result error" style="display: none;"></div>
    </div>

    <script>
        // MCP client configuration
        const mcpEndpoint = '/api/mcp'; // Your MCP server endpoint
        
        async function callMCPTool(toolName, args = {}) {
            try {
                const response = await fetch(`${mcpEndpoint}/${toolName}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(args)
                });
                
                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
                }
                
                return await response.json();
            } catch (error) {
                showError(`Error calling ${toolName}: ${error.message}`);
                throw error;
            }
        }
        
        async function loadSchema() {
            try {
                showLoading('Loading schema...');
                const result = await callMCPTool('introspect_schema', { 
                    includeDeprecated: false 
                });
                
                document.getElementById('schemaContent').textContent = 
                    result.content[0].text;
                document.getElementById('schema').style.display = 'block';
                hideError();
            } catch (error) {
                console.error('Schema loading failed:', error);
            }
        }
        
        async function executeQuery() {
            const query = document.getElementById('queryInput').value.trim();
            if (!query) {
                showError('Please enter a GraphQL query');
                return;
            }
            
            try {
                showLoading('Executing query...');
                const result = await callMCPTool('execute_query', { 
                    query,
                    variables: {}
                });
                
                const resultDiv = document.getElementById('queryResult');
                resultDiv.innerHTML = `
                    <h3>Query Result</h3>
                    <pre>${JSON.stringify(result.content[0], null, 2)}</pre>
                `;
                resultDiv.style.display = 'block';
                hideError();
            } catch (error) {
                console.error('Query execution failed:', error);
            }
        }
        
        function showError(message) {
            const errorDiv = document.getElementById('error');
            errorDiv.textContent = message;
            errorDiv.style.display = 'block';
        }
        
        function hideError() {
            document.getElementById('error').style.display = 'none';
        }
        
        function showLoading(message) {
            // You could implement a loading indicator here
            console.log(message);
        }
        
        function clearResults() {
            document.getElementById('schema').style.display = 'none';
            document.getElementById('queryResult').style.display = 'none';
            hideError();
        }
    </script>
</body>
</html>
```

## Test your AI tool integration

### Functional tests

Schema introspection:
- [ ] Ask AI to show GraphQL schema: "Display my GraphQL API schema"
- [ ] Verify all types, queries, mutations appear correctly
- [ ] Check descriptions and deprecation warnings
- [ ] Confirm field arguments and return types

Query execution:
- [ ] Test simple queries: "Execute: `query { users { id name } }`"
- [ ] Test queries with variables: "Get user by ID with this query..."
- [ ] Test mutations: "Create a new user with this mutation..."
- [ ] Verify error handling for invalid queries

Query analysis:
- [ ] Request complexity analysis: "Analyze this query's complexity and suggest optimizations"
- [ ] Test performance suggestions: "How can I make this query faster?"
- [ ] Validation checks: "Is this GraphQL query syntactically correct?"

Error scenarios:
- Invalid GraphQL endpoint → Should show clear error message
- Expired authentication → Should prompt for new credentials
- Malformed queries → Should explain syntax errors
- Network issues → Should handle gracefully with retry suggestions

### Performance tests

Response times:
- Schema introspection: < 5 seconds for typical APIs
- Simple queries: < 2 seconds
- Complex queries: Monitor and optimize based on API performance
- Tool discovery: < 1 second

Resource usage:
- Monitor memory usage during long conversations
- Check for connection leaks
- Verify proper cleanup of resources
- Test with multiple concurrent AI clients

Rate limiting:
- Verify your GraphQL API rate limits are respected
- Test behavior under rate limit conditions
- Implement proper backoff strategies
- Monitor API usage patterns

## Resources

- MCP documentation: [modelcontextprotocol.io](https://modelcontextprotocol.io)
- Claude Desktop support: Check Claude Desktop logs and configuration.
- VS Code issues: Review VS Code MCP documentation and GitHub issues.
- Continue.dev community: Join Discord for support.
- GraphQL debugging: Use GraphiQL or similar tools.

Remember to always test in development environments first and follow security best practices when deploying to production!
