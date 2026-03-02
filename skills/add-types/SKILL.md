---
name: add-types
description: Create TypeScript interface types for an API response. Use when adding type definitions for a new resource or endpoint.
---

# Create New Types

## 1. Gather Requirements

Ask the user:

- Which API resource or entity? (e.g., `user`, `order`, `product`)
- What is the endpoint URL to inspect?
- Are there nullable fields or optional fields?

## 2. Inspect the Actual API Response

Always fetch the real endpoint before writing types:

```bash
# List endpoint
curl <api-base-url>/{resource}s?limit=1 | jq '.'

# Detail endpoint
curl <api-base-url>/{resource}s/{id} | jq '.'
```

Pay attention to:

- Exact field names and casing
- Fields that can be `null`
- Fields that may be absent
- Nested object shapes
- Arrays of primitives vs arrays of objects
- Number vs string for IDs

## 3. Type File Structure

File: `src/api/types/{resource}.d.ts`

Type files contain **only interfaces and type aliases** — no functions, no constants, no imports.

If the resource shares common shapes (e.g. `Pagination`, `ApiInfo`) with other resources, define those in `src/api/types/common.d.ts` and keep each resource file focused on its own interfaces.

```typescript
// Main resource interface
export interface {Resource} {
  id: number | string;
  // Nullable field — the API can return null
  description: string | null;
  // Optional field — may not be present in the response
  extraField?: string;
  // Timestamp
  updated_at?: string;
  // Arrays of primitives
  tags?: string[] | null;
  // Arrays of objects
  related?: {Resource}Ref[];
}

// Nested reference type (if needed)
export interface {Resource}Ref {
  id: number;
  title: string;
}

// List response wrapper
export interface {Resource}ListResponse {
  data: {Resource}[];
  // include pagination/info fields matching the actual response
}

// Detail response wrapper
export interface {Resource}DetailResponse {
  data: {Resource};
}
```

## 4. Nullable vs Optional

```typescript
// Field returned as null by the API
description: string | null

// Field that may not be present in the response
bio?: string

// Both
avatar?: string | null
```

## 5. Rules

- Use `import type` when importing these interfaces in other files
- No `any` — every field must have an explicit type
- Export all interfaces so they can be imported elsewhere
- No runtime code in type files — interfaces and type aliases only
- Place in `src/api/types/{resource}.d.ts`

## 6. After Creation

Import and use in your code:

```typescript
import type {
  {Resource}ListResponse,
  {Resource}DetailResponse,
} from '@/api/types/{resource}.d';

const body: {Resource}ListResponse = await response.json();
```

Run type check:

```bash
npx tsc --noEmit
```
