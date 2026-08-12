# Supported languages

Vergil supports a set of first-class languages under one shared model:
**containerized, multi-version toolchains** with a common validation pipeline
(lint, typecheck, test + coverage, dependency audit). Each language is added the
same way — a registry entry, a strictness policy, prebuilt container images, and
reusable CI — so every repository is validated identically regardless of the
language it is written in.

This page is the fleet-level summary. The authoritative, per-language **standards**
(toolchains, strictness rules, naming, testing and coverage) live in
`vergil-tooling` and are linked below rather than restated here.

| Language | Standards |
| --- | --- |
| Python | [Python standards](https://vergil-project.github.io/vergil-tooling/standards/development/python/overview/) |
| Go | [Go standards](https://vergil-project.github.io/vergil-tooling/standards/development/go/overview/) |
| Java | [Java standards](https://vergil-project.github.io/vergil-tooling/standards/development/java/overview/) |
| Rust | [Rust standards](https://vergil-project.github.io/vergil-tooling/standards/development/rust/overview/) |
| C++ | [C++ standards](https://vergil-project.github.io/vergil-tooling/standards/development/cpp/overview/) |
| TypeScript _(new)_ | [TypeScript standards](https://vergil-project.github.io/vergil-tooling/standards/development/typescript/overview/) |

## TypeScript

TypeScript is the newest first-class language, and the first new language added
since the extension model matured with C++. It follows the same containerized,
multi-version pattern as the rest of the fleet. The v1 shape:

- **Node.js runtime, single canonical `tsc`.** TypeScript has no Clang-vs-GCC
  analog — there is exactly one typechecker. So `TYPECHECK`, `LINT`, and `AUDIT`
  each run **once**, and only the test stage fans out per Node major. That makes
  the expensive matrix axis the **Node version**, not the compiler — and makes
  TypeScript lighter on merge gates than C++.
- **Multi-version container images.** The toolchain ships as prebuilt stable
  images for the supported Node majors — **`node-24`** (primary) and **`node-22`**
  — never built from source.
- **npm for install and audit.** Install is `npm ci` (a clean, lockfile-pinned
  install from `package-lock.json`); the dependency audit is `npm audit`. npm was
  chosen for universality at zero adoption barrier.
- **Warnings to 11, via a shareable strict base tsconfig.** Strictness is the
  baseline, not a luxury: consumers extend a shared strict base `tsconfig` (`strict`
  plus a curated set of extras promoted to errors), so every repository typechecks
  clean against the same rules.
- **100% coverage with Vitest + V8.** The house line is held at 100% via Vitest's
  V8 coverage, with explicit `/* v8 ignore */` markers as the acknowledged-gap
  valve — gaps are declared, never silently dropped.
- **ESM-first.** v1 targets native ES modules (`nodenext` resolution, `es2022`
  target); CommonJS and plain untyped JavaScript are out of scope for v1.

For the full toolchain, strictness rules, and testing policy, see the
[TypeScript standards](https://vergil-project.github.io/vergil-tooling/standards/development/typescript/overview/)
in `vergil-tooling`:

- [Toolchain and strictness](https://vergil-project.github.io/vergil-tooling/standards/development/typescript/toolchain-and-strictness/)
- [Testing and coverage](https://vergil-project.github.io/vergil-tooling/standards/development/typescript/testing-and-coverage/)
- [Naming conventions](https://vergil-project.github.io/vergil-tooling/standards/development/typescript/naming-conventions/)
