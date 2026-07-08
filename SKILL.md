---
name: fnpack-dev
version: 1.2.5
description: Develop and package apps for 飞牛 fnOS NAS using fnpack CLI. Use when creating, configuring, building fnOS .fpk packages, writing lifecycle scripts, setting app entries, or configuring wizards.
---

# fnpack Development — v1.2.3

Develop and package 飞牛 fnOS applications using `fnpack` CLI. Documentation: https://developer.fnnas.com/docs/cli/fnpack/

## Quick Start

```bash
# v1.2.3
fnpack create MyApp                           # Standard app
fnpack create MyApp --template docker         # Docker app
fnpack create MyApp --without-ui true         # Service only, no desktop icon
fnpack create MyApp --template docker --without-ui true  # Docker service only

# Edit generated files (see REFERENCE.md for all format specs)
fnpack build                                  # Generate .fpk
```

## CI/CD Integration

For GitHub Actions, use `mengzhuo/setup-fnpack@v1` to install fnpack in CI:

```yaml
- uses: mengzhuo/setup-fnpack@v1
- run: fnpack build
```

## Project Structure

```
myapp/
├── manifest              # App metadata — key=value format
├── ICON.PNG              # 64x64 px, sRGB, ≤1024KB
├── ICON_256.PNG          # 256x256 px
├── app/                  # App files
│   ├── ui/config         # Desktop entry definitions (JSON)
│   ├── ui/index.cgi      # CGI entry for static/light UI
│   └── docker/           # Docker template only
├── cmd/                  # Lifecycle scripts (bash): main, install_*, upgrade_*, uninstall_*, config_*
├── config/
│   ├── privilege         # Run-as user model (JSON)
│   └── resource          # Shared dirs, system links, docker project (JSON)
└── wizard/               # User input forms (JSON arrays): install, upgrade, uninstall, config
```

## Core Workflow

### 1. Manifest (`key=value`)

Minimum: `appname`, `version`, `display_name`, `source=thirdparty`, `platform`. Common additions: `service_port`, `ctl_stop=false` (static apps), `install_type=root`, `os_min_version`. Full field reference in REFERENCE.md.

### 2. Privilege (`config/privilege` JSON)

```json
{ "defaults": { "run-as": "package" }, "username": "myapp_user", "groupname": "myapp_group" }
```
- `run-as: "package"` — dedicated app user (default choice). Use `join-groups` for device access.
- `run-as: "root"` — only for privileged setup. Drop privileges (`runuser -u $TRIM_USERNAME -- ...`) before running services.

### 3. Resources (`config/resource` JSON)

Three types: `data-share` (user-accessible shared dirs, accessed via `$TRIM_DATA_SHARE_PATHS`), `usr-local-linker` (expose binaries/libs to `/usr/local/`), `docker-project` (declare Docker Compose stack). Only declare what the app actually needs.

### 4. Lifecycle Scripts (`cmd/`)

All bash. `cmd/main` handles `start`/`stop`/`status`. Callbacks: `install_init|callback`, `upgrade_init|callback`, `uninstall_init|callback`, `config_init|callback`. Exit codes: `0`=success/running, `1`=failure, `3`=not running (status). Write user errors to `$TRIM_TEMP_LOGFILE` before non-zero exit.

**Always use env vars, never hardcode paths**: `$TRIM_APPDEST`, `$TRIM_PKGETC`, `$TRIM_PKGVAR`, `$TRIM_PKGTMP`, `$TRIM_PKGHOME`, `$TRIM_SERVICE_PORT`.

### 5. App Entry (`app/ui/config` JSON)

Defines desktop icons and file type associations:
```json
{ ".url": { "AppName.Main": { "title": "...", "icon": "images/icon_{0}.png", "type": "iframe", "protocol": "http", "port": "8080", "url": "/", "allUsers": true } } }
```
- `type`: `iframe` (in fnOS desktop) or `url` (external browser)
- `fileTypes`: `["txt", "md"]` for file associations (app gets `?path=<filepath>` appended)
- `noDisplay: true`: hide from desktop but keep file association
- For CGI/static apps: entry URL is `/cgi/ThirdParty/<appname>/index.cgi/`, no port

### 6. Wizard Forms (`wizard/install|upgrade|uninstall|config` JSON)

Only use when user input is genuinely needed. JSON array of steps with `items[]`. Field types: `text`, `password`, `radio`, `checkbox`, `select`, `switch`, `tips`. Custom fields use `wizard_` prefix (never `TRIM_`). Values become env vars. Add `rules` for validation. See REFERENCE.md for full type details.

### 7. Icons

`ICON.PNG` (64×64) and `ICON_256.PNG` (256×256) at project root. Entry icons in `app/ui/images/icon_{64,256}.png`. Round-rect style, sRGB, ≤1024KB.

## App Patterns

| Pattern | Key Settings |
|---|---|
| **Static CGI** | `ctl_stop=false`, no `service_port`, entry via `index.cgi` URL |
| **Service** | `service_port` declared, `cmd/main` manages process |
| **Docker** | `--template docker`, `config/resource` declares `docker-project` |
| **Notepad (Full-stack)** | Unified Gateway, Node.js backend + frontend, `data-share`, Unix socket |

### Notepad Pattern — Full-stack Native App

The recommended pattern for UI apps with a backend service. Key decisions:

- **Gateway access**: `gatewayPrefix="/app/{appname}"`, `gatewaySocket="app.sock"` (no `service_port`)
- **Runtime**: `install_dep_apps=nodejs_v22`, `cmd/main` adds Node.js to `PATH` and starts server on `$TRIM_APPDEST/app.sock`
- **Persistence**: `config/resource` declares `data-share`, server writes to `$TRIM_DATA_SHARE_PATHS`
- **Auth**: server reads `X-Trim-Userid` header forwarded by gateway for per-user isolation
- **Build**: `npm run build:frontend` → copy backend + dist to `app/server/` → `fnpack build`
- **Entry**: `type: "iframe"`, `url: "/app/{appname}"`, `gatewayPrefix` + `gatewaySocket`

See [REFERENCE.md](REFERENCE.md) for full Notepad implementation walkthrough.

## Key Rules

- Use `$TRIM_*` env vars for ALL paths, never hardcode `/var/apps/...`
- Default `run-as: "package"`, not root
- Quote all shell variables
- Validate wizard/user values in scripts before use
- Keep field names stable across versions
- Make lifecycle scripts idempotent (may be re-executed)
- Minimal wizards: default what you can, ask only what you must

Full format specs, env var table, wizard field types, CGI template, build checks: see [REFERENCE.md](REFERENCE.md).
