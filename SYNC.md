# Sync Runbook (Client Repo)

This repository (`rustdesk`) depends on submodule `libs/hbb_common`.

Use this runbook whenever you want to sync with upstream while preserving custom self-hosted defaults (`RENDEZVOUS_SERVER`, `RS_PUB_KEY`).

## Remotes (expected)

- `origin` -> `https://github.com/tdimitriou/rustdesk.git`
- `upstream` -> `https://github.com/rustdesk/rustdesk.git`

Submodule `libs/hbb_common` should have:

- `fork` -> `https://github.com/tdimitriou/hbb_common.git`
- `upstream` -> `https://github.com/rustdesk/hbb_common.git`

## 1) Sync hbb_common first

From `libs/hbb_common`:

```bash
git checkout main
git fetch upstream
git fetch fork
git merge upstream/main
# If conflicts occur, keep custom build/config env support:
# - build.rs (RENDEZVOUS_SERVER, RS_PUB_KEY via cargo:rustc-env)
# - src/config.rs (option_env! for RENDEZVOUS_SERVER and RS_PUB_KEY)
git push fork main
```

## 2) Sync rustdesk

From repository root:

```bash
git checkout master
git fetch upstream
git merge upstream/master
git submodule update --init --recursive
git add libs/hbb_common
git commit -m "chore: bump hbb_common submodule"
git push origin master
```

## 3) Verify customization is still active

Before running CI:

- `libs/hbb_common/build.rs` contains `RENDEZVOUS_SERVER` and `RS_PUB_KEY` env forwarding.
- `libs/hbb_common/src/config.rs` uses `option_env!("RENDEZVOUS_SERVER")` and `option_env!("RS_PUB_KEY")`.
- `.github/workflows/flutter-build.yml` maps secrets:
  - `RENDEZVOUS_SERVER`
  - `RS_PUB_KEY`

## 4) Build/release order

1. Run `Flutter Nightly Build`
2. If green, run `Flutter Stable Build`
3. Use `Flutter Tag Build` only for immutable versioned snapshots

## Notes

- Keep server export data out of this repo.
- Do not remove the `hbb_common` submodule path (`libs/hbb_common`) because Cargo expects it.
