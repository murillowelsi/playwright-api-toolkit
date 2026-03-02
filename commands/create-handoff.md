You are creating a handoff document for session continuity. Capture everything needed for a fresh session to continue seamlessly.

## When to Use

- Context window getting full
- Taking a break
- Complex task spanning sessions
- Before risky operations

---

## Gather Context

```bash
git branch --show-current
git status --short
git log --oneline -10
git diff --stat
git stash list
```

---

## Handoff Document

Create at `docs/handoffs/YYYY-MM-DD-<topic>.md`:

```markdown
# Handoff: [Topic]

**Date**: YYYY-MM-DD HH:MM
**Branch**: `branch-name`
**Goal**: [What we were accomplishing]

---

## Original Request

> [User's original request verbatim]

[Any clarifications]

---

## Work Completed

### Files Created
- `path/to/file.ts` - [Purpose]

### Files Modified
- `path/to/file.ts:42-67` - [What changed]

### Commands Run
```bash
npm install some-package
npm test
```

### Decisions Made
1. **[Decision]**: [Reasoning]

### Discoveries
- [Important finding about the API]

---

## Work Remaining

### Next Steps
1. [ ] [Task] - `file:line` reference
2. [ ] [Task]

### Blocked
- [ ] [Task] - Blocked by: [reason]

---

## What Didn't Work

1. **[Approach]**: [Why it failed]
   - Error: `[message]`

---

## Critical Context

### Key Files
1. `src/tests/api/books.api.spec.ts` - [Why important]
2. `src/api/types/book.d.ts` - [Current type interface]
3. `src/api/api-clients/books-api-client.ts` - [Client methods]

### API Notes
- [Any discovered API behaviors, quirks, or inconsistencies]
- [Endpoints that return unexpected status codes or shapes]

### Assumptions
- [Things assumed but not verified about the API]

---

## Current State

- **Tests Pass**: ✅ / ❌
- **TypeScript Clean**: ✅ / ❌

### Uncommitted Changes
```
[git status output]
```

### Temporary Code
- `file:line` - `// TODO: remove debug`

---

## Resume Instructions

1. Read: [key files]
2. Start with: [first action]
3. Run: `npm test && npx tsc --noEmit`

---

## Related

- PR: [URL]
- Issue: #123
```

---

## Quick Handoff

For simpler tasks:

```markdown
# Quick Handoff: [Topic]

**Branch**: `branch-name`
**Status**: [In progress / Blocked]

## Done
- [x] Thing 1

## Next
- [ ] Thing 2 (`file:line`)

## Key Files
- `src/tests/api/books.api.spec.ts`
- `src/api/types/book.d.ts`

## Notes
- [Important context about the API]
```

---

## After Creating

1. Save to `docs/handoffs/`
2. Optionally commit:
   ```bash
   git add docs/handoffs/
   git commit -m "docs: add handoff for [topic]"
   ```
3. Tell user where saved and how to resume

---

## Resume From Handoff

When user provides handoff:

1. Read the file
2. Verify state matches (branch, git status)
3. Read key files listed
4. Summarize what's done/remaining
5. Confirm before proceeding
6. Start with first "Work Remaining" item
