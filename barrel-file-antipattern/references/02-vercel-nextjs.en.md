# How Next.js Optimized Package Imports

> Original: [How we optimized package imports in Next.js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
> Author: Shu Ding (Software Engineer, Vercel)
> Published: October 13, 2023

40% faster cold boots and 28% faster builds.

---

## What Are Barrel Files?

A pattern where multiple modules are grouped and exported from a single file. It provides a centralized location to access grouped modules, making imports more convenient.

```js
// index.js
export { default as module1 } from './module1';
export { default as module2 } from './module2';
export { default as module3 } from './module3';

// Using the barrel
import { module1, module2, module3 } from './utils';
```

Some popular icon and component libraries have **up to 10,000 re-exports in a single entry barrel file**.

## The Problem with Barrel Files

Every `require(...)` and `import '...'` carries a hidden cost in the JavaScript runtime. Even if you only need a single export from a barrel file that imports thousands of other things, you still pay the cost of importing all those unnecessary modules.

Some popular React packages take **200-800ms just to import**. In extreme cases, it can take several seconds.

Both local development and production performance are affected. Serverless environments are hit even harder since they need to re-import everything on each app startup.

## Can't Tree Shaking Solve This?

Tree shaking is a *bundler* feature (Webpack, Rollup, etc.), not a JavaScript runtime feature. If a library is marked as `external`, it remains a black box.

Bundling a library together with app code does allow tree shaking to work via the `sideEffects` setting, but it significantly slows down builds because the entire module graph needs to be compiled and analyzed.

## First Attempt: modularizeImports

Next.js's initial approach. It required manually configuring the mapping between exported names and actual module paths.

```js
// Config: transform to my-lib/{{member}}
// import { module2 } from 'my-lib'
// → import module2 from 'my-lib/module2'
```

Problems:
- Depends on the library's internal directory structure
- Requires excessive manual configuration
- Library version updates can change internal structure, invalidating the transforms
- Doesn't scale to millions of npm packages

## The New Solution: optimizePackageImports

Introduced in Next.js 13.5. It automatically analyzes barrel files and transforms them into direct imports.

```js
// next.config.js
module.exports = {
  experimental: {
    optimizePackageImports: ["my-lib"]
  }
}
```

How it works:
1. Analyzes the entry file to determine if it's a barrel file
2. Scans in a single pass to auto-generate the import mapping
3. Recursively handles nested barrel files and wildcard exports (`export * from`)
4. Stops the process when it reaches a file that isn't a barrel file

Cheaper than tree shaking — it only needs to scan the entry barrel file once.

Pre-configured for common libraries like `lucide-react`, `@headlessui/react`, and others.

## Performance Improvements

### Local Development (M2 MacBook Air)

| Library | Before | After | Difference |
|---------|--------|-------|------------|
| @mui/material | 7.1s (2,225 modules) | 2.9s (735 modules) | -4.2s |
| recharts | 5.1s (1,485 modules) | 3.9s (1,317 modules) | -1.2s |
| @material-ui/core | 6.3s (1,304 modules) | 4.4s (596 modules) | -1.9s |
| react-use | 5.3s (607 modules) | 4.4s (337 modules) | -0.9s |
| lucide-react | 5.8s (1,583 modules) | 3.0s (333 modules) | -2.8s |
| @material-ui/icons | 10.2s (11,738 modules) | 2.9s (632 modules) | -7.3s |
| @tabler/icons-react | 4.5s (4,998 modules) | 3.9s (349 modules) | -0.6s |
| rxjs | 4.3s (770 modules) | 3.3s (359 modules) | -1.0s |

When using multiple libraries, these numbers add up fast.

### Production Build

On a Next.js App Router page benchmark using `lucide-react` and `@headlessui/react`, `next build` ran **roughly 28% faster**.

### Cold Boot

- Local: Node.js server started **roughly 10% faster**
- Serverless (Vercel): Combined with other improvements, up to **40% faster cold starts**

### Recursive Barrel Files

A module with 4 levels of 10 `export *` expressions (10,000 modules total):
- Before: ~30 seconds
- After: ~7 seconds
- Some customers with 100,000+ modules saw **90% faster reloads**
