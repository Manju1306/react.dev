# I Migrated React's Official Docs from Yarn to pnpm — Here's the Bug It Exposed

I forked [react.dev](https://react.dev) — React's official documentation platform used by **2M+ developers monthly** — and migrated it from **Yarn 1.x to pnpm**.

What I expected: a straightforward find-and-replace.
What I got: **16 files changed, 5 errors, and a latent bug that Yarn had been silently hiding.**

👉 **Full code changes:** [github.com/Manju1306/react.dev](https://github.com/Manju1306/react.dev)

---

## Why pnpm in 2024?

pnpm is the default for Vue, Vite, SvelteKit, Nuxt, Astro, and Turborepo. **35%+ adoption** and growing 50-60% YoY.

- **4x faster installs** — content-addressable store with hard links, not copies
- **10-30 GB saved** across projects — one copy of each package on disk
- **Strict dependency resolution** — no phantom dependencies
- **Script security** — pnpm v11 blocks postinstall scripts by default

That strict resolution is where it got interesting.

---

## The Bug Yarn Was Hiding

react.dev's MDX compiler uses `eval()` to execute compiled code in a sandbox with a custom `fakeRequire`. It handled `react/jsx-runtime` (production) but **not** `react/jsx-dev-runtime` (development).

With Yarn's flat `node_modules`, the sandbox accidentally resolved the dev runtime through Node's module resolution chain. pnpm's strict symlinked structure broke that implicit path, exposing the bug:

```
TypeError: (0, _jsxDevRuntime.jsxDEV) is not a function
```

**A real bug, shipping undetected.** Two lines fixed it.

This is exactly why strict dependency resolution matters — it tells you the truth about your dependency graph.

---

## What Changed

| File | Change |
|---|---|
| `package.json` | `packageManager` + 6 scripts |
| `.npmrc` | Created — `shamefully-hoist=true` for eval'd MDX |
| `.husky/pre-commit` | `yarn` → `pnpm` |
| `src/utils/compileMDX.ts` | Lockfile path + jsx-dev-runtime fix |
| `.github/workflows/*` | CI cache, install, lockfile paths |
| `scripts/*` | Usage comments + error messages |
| `README.md`, `CLAUDE.md` | Updated dev commands |
| `pnpm-lock.yaml` | Created (9,980 lines) |
| `yarn.lock` | Deleted (8,731 lines) |

---

## 5 Errors, 5 Fixes

| # | Error | Root Cause |
|---|---|---|
| 1 | `This project is configured to use yarn` | `packageManager` field gates pnpm |
| 2 | `yarn: command not found` (exit 127) | Husky pre-commit hook |
| 3 | `ENOENT: yarn.lock` | MDX compiler reads lockfile for cache hash |
| 4 | `jsxDEV is not a function` | Missing dev runtime in `fakeRequire` |
| 5 | `ERR_PNPM_IGNORED_BUILDS` | pnpm v11 blocks postinstall scripts |

---

## Key Takeaways

- **Migration is not `s/yarn/pnpm/g`.** Lockfiles, CI cache keys, git hooks, build pipelines, error messages — all carry package manager references.
- **`eval()`-based builds are fragile across package managers.** If your build uses `new Function()` with a custom `require`, audit every module it resolves.
- **pnpm's strictness is a feature.** It surfaces bugs that flat hoisting hides.
- **`--cwd` → `--dir`.** Same semantics, different flag. Easy to miss.

---

## Explore the Full Migration

🔗 **[github.com/Manju1306/react.dev](https://github.com/Manju1306/react.dev)**

Check the commits, diffs, and the `MIGRATION-YARN-TO-PNPM.md` for the complete step-by-step guide with architecture context, the jsx-dev-runtime deep dive, and senior-level takeaways.

---

If you're still on Yarn 1.x, the migration is worth it — not just for speed, but for dependency integrity. pnpm doesn't just install your packages faster. It tells you the truth about your dependency graph.

And sometimes, the truth is a bug you didn't know you had.

---

*Have you migrated to pnpm? What did it surface in your codebase?*
