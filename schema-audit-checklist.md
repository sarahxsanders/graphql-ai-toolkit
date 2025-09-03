Use the following checklists to audit your GraphQL schema for AI-readiness.

## Type-level audit

- [ ]  Every type has a description explaining what it represents
- [ ]  Business context included (e.g., “Represents a user account in the system”)
- [ ]  Relationship clarity (e.g., “Links to Posts via the author field”)
- [ ]  Lifecycle information where relevant (e.g., “Created when user registers, deleted after 30 days of inactivity”)

## Field-level audit

- [ ]  Every field has a description explaining its purpose
- [ ]  Data format specified for complex fields (e.g., “ISO 8601 timestamp”, “URL format”, “Email format”)
- [ ]  Nullability explained (e.g., “Null when user hasn’t set a bio”)
- [ ]  Validation rules documented (e.g., “1-280 character”, “Must be unique”)
- [ ]  Performance implications noted for expensive fields

## Argument-level audit

- [ ]  Every argument has a description explaining its purpose
- [ ]  Valid value ranges specified (e.g., “1-100”, “Enum values: ACTIVE, INACTIVE”)
- [ ]  Default values documented (e.g., “Defaults to 10 if not provided”)
- [ ]  Required vs. optional clearly marked
- [ ]  Examples provided for complex arguments

## Connection & pagination audit

- [ ]  Pagination pattern explained in connection descriptions
- [ ]  Cursor format documented (e.g., “Base64-encoded timestamp”)
- [ ]  Page size limits specified (e.g., “Maximum 100 items per page”)
- [ ]  Sort order documented (e.g. “Ordered by createdAt descending”)

## Custom scalars & enums

- [ ]  Custom scalars have format examples (e.g., DateTime: “2025-03-15T10:30:00Z”)
- [ ]  Enum values all have descriptions explaining when to use each
- [ ]  Validation rules specified for custom scalars

## Union & interface audit

- [ ]  Union types explain when each type appears
- [ ]  Interface implementations all listed
- [ ]  Discriminating fields documented (how to tell types apart)

## Directive audit

- [ ]  Custom directives have usage examples
- [ ]  Authorization directives explain access rules
- [ ]  Deprecation directives include migration guidance

## AI-specific checks

- [ ]  Run schema through several models: Can it explain your API correctly?
- [ ]  Test query generation: Ask AI to generate queries for common use cases.
- [ ]  Check for ambiguity: Are there any fields that could be interpreted multiple ways?
- [ ]  Verify examples work: Check that all embedded code samples are syntactically correct.
