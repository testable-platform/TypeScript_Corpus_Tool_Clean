# TypeScript Order Platform -- Monolith (TS_V20_ESBUILD_NPM_MONO)

Tool-evaluation repository for **Node 20**, bundled with **esbuild**,
managed with **npm**, in a **Monolith** layout.

This is branch **TS_V20_ESBUILD_NPM_MONO** of the consolidated `typescript-corpus` repository, which holds all 216 TypeScript branches across every Node version, bundler, package manager and architecture combination in this corpus.

## Project type

- **Language:** TypeScript 5.9.3
- **Runtime:** Node 20 (verified against 20.20.2)
- **Scenario:** 1 - Monolithic
- **Architecture:** Monolith
- **Module layout:** flat
- **Bundler:** esbuild 0.28.2 (esbuild transform and linker)
- **Package manager:** npm 11.19.1
- **Source root:** `src`

Each tool is pinned to the release that was resolved for Node 20; `dataset.json`
records the tools that were skipped and why. Every version in this
repository was resolved against the live npm registry and then executed on a
real Node 20.20.2 interpreter. None was written from memory.

## Build

```bash
npm i -g npm@11.19.1
npm ci
npm run build
npm run bundle
```

`npm run build` type-checks with `tsc --noEmit` and emits CommonJS + declarations to
`dist/`; `npm run bundle` then bundles with **esbuild 0.28.2 (esbuild transform and linker)** and **executes the bundle**.
Emitting is not proof; running it is.

## Run

```bash
node dist/src/index.js
```

## Test

```bash
npm run test        # mocha over tests/
npm run coverage    # c8 (primary) AND nyc + ts-node (cross-check)
```

Both coverage tools must report non-zero. They deliberately disagree: c8 reads
V8 coverage of the emitted output and remaps it, while nyc instruments the
TypeScript AST directly through `ts-node/register`. Identical numbers would mean
one of them is not an independent second opinion.

`nyc` here never reports through a source-map remap. Doing so silently yields
0% -- the file is remapped to `.ts`, an `--include` written against `dist/**`
stops matching, and the report empties while the process still exits 0.

## Architecture

**Monolith.** One deployable package. `package.json` declares **no**
`workspaces` field, the module tree under `src/` is flat, and there is no
`services/` directory. Those are exactly the properties an auditor reads to
classify a repository, so they are the ones held true here.

```
src/
  index.ts            public surface + sample runner
  models/             domain records and tax table (leaf layer)
  services/           pricing rules, order service, the duplicate pair
  platform/           integrations that use the planted dependency pins
  analysis/           planted fixtures -- never imported by real code
```

`dependency-cruiser` enforces the layering: `models/` may not import
`services/`, and nothing outside `analysis/` may import `analysis/`.


## Tool test-data folders

This repo carries only `Tool Clean (Synthetic Data)/` from testable-platform/typescript-corpus's original per-branch task-data folders.

### `Tool Clean (Synthetic Data)/`
25 of the 40 tools each carry 5 generated fixture packages, one per Node family (only `node20` kept in this repo), engineered to be clean so the tool should report
zero findings: the **Tool Clean (100% pass)** condition. `covgate` is the
exception within the exception: because it builds a native binary per Node
family, each of its 5 node-version folders (`node12/`...`node26/`) is itself a
full self-contained package with its own git history, plus a 6th git history
at the `covgate/` root -- 6 separate repositories in total. `diff-cover` and
`pydriller` operate on git history rather than language syntax, so each
carries one real git repository's worth of history instead of 5 per-version
copies. 13 other tools (`Bearer CLI`, `CVE Lite CLI`, `DepScout`, `Dependabot`, `GitHub API`, `Grype`, `ORT`, `OSV-Scanner`, `SonarJS`, `TraceGraph`, `git-hot`, `red-dragon`, `reson`) carry a single flat `src/` fixture rather than per-Node copies. All 8 of these git histories (6 covgate + diff-cover + pydriller) are
restored from `_git-bundles/` via `restore-git.ps1` rather than kept as live
`.git` folders, so a plain file copy never silently drops their content as a
submodule-style gitlink.

