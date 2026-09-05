# DeskLink hbb_common AI Handoff

Last verified: 2026-09-03 UTC.

This repository is the `hbb_common` submodule used by the DeskLink client. The
DeskLink remote is `desklink` (`Kunlun-Hub/hbb_common`); `origin` is upstream
RustDesk. The parent repository pins an exact commit, so changes here are not in
a client release until the parent submodule pointer is committed and tagged.

## Current DeskLink State

- Current branded commit: `99ff3f1b04bc398de5bd554733f41948e7fd066b`
- Parent `zto57` release points to that commit.
- Embedded server configuration in `src/config.rs`:
  - `RENDEZVOUS_SERVERS = ["10.202.22.90:21116"]`
  - `RS_PUB_KEY = "I17NeGkLtixLRMrtxXvY5jQ3gDiL8yqtktCrMVwb69I="`
  - `DEFAULT_API_SERVER = "http://10.202.22.90:21114"`
  - `DEFAULT_RELAY_SERVER = "10.202.22.90:21117"`
- Default remote display options:
  - `view_style = adaptive`
  - `image_quality = custom`
  - `custom_image_quality = 85`
  - `custom-fps = 60`, accepted range `5..=120`

These are defaults only. Stored user settings and policy overrides still win.

## Editing Rules

- Keep shared protocol/config changes compatible with the parent client.
- Avoid `unwrap()`/`expect()` in production code. Tests and poisoned-lock
  handling are the established exceptions.
- Do not run or apply repository-wide formatting changes. This checkout has
  upstream files that do not match the current formatter.
- Do not rename option keys. They are persisted configuration contracts.
- When adding protobuf fields, preserve field numbers and regenerate all parent
  client bridges/artifacts required by the existing workflow.
- Never replace the embedded public key without coordinating the persisted
  hbbs private key and a new client release. A mismatch breaks secure TCP.

## Tests

From this repository:

```bash
rustfmt +1.98.0 --edition 2021 --check src/config.rs
cargo +1.98.0 test desklink_server_defaults_match_zto56_release --lib
cargo +1.98.0 test desklink_default_remote_display_options --lib
```

The first test name is historical but checks the current endpoints and public
key. Also run the parent repository checks because behavior such as public-server
classification and FPS QoS lives outside this submodule.

## Commit and Parent Update

```bash
git status --short
git add <only-touched-files>
git commit -m "..."
git push desklink HEAD:master
cd ../..
git add libs/hbb_common
git commit -m "..."
git push preview master
```

Verify the exact commit is reachable from `Kunlun-Hub/hbb_common` before tagging
the parent. The local branch name/tracking branch may differ; rely on the remote
URL and exact SHA, not the displayed branch label.

## Security

Server addresses and the public verification key are intentionally public.
Private keys, Docker credentials, API tokens, SSH passwords, and signing secrets
must never be added here or to the parent repository.
