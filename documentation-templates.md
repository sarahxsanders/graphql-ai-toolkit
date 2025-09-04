# GraphQL documentation templates

Use the following templates as a starting point for creating AI-ready GraphQL documentation. These templates are intended to be getting started
suggestions, and not prescribed best practices for every use case.

## Table of contents

- [API overview template](#api-overview-template)
- [Query pattern template](#query-pattern-template)

## API overview template

For your API overviews:

- Start with a 'Quick start' page that explains the very basics of your API.
- Then explain your domain model: what entities exist and how they are used/relate.
- Link to common query patterns. This can be a query pattern library, a few high-quality example queries, or a link to other use case guides with queries embedded in them.

Example:

```markdown
# [API name] GraphQL API

## Quick start

This API uses GraphQL with cursor-based pagination, follows Relay specifications, and 
requires authentication via Bearer tokens.

**Endpoint:** https://api.example.com/graphql
**Auth:** Bearer token in Authorization header
**Rate Limits:** 1000 requests/hour per user

## Domain model

- **Users**: Application users with profiles and preferences
- **Posts**: User-generated content with rich media support  
- **Comments**: Threaded discussions on posts
- **Reactions**: Likes, shares, and other engagement actions

## Common query patterns

See [Query Patterns](#query-patterns) for complete, working examples of typical use 
cases.
```

## Query pattern template

When providing query examples, outline the query's:

- Use case
- Performance implications
- Rate limit implications
- Complexity score
- Full query, variable example, and response example

Example:

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

### Variables

```json
{
  "userId": "user_123",
  "first": 20,
  "after": "cursor_abc123"
}

### Response

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
