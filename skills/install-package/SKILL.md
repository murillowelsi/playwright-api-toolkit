---
name: install-package
description: Add a new npm package dependency to the project.
---

# Add Dependency

## 1. Gather Requirements

Ask the user:
- Package name
- Is it a dependency or devDependency?
- Preferred version (or use latest)

## 2. Install the Package

```bash
# Regular dependency
npm install package-name

# Dev dependency
npm install --save-dev package-name
```

## 3. Verify Installation

```bash
npm list package-name
```

## 4. Import and Use

```typescript
// In utils/ or src/
import { something } from 'package-name';
```

## Common Packages for This Project

### Testing / Assertions

```bash
npm install --save-dev @playwright/test   # Already installed
npm install dotenv                        # Already installed
```

### Type Utilities

```bash
npm install --save-dev @types/node        # Already installed
npm install --save-dev type-fest          # Advanced TypeScript types
```

### Utilities

```bash
npm install --save-dev @faker-js/faker    # Fake data generation for factories
npm install dayjs                         # Date manipulation
```

## Rules

- Use `--save-dev` for packages only needed during testing (most packages in this project)
- Only use regular `dependencies` for packages needed at runtime (e.g. `dotenv`)
- Prefer latest stable unless there is a compatibility issue
- Check `package.json` first — verify the package isn't already installed before adding

## After Installation

- Confirm the package is listed in `package.json`
- Run `npx tsc --noEmit` to verify imports are valid
- Run `npm test` to confirm nothing is broken
