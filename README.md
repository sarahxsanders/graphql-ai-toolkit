_A practical toolkit for designing GraphQL APIs, documentation, and developer tooling that work seamlessly with AI-powered development workflows. Created for GraphQLConf 2025._

## Quick start

- [Schema audit checklist](schema-audit-checklist.md)
- [Documentation templates](documentation-templates.md)
- [Error message examples](error-message-examples.md)
- [Query examples](query-examples.md)
- [MCP integration guide](mcp-integration/basic-server.md)

## The methodology

By creating self-explanatory schemas, context-rich documentation, and better developer tooling, we can create better GraphQL developer experiences.

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

### Enhance GraphiQL

Your GraphiQL instance might need a makeover! 

- Natural language queries: Add an input bar that converts "show me all users with recent posts" into proper GraphQL syntax. This lets developers think in outcomes, not query language.
- Performance warnings: Show real-time feedback like "this query hits 500+ records" or "complexity: 127/100" directly in the editor. Catch expensive operations before they hit production.
- Copy as curl/fetch: One-click export to executable code formats that AI tools can immediately use.
- Mock data generation: Generate realistic responses from any query without hitting your backend. This can help AI understand response shapes and enables offline development.

### Model Context Protocol (MCP)

The future of AI-API integration is here. GraphQL's typed schema and introspection make
it a natural fit for Model Context Protocol.

#### Example MCP + GraphQL workflow

1. Developer asks Claude: "Check if my user feed is optimal".
2. AI introspects your GraphQL schema.
3. AI validates query structure and suggests improvements.
4. Developer ships optimized code.

## Additional resources

- [Docker State of Application Development Report](https://www.docker.com/blog/2025-docker-state-of-app-dev/): Read developer experience insights from thousands of surveys devleopers
- [Docker MCP Catalog and Toolkit](https://docs.docker.com/ai/mcp-catalog-and-toolkit/): Learn about Docker's MCP product offerings
- [Docker setup for GraphQL MCP server](mcp-integration/mcp-integration/docker-setup.md)

## Have feedback?

I'd love to hear from you! If you have suggestions, encounter issues, or want to contribute improvements to the docs in this toolkit:

- Submit an issue: [Create a new issue](../../issues/new) in this repository
- Provide feedback: Use the issue title format `[Feedback] Your topic here`
- Report bugs: Use the issue title format `[Bug] Description of the problem`
- Request features: Use the issue title format `[Feature Request] Your idea here`

When submitting feedback, please include:
- The section of the guide you're referencing
- Your use case or what you're trying to accomplish
- Any error messages or unexpected behavior
- Your environment details (if relevant)

Your feedback helps me improve this guide for everyone. Thank you for contributing!

## About

Created by Sarah Sanders for GraphQLConf 2025.
