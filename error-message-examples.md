# AI-friendly GraphQL error messages

Use the following error message examples to write AI-friendly GraphQL error messages.

## Table of contents

- [Authentication errors](#authentication-errors)
- [Validation errors](#validation-errors)
- [Rate limiting](#rate-limiting)
- [Invalid argument error example](#invalid-argument-error-example)
- [Type mismatch error example](#type-mismatch-error-example)
- [Missing required field error example](#missing-required-field-error-example)
- [Depth limit error example](#depth-limit-error-example)
- [Query complexity limits](#query-complexity-limits)
- [Unknown field errors](#unknown-field-errors)
- [Enum validation errors](#enum-validation-errors)
- [Input object validation errors](#input-object-validation-errors)
- [Permission/authorization errors](#permissionauthorization-errors)

## Authentication errors

AI optimizations: 

- Explicit solution with exact syntax (`Authorization: Bearer YOUR_TOKEN`)
- Machine-readable error code for categorization
- Direct link to relevant documentation
- Concise yet complete information

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

AI optimizations:

- Clear boundaries (1-100) that AI can use for automatic correction
- Specific field identification for targeted fixes
- Practical example with recommended value
- Contextual documentation link

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

AI optimizations:

- Parseable ISO timestamp for automated scheduling
- Actionable algorithmic suggestion (exponential backoff)
- Clear state information for retry logic

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

AI optimizations:

- "Did you mean" suggestion for immediate correction
- Complete working example with sample values
- Full argument inventory with type information
- Explains the underlying pattern (cursor-based pagination)

```
Unknown argument "limit" on field "posts".
Did you mean "first"? This field uses cursor-based pagination.

Example: posts(first: 10, after: "cursor123")

Cursor pagination guide: /docs/pagination-patterns
Available arguments: first (Int), after (String), orderBy (PostOrderBy)
```

## Type mismatch error example

AI optimization:

- Multiple valid examples with use-case annotations
- Explicit anti-pattern explanation (quotes around numbers)
- Recommended values for different scenarios

```
Field "first" expects an integer, but received string "abc".
Valid examples:

first: 10 (recommended for most use cases)
first: 50 (maximum allowed value)

Common mistake: Don't use quotes around numbers in GraphQL (first: "10" vs. first: 10)
```

## Missing required field error example

AI optimizations:

- Complete, copy-pasteable solution
- Shows full response structure including error handling
- Demonstrates the mutation pattern comprehensively

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

AI optimizations:

- Explains the root cause with a concrete example
- Multiple numbered solutions for different scenarios
- External tool for validation before execution
- Current vs. maximum values for calibration

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

AI optimizations:

- Quantified cost analysis per field
- Specific optimization suggestions with numbers
- Actionable reduction strategies
- Tool reference for pre-validation

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

AI optimizations:

- Multiple correction suggestions
- Complete field inventory with type information
- Shows nested field access pattern
- Direct schema documentation link

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

AI optimizations:

- Complete enum inventory with explanations
- Working example in context
- Clear value mapping (DESCENDING → DESC)

```
Value "DESCENDING" is not valid for enum "SortOrder".

Valid options:
- ASC (ascending order)
- DESC (descending order)

Example: posts(orderBy: { createdAt: DESC })
Sorting guide: /docs/sorting
```

## Input object validation errors

AI optimizations:

- Exact format specification (ISO 8601)
- Multiple failure examples with explanations
- Clear success example
- Visual indicators (✓/✗) for quick scanning

```
Field "CreatePostInput.publishedAt" expected type "DateTime" but got "2025-13-45".

DateTime format: ISO 8601 (YYYY-MM-DDTHH:MM:SSZ)
Valid example: "2025-03-15T10:30:00Z"

Common mistakes:
✗ "2025-13-45" (invalid month/day)
✗ "March 15, 2025" (wrong format)
✓ "2025-03-15T10:30:00Z" (correct format)

DateTime formatting guide: /docs/datetime
```

## Permission/authorization errors

AI optimizations:

- Clear permission requirements
- Current permission state for comparison
- Actionable next steps
- Structured role and scope information

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
