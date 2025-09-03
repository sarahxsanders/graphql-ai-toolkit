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

Common mistake: Don't use quotes around numbers in GraphQL (first: "10" vs. first: 10)
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

## Query complexity limits

```
Query exceeds complexity limit (cost: 245, maximum: 200).

Expensive fields in your query:
- user.posts: 50 points (consider using first: 5 instead of first: 20)
- posts.comments: 120 points (consider removing or paginating)

Optimization tips:
1. Reduce pagination sizes (first/last arguments)
2. Remove expensive nested fields
3. Use our query cost calculator: /tools/query-cost

Query optimization guide: /docs/performance
```

## Unknown field errors

```
Cannot query field "authorName" on type "Post".

Did you mean one of these?
- author { name }
- author { displayName }

Available fields on Post:
- id, title, content, createdAt
- author (returns User type)
- tags (returns [String])

Schema explorer: /docs/schema/post
```

## Enum validation errors

```
Value "DESCENDING" is not valid for enum "SortOrder".

Valid options:
- ASC (ascending order)
- DESC (descending order)

Example: posts(orderBy: { createdAt: DESC })
Sorting guide: /docs/sorting
```

## Input object validation errors

```
Field "CreatePostInput.publishedAt" expected type "DateTime" but got "2025-13-45".

DateTime format: ISO 8601 (YYYY-MM-DDTHH:MM:SSZ)
Valid example: "2025-03-15T10:30:00Z"

Common mistakes:
❌ "2025-13-45" (invalid month/day)
❌ "March 15, 2025" (wrong format)
✅ "2025-03-15T10:30:00Z" (correct format)

DateTime formatting guide: /docs/datetime
```

## Permission/authorization errors

```
You don't have permission to access field "User.email".

This field requires:
- Role: ADMIN or USER_MANAGER
- Scope: "user:email:read"

Your current permissions:
- Role: USER
- Scopes: "user:profile:read", "posts:read"

Permissions guide: /docs/authorization
Contact your admin to request additional permissions.
```
