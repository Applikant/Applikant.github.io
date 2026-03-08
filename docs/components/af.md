# af – Frontend Connector

Permanently running OTP application on the git-in server. Serves as a **local proxy** between the short-lived escripts (`as`, `ah`) and the central Manager (`am`).

## Motivation

`ah` and `as` are escripts — on every invocation, a new Erlang node starts, connects via Erlang distribution, syncs global process names, makes a single call, and exits. This is slow.

`af` solves this: it runs permanently, keeps the connection to the Manager open, and accepts local requests from `as` and `ah` instantly.

```
Without af:                     With af:

as ──[Erlang Dist]──▶ am       as ──[local]──▶ af ──[Erlang Dist]──▶ am
ah ──[Erlang Dist]──▶ am       ah ──[local]──▶ af ──[Erlang Dist]──▶ am

(per call: start node,          (af runs permanently, connection
 connect, sync, query)           established, local calls instant)
```

## Responsibilities

| Responsibility | Description |
|---|---|
| **Proxy for `as`** | Forwards authorization requests to the Manager |
| **Proxy for `ah`** | Forwards git hook requests to the Manager |
| **Permanent connection** | Maintains the Erlang distribution link to `am` |
| **authorized_keys management** | Writes SSH keys with forced commands when notified by the Manager |

## Globally Registered Services

| Process | Module | Called By |
|---|---|---|
| `af_auth` | `af_auth` | `as` — `af_auth:check_access(User, Repo, Access)` |
| `af_hook_server` | `af_hook_server` | `ah` — `af_hook_server:handle_hook(HookData)` |
| `af_repo_manager` | `af_repo_manager` | `aw` — `af_repo_manager:create_repo(Name)`, `af_repo_manager:list_repos()` |

## Erlang Node

| Setting | Value |
|---|---|
| **Name** | `af@<hostname>` (short names) |
| **Cookie** | `applikant_cookie` |

## Status

!!! warning "Work in progress"
    `af_auth` allows everything (no real permission checks). `af_hook_server` forwards hook data to `am`. `af_repo_manager` creates bare repos with `ah` hooks. Real authorization forwarding to `am` is not yet implemented.

## Build & Run

```bash
cd applikant.git-in/af
rebar3 shell
```

