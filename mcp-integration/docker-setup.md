# Docker setup for GraphQL MCP servers

Connect your GraphQL API to AI tools using Docker's MCP infrastructure, a fast path from API to AI-powered automation.

## Table of contents

- [Why Docker for GraphQL MCP](#why-docker-for-graphql-mcp)
- [Quick start with MCP gateway](#quick-start-with-mcp-gateway)
- [Containerize your GraphQL server](#containerize-your-graphql-server)
- [Connect to the MCP gateway](#connect-to-the-mcp-gateway)
- [Test with AI clients](#test-with-ai-clients)
- [Contribute to MCP registry](#contribute-to-mcp-registry)
- [Production deployment](#production-deployment)
- [Resources](#resources)

## Why Docker for GraphQL MCP

The [Docker MCP gateway](https://github.com/docker/mcp-gateway) provides the infrastructure you need to connect GraphQL APIs to AI tools:

- **Unified access point**: Connect multiple GraphQL endpoints through a single gateway
- **Zero configuration for clients**: AI tools like Claude, Cursor, and VS Code connect once to access all your APIs
- **Community ecosystem**: Discover GraphQL MCP servers in the [MCP registry](https://github.com/docker/mcp-registry)
- **Security by default**: Containers provide isolation, resource limits, and credential management

## Quick start with MCP gateway

### Clone and run the gateway

```bash
# Get the MCP gateway
git clone https://github.com/docker/mcp-gateway
cd mcp-gateway

# Start the gateway
docker compose up -d

# Verify it's running
docker compose logs gateway
```

The gateway is now ready to manage your GraphQL MCP servers.

### Add your GraphQL server

The gateway can run any MCP server from the catalog. Try the GitHub GraphQL server:

```
# Pull an existing GraphQL MCP server from the catalog
docker pull mcp/github:latest

# Or use your own GraphQL server (see next section)
docker pull your-username/your-graphql-mcp:latest
```

## Containerize your GraphQL server

### Create your MCP server

First, ensure your GraphQL server implements the MCP protocol. Here's a minimal example:

```typescript
// src/server.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import { GraphQLClient } from 'graphql-request';

const server = new Server(
  {
    name: 'my-graphql-mcp',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// Define GraphQL tools
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: 'execute_query',
        description: 'Execute a GraphQL query',
        inputSchema: {
          type: 'object',
          properties: {
            query: { type: 'string' },
            variables: { type: 'object' },
          },
          required: ['query'],
        },
      },
    ],
  };
});

const transport = new StdioServerTransport();
server.connect(transport);
```

### Create Dockerfile

```
FROM node:18-alpine

WORKDIR /app

# Copy and install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy source
COPY . .

# Build if using TypeScript
RUN npm run build

# Security: Run as non-root
USER node

# MCP servers use stdio transport
CMD ["node", "dist/server.js"]
```

### Build and test

```bash
# Build your image
docker build -t my-graphql-mcp .

# Test locally
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | \
  docker run -i my-graphql-mcp
```

## Connect to the MCP gateway

### Configure the gateway

Add your GraphQL server to the gateway's configuration:

```yaml
# mcp-gateway/config/servers.yaml
servers:
  - name: my-graphql-api
    image: my-graphql-mcp:latest
    env:
      GRAPHQL_ENDPOINT: https://api.example.com/graphql
      AUTH_TOKEN: ${GRAPHQL_TOKEN}
    
  # Add multiple GraphQL endpoints
  - name: github-graphql
    image: mcp/github:latest
    env:
      GITHUB_TOKEN: ${GITHUB_TOKEN}
      
  - name: shopify-graphql
    image: my-shopify-mcp:latest
    env:
      SHOP_DOMAIN: myshop.myshopify.com
      API_TOKEN: ${SHOPIFY_TOKEN}
```

### Restart the gateway

```bash
cd mcp-gateway
docker compose restart
```

Your GraphQL servers are now available through the gateway!

## Test with AI clients

### Claude Desktop

The gateway automatically configures Claude Desktop:

```json
{
  "mcpServers": {
    "docker-gateway": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "mcp-gateway"],
      "env": {
        "GATEWAY_URL": "http://localhost:8080"
      }
    }
  }
}
```

Test it with the following prompts:

- "Use my GraphQL API to fetch user data"
- "Execute a mutation to create a new post"
- "Show me the schema for my GraphQL endpoint"

### VS Code

Enable in VS Code settings:

```
{
  "mcp": {
    "servers": {
      "docker-gateway": {
        "command": "docker",
        "args": ["mcp", "gateway", "run"],
        "type": "stdio"
      }
    }
  }
}
```

## Contribute to MCP registry

Share your GraphQL MCP server with the community through the [Docker MCP registry](https://github.com/docker/mcp-registry).

### Prepare your server

Ensure your GraphQL MCP server:

- Has comprehensive tool descriptions
- Includes error handling with AI-friendly messages
- Supports common GraphQL patterns (introspection, pagination, mutations)
- Has a clear README with examples

### Submit to registry

```bash
# Fork the registry
git clone https://github.com/docker/mcp-registry
cd mcp-registry

# Create your server definition
mkdir servers/graphql/your-api
```

Create `servers/graphql/your-api/metadata.yaml`:

```
name: your-graphql-api
namespace: community  # or 'mcp' if Docker-built
description: MCP server for YourAPI GraphQL endpoint
image: docker.io/yourusername/your-graphql-mcp
categories:
  - graphql
  - api
  - developer-tools
tools:
  - name: execute_query
    description: Execute GraphQL queries
  - name: introspect_schema
    description: Get the complete GraphQL schema
  - name: analyze_complexity
    description: Analyze query performance
configuration:
  - name: GRAPHQL_ENDPOINT
    description: Your GraphQL API endpoint
    required: true
  - name: AUTH_TOKEN
    description: Authentication token
    required: false
    secret: true
```

### Submit pull request

```bash
git add .
git commit -m "Add YourAPI GraphQL MCP server"
git push origin main
```

Once approved, your server will be available:

- In Docker Hub under `mcp/your-graphql-api` (if Docker-built)
- In Docker Desktop's MCP Toolkit
- To millions of developers worldwide

## Production deployment

### Use Docker Compose

```yaml
version: '3.8'

services:
  mcp-gateway:
    image: docker/mcp-gateway:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/config
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - LOG_LEVEL=info
      
  graphql-mcp:
    image: my-graphql-mcp:latest
    environment:
      - GRAPHQL_ENDPOINT=${GRAPHQL_ENDPOINT}
    secrets:
      - graphql_token
    restart: unless-stopped
    
secrets:
  graphql_token:
    file: ./secrets/token.txt
```

### Monitor and debug

```
# View gateway logs
docker logs mcp-gateway

# Check connected servers
curl http://localhost:8080/servers

# Test a specific tool
curl -X POST http://localhost:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"server": "my-graphql-api", "tool": "execute_query", "args": {...}}'
```

## Resources

- [Docker MCP gateway](https://github.com/docker/mcp-gateway): Run and manage MCP servers
- [MCP registry](https://github.com/docker/mcp-registry): Share your GraphQL servers with the community
- [Contributing guide](https://github.com/docker/mcp-registry/blob/main/CONTRIBUTING.md): Submit your server to the registry
- [MCP specification](https://spec.modelcontextprotocol.io): Protocol details
- [Docker MCP catalog](https://hub.docker.com/mcp): Browse available servers
- [MCP toolkit docs](https://docs.docker.com/ai/mcp-catalog-and-toolkit/): Docker Desktop integration
- [GitHub issues](https://github.com/docker/mcp-gateway/issues): Report bugs or request features
- [Docker community](https://forums.docker.com): Ask questions and share experiences
