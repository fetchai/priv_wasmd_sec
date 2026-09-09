# Confidential Security Hotfix

⚠️ **This repository is confidential.**

This repository contains a **private security hotfix for a critical severity CosmWasm vulnerability** that can lead to **fund loss**. The issue is exploitable in practice, and has been locally reproduced.

Details are intentionally limited during the private disclosure window.
Please **do not share, fork, or discuss publicly** until disclosure.

Thank you for your continued dedication to maintaining a safe and secure ecosystem.

---

## Upgrade Guidance

To reduce the risk of premature disclosure, it is **strongly recommended** that this fix is deployed via **compiled binaries distributed directly to validators**, rather than public source changes, until the disclosure window closes.

This upgrade must be performed as a coordinated upgrade.

## Timeline

The private disclosure window for this vulnerability is 2 weeks, beginning Thursday, September 10th. After the disclosure window closes, the fixes will be merged into the public repo at 10am EST on Thursday, September 24th 2026 and released in a patch release.

### Hotfix Tags

Use the tag matching your release line. `main` tracks upstream and does not contain the fix.

| Your release line | Tag | Branch | wasmvm replacement |
|---|---|---|---|
| `v0.54.x` | `v0.54.10` | `security/v0.54.x` | `github.com/CosmWasm/priv_wasmvm_sec/v2 v2.2.9` |
| `v0.60.x` | `v0.60.9` | `security/v0.60.x` | `github.com/CosmWasm/priv_wasmvm_sec/v2 v2.3.5` |
| `v0.61.x` | `v0.61.15` | `security/v0.61.x` | `github.com/CosmWasm/priv_wasmvm_sec/v3 v3.0.8` |
| `v0.70.x` | `v0.70.4` | `security/v0.70.x` | `github.com/CosmWasm/priv_wasmvm_sec/v3 v3.0.8` |

Chains on a release line not listed above should upgrade to the closest version that is.

Release candidates are published ahead of the final tags, as `-rc.N` suffixes on the same versions.

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

Add `replace` directives for both this repository and the private `wasmvm` repository, using the tag and wasmvm dependency from the table above.

For example, if you are on `v0.60.x`:

```go
replace (
	github.com/CosmWasm/wasmd => github.com/CosmWasm/priv_wasmd_sec v0.60.9
	github.com/CosmWasm/wasmvm/v2 => github.com/CosmWasm/priv_wasmvm_sec/v2 v2.3.5
)
```

Two directives are required: the patched `wasmd` depends on a `wasmvm` version that is available only in the private `wasmvm` repository.

The `/v2` or `/v3` suffix appears on **both sides** of the `wasmvm` directive and is required on both. It is `/v2` for the `v0.54.x` and `v0.60.x` lines and `/v3` for the `v0.61.x` and `v0.70.x` lines. Omitting it on the right-hand side fails with `version "v2.3.5" invalid: should be v0 or v1, not v2`.

Then, tidy using the `GOPRIVATE` variable:

```bash
GOPRIVATE=github.com/CosmWasm/priv_wasmd_sec,github.com/CosmWasm/priv_wasmvm_sec go mod tidy
```

---

### 3. Build and Deploy

Rebuild your node binary using your standard process, distribute the compiled binary to validators, and perform a coordinated upgrade.

---

## Notes

- Do not mirror this repository to public infrastructure
- Do not copy this repository to a public Github repository
- Do not reference this fix in public changelogs or releases before disclosure
