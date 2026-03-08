# Development Status

!!! info "Work in Progress"
    Applikant is in an early stage of development. The tables below reflect the current state of implementation. This documentation describes the **concept and planned architecture** — not all features are functional yet.

Current implementation status of all Applikant features.

## Components

| Component | Status | Notes |
|---|---|---|
| **am** – Manager | 🟡 Partial | Supervision tree, hook processing, node observer, user DB with SSH key management done. Permission checks not yet enforced. |
| **af** – Frontend Connector | 🟡 Partial | OTP app with `af_auth` (allows everything), `af_hook_server`, `af_repo_manager`. Real permission forwarding to `am` missing. |
| **as** – SSH Handler | 🟡 Partial | Parses SSH commands, connects to `af`. Actual git execution TODO. |
| **ah** – Git Hooks | ✅ Working | Connects to `af`, sends hook data, receives response via Manager. |
| **aw** – Web Frontend | 🟡 Partial | Cowboy running, Elm UI with Dashboard, Repository CRUD, User + SSH key management. Git HTTP read-only foundation. |
| **ab** – Builder | 🟡 Partial | Docker image builds, `build.sh` works. Manager integration missing. |

## Features

| Feature | Status |
|---|---|
| Git hook communication (ah → af → am) | ✅ Implemented |
| Hook processing (pre-receive, update, post-update) | ✅ Implemented |
| Supervision tree with hook handler | ✅ Implemented |
| Node monitoring | ✅ Implemented |
| Cowboy HTTP server | ✅ Implemented |
| Elm web interface (Dashboard, Repos, Users) | ✅ Implemented |
| Repository management via web (create, list, detail) | ✅ Implemented |
| User management with SSH keys (create, list, add/edit/remove keys) | ✅ Implemented |
| REST API (repos, users, SSH keys) | ✅ Implemented |
| Git HTTP protocol (read-only clone/fetch) | 🟡 Foundation |
| SSH forced command parsing | 🟡 Scaffold |
| Permission enforcement (read/write per repo) | ❌ Not started |
| Automatic `authorized_keys` management | ❌ Not started |
| Builder triggered by Manager | ❌ Not started |
| Webhooks | ❌ Not started |

## Roadmap

1. **Permission enforcement** — Implement real permission checks (read/write per repo per user) in `am`, forward from `af`
2. **SSH execution** — Complete `as` to actually run git commands after authorization
3. **`authorized_keys` automation** — `af` manages SSH keys based on Manager data
4. **Builder integration** — Manager triggers `ab` after push
5. **Webhooks** — Notify external services on push/merge events

