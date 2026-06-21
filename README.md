# Build FOSS Android Apps

Automated builds of FOSS Android applications for **arm64-v8a**, signed with a personal keystore and published as GitHub Releases.

> **Caution: Signed with my own keystore. Use at your own risk.**

## How it works

Every Saturday at 06:30 UTC (or on manual trigger), the workflow:

1. Checks each app's latest upstream release via GitHub API
2. Compares against [`config.json`](config.json) to skip apps that haven't changed
3. Downloads the prebuilt upstream APKs, strips every native-lib ABI except `arm64-v8a`, then re-signs with the keystore stored in repository secrets
4. Creates a single GitHub Release (tagged `v1`, `v2`, `v3`...) containing all newly built APKs
5. Commits the updated `config.json` back to the repo

A new release is only created when at least one app has a new upstream version. `config.json` is only updated after the release is confirmed.

## Apps

| App | Upstream | Flavors |
|-----|----------|---------|
| [KeePassDX](https://github.com/Kunzisoft/KeePassDX) | `Kunzisoft/KeePassDX` | `libre`, `free` (libre + `.free` package) |

## Configuration (`config.json`)

Each app is one entry. The key is the upstream `owner/repo`, and `urls` is a list of
GitHub release-asset templates where `{TAG}` is replaced by the resolved release tag:

```json
{
  "Kunzisoft/KeePassDX": {
    "version": "4.4.5",
    "urls": [
      "https://github.com/Kunzisoft/KeePassDX/releases/download/{TAG}/KeePassDX-{TAG}-libre.apk",
      "https://github.com/Kunzisoft/KeePassDX/releases/download/{TAG}/KeePassDX-{TAG}-free.apk"
    ]
  }
}
```

To add an app you only add a new entry with its `urls` — everything else is derived:

- The **repo to version-check** is parsed from the first URL.
- `version` is filled in / bumped automatically by the workflow after each release.

## APK naming

The output name is derived from the upstream asset name: the version token is dropped,
the rest is joined with `_`, and `arm64v8a` + the version are appended.

```
KeePassDX-{TAG}-libre.apk  ->  KeePassDX_libre_arm64v8a_{TAG}.apk
KeePassDX-{TAG}-free.apk   ->  KeePassDX_free_arm64v8a_{TAG}.apk
```

## Workflow structure

Everything lives in a single workflow, [`build.yaml`](.github/workflows/build.yaml), split into two jobs:

```
build.yaml
├── prerequisites   ← verifies the keystore, checks upstream versions vs config.json
└── release         ← runs only when something changed
```

**`prerequisites`** owns:
- Verifying the keystore is readable with the configured password/alias
- Version checks (`config.json` vs GitHub API), gating the `release` job

**`release`** owns (runs only when an upstream version changed):
- Downloading the prebuilt upstream APKs for the exact release tag
- Decompiling with [apktool](https://github.com/iBotPeaches/Apktool), removing all `lib/` ABIs except `arm64-v8a`, recompiling, zipaligning, and re-signing
- Creating the GitHub Release with composed notes
- Committing the `config.json` bump

Keeping it in one workflow avoids a reusable-workflow call (an extra runner) and the artifact upload/download round-trip between jobs.

## Secrets required

| Secret | Purpose |
|--------|---------|
| `KEYSTORE_BASE64` | Base64-encoded keystore file |
| `STORE_PASSWORD` | Keystore password |
| `KEY_ALIAS` | Key alias |
| `KEY_PASSWORD` | Key password |
