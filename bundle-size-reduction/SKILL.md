---
name: bundle-size-reduction
description: Diagnose and shrink a frontend JavaScript bundle — analyze what's in it, remove dead weight, code-split, tree-shake, and swap heavy dependencies — use when a web app's JS is too large, first-load or Time-to-Interactive is slow, Lighthouse flags unused JS, or a bundle-size budget is exceeded.
---

# Bundle Size Reduction

Large JavaScript bundles delay parsing, hydration, and interactivity, especially on mobile. This skill provides a measurement-first workflow to see exactly what ships to the browser, then apply the highest-leverage reductions — removing or replacing heavy modules, splitting code, and eliminating dead code — while verifying the app still works.

## When to use this skill

- Initial JS payload is large (hundreds of KB+ gzipped) or growing over time.
- Lighthouse/PageSpeed flags "reduce unused JavaScript" or slow Time-to-Interactive.
- A CI bundle-size budget failed or you want to add one.
- A single dependency is suspected of bloating the build.
- You are preparing a performance-sensitive release.

## Instructions

1. **Measure the current bundle.** Build for production and generate a bundle report. Record total and per-chunk sizes as **gzipped/brotli** bytes (raw bytes overstate impact). Tools: `webpack-bundle-analyzer`, `rollup-plugin-visualizer`, `vite build --report`, `source-map-explorer`, or `npx bundlephobia`.
2. **Find the biggest contributors.** Sort modules by size. Look for: moment.js/large date libs, lodash imported wholesale, entire icon sets, duplicate copies of a library at different versions, polyfills for browsers you no longer support, and large images/fonts inlined into JS.
3. **Attack the largest items first**, one at a time:
   - **Replace heavy deps** with lighter alternatives (e.g. `date-fns`/native `Intl` over moment, `lodash-es` with named imports over full `lodash`).
   - **Tree-shake**: import only what you use (`import debounce from 'lodash-es/debounce'`), ensure `"sideEffects": false` where safe, and use ES modules.
   - **Code-split**: lazy-load routes and rarely-used features via dynamic `import()` so they are not in the initial chunk.
   - **De-duplicate**: dedupe transitive versions (`npm dedupe`, resolutions/overrides).
4. **Trim polyfills and targets.** Set an explicit `browserslist` matching real users so the compiler stops shipping legacy transforms and polyfills.
5. **Defer the non-critical.** Move analytics, chat widgets, and below-the-fold code out of the critical path; load on interaction or idle.
6. **Rebuild and re-measure** after each change. Confirm the reduction in gzipped size and that nothing broke (lazy chunks load, features work).
7. **Add a budget to CI.** Enforce a size limit (e.g. `size-limit`, bundlesize, or the bundler's `performance.maxAssetSize`) so regressions fail the build.

## Examples

Turning a full-library import into a tree-shakeable one:

```js
// Before — pulls in all of lodash (~70KB gzipped)
import _ from 'lodash';
_.debounce(fn, 200);

// After — only the function you use
import debounce from 'lodash-es/debounce';
debounce(fn, 200);
```

Route-level code splitting so a heavy admin page is not in the initial bundle:

```js
import { lazy, Suspense } from 'react';
const AdminDashboard = lazy(() => import('./AdminDashboard')); // its own chunk

<Suspense fallback={<Spinner />}>
  <AdminDashboard />
</Suspense>
```

Enforcing a budget in CI with size-limit:

```json
{
  "size-limit": [
    { "path": "dist/main.*.js", "limit": "150 KB" }
  ]
}
```

## Checklist

- [ ] Bundle was analyzed and sizes recorded in gzipped/brotli bytes.
- [ ] The largest contributors were identified and addressed first.
- [ ] Heavy dependencies were replaced or imported narrowly (tree-shaken).
- [ ] Non-critical code is code-split / lazy-loaded out of the initial chunk.
- [ ] Duplicate library versions were de-duplicated.
- [ ] `browserslist`/targets trimmed unnecessary polyfills.
- [ ] App re-tested: lazy chunks load and features still work.
- [ ] A bundle-size budget is enforced in CI.
