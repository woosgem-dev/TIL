# How We Achieved 75% Faster Builds by Removing Barrel Files

> Original: [How We Achieved 75% Faster Builds by Removing Barrel Files](https://www.atlassian.com/blog/atlassian-engineering/faster-builds-when-removing-barrel-files)
> Author: Tim Sebastian (Principal Software Engineer, Atlassian)
> Published: June 26, 2025

Removing JavaScript barrel files from the Jira frontend codebase cut build times by 75%, and significantly improved TypeScript highlighting and unit test speeds. This large-scale automated effort also boosted CI efficiency and made code navigation much better for developers.

---

## When a "Best Practice" Becomes a Performance Bottleneck

Atlassian's Jira frontend codebase had grown to include thousands of internal packages. At that scale, architectural decisions that worked fine in small projects can become serious performance bottlenecks.

Barrel files are a widely used JavaScript pattern — a single entry point for a module.

```js
// Individual component files
// components/Button/Button.js
export const Button = () => { /* component logic */ };

// components/Modal/Modal.js
export const Modal = () => { /* component logic */ };

// Barrel file: components/index.js
export { Button } from './Button/Button';
export { Modal } from './Modal/Modal';

// Barrel file approach — clean imports through the barrel
import { Button, Modal } from './components';
```

The problem got worse with cascading barrel files across the directory hierarchy:

```js
// Deep component
export const UserCard = () => { /* component logic */ };

// Level 1 barrel: features/UserManagement/components/index.js
export { UserCard } from './UserCard/UserCard';

// Level 2 barrel: features/UserManagement/index.js
export { UserCard, UserList } from './components';

// Top-level barrel: features/index.js
export { UserCard, UserList, userManagementUtils } from './UserManagement';

// Final import — looks clean but creates a massive dependency chain
import { UserCard } from './features';
```

As the codebase grew, they noticed concerning trends in the development experience:

- **Local TypeScript highlighting** took up to 2 minutes, leading many developers to assume the system was completely broken
- **Running a single unit test** took several minutes locally
- **CI builds** were running far more tests than necessary

---

## Root Cause Investigation: Following the Data

They suspected barrel files were causing dependency graph bloat.

```js
// components/index.js (barrel file)
export { Button } from './Button/Button';
export { DataTable } from './DataTable/DataTable'; // Heavy component with many dependencies
export { Chart } from './Chart/Chart';
// ... 50+ component exports

// You only need Button
import { Button } from './components';
// → But DataTable, Chart, and all 50 others get parsed
```

Even when importing just `Button`, tools like TypeScript and Jest have to process the entire barrel file. A single component import triggers a chain that reads, parses, and resolves dependencies for tens of thousands of unrelated modules.

They decided to try a "good enough" codemod across the entire codebase, and the results were clearer than expected:

- **Local test performance**: Single test runs became much faster
- **TypeScript processing**: Highlighting speed improved significantly
- **CI test selection**: Up to 70%+ improvement
- **Bundle generation**: Bundle sizes shrank slightly and bundle times decreased

---

## Engineering the Solution: Automating a Large-Scale Refactor

### Building the Technical Foundation

They used an internal tool called `factsmap` to collect import/export metadata across the codebase.

Where chains like `a -> b -> c -> d` existed before, they created direct imports like `a -> d`:

```js
// Before: Import chain through barrel files
import { Button } from './ui';
// -> ui/index.js -> components/index.js -> Button/Button.js

// After: Direct import
import { Button } from './components/Button/Button';
```

They built an ESLint rule that acts as both a linter and a code transformer:

```js
module.exports = {
  meta: { fixable: "code" },
  create(context) {
    return {
      ImportDeclaration(node) {
        if (isBarrelFileImport(node.source.value)) {
          context.report({
            node,
            message: "Avoid barrel file imports",
            fix(fixer) {
              const directImport = resolveToDirectImport(node);
              return fixer.replaceText(node, directImport);
            }
          });
        }
      }
    };
  }
};
```

### 3-Wave Landing Strategy

They needed to change over 90,000 files without blocking the daily work of 1,000+ developers.

The key insight was that **up to 80% of the codebase is dormant at any given time**.

They built a script that pulled active branch and changed file data from VCS to automatically avoid conflicts.

- **Wave 1**: Targeted dormant packages, which made up 80% of the codebase (areas with no active PRs). Conflict avoidance at the package level.
- **Wave 2**: Targeted individual files rather than entire packages. Only files not being modified by pull requests.
- **Wave 3**: The remaining few hundred files. Landed on a first-come-first-served basis, racing directly with developers.

Within days, they landed changes across 90,000+ files with minimal conflicts.

### Cleanup Phase

After each wave, they identified files that no longer appeared in any dependency graph and automatically deleted them. Thousands of obsolete files that no longer served a purpose were cleaned up.

---

## Measuring the Impact

- **TypeScript highlighting** speed improved by over 30%
- **Local unit tests** became roughly 50% faster on average, with up to 10x improvement in certain packages
- **CI unit tests**: 1,600 per build -> 200 (88% reduction), average execution time down 73%
- **CI integration tests**: 130 per build -> 20 (85% reduction)
- **VR tests**: 50 -> 25 (50% reduction)
- **Overall build time**: **75% reduction** in build minutes consumed per commit

---

## Unexpected Benefits

- **Better IDE navigation**: Clicking an import takes you directly to the source file (instead of a barrel chain)
- **Clear dependency relationships**: It became transparent what code actually depends on what
- **Simpler build tooling**: Reduced complexity in bundling and analysis tools
- **Further optimization opportunities**: Enabled dynamic CI pipelines through better test selection

---

## Trade-offs

- Packages can no longer easily control their "public API" through barrel files, losing a layer of encapsulation
- Refactoring becomes more fragile since moving a source file means updating all direct imports
- Direct imports can lead to deeper cross-module dependencies, potentially increasing coupling

Atlassian's stance: **When abstraction and performance conflict, choose performance. Prefer direct relationships for internal code. Trust measurements over conventions.**

---

## Key Takeaways

- Question "best practices" when they don't fit your scale
- Measure everything before and after making changes
- Consider the compounding effect of seemingly small decisions
- Sometimes the biggest wins come not from adding optimizations, but from removing complexity
