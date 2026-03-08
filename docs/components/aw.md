# aw – Web Frontend

# aw – Web Frontend

Web interface of the Applikant system. An Erlang/OTP backend based on [Cowboy](https://github.com/ninenines/cowboy) with an **Elm** frontend.

## Role in the System

The web frontend is a **presentation and management layer**:

- **Display** repositories, users and permissions
- **Manage** — create repos, add users, configure permissions
- **Git HTTP protocol (read-only)** — repositories can be cloned/fetched via HTTP
- **No write operations** — pushes run exclusively via SSH

## HTTP Routes

| Route | Handler | Description |
|---|---|---|
| `/:user/:repo/info/refs` | `aw_web_git` | Git Smart HTTP — fetch refs |
| `/:user/:repo/git-upload-pack` | `aw_web_git` | Git Smart HTTP — upload-pack |
| `/api/repos` | `aw_web_api` | REST API — list repos, create repo |
| `/api/repos/:name` | `aw_web_api_repo` | REST API — repo details |
| `/api/users` | `aw_web_api_users` | REST API — list users, create user |
| `/api/users/:id` | `aw_web_api_user` | REST API — user details |
| `/api/users/:id/keys` | `aw_web_api_user` | REST API — add SSH key |
| `/api/users/:id/keys/:kid` | `aw_web_api_user` | REST API — update/remove SSH key |
| `/` | `cowboy_static` | Landing page (`index.html` + Elm app) |
| `/[...]` | `cowboy_static` | Static files (app.js, etc.) |

## Modules

| Module | Description |
|---|---|
| `aw_app` | OTP application — starts supervisor and Cowboy |
| `aw_sup` | Top-level supervisor |
| `aw_config` | Reads `aw.config`, sets application env |
| `aw_web` | Cowboy router & dispatch |
| `aw_web_git` | Git Smart HTTP handler (`git-upload-pack`) |
| `aw_web_api` | REST API — repo list and creation |
| `aw_web_api_repo` | REST API — single repo details |
| `aw_web_api_users` | REST API — user list and creation |
| `aw_web_api_user` | REST API — user details, SSH key CRUD |
| `aw_web_catch_all` | Catch-all handler (debug) |
| `aw_web_session_wrapper` | Cookie-based session management |
| `aw_git` | Executes `git-upload-pack` as OS port, streams response |

## Elm Frontend

The web UI is built with **Elm 0.19** and compiled to `app.js`:

```bash
cd applikant.webfrontend/elm
bash build-elm.sh
# Output: ../aw/apps/aw/priv/static/app.js
```

The Elm source is in `applikant.webfrontend/elm/src/` and split into modules:

| Module | Description |
|---|---|
| `Main.elm` | Init, Update, HTTP, View shell (header, nav, routing) |
| `Types.elm` | All types (Page, Model, Msg, RepoInfo, UserInfo, SshKeyInfo) |
| `Shared.elm` | Shared view components (cards, alerts, badges, footer) |
| `Page/Dashboard.elm` | Dashboard page |
| `Page/Repositories.elm` | Repository list and detail views |
| `Page/Users.elm` | User list, detail, SSH key management |

## Configuration

`priv/aw.config`:

| Key | Description | Default |
|---|---|---|
| `http_port` | Cowboy HTTP port | `8008` |
| `version` | Version string | `false` |
| `git_hash` | Git hash of the build | `false` |

## Build & Run

```bash
cd applikant.webfrontend/aw

# Build Elm frontend
cd ../elm && bash build-elm.sh && cd ../aw

# Start Erlang backend
rebar3 shell
```

Then open [http://localhost:8008](http://localhost:8008).

## Erlang Node

| Setting | Value |
|---|---|
| **Name** | `aw@<hostname>` |
| **Cookie** | `applikant_cookie` |
| **Port** | `8008` |

