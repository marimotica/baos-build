# baos-build — BAOS CI/CD orchestrator

## Purpose

This repository is the build orchestration hub for the
[Boat Assistant OS (BAOS)](https://github.com/marimotica) project.

It is responsible for:
- Polling upstream Home Assistant repositories for new releases
- Triggering full BAOS image builds when upstream components change
- Receiving build dispatch events from BAOS component repos
- Publishing BAOS releases with pre-built images

## How builds are triggered

### 1. Upstream HA release

A scheduled job (every 6 hours) polls the following upstream repos:

| Upstream repo | What it tracks |
|---|---|
| `home-assistant/operating-system` | HAOS base OS images (Buildroot) |
| `home-assistant/core` | HA Core (Python automation engine) |
| `home-assistant/supervisor` | HA Supervisor (container orchestrator) |
| `home-assistant/frontend` | HA web frontend |
| `home-assistant/iOS` | HA Companion for iOS |
| `home-assistant/android` | HA Companion for Android |

When a new release is detected, `upstream-versions.json` is updated
(committed) and a build is dispatched.

### 2. BAOS component change

Any push to `main` in a `marimotica/ba-*` repo fires a
`repository_dispatch` event of type `baos-component-update` here.

### 3. Manual

Use the "Run workflow" button on `.github/workflows/build.yml`.

## Required secrets

| Secret | Where to set | Description |
|---|---|---|
| `BAOS_BUILD_TOKEN` | Org-level or repo-level | PAT with `repo` and `workflow` scopes on `marimotica/baos-build`. Used by `check-upstream.yml` to push commits and dispatch events, and by component repos to dispatch events here. |

## Self-hosted runner

HAOS uses Buildroot which requires significant compute. Register a
self-hosted runner with the label **`self-hosted-baos-builder`**.

Minimum specs:
- 8 cores
- 16 GB RAM
- 100 GB disk
- Ubuntu 22.04

Register via: `Settings → Actions → Runners → New self-hosted runner`
(organisation level recommended so all `marimotica` repos can use it).

## Version tracking

`upstream-versions.json` is the source of truth for which upstream
versions BAOS is currently built against. Every version bump is a git
commit, giving a full audit trail.
