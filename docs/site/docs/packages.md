# Package repository

Vergil products are published as signed OS packages — `.deb` for Ubuntu and
`.rpm` for RHEL — through one org-wide package repository. This page is the
fleet-level summary. The full install commands live in the
[`packages` README](https://github.com/vergil-project/packages#readme), and the
build side lives in vergil-tooling's
[Package Config Reference](https://vergil-project.github.io/vergil-tooling/latest/reference/package-config/)
and [CI Architecture](https://vergil-project.github.io/vergil-tooling/latest/guides/ci-architecture/)
guide.

The packages are built, signed, published and install-tested for consumers
outside Vergil. Vergil itself does not install from them: every Vergil
install of vergil-tooling uses `uv` from git (see
[Who installs which way](#who-installs-which-way)).

## Where it lives

The repository is served from GitHub Pages at
<https://vergil-project.github.io/packages>:

| Format | Path | Releases |
| --- | --- | --- |
| apt | `/deb` | Ubuntu 24.04 (`noble`), Ubuntu 26.04 (`resolute`) |
| rpm | `/rpm/el9`, `/rpm/el10` | RHEL 9, RHEL 10 |

Every release is published for both **amd64** and **arm64**. The indexed
products today are `vergil-tooling`, `vergil-python` (the pinned CPython
runtime the Python products run on), and `vergil-archive-keyring` (built by
the `packages` repo itself).

## Bootstrapping a host

A host trusts the repository in four steps:

1. Fetch the public key from `keys/vergil.asc` on the repository site.
2. Verify that it holds exactly one primary key, with fingerprint
   `B3A1D804DC03AB036566A4E367861822166ECABE`.
3. Add a temporary bootstrap source signed by that key and install
   **`vergil-archive-keyring`**. From then on the keyring package owns the key
   and the permanent apt source / dnf repo file.
4. Remove the bootstrap source and key.

The exact apt and dnf commands are in the
[`packages` README](https://github.com/vergil-project/packages#readme) and are
not repeated here. When `vrg-package` builds or install-tests a Python
product, it performs this bootstrap itself to install the `vergil-python`
runtime, and refuses any key whose primary fingerprint differs from the
pinned one.

## Signing model

The org has one OpenPGP identity. Its **primary key is RSA-4096, certify-only,
and held offline** by a human. CI never sees it. CI holds only a **signing
subkey**, which signs every `.rpm` and the apt (`InRelease` / `Release.gpg`)
and dnf (`repomd.xml`) metadata.

Trust the **primary** fingerprint, not the subkey. The subkey rotates: a new
one is minted offline and shipped as a new `vergil-archive-keyring` release, so
a host that trusts the primary picks it up through an ordinary upgrade.

The subkey lives in GitHub environments restricted by branch:
**`package-signing`** (admits only `main`) in each product repo, where a
release signs its packages, and **`index-signing`** (admits only `develop`) in
`packages`, where the index is signed. Sigstore build-provenance attestations
are an extra layer on top of the signatures, never the trust root.

## Who installs which way

Every Vergil install of vergil-tooling is `uv tool install` from git, on a
uv-managed Python 3.15. A release install resolves its tag to one commit
and constrains its dependencies to that commit's own `uv.lock`. None of
them reads the package repository.

| Environment | How vergil-tooling is installed |
| --- | --- |
| Agent VMs (Lima and cloud) | `vrg-vm` resolves the identity's version to one commit with `git ls-remote` and installs that commit from git, constrained by its `uv.lock` |
| Dev containers | The per-branch cache build installs every ref from git on the image's uv-managed Python 3.15 (`VERGIL_CORE_PYTHON_DIR`), with interpreter downloads off; a release pin is constrained by its commit's `uv.lock`, a dev ref is not (see [CI architecture](https://vergil-project.github.io/vergil-tooling/latest/guides/ci-architecture/#tier-1-local-pre-commit) and [How the cache tracks vergil-tooling releases](https://vergil-project.github.io/vergil-tooling/latest/reference/cli-tools-overview/#how-the-cache-tracks-vergil-tooling-releases)) |
| CI (GitHub-hosted runners and container jobs) | The vergil-actions setup action installs from git, constrained by the resolved commit's `uv.lock` |
| macOS host | `uv tool install --no-config --python 3.15 --managed-python` of the exact release tag, constrained by the tag's `uv.lock`; the release's consumer refresh runs it, and a first install follows vergil-tooling's [Getting Started](https://vergil-project.github.io/vergil-tooling/latest/getting-started/) |

On a VM, a version with no release tag, a failed lock export or a failed
install fails provisioning: `vrg-vm` never falls back to another version
or to an unconstrained install. VMs made before the move to `uv` installed
vergil-tooling as an apt package. Once the `uv` install passes its check,
`vrg-vm update` removes the vergil packages, the vergil apt source and its
key from them, and `vrg-vm create` no longer sets up the package
repository. See vergil-tooling's
[VM spec reference](https://vergil-project.github.io/vergil-tooling/latest/reference/vm-spec/#vergil-tooling-inside-the-vm).

Packaged Python products depend on the `vergil-python` runtime series they were
built against. `vergil-python` publishes one package per listed CPython
series, `vergil-python<x.y>`, and every release publishes every listed series.
A package's version is `<cpython>+<pbs_tag>.<release>`, where `<release>` is
the vergil-python release that built it (for example `3.15.1+20261021.1.0.4`);
releases before v1.0.4 lack the `.<release>` suffix. A product depends on
`vergil-python<x.y> (>= <installed version>)`, the runtime version it was
built against, so later patches and rebuilds of the same series satisfy it.

The series is derived: it is the newest window version with a published
runtime. A product can instead pin an exact `x.y.z` with
`[package.python] runtime`, and then the build fails if the series package
serves any other patch. For a pinned product, the `vergil` updater in
`vrg-update-deps` bumps the pin to the patch the series package serves, only
once every package index the build and install-test read from serves it, and
never outside the pin's series: a newer series is reported as held back. See
vergil-tooling's
[Package Config Reference](https://vergil-project.github.io/vergil-tooling/latest/reference/package-config/)
and
[Dependency Update Config](https://vergil-project.github.io/vergil-tooling/latest/reference/dependency-update-config/).

## Publishing a product

A repo opts in by adding a `[package]` table to its `vergil.toml` (builder,
vendor, summary, smoke test, and target selection). With it present, PR CI
builds and install-tests the packages, and a release builds, signs, attests
and attaches them to the GitHub Release, then notifies `packages` to re-index.
Without it, nothing changes. The keys are documented in the
[Package Config Reference](https://vergil-project.github.io/vergil-tooling/latest/reference/package-config/);
the pipeline is summarized under
[Release packaging](ci.md#release-packaging).
