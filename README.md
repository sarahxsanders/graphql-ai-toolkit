A practical toolkit for designing GraphQL APIs that work seamlessly with AI-powered development workflows. Created for GraphQLConf 2025.

## The problem

75% of developers now use AI tools (up from 46% last year), but most GraphQL APIs aren't
designed for AI consumers. The result? Frustrated developers with "almost right" AI-generated queries
that fail on first attempt.

The new developer workflow: Ask AI -> Generate -> Iterate -> Ship

Your GraphQL API should support this reality.

## Quick start

- Audit your schema -> Schema audit checklist
- Improve your docs -> Documentation templates
- Fix your errors -> Error message examples
- Add code examples -> Query examples
- Connect with AI -> MCP integration guide

## The three layers

### 1. Self-explanatory schemas

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

### 2. Context-rich documentation

Structure docs for AI comprehension with complete, working examples:

- Use cases with performance context
- Complete queries with realistic variables
- Expected response examples
- Common mistake examples

### 3. Intelligent error messages

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

## Resources

| Resource | Purpose | Best for |
|----------|---------|----------|
| Schema audit checklist | Comprehensive review guide | Making existing schemas AI-friendly |
| Documentation templates | Structured doc formats | Creating new AI-optimized documentation |
| Error message templates | Before/after error improvements | Implementing helpful error responses |
| Query examples | Complete code samples | Providing AI with working patterns to copy |
| MCP integration | Model Context Protocol setup | Connecting GraphQL APIs to AI tools |

## Model Context Protocol (MCP)

The future of AI-API integration is here. GraphQL's typed schema and introspection make
it a natural fit for Model Context Protocol.

### Example MCP + GraphQL workflow

1. Developer asks Claude: "Check if my user feed is optimal"
2. AI introspects your GraphQL schema
3. AI validates query structure and suggests improvements
4. Developer ships optimized code

## Contribute

Found a pattern that works well with AI? Have examples of AI-friendly GraphQL designs? We'd love your contributions!

1. Fork this repository
2. Add your examples or improvements
3. Submit a pull request

## About

Created by Sarah Sanders for GraphQLConf 2025.
