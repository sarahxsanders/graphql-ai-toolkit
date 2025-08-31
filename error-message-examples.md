## Authentication errors

```json
{
  "errors": [{
    "message": "Authentication required",
    "extensions": {
      "code": "UNAUTHENTICATED",
      "suggestion": "Include 'Authorization: Bearer YOUR_TOKEN' header",
      "docs": "https://docs.example.com/auth"
    }
  }]
}
```

## Validation errors

```json
{
  "errors": [{
    "message": "Field 'first' must be between 1 and 100",
    "extensions": {
      "code": "VALIDATION_ERROR",
      "field": "first",
      "suggestion": "Use first: 10 for typical pagination",
      "docs": "https://docs.example.com/pagination"
    }
  }]
}
```

## Rate limiting

```json
{
  "errors": [{
    "message": "Rate limit exceeded",
    "extensions": {
      "code": "RATE_LIMITED",
      "resetTime": "2025-03-15T10:35:00Z",
      "suggestion": "Implement exponential backoff or reduce request frequency"
    }
  }]
}
```

## Invalid argument error example

```
Unknown argument "limit" on field "posts".
Did you mean "first"? This field uses cursor-based pagination.

Example: posts(first: 10, after: "cursor123")

Cursor pagination guide: /docs/pagination-patterns
Available arguments: first (Int), after (String), orderBy (PostOrderBy)
```

## Type mismatch error example

```
Field "first" expects an integer, but received string "abc".
Valid examples:

first: 10 (recommended for most use cases)
first: 50 (maximum allowed value)

Common mistake: Don't use quotes around numbers in GraphQL
❌ first: "10"
✅ first: 10
```

## Missing required field error example

```
The "createPost" mutation returns data that must be selected.

You need to specify which fields you want back:

mutation {
createPost(input: { title: "Hello", content: "World" }) {
post {
id
title
createdAt
}
errors {
field
message
}
}
}

Mutation patterns guide: /docs/mutations
```

## Depth limit error example

```
Your query is too deeply nested (depth: 12, maximum: 8).

This usually happens with recursive relationships like:
post > author > posts > author > posts...

Solutions:

1. Use fragments to structure your query
2. Fetch data in multiple requests
3. Use our pre-built query patterns: /docs/query-patterns

Query depth calculator: /tools/depth-calculator
```
