# Build FOSS Android Apps

Automated builds of FOSS Android applications for **arm64-v8a**, signed with a personal keystore and published as GitHub Releases.

> **Caution: Signed with my own keystore. Use at your own risk.**

## How it works

Every Saturday at 06:30 UTC (or on manual trigger), the workflow:

1. Checks each app's latest upstream release via GitHub API
2. Compares against [`versions.json`](versions.json) to skip apps that haven't changed
3. Builds only what's new, signs with the keystore stored in repository secrets
4. Creates a single GitHub Release (tagged `v1`, `v2`, `v3`...) containing all newly built APKs
5. Commits the updated `versions.json` back to the repo

A new release is only created when at least one app has a new upstream version. `versions.json` is only updated after the release is confirmed.

## Apps

| App | Upstream | Flavors |
|-----|----------|---------|
| [KeePassDX](https://github.com/Kunzisoft/KeePassDX) | `Kunzisoft/KeePassDX` | `libre`, `free` (libre + `.free` package) |

## APK naming

```
KeePassDX_libre_arm64v8a_v4.1.0.apk
KeePassDX_free_arm64v8a_v4.1.0.apk
```

## Workflow structure

```
build.yaml                        ← orchestrator (schedule, secrets, release)
└── kunzisoft_keepassdx.yaml      ← builds KeePassDX, stages APKs as temp artifacts
```

**`build.yaml`** owns:
- Version checks (`versions.json` vs GitHub API)
- Passing secrets to app workflows
- Creating the GitHub Release with composed notes
- Committing version bumps

**App workflows** own:
- Cloning the upstream repo at the exact release tag
- Patching and building the APK
- Staging the renamed APK as a 1-day artifact for the release job to pick up

## Secrets required

| Secret | Purpose |
|--------|---------|
| `KEYSTORE_BASE64` | Base64-encoded keystore file |
| `STORE_PASSWORD` | Keystore password |
| `KEY_ALIAS` | Key alias |
| `KEY_PASSWORD` | Key password |
