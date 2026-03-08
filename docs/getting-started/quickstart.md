# Quick Start

!!! warning "Work in Progress"
    These instructions reflect the current development state. Some steps may not work fully yet.

Get Applikant running on your local machine in a few minutes.

## Prerequisites

- **Erlang/OTP 27** or later
- **Rebar3**
- **Elm 0.19.1** (for the web frontend)
- **Git**

## 1. Start the Manager

```bash
cd applikant.manager/am
rebar3 shell
```

You should see:

```
===> Booted am
===> Booted sasl
```

The Manager is now running as `am@<hostname>`.

## 2. Start the Web Frontend

In a **new terminal**:

```bash
# Build the Elm frontend
cd applikant.webfrontend/elm
bash build-elm.sh

# Start the Cowboy server
cd ../aw
rebar3 shell
```

You should see:

```
===> Booted cowlib
===> Booted ranch
===> Booted cowboy
===> Booted aw
```

Open [http://localhost:8008](http://localhost:8008) in your browser.

The web interface lets you:

- View the Dashboard
- Create and browse git repositories
- Manage users and their SSH keys

## 3. Start the Frontend Connector (optional)

In a **new terminal**:

```bash
cd applikant.git-in/af
rebar3 shell
```

This starts the `af` node, which `as` and `ah` will connect to for authorization.

## 4. Build the SSH Handler (optional)

```bash
cd applikant.git-in/as
rebar3 escriptize
```

The escript is at `_build/default/bin/as`. See [SSH Setup](ssh-setup.md) for configuring `authorized_keys`.

## Verify Cluster

In any running Erlang shell, check that nodes are connected:

```erlang
1> nodes().
['am@myhost', 'aw@myhost']
```

!!! tip
    All nodes must use the same cookie (`applikant_cookie`) and short names (`-sname`).

