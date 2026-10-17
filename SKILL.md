---
name: modern-web-toolchain
description: Assess and migrate a JS/TS repository to the fast native toolchain — Bun (package manager + script runner), TypeScript 7/tsgo (typecheck), Vite+ (vitest-compatible test runner + oxlint + oxfmt) replacing Vite/ESLint/Prettier. Also surfaces OneStack as an optional suggestion when the repo is Expo/React Native. Use when the user asks to speed up installs, typecheck, tests, lint or CI; to modernize the toolchain; or invokes this skill by name.
---

# Modern Web Toolchain Migration

These are simple, low-cost, low-regression migrations — the hard part is not the tools, it's the rules that keep the formatter from reformatting the whole repo.

Reference migrations with measured stats (cite them when pitching):
- https://github.com/fdussert/fremkit/pull/42
- https://github.com/fdussert/fremkit-sietch/pull/13

## The four migrations

| From | To | Measured gain | Notes |
|------|----|---------------|-------|
| pnpm/npm/yarn | **Bun** (`bun@1.4.x`, `bun.lock`) | ~9× cold install | `oven-sh/setup-bun@v2` in CI, `bun install --frozen-lockfile` |
| TypeScript 5 `tsc` | **TypeScript 7** (`typescript@7`, native Go rewrite) | ~7–10× typecheck | Keep the `tsc -p tsconfig.json --noEmit` script — the binary is now native |
| Vite/Vitest | **Vite+** (`vite-plus`, `vp test`) | test-run dependent | `vitest` imports keep working; `vitest.config.ts` → `vite.config.ts`. Jest repos need explicit validation — `vp test` is vitest-compatible, not jest-compatible; check module mocking/`jest.*` API usage before promising drop-in |
| ESLint + Prettier | **oxlint + oxfmt via Vite+** (`vp lint`, `vp fmt`) | orders of magnitude | See "Formatter discipline" — this is the risky part |

## Assessment first — never migrate blind

Before touching anything, inventory and report:

1. **Package manager**: `packageManager` field, lockfile (`pnpm-lock.yaml`, `package-lock.json`, `yarn.lock`, `bun.lock`). Confirm Bun is on PATH (`bun --version`).
2. **TypeScript version** in devDependencies; whether `tsc` is invoked directly or wrapped.
3. **Test runner**: vitest/jest config files, test glob, test count.
4. **Lint/format**: `.eslintrc*`, `eslint.config.*`, `.prettierrc*`, `prettier` in scripts or `lint-staged`/`husky` hooks.
5. **Spawned binaries in scripts/tests**: anything referencing `node_modules/.bin/tsx`, `tsx`, `npx`, `node ` — these break silently when the runner changes. Provide an env override per repo (pattern: `process.env.<REPO>_TEST_RUNNER ?? 'bun'` — derive the name, don't reuse it).
6. **CI workflows**: setup steps, cache keys, pinned versions — update `packageManager`, lockfile name, setup action, and drop now-dead steps (e.g. `actions/setup-node` once nothing invokes `node`/`npx`; drop stale `engines.node` if nothing consumes it).
7. **Workspaces and npm-isms**: `pnpm-workspace.yaml` must become `workspaces` in `package.json` for bun; pnpm `catalog:`/`overrides`, `.npmrc` settings, and corepack/`packageManager` pins each need an explicit port or a documented drop.
8. **Docs**: every `pnpm`/`vitest`/`eslint`/`prettier` reference in README/CONTRIBUTING/AGENTS/widget docs — grep for the old names, including inside nested `*/README.md` files (the reference PR missed one stale `pnpm exec vitest` in a nested widget README; don't repeat that).

## Migration rules

- **One PR, minimal file set.** Three buckets only: core migration files, lint-mandated cleanups the new linter requires, docs. No drive-by refactors.
- **Keep `lint` separate from `fmt`.** Add `vp lint` to CI immediately — it catches real issues cheaply. Do NOT enforce `vp fmt` in the same change.
- **Formatter discipline — the actual hard part.** Configure `fmt` in `vite.config.ts` with `ignorePatterns` covering every path whose bytes matter — published artifacts, generated dirs, vendored code, content-hashed packages, the lockfile, markdown. Derive the list from the repo's own layout; in the reference repo it was `widgets/**`, `themes/**`, `tools/vendor/**`, `dist/**`, `bun.lock`, `**/*.md`. Enabling the formatter means a ~20-file reformat; that is a change of its own, never smuggled into a tooling migration. Say so explicitly in the PR body.
- **Pin versions**: exact `bun-version` in CI, exact `typescript` in devDependencies.
- **Preserve lockfile integrity**: delete the old lockfile, commit the new one, verify `bun install --frozen-lockfile` passes.
- **Verify end-to-end locally** before pushing: install, typecheck, lint, full test suite, build. Then let CI confirm.
- **Squash to a single commit** if the work was iterated.

## Measure and report

Time before/after with `hyperfine` (1 warmup, medians): cold install, typecheck, test, lint, validate/build. Compare the CI job duration against the base branch's recent runs. Numbers belong in the PR body — the pitch is "low cost, low regression risk, measured wins," not vibes.

## OneStack — assess and suggest only

If the repository is an **Expo / React Native** project (signals: `expo` in dependencies, `app.json`/`app.config.*`, `metro.config.js`), evaluate https://onestack.dev/ as a Metro replacement (~5.4× measured). This is a **suggestion to surface, never an action**: assess feasibility, report fit and expected gain, and let the human decide. Do not migrate, install, or commit anything for OneStack without explicit approval. If the repo is not Expo/RN, don't mention it.

## PR shape

Title: `Move the registry tooling to bun, TypeScript 7 and Vite+` (adapt to repo). Body: what each tool replaces, the measurements table, the CI comparison, and an explicit statement of what is configured-but-not-enforced (formatter) and why.
