# Package repository

Vergil products are published as signed OS packages — `.deb` for Ubuntu and
`.rpm` for RHEL — through one org-wide package repository. This page is the
fleet-level summary. The full install commands live in the
[`packages` README](https://github.com/vergil-project/packages#readme), and the
build side lives in vergil-tooling's
[Package Config Reference](https://vergil-project.github.io/vergil-tooling/reference/package-config/)
and [CI Architecture](https://vergil-project.github.io/vergil-tooling/guides/ci-architecture/)
guide.

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
not repeated here. On a vergil-managed VM, `vrg-package` performs this
bootstrap itself and refuses any key whose primary fingerprint differs from the
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

| Environment | How vergil-tooling is installed |
| --- | --- |
| Agent VMs (Lima and cloud) | From the package repository, with `apt` |
| Dev containers | `uv tool install` (the container cache) |
| macOS host | `uv tool install` |

On a VM, `vrg-vm` pins `vergil-tooling` to the identity's resolved version and
fails provisioning if the repository has no matching package; it never falls
back to a `uv` install. See vergil-tooling's
[VM spec reference](https://vergil-project.github.io/vergil-tooling/reference/vm-spec/#vergil-tooling-inside-the-vm).

Packaged Python products depend on the `vergil-python` runtime they were built
against. The dependency names an exact CPython patch (it is part of the package
name, `vergil-python3.Y.Z`) with a `>=` floor on the python-build-standalone
build, so a newer build of the same patch satisfies it.

## Publishing a product

A repo opts in by adding a `[package]` table to its `vergil.toml` (builder,
vendor, summary, smoke test, and target selection). With it present, PR CI
builds and install-tests the packages, and a release builds, signs, attests
and attaches them to the GitHub Release, then notifies `packages` to re-index.
Without it, nothing changes. The keys are documented in the
[Package Config Reference](https://vergil-project.github.io/vergil-tooling/reference/package-config/);
the pipeline is summarized under
[Release packaging](ci.md#release-packaging).
