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

The private disclosure window for this vulnerability is 2 weeks, beginning Wednesday, September 9th. After the disclosure window closes, the fixes will be merged into the public repo at 10am EST on Wednesday, September 23rd 2026 and released in a patch release.

### Hotfix Branches

The fix is provided on the following branch. `main` tracks upstream and does not contain the fix.

- `security/v0.60.x` for the `v0.60.x` release line

No hotfix tag has been published yet, so pin by commit from the head of that branch.

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

Add `replace` directives pointing to this repository and to the private `wasmvm` repository, using the branch that matches your release line.

If you are on `v0.60.x`:

```go
replace (
	github.com/CosmWasm/wasmd => github.com/CosmWasm/priv_wasmd_sec <commit-sha>
	github.com/CosmWasm/wasmvm/v2 => github.com/CosmWasm/priv_wasmvm_sec v2.3.5-rc.2
)
```

Two directives are required: the patched `wasmd` depends on `wasmvm v2.3.5-rc.2`, which is available only in the private `wasmvm` repository.

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
