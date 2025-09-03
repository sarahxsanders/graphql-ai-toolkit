# Docker MCP Setup for GraphQL servers

Use Docker MCP Toolkit to containerize, test, and deploy your GraphQL MCP servers with enhanced security and easier distribution.

## When to use Docker for your GraphQL MCP server

Consider Docker when you need:

- Easy distribution: Share your MCP server with others without dependency issues
- Isolated testing: Run multiple GraphQL MCP servers without conflicts
- Production deployment: Deploy securely with built-in resource limits
- AI client integration: Connect to Claude Desktop, VS Code, and other MCP clients through a unified gateway

## Prerequisites

- Docker Desktop 4.45.0+ with MCP Toolkit enabled
- Your GraphQL MCP server
- AI client for testing

## Enable Docker MCP Toolkit

1. Update Docker Desktop to version 4.45.0 or later
2. Enable MCP Toolkit:
   - Open Docker Desktop Settings
   - Navigate to Beta features  
   - Enable "Docker MCP Toolkit"
   - Apply changes and restart Docker Desktop
3. Verify: Look for "MCP Toolkit" in Docker Desktop's sidebar

## Containerize your GraphQL MCP server

### Create Dockerfile

In your existing GraphQL MCP server project, create a `Dockerfile`:

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy TypeScript config and source
COPY tsconfig.json ./
COPY src/ ./src/

# Build TypeScript
RUN npm run build

# Create non-root user for security
RUN addgroup -g 1001 -S mcp && \
    adduser -S mcpuser -u 1001 -G mcp

# Switch to non-root user
USER mcpuser

# Health check (optional)
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD node -e "console.log('Server healthy')" || exit 1

# Start MCP server
CMD ["npm", "start"]
```

### Build container image

```bash
# Build your GraphQL MCP server image
docker build -t my-graphql-mcp .
```

## Run with Docker MCP toolkit

### Option A: Run directly with Docker

```bash
# Run your containerized GraphQL MCP server
docker run -d \
  --name my-graphql-mcp \
  -e GRAPHQL_ENDPOINT=https://api.yourdomain.com/graphql \
  -e AUTH_TOKEN=your_token_here \
  my-graphql-mcp

# Test the server
docker logs my-graphql-mcp
```

### Option B: Use Docker MCP Toolkit gateway

The Docker MCP Toolkit acts as a gateway, allowing you to manage multiple MCP servers and connect them to AI clients easily.

1. Add your server to the toolkit:

   ```bash
   # Using Docker MCP CLI (if available)
   docker mcp server add my-graphql-mcp
   ```

2. Or manage through Docker Desktop:

   - Open Docker Desktop → MCP Toolkit
   - Your running server should appear in the local servers section
   - Configure environment variables through the UI

## Connect to AI clients

### Connect Claude Desktop

1. Open Docker Desktop → MCP Toolkit → Clients tab
2. Find Claude Desktop in the client list
3. Click "Connect". This automatically configures Claude's MCP settings
4. Restart Claude Desktop to load the new configuration

Your Claude Desktop will now have access to your GraphQL MCP server tools through the Docker gateway.

### Connect VS Code

1. Create MCP configuration in your project:

   ```bash
   # Initialize MCP for VS Code
   docker mcp init vscode
   ```

2. This creates `.vscode/mcp.json`:

   ```json
   {
     "mcp": {
       "servers": {
         "MCP_DOCKER": {
           "command": "docker",
           "args": ["mcp", "gateway", "run"],
           "type": "stdio"
         }
       }
     }
   }
   ```

3. Open VS Code in your project and use Agent mode to interact with your GraphQL tools

### Connect other clients

Docker MCP Toolkit supports:

- Cursor: Similar to VS Code setup
- Continue.dev: Automatic detection through the gateway
- Gordon: Built-in client in Docker Desktop

## Test Your GraphQL tools

### Claude Desktop

Ask Claude to interact with your GraphQL API:

- "Introspect the GraphQL schema and show me available queries"
- "Execute a query to get user information for ID 123"  
- "Validate this GraphQL query: `query { users { name email } }`"
- "Analyze the complexity of this query and suggest optimizations"

### VS Code agent mode

In VS Code, open a chat and select Agent mode, then:

- "Use the GraphQL server to show me the API schema"
- "Execute a GraphQL mutation to create a new user"
- "Help me write an optimized query for the product catalog"

## Environment configuration

### Using Docker Compose (Recommended)

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  graphql-mcp:
    build: .
    container_name: my-graphql-mcp
    environment:
      - GRAPHQL_ENDPOINT=${GRAPHQL_ENDPOINT}
      - AUTH_TOKEN=${AUTH_TOKEN}
      - SERVER_NAME=my-graphql-api
    restart: unless-stopped
    networks:
      - mcp-network

networks:
  mcp-network:
    driver: bridge
```

Create `.env`:

```bash
GRAPHQL_ENDPOINT=https://api.yourdomain.com/graphql
AUTH_TOKEN=your_api_token_here
```

Run with:

```bash
docker-compose up -d
```

### Using Docker secrets (Production)

For sensitive credentials:

```bash
# Create secrets
echo "your_api_token" | docker secret create graphql_token -

# Update docker-compose.yml
services:
  graphql-mcp:
    build: .
    environment:
      - GRAPHQL_ENDPOINT=https://api.yourdomain.com/graphql
      - AUTH_TOKEN_FILE=/run/secrets/graphql_token
    secrets:
      - graphql_token

secrets:
  graphql_token:
    external: true
```

Update your server code to read from file:

```typescript
const AUTH_TOKEN = process.env.AUTH_TOKEN || 
  (process.env.AUTH_TOKEN_FILE ? 
    fs.readFileSync(process.env.AUTH_TOKEN_FILE, 'utf8').trim() : 
    undefined);
```

## Debug and monitor

### View Logs

```bash
# Container logs
docker logs my-graphql-mcp

# Follow logs in real-time
docker logs -f my-graphql-mcp

# Docker Compose logs
docker-compose logs graphql-mcp
```

### Resource monitoring

```bash
# Check resource usage
docker stats my-graphql-mcp

# Inspect container details
docker inspect my-graphql-mcp
```

### Health checks

Test your server health:

```bash
# Test container health
docker exec my-graphql-mcp node -e "console.log('Health check passed')"

# Test GraphQL connectivity
docker exec my-graphql-mcp curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"query":"{ __typename }"}' \
  $GRAPHQL_ENDPOINT
```

## Publishing (Optional)

### Submit to Docker MCP Catalog

To share your GraphQL MCP server with the community:

1. Prepare your server:

   - Ensure it follows MCP best practices
   - Add comprehensive documentation
   - Test thoroughly with multiple GraphQL APIs

2. Submit to Docker MCP Registry:

   - Visit [github.com/docker/mcp-registry](https://github.com/docker/mcp-registry)
   - Follow the submission guidelines
   - Choose Docker-built (recommended) or community-built tier

3. Benefits of publishing:

   - Automatic security scanning and signing
   - Distribution through Docker Hub
   - Discovery in Docker MCP Catalog
   - Access to millions of developers

## Troubleshoot

### Common issues

Server not appearing in Docker MCP Toolkit:

- Verify container is running: `docker ps`
- Check logs for startup errors: `docker logs container-name`
- Ensure MCP Toolkit is enabled in Docker Desktop

GraphQL connection failures:

- Test endpoint accessibility from container: `docker exec container-name curl endpoint`
- Verify environment variables: `docker exec container-name env | grep GRAPHQL`
- Check authentication credentials and format

AI client not connecting:

- Restart AI client after connecting through Docker MCP Toolkit
- Verify MCP configuration files were created correctly
- Check Docker MCP Toolkit gateway is running

## More resources

- Docker MCP documentation: [docs.docker.com/ai/mcp-catalog-and-toolkit/](https://docs.docker.com/ai/mcp-catalog-and-toolkit/)
- MCP specification: [spec.modelcontextprotocol.io](https://spec.modelcontextprotocol.io/)
- Docker Community: [forums.docker.com](https://forums.docker.com)
