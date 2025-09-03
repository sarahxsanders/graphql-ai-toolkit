Use the following templates as a starting point for creating AI-ready GraphQL documentation. These templates are intended to be getting started
suggestions, and not prescribed best practices for every use case.

## API overview template

```markdown
# [API name] GraphQL API

## Quick start

This API uses GraphQL with cursor-based pagination, follows Relay specifications, and 
requires authentication via Bearer tokens.

**Endpoint:** https://api.example.com/graphql
**Auth:** Bearer token in Authorization header
**Rate Limits:** 1000 requests/hour per user

## Core domain model

- **Users**: Application users with profiles and preferences
- **Posts**: User-generated content with rich media support  
- **Comments**: Threaded discussions on posts
- **Reactions**: Likes, shares, and other engagement actions

## Common query patterns

See [Query Patterns](#query-patterns) for complete, working examples of typical use 
cases.
```

## Query pattern template

```markdown
## User feed query

**Use Case:** Display a user's personalized content feed
**Performance:** ~100ms average, cacheable for 5 minutes
**Rate Limit:** 60 requests/hour per user
**Complexity Score:** 8/10

### Query

```graphql
query UserFeed($userId: ID!, $first: Int = 10, $after: String) {
  user(id: $userId) {
    id
    name
    feed(first: $first, after: $after) {
      edges {
        node {
          id
          title
          content
          createdAt
          author {
            id
            name
            avatar
          }
          reactions {
            totalCount
            viewerReaction
          }
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

## Variables

```json
{
  "userId": "user_123",
  "first": 20,
  "after": "cursor_abc123"
}

## Response

```json
{
  "data": {
    "user": {
      "id": "user_123",
      "name": "Jane Doe",
      "feed": {
        "edges": [...],
        "pageInfo": {
          "hasNextPage": true,
          "endCursor": "cursor_def456"
        }
      }
    }
  }
}
```

## Performance notes

- Use fragments for repeated user information
- Limit first parameter to 50 for optimal performance
- Consider using `reactions { totalCount }` instead of full reaction details for list views

## Common mistakes

### Don’t do this

- `feed(limit: 100)`: this API uses cursor pagination, not limit/offset
- Query without pageInfo: you'll lose pagination context

### Do this

- Always include pageInfo for proper pagination handling
