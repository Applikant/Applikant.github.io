# Development Status

Current implementation status of all Applikant features.

## Components

| Component | Status | Notes |
|---|---|---|
| **am** – Manager | 🟡 Partial | Supervision tree, hook processing, node observer done. User/permission management missing. |
| **af** – Frontend Connector | 🟡 Partial | OTP skeleton with `af_auth` (allows everything). Real forwarding to `am` missing. |
| **as** – SSH Handler | 🟡 Partial | Parses SSH commands, connects to `af`. Actual git execution TODO. |
| **ah** – Git Hooks | ✅ Working | Connects to Manager, sends hook data, receives response. |
| **aw** – Web Frontend | 🟡 Partial | Cowboy running, static files served, git HTTP read-only foundation. Elm landing page done. |
| **ab** – Builder | 🟡 Partial | Docker image builds, `build.sh` works. Manager integration missing. |

## Features

| Feature | Status |
|---|---|
| Git hook communication (ah ↔ am) | ✅ Implemented |
| Hook processing (pre-receive, update, post-update) | ✅ Implemented |
| Supervision tree with hook handler | ✅ Implemented |
| Node monitoring | ✅ Implemented |
| Cowboy HTTP server | ✅ Implemented |
| Git HTTP protocol (read-only clone/fetch) | 🟡 Foundation |
| Elm web interface | 🟡 Landing page |
| SSH forced command parsing | 🟡 Scaffold |
| User and permission management | ❌ Not started |
| Repository management via web | ❌ Not started |
| Automatic `authorized_keys` management | ❌ Not started |
| Builder triggered by Manager | ❌ Not started |
| Webhooks | ❌ Not started |

## Roadmap

1. **Authorization** — Implement real permission checks in `am`, forward from `af`
2. **SSH execution** — Complete `as` to actually run git commands after authorization
3. **User management** — CRUD for users, SSH keys, permissions in `am`
4. **Web UI** — Elm pages for repository browsing, user management
5. **Builder integration** — Manager triggers `ab` after push
6. **`authorized_keys` automation** — `af` manages SSH keys based on Manager data

