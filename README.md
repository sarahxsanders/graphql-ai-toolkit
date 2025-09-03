_A practical toolkit for designing GraphQL APIs, documentation, and developer tooling that work seamlessly with AI-powered development workflows. Created for GraphQLConf 2025._

## Quick start

- Audit your schema -> [Schema audit checklist](schema-audit-checklist.md)
- Improve your docs -> [Documentation templates](documentation-templates.md)
- Fix your errors -> [Error message examples](error-message-examples.md)
- Add code examples -> [Query examples](query-examples.md)
- Connect with AI -> [MCP integration guide](mcp-integration\basic-server.md)

## The methodology

To make your GraphQL API, documentation, and developer tooling AI-ready, use the following
guidelines.

### Self-explanatory schemas

Transform your schema from technically correct to AI-readable:

```graphql
# Before: AI has to guess
type User {
  posts(first: Int, after: String): PostConnection
}

# After: AI understands immediately  
type User {
  """
  User's posts with cursor-based pagination.
  
  Example: posts(first: 10, after: "cursor123")
  Limits: first must be 1-50 (default: 10)
  Performance: Consider fragments for large result sets
  """
  posts(
    "Number of posts to fetch (1-50, default: 10)"
    first: Int
    "Pagination cursor - use endCursor from pageInfo" 
    after: String
  ): PostConnection
}
```

### Context-rich documentation

Structure docs for AI comprehension with complete, working examples:

- Use cases with performance context
- Complete queries with realistic variables
- Expected response examples
- Common mistake examples

### Intelligent error messages

Turn every error into a teaching moment:

```text
# Instead of:
Unknown argument 'offset' on field 'posts'

# Provide:
Unknown argument 'offset' on field 'posts'. 
Did you mean 'after'? This field uses cursor-based pagination.

Example: posts(first: 10, after: "cursor123")
Learn more: /docs/pagination-patterns
```

## Model Context Protocol (MCP)

The future of AI-API integration is here. GraphQL's typed schema and introspection make
it a natural fit for Model Context Protocol.

### Example MCP + GraphQL workflow

1. Developer asks Claude: "Check if my user feed is optimal".
2. AI introspects your GraphQL schema.
3. AI validates query structure and suggests improvements.
4. Developer ships optimized code.

## Additional resources

- [Docker State of Application Development Report](https://www.docker.com/blog/2025-docker-state-of-app-dev/): Read developer experience insights from thousands of surveys devleopers
- [Docker MCP Catalog and Toolkit](https://docs.docker.com/ai/mcp-catalog-and-toolkit/): Learn about Docker's MCP product offerings

## Contribute

Found a pattern that works well with AI? Have examples of AI-friendly GraphQL designs? We'd love your contributions!

1. Fork this repository.
2. Add your examples or improvements.
3. Submit a pull request.

## About

Created by Sarah Sanders for GraphQLConf 2025.
