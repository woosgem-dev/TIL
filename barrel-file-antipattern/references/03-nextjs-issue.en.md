# [Next.js Issue #12557] Tree Shaking Doesn't Work with TypeScript Barrel Files

> Original: [Tree shaking doesn't work with Typescript barrel files · Issue #12557](https://github.com/vercel/next.js/issues/12557)
> Author: majelbstoat
> Date: May 6, 2020
> Labels: TypeScript, Webpack, good first issue
> Status: Closed (Resolved)

---

## Bug Report

### Description

When using barrel files to re-export components from a single location, tree shaking does not work correctly.

### Steps to Reproduce

Based on Next 9.3.6. Component structure:

```
components/
  Header/
    Header.tsx
  Sidebar/
    Sidebar.tsx
  index.ts
```

`index.ts` barrel file:

```ts
export * from './Header/Header.tsx'
export * from './Sidebar/Sidebar.tsx'
// ...
```

Usage in `_app.tsx`:

```ts
import { Header, Sidebar } from "../components"
```

Around 100 components are defined, and only a few are used in `_app.tsx`. Yet analyzing the bundle reveals a very large shared chunk that **includes every single component**.

**Switching to direct imports:**

```ts
import { Header } from "../components/Header/Header"
import { Sidebar } from "../components/Sidebar/Sidebar"
```

The common bundle and app page sizes **shrink significantly**.

### Expected Behavior

App page size should be the same regardless of which import strategy is used.

---

## Key Community Comments

This issue got 151+ thumbs-up reactions — clearly a common pain point for Next.js users.

### Core Discussion Points

**1. `sideEffects: false` is not enough**

Several users suggested that adding `sideEffects: false` to `package.json` would fix the problem, but this alone does not fully resolve it. Even when Webpack performs tree shaking, it still has to parse and analyze every module referenced by the barrel file, which carries its own cost.

**2. The `_app.tsx` problem is especially bad**

Since `_app.tsx` is shared across all pages, importing through a barrel file here inflates the **bundle size for every page**. This largely negates the benefits of code splitting.

**3. Internal components vs external libraries**

The problem is not limited to external npm packages; it happens just as much with internal project components managed through barrel files. External libraries can be addressed with `optimizePackageImports`, but developers have to deal with internal barrel files on their own.

**4. Workaround: direct imports**

Importing directly from source files instead of barrel files is the only reliable workaround. Users who made the switch reported significant reductions in bundle size.

### Resolution

This issue directly motivated the `optimizePackageImports` feature introduced in Next.js 13.5. This feature automatically transforms barrel file imports into direct imports at the compiler level, giving developers the same performance gains without having to manually change import paths.

That said, this solution targets **external packages**. For internal barrel files within a project, the recommendation is still to use direct imports or eliminate barrel files altogether.
