# TypeScript Order Platform -- Microservices (TS_V12_VITE_PNPM_MICRO)

Tool-evaluation repository for **Node 12**, bundled with **vite**,
managed with **pnpm**, in a **Microservices** layout.

This is branch **TS_V12_VITE_PNPM_MICRO** of the consolidated `typescript-corpus` repository, which holds all 216 TypeScript branches across every Node version, bundler, package manager and architecture combination in this corpus.

## Project type

- **Language:** TypeScript 5.0.4
- **Runtime:** Node 12 (verified against 12.22.12)
- **Scenario:** 2 - Microservices
- **Architecture:** Microservices
- **Module layout:** workspace
- **Bundler:** Vite 2.9.18 (Rollup linker + its own bundled esbuild transform)
- **Package manager:** pnpm 6.35.1
- **Source root:** `packages/domain/src`

Node 12 is end-of-life, and that is deliberate: it pins the entire toolchain to
the last release of each tool that still supports it. Every version in this
repository was resolved against the live npm registry and then executed on a
real Node 12.22.12 interpreter. None was written from memory.

## Build

```bash
npm i -g pnpm@6.35.1
pnpm install --frozen-lockfile
pnpm run build
pnpm run bundle
```

`pnpm run build` type-checks with `tsc --noEmit` and emits CommonJS + declarations to
`dist/`; `pnpm run bundle` then bundles with **Vite 2.9.18 (Rollup linker + its own bundled esbuild transform)** and **executes the bundle**.
Emitting is not proof; running it is.

## Run

```bash
node dist/packages/domain/src/index.js
```

Each service is independently runnable:

```bash
node dist/services/gateway-service/src/index.js
node dist/services/pricing-service/src/index.js
```

## Test

```bash
pnpm run test        # mocha over tests/
pnpm run coverage    # c8 (primary) AND nyc + ts-node (cross-check)
```

Both coverage tools must report non-zero. They deliberately disagree: c8 reads
V8 coverage of the emitted output and remaps it, while nyc instruments the
TypeScript AST directly through `ts-node/register`. Identical numbers would mean
one of them is not an independent second opinion.

`nyc` here never reports through a source-map remap. Doing so silently yields
0% -- the file is remapped to `.ts`, an `--include` written against `dist/**`
stops matching, and the report empties while the process still exits 0.

## Architecture

**Microservices.** Three independently deployable services over a shared domain
package, wired through a workspace root.

```
packages/
  domain/         @orderkit/domain     -- models, services, analysis fixtures
  contracts/      @orderkit/contracts  -- inter-service message types
services/
  gateway-service/    accepts payloads, emits order.submitted
  order-service/      validates and prices, emits order.priced / order.rejected
  pricing-service/    owns tier and volume rates (leaf -- calls no one)
```

Each service has its own `package.json`, its own `src/index.ts` entry point and
its own start script. The workspace root lists them under `workspaces`.

**Documented inter-service call path:**

```
http.request -> gateway-service -> order.submitted
                order-service   -> pricing.quote -> pricing-service
                pricing-service -> pricing.rate  -> order-service
                order-service   -> order.priced | order.rejected
```

This matters because of what the JavaScript corpus got wrong: FlintAtlas's sheet
said `Microservices` while the repository was a flat monolith with no
workspaces, and its own README and `dataset.json` both said "monolith". Nothing
in that repo compared the two. Here `Tool Triggering (Synthetic Data)/full_check.ts` fails the build if the
declared architecture and the actual layout disagree.


## Tool test-data folders

This repo carries only `Tool Clean (Synthetic Data)/` from testable-platform/typescript-corpus's original per-branch task-data folders.

### `Tool Clean (Synthetic Data)/`
25 of the 40 tools each carry 5 generated fixture packages, one per Node family (only `node12` kept in this repo), engineered to be clean so the tool should report
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

