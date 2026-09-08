# Confidential Security Hotfix

⚠️ **This repository is confidential.**

This repository contains a **private security hotfix for a critical severity CosmWasm vulnerability** that can lead to **fund loss**. The issue is exploitable in practice, and has been locally reproduced.

The vulnerability is in the Wasmer Singlepass compiler used by `libwasmvm`. Any chain running `x/wasm` with contract upload or instantiation reachable by an attacker is affected.

Details are intentionally limited during the private disclosure window.
Please **do not share, fork, or discuss publicly** until disclosure.

Thank you for your continued dedication to maintaining a safe and secure ecosystem.

---

## Upgrade Guidance

To reduce the risk of premature disclosure, it is **strongly recommended** that this fix is deployed via **compiled binaries distributed directly to validators**, rather than public source changes, until the disclosure window closes.

This upgrade must be performed as a coordinated upgrade.

If you cannot complete this upgrade before the disclosure window closes, apply the interim mitigation instead: rebuild your existing binary as position-independent (`-buildmode=pie`, and `-static-pie` in place of `-static` in `extldflags`) and set `kernel.randomize_va_space=2` on every validator host. That reduces exploitability but does not remove the vulnerability.

## Timeline

The private disclosure window for this vulnerability is open now. After the window closes, the fixes will be merged into the public repositories and released as patch releases.

⚠️ **Disclosure date: [NEEDS CONFIRMING before this doc is shared]**

### Hotfix Branches

⚠️ **The fix is not on `main`.** `main` tracks upstream. The patched code lives on the `security/*` branches below. Check out the branch matching your release line before you build.

The fix is delivered on `security/*` branches. This repository carries:

| Release line | Branch | wasmvm dependency |
|---|---|---|
| `v0.60.x` | `security/v0.60.x` | `github.com/CosmWasm/wasmvm/v2 v2.3.5-rc.2` |

> **`v0.61.x` and `v0.70.x` are not yet cut.** If your chain is on either line, contact us before starting. Do not attempt to port the `v0.60.x` branch yourself.

⚠️ **No hotfix tag has been published in this repository yet.** Until one exists, pin by commit as shown below.

---

## Applying the Hotfix

### 1. Update your Git config to use private repositories

#### SSH Instructions

First, configure your machine to use SSH for Git. More details can be found here: https://docs.github.com/en/authentication/connecting-to-github-with-ssh.

To use SSH in `go mod` downloads, add these lines to `~/.gitconfig`:

```md
[url "ssh://git@github.com/"]
    insteadOf = https://github.com/
```

#### HTTPS Instructions

If you choose to use HTTPS, please follow the instructions here: https://go.dev/doc/faq#git_https.

### 2. Update `go.mod`

You need **two** `replace` directives. The patched `wasmd` depends on a `wasmvm` release candidate that exists only in the private `wasmvm` repository, so replacing `wasmd` alone will not resolve.

If you are on `v0.60.x`:

```go
replace (
	github.com/CosmWasm/wasmd => github.com/CosmWasm/priv_wasmd_sec <commit-sha>
	github.com/CosmWasm/wasmvm/v2 => github.com/CosmWasm/priv_wasmvm_sec v2.3.5-rc.2
)
```

Take `<commit-sha>` from the head of `security/v0.60.x`.

Then, tidy using the `GOPRIVATE` variable:

```bash
GOPRIVATE=github.com/CosmWasm/priv_wasmd_sec,github.com/CosmWasm/priv_wasmvm_sec go mod tidy
```

### 3. Verify you picked up the fix

Confirm the resolved `wasmvm` version and that the compiled engine is the patched one:

```bash
go list -m github.com/CosmWasm/wasmvm/v2   # expect v2.3.5-rc.2
```

The patched `libwasmvm` is built against **Wasmer 7.4.0**. Prebuilt shared libraries for `linux/amd64`, `linux/arm64` and `darwin` are committed to the `wasmvm` repository, so a Rust toolchain is not required for a standard build.

---

### 4. Build and Deploy

Rebuild your node binary using your standard process, distribute the compiled binary to validators, and perform a coordinated upgrade.

Build the binary as position-independent so that ASLR applies:

```bash
-buildmode=pie
# and, if you build statically, use -static-pie rather than -static in extldflags
```

Set `kernel.randomize_va_space=2` on every validator host. Confirm the resulting binary is position-independent before distributing it:

```bash
readelf -h ./build/<your-binary> | grep Type   # expect DYN
```

---

## Notes

- Do not mirror this repository to public infrastructure
- Do not copy this repository to a public Github repository
- Do not reference this fix in public changelogs or releases before disclosure
- Do not publish the patched binary's build instructions or version string ahead of disclosure. A version string that does not correspond to any public tag identifies a chain as carrying a private security build.

---

## Upstream documentation

This README has been replaced with the hotfix instructions because that is what you need here. The upstream project README and full build documentation are at https://github.com/CosmWasm/wasmd.
