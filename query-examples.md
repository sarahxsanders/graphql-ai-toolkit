# How to structure GraphQL query examples

Guidelines for creating AI-friendly documentation with complete, working examples.

## Table of contents

- [The problem](#the-problem)
- [Query example template](#query-example-template)
- [Example: Before vs. after](#example-before-vs-after)
- [Key principles](#key-principles)
- [Mutation example structure](#mutation-example-structure)
- [Document organization tips](#document-organization-tips)
- [Test your examples](#test-your-examples)
- [Quick checklist](#quick-checklist)

## The problem

Schema fragments as query examples don't show context:

```graphql
type User {
  posts(first: Int, after: String): PostConnection
}
```

AI benefits from complete, working examples with context to generate correct
queries.

## Query example template

Use this template for query examples in your documentation:

### Context header

```markdown
## [Query name]

**Use case:** [Brief description of when to use this query]
**Performance:** [Timing and caching info]  
**Rate limit:** [Any applicable limits]
**Complexity:** [Query cost if available]
```

### Complete query

Always include:

- Realistic operation name
- All variables with types and defaults
- Complete field selection
- Comments explaining non-obvious parts

```graphql
query YourOperationName($var1: Type!, $var2: Type = defaultValue) {
  field(arg: $var1) {
    # Include enough fields to be realistic
    subfield1
    subfield2
    
    # Show relationships
    relatedField {
      id
      name
    }
  }
}
```

### Variables section

```markdown
```json
{
  "var1": "realistic_example_value",
  "var2": "another_example"
}
```

### Response

```markdown
```json
{
  "data": {
    "field": {
      // Show the shape of successful response
    }
  }
}
```

### Error examples (if needed)

```markdown
```json
{
  "errors": [{
    "message": "Specific error message",
    "extensions": {
      "suggestion": "How to fix it"
    }
  }]
}
```

## Example: Before vs. after

### Before

Instead of documenting query examples like this:

```markdown
## Get user posts

{
  user(id: ID) {
    posts {
      title
    }
  }
}
```

### After

Document query examples like this:

```markdown
## Get user posts

- **Use case**: Display user profile with recent activity
- **Performance**: ~100ms, cacheable for 5 minutes
- **Rate limit**: 1000 requests/hour per user

```graphql
query GetUserPosts($userId: ID!, $limit: Int = 10) {
  user(id: $userId) {
    id
    name
    posts(first: $limit, orderBy: { createdAt: DESC }) {
      edges {
        node {
          id
          title
          excerpt
          createdAt
        }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
}

Variables:

```json
{
  "userId": "user_123",
  "limit": 5
}

Response:

```json
{
  "data": {
    "user": {
      "id": "user_123",
      "name": "Jane Developer",
      "posts": {
        "edges": [
          {
            "node": {
              "id": "post_456",
              "title": "My Latest Post",
              "excerpt": "This is a summary...",
              "createdAt": "2025-03-15T10:30:00Z"
            },
            "cursor": "cursor_abc123"
          }
        ],
        "pageInfo": {
          "hasNextPage": true,
          "endCursor": "cursor_abc123"
        }
      }
    }
  }
}

```

## Key principles

### Make examples complete

Don't:

```graphql
{
  posts {
    title
  }
}
```

Do:

```graphql
query RecentPosts($limit: Int = 20) {
  posts(first: $limit, orderBy: { createdAt: DESC }) {
    edges {
      node {
        id
        title
        excerpt
        createdAt
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### Include realistic variables

Don't:

```json
{
  "id": "123"
}
```

Do:

```json
{
  "userId": "user_clkj2n4x80001",
  "limit": 20,
  "includeInactive": false
}
```

### Show response structure

Always include the expected response shape so AI knows what success looks like.

### Add performance context

```markdown
**Performance:** ~50ms, cacheable for 10 minutes
**Query cost:** 15 points (limit: 1000)
**Best used for:** Dashboard views, user profiles
**Avoid for:** Real-time data, frequently changing content
```

### Document relationships

```graphql
query PostWithAuthor($postId: ID!) {
  post(id: $postId) {
    id
    title
    # Always show how to traverse relationships
    author {
      id
      name
      # Include enough fields to be useful
      avatar
      isVerified
    }
    # Show connection patterns
    comments(first: 5) {
      edges {
        node {
          id
          content
          author {
            name
          }
        }
      }
    }
  }
}
```

## Mutation example structure

### Template

```markdown
### [Mutation name]
**Use case:** [What this accomplishes]
**Permissions:** [Required roles/scopes]
**Side effects:** [What else happens]
```

### Complete example

```markdown
#### Create Blog Post
**Use case:** Publish new content with validation
**Permissions:** Requires "content:write" scope
**Side effects:** Sends notifications to followers, updates search index

```graphql
mutation CreatePost($input: CreatePostInput!) {
  createPost(input: $input) {
    # Always show success data
    post {
      id
      title
      slug
      status
      createdAt
    }
    
    # Always include error handling
    userErrors {
      field
      message
      code
    }
  }
}

Input:

```json
{
  "input": {
    "title": "Getting Started with GraphQL",
    "content": "Content here...",
    "status": "PUBLISHED",
    "tags": ["tutorial", "graphql"]
  }
}

Success response:

```json
{
  "data": {
    "createPost": {
      "post": {
        "id": "post_789",
        "title": "Getting Started with GraphQL",
        "slug": "getting-started-with-graphql",
        "status": "PUBLISHED",
        "createdAt": "2025-03-15T10:30:00Z"
      },
      "userErrors": []
    }
  }
}

Validation error:

```json
{
  "data": {
    "createPost": {
      "post": null,
      "userErrors": [
        {
          "field": "title",
          "message": "Title must be between 5 and 100 characters",
          "code": "TOO_SHORT"
        }
      ]
    }
  }
}
```

## Document organization tips

### Group by use case, not schema

Instead of "User type queries", use "Profile management queries".

### Start with most common

Put your most frequently used queries first.

### Cross-reference related examples

```markdown
**See also:**
- [Update User Profile](mutations.md#update-profile)
- [User Authentication](auth-queries.md#login)
- [Pagination Best Practices](pagination.md)
```

### Include anti-patterns

```markdown
## Common mistakes

### Don't: Query without error handling

```graphql
mutation CreatePost($input: CreatePostInput!) {
  createPost(input: $input) {
    post {
      id
    }
    # Missing: userErrors field
  }
}

### Do: Always include error fields

```graphql
mutation CreatePost($input: CreatePostInput!) {
  createPost(input: $input) {
    post {
      id
      title
    }
    userErrors {
      field
      message
    }
  }
}
```

## Test your examples

### Validate all examples

Every code example should be:

- Syntactically correct
- Executable against your schema
- Using realistic data

### AI testing

Paste your examples into several models and ask:

- "Generate a similar query for [different use case]"
- "What would the variables be for this query?"
- "How would I modify this for [variation]?"

If AI struggles or generates incorrect queries, your examples need more context.

## Quick checklist

For each query example, ensure you have:

- [ ] Clear use case description
- [ ] Complete, named operation
- [ ] All variables with realistic values
- [ ] Enough fields to be useful
- [ ] Expected response structure
- [ ] Performance/caching context
- [ ] Error scenarios (for mutations)
- [ ] Related query references

After reading your example, AI should be able to generate correct, similar queries for related use cases.
