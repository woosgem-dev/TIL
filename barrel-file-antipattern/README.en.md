# Barrel Files & TypeScript Conventions

> 2025-02-15 Study Notes
> Topics: Barrel file anti-pattern, import strategy, TypeScript conventions

---

## 1. What Is a Barrel File

A JavaScript convention where multiple modules are re-exported from a single `index.ts` to simplify import paths.

```ts
// components/index.ts (barrel file)
export { Button } from './Button/Button'
export { Modal } from './Modal/Modal'

// Consumer
import { Button, Modal } from '@/components'
```

---

## 2. Why It's an Anti-Pattern

### 2-1. Build Performance (Verified Data)

**Atlassian Jira (90,000 files)**

| Metric | Improvement |
|--------|-------------|
| TypeScript highlighting | 30%+ faster |
| Local unit tests | 50% faster on average, some packages 10x |
| CI unit tests | 1,600 → 200 (88% reduction) |
| CI integration tests | 130 → 20 (85% reduction) |
| Overall build time | **75% reduction** |

> Details: [references/01-atlassian.en.md](./references/01-atlassian.en.md)

**Next.js / Vercel Benchmark (M2 MacBook Air)**

| Library | Before | After | Difference |
|---------|--------|-------|------------|
| @mui/material | 7.1s (2,225 modules) | 2.9s (735 modules) | -4.2s |
| @material-ui/icons | 10.2s (11,738 modules) | 2.9s (632 modules) | -7.3s |
| lucide-react | 5.8s (1,583 modules) | 3.0s (333 modules) | -2.8s |

Production builds 28% faster, serverless cold boot 40% faster.

> Details: [references/02-vercel-nextjs.en.md](./references/02-vercel-nextjs.en.md)

### 2-2. Breaks Tree Shaking

- Next.js Issue #12557: Using only 2 components from a barrel of 100 still bundles all of them
- Switching to direct imports slashed bundle size
- `sideEffects: false` alone is insufficient (module parsing cost remains)

> Details: [references/03-nextjs-issue.en.md](./references/03-nextjs-issue.en.md)

### 2-3. Root Cause

The TypeScript compiler and Jest must **load every module inside a barrel file** just to resolve a single import. Import one Button, but if the barrel has 50 components, all 50 get parsed. **Nested barrels** — where one barrel references another — make this cost snowball.

```
import { Button } from '@/components'
  → components/index.ts (loads all 30 re-exports)
    → Button/index.ts (another load)
      → Button.tsx (finally arrives)
```

One import traverses 3 files. With 30 components, that's 61 files processed.

### 2-4. Circular Dependency Risk

Barrel files importing from each other easily create circular dependencies, leading to runtime `undefined` errors that are notoriously hard to trace.

### 2-5. Maintenance Overhead

Every time a component is added or removed, you have to update the export line in `index.ts`. Forget, and you end up with dead exports or new components that can't be imported.

---

## 3. Why Design Systems Get Away With It

Why design system libraries (`import { Button } from '@some-ds/components'`) can use barrels:

1. **Pre-built artifacts**: The consumer's TypeScript compiler doesn't parse the full source — it reads pre-built `.js` and `.d.ts` files
2. **Bundler optimizations are baked in**: `sideEffects: false`, `exports` field, per-component build output
3. **Public API contract**: Hides internal structure, ensuring consumer code doesn't break across version changes

**The key**: It comes down to who pays the resolve cost and when. Libraries have already resolved everything at publish time. Internal barrels make the developer pay that cost on every single save.

---

## 4. AI Agents and Barrels

### Impact on Agent Performance

No official test results **exist**. Neither Anthropic, OpenAI, nor any other AI company has published a barrel file vs. direct import comparison for agent performance.

**For grep-based agents, barrels aren't a major obstacle:**
- `grep -r "export.*Button" --include="*.tsx"` → finds the source regardless of barrels

**Where barrels actually annoy agents:**
- Tracing an import means opening files one by one (tool call + token cost per hop)
- When every file is named `index.ts`, a directory listing tells you nothing about what each file does — you have to open each one

**Conclusion**: The case against barrels is far stronger on **build performance and tree shaking** grounds than on agent efficiency.

---

## 5. Recommended Architecture

### 5-1. App Internal Code

```
components/
  Button/
    Button.tsx          ← named export: export function Button()
    Button.styles.ts
    Button.test.tsx
  Modal/
    Modal.tsx
    Modal.styles.ts
```

- **No index.ts barrel**
- Direct path import: `import { Button } from '@/components/Button/Button'`
- tsconfig paths: `"@/*": ["./src/*"]`
- Exclude index.ts from auto-import: `autoImportFileExcludePatterns: ["**/index.ts"]`

### 5-2. Monorepo Package Boundaries

```json
{
  "name": "@pkg/ui",
  "exports": {
    "./Button": "./src/Button/Button.tsx",
    "./Modal": "./src/Modal/Modal.tsx"
  }
}
```

- Provide granular entry points via subpath exports
- Consumer: `import { Button } from '@pkg/ui/Button'`
- Do not use a single-entry barrel (`".": "./src/index.ts"`)

### 5-3. External Libraries

- Let Next.js `optimizePackageImports` handle it (compiler-level barrel → direct import transformation)
- Use direct imports when the package structure allows it

---

## 6. index.ts vs ComponentName.tsx in Component Directories

```
# index.ts pattern — file list tells you nothing
components/Button/index.ts
components/ListItem/index.ts
components/Modal/index.ts

# Explicit filenames — immediately clear
components/Button/Button.tsx
components/ListItem/ListItem.tsx
components/Modal/Modal.tsx
```

- No more editor tabs full of indistinguishable `index.ts` files
- Stack traces immediately identify the file
- AI agents can determine a file's role from the name alone
- `Button/Button` looks redundant, but auto-import handles it — you almost never type it manually

---

## 7. Cleaning Up Paths with tsconfig paths

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

`import { Button } from '@/components/Button/Button'` — the path maps 1:1 to the file system, making it **the most predictable structure**. AI agents can locate the file just by reading the path.

---

## 8. Export Style: Named vs Default

**Always use named exports:**

```ts
// CORRECT
export function Button() { ... }

// WRONG
export default function Button() { ... }
```

Why named exports:
- Prevents consumers from renaming the import (codebase-wide consistency)
- Better auto-import accuracy
- Per-export tree shaking
- Easy to trace usage via grep/AST

### React.lazy Workaround

```ts
const Page = React.lazy(() =>
  import('./Page').then(module => ({ default: module.Page }))
)

// Or a helper function
function lazyNamed<T extends Record<string, any>>(
  factory: () => Promise<T>,
  name: keyof T
) {
  return React.lazy(() =>
    factory().then(module => ({ default: module[name] }))
  )
}
```

### Vue Exception

`.vue` SFCs structurally only support default export (framework constraint). This is acceptable because:

- One file = one component, so the filename is the component name
- Tools like `unplugin-vue-components` enforce naming based on the filename
- Composables, utils, and other `.ts` files still use named exports

```ts
// CORRECT — name matches filename
import Button from '@/components/Button/Button.vue'

// WRONG — name differs from filename
import Btn from '@/components/Button/Button.vue'
```

---

## 9. Function Declarations: function Keyword vs Arrow

**Use the function keyword for top-level declarations:**

```ts
// CORRECT
export function Button() { ... }
export function useAuth() { ... }

// WRONG
export const Button = () => { ... }
```

Why the function keyword:
- Hoisting allows flexible ordering within a file
- Stack traces always show the function name
- grep-friendly: `function Button` is an unambiguous pattern
- Clearly signals "this is a named, reusable unit"

**Arrow functions are for callbacks and inline usage:**

```ts
const items = list.map((item) => transform(item))
useEffect(() => { ... }, [])
```

---

## 10. interface vs type

### Why Agents Default to interface

Training data is overwhelmingly interface-heavy. The TypeScript docs recommended "use interface if you're not sure" for a long time, and most open-source projects followed suit.

### Functional Differences

```ts
// Only interface can do this
interface A { x: string }
interface A { y: number }  // declaration merging — extends with the same name

// Only type can do this
type Result = Success | Failure           // union
type Mapped = { [K in Keys]: boolean }    // mapped type
type Inferred = ReturnType<typeof fn>     // conditional/utility
```

### Advantages of type-first

**Consistency**: type handles everything — unions, intersections, mapped types, conditionals, object shapes. interface only handles object shapes, so you end up mixing both anyway. That creates a constant "should this be interface or type?" decision tax.

**Safety (no accidental declaration merging)**:

```ts
// Accidentally use the same name in another file
interface Config { debug: boolean }
interface Config { logLevel: string }
// → Silently merged. No error.

type Config = { debug: boolean }
type Config = { logLevel: string }
// → Immediate compile error. Safe.
```

**Simpler agent instructions**: Eliminates the unnecessary "interface or type?" branching logic.

**More readable error messages (better for debugging)**:

```ts
// interface error — shows only the name
Type 'X' is not assignable to type 'ButtonProps'
// → You have to look up ButtonProps to understand what's wrong

// type error — shows the expanded structure
Type 'X' is not assignable to type '{ label: string } & { size: "sm" | "md" | "lg" }'
// → You can fix it directly from the error message
```

For agents too — the error message alone is enough to make the fix without additional tool calls.

### Where interface Wins

- `extends` reads better than `&`
- Declaration merging is necessary when extending library types

### Verdict

**Prefer type in application code.** Better consistency, safety, error messages, and simpler agent instructions. Use interface only when you actually need declaration merging.

---

## 11. Next.js's Solution: optimizePackageImports

Solves the barrel problem at the compiler level:

1. Analyzes barrel entry files at build time
2. Generates export name → actual file path mappings
3. Rewrites imports automatically: `from 'lib'` → `from 'lib/dist/esm/Button'`
4. Recursively handles nested barrels and `export *`
5. Bypasses barrels entirely — cheaper than tree shaking

Pre-configured for common libraries like lucide-react and @headlessui/react.

---

## 12. Atlassian's Migration Strategy (90,000 Files)

Carried out while 1,000+ developers were actively working on the codebase.

**Technical Foundation:**
- `factsmap` internal tool to collect import/export metadata across the entire codebase
- Barrel chain resolution: `a → b → c → d` → `a → d`
- Used ESLint fixable rules as codemods (dual purpose: lint + auto-transform)
- Parallelized ESLint runner executed across the full codebase

**3-Wave Landing:**
- **Wave 1**: Targeted dormant packages (no active PRs) — 80% of the codebase
- **Wave 2**: Targeted individual files rather than packages (excluding hot files)
- **Wave 3**: Remaining few hundred files — raced against developers modifying them in real time

Used VCS data to identify files being changed in active branches, automatically avoiding conflicts.

**Cleanup**: Automatically deleted thousands of files that no longer appeared in the dependency graph.

---

## 13. Applying This as an Agent Guide

### Reference from CLAUDE.md

```markdown
# CLAUDE.md
See @docs/typescript-import-conventions.md for import and module rules.
```

### Core Principles

- Agents learn from and mimic existing codebase patterns, so **explicit rules** matter
- Rules + CORRECT/WRONG examples let agents follow via pattern matching
- Write short, focusing on "what to do" rather than "why"

---

## 14. One-Line Summary

| Area | Rule |
|------|------|
| App internal | No barrels. Direct path imports |
| Package boundaries | Provide entry points via subpath exports |
| External libraries | Let optimizePackageImports handle it |
| File structure | `ComponentName/ComponentName.tsx` (no index.ts) |
| Exports | Named export only (Vue .vue exception) |
| Function declarations | function keyword for top-level |
| Types | Prefer type (interface only when merging is needed) |

**"What Atlassian had to painstakingly remove across 90,000 files — just don't create in the first place."**

---

## References

- [Atlassian: 75% Faster Builds by Removing Barrel Files](https://www.atlassian.com/blog/atlassian-engineering/faster-builds-when-removing-barrel-files) — [Summary](./references/01-atlassian.en.md)
- [Vercel: How We Optimized Package Imports in Next.js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js) — [Summary](./references/02-vercel-nextjs.en.md)
- [Next.js GitHub Issue #12557: Tree Shaking and Barrel Files](https://github.com/vercel/next.js/issues/12557) — [Summary](./references/03-nextjs-issue.en.md)
