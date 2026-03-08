# Architecture Overview

!!! note "Concept"
    This page describes the **target architecture**. Not all connections and features are fully implemented yet — see the [Development Status](../development/status.md) for details.

Applikant is a **distributed system** built from several loosely coupled Erlang/OTP components. Each component runs as its own Erlang node, and they communicate via Erlang's built-in distribution mechanism.

## System Diagram

```mermaid
graph TB
    subgraph "git-in Server"
        as[as – SSH Handler<br/><small>escript</small>]
        ah[ah – git Hooks<br/><small>escript</small>]
        af[af – Frontend Connector<br/><small>OTP app, permanent</small>]
        as --> af
        ah --> af
    end

    subgraph "Manager Server"
        am[am – Manager<br/><small>OTP app</small>]
    end

    subgraph "Web Server"
        aw[aw – Web Frontend<br/><small>OTP app, Cowboy + Elm</small>]
    end

    subgraph "Build Server"
        ab[ab – Builder<br/><small>Docker-in-Docker</small>]
    end

    af -->|Erlang Distribution| am
    aw -->|Erlang Distribution| am
    am -->|triggers| ab

    User([fa:fa-user User]) -->|SSH| as
    User -->|HTTP :8008| aw

    style am fill:#7c3aed,color:#fff,stroke:#5b21b6
    style af fill:#e2e8f0,stroke:#64748b
    style as fill:#e2e8f0,stroke:#64748b
    style ah fill:#e2e8f0,stroke:#64748b
    style aw fill:#dbeafe,stroke:#3b82f6
    style ab fill:#fef3c7,stroke:#d97706
```

## Data Flow

### Git Push via SSH

```mermaid
sequenceDiagram
    participant U as User
    participant SSH as SSH Server
    participant as as as (escript)
    participant af as af (OTP)
    participant am as am (Manager)
    participant ah as ah (escript)

    U->>SSH: git push
    SSH->>as: forced command (user-id)
    as->>af: check_access(user, repo, write)
    af->>am: forward request
    am-->>af: {ok, allowed}
    af-->>as: {ok, allowed}
    as->>SSH: exec git-receive-pack
    Note over SSH: git executes hooks
    SSH->>ah: pre-receive hook
    ah->>am: {git_hook, {pre-receive, ...}}
    am-->>ah: ok
```

### Git Clone via HTTP (read-only)

```mermaid
sequenceDiagram
    participant U as User
    participant aw as aw (Web Frontend)
    participant Git as git-upload-pack

    U->>aw: GET /user/repo/info/refs
    aw->>Git: git-upload-pack --advertise-refs
    Git-->>aw: ref list
    aw-->>U: Smart HTTP response
    U->>aw: POST /user/repo/git-upload-pack
    aw->>Git: git-upload-pack (stdin)
    Git-->>aw: packfile
    aw-->>U: packfile stream
```

## Deployment

All components can run on a **single machine** or be **distributed across multiple hosts**. The only requirement is that the Erlang nodes can reach each other and share the same cookie.

### Single Machine

```
┌─────────────────────────────────────────┐
│  Host                                   │
│                                         │
│  af@host  ←→  am@host  ←→  aw@host     │
│                  ↓                      │
│              ab (Docker)                │
└─────────────────────────────────────────┘
```

### Multi-Machine

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  git-in      │   │  manager     │   │  web         │
│  af@git-in   │──▶│  am@manager  │◀──│  aw@web      │
│  as, ah      │   │              │   │  Port 8008   │
└──────────────┘   └──────┬───────┘   └──────────────┘
                          │
                   ┌──────▼───────┐
                   │  builder     │
                   │  ab (Docker) │
                   └──────────────┘
```

## Design Principles

- **Manager is the single source of truth** — all authorization decisions flow through `am`
- **SSH is the only write path** — pushes always go through SSH, never HTTP
- **Escripts are thin wrappers** — `as` and `ah` do as little as possible, delegating to `af`/`am`
- **Fault tolerance via OTP** — supervisors restart failed components automatically
- **Loose coupling** — components can be started and stopped independently

