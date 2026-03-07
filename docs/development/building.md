# Building

How to build each Applikant component.

## Prerequisites

| Tool | Version | Purpose |
|---|---|---|
| Erlang/OTP | 27+ | Backend |
| Rebar3 | 3.x | Erlang build tool |
| Elm | 0.19.1 | Web frontend |
| Docker | 20+ | Builder, containerization |

## Components

### am – Manager

```bash
cd applikant.manager/am
rebar3 compile       # compile only
rebar3 shell         # compile + start interactive shell
rebar3 as prod tar   # production release
```

### af – Frontend Connector

```bash
cd applikant.git-in/af
rebar3 compile
rebar3 shell
```

### as – SSH Handler

```bash
cd applikant.git-in/as
rebar3 escriptize
# Output: _build/default/bin/as
```

### ah – Git Hooks

```bash
cd applikant.git-in/ah
rebar3 escriptize
# Output: _build/default/bin/ah
```

### aw – Web Frontend

```bash
# 1. Build Elm frontend
cd applikant.webfrontend/elm
bash build-elm.sh
# Output: ../aw/apps/aw/priv/static/app.js

# 2. Build and start Erlang backend
cd ../aw
rebar3 shell
```

### ab – Builder

```bash
cd applikant.builder
docker build -t applikant/builder .
```

## Docker Images

### git-in (as + ah)

```bash
cd applikant.git-in
docker build -t applikant/git-in .
```

### Web Frontend

```bash
cd applikant.webfrontend
bash docker-build.sh
```

### Builder

```bash
cd applikant.builder
docker build -t applikant/builder .
```

