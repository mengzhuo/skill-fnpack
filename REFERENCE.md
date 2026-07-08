# fnpack Reference

> **Version:** v1.2.3 — Based on [fnpack CLI docs](https://developer.fnnas.com/docs/cli/fnpack/)

## Full Manifest Fields *(v1.2.3)*

| Field | Type | Req | Since | Description |
|---|---|---|---|---|
| `appname` | string | Yes | v1.2.3 | Unique app identifier |
| `version` | string | Yes | v1.2.3 | App version (e.g. `1.0.0`, `2.1.3-beta`) |
| `display_name` | string | Yes | v1.2.3 | Name shown in App Center and UI |
| `desc` | string | No | v1.2.3 | App description (HTML allowed) |
| `source` | string | Yes | v1.2.3 | `thirdparty` for 3rd-party apps |
| `platform` | string | Yes | v1.2.3 | `x86`, `arm`, or `all` (use `all` only if no arch-specific binaries) |
| `maintainer` | string | No | v1.2.3 | Developer/team name |
| `maintainer_url` | string | No | v1.2.3 | Developer website |
| `distributor` | string | No | v1.2.3 | Publisher (if different from maintainer) |
| `distributor_url` | string | No | v1.2.3 | Publisher website |
| `os_min_version` | string | No | v1.2.3 | Minimum fnOS version (e.g. `1.2.0`) |
| `os_max_version` | string | No | v1.2.3 | Maximum fnOS version |
| `ctl_stop` | bool | No | v1.2.3 | `true`=show start/stop, `false`=hide (for static/config apps) |
| `install_type` | string | No | v1.2.3 | Empty=user selects volume, `root`=system partition |
| `install_dep_apps` | string | No | v1.2.3 | Deps separated by `:`, use `>` for min version: `db>2.2.2:cache` |
| `desktop_uidir` | string | No | v1.2.3 | UI directory relative to app dir (default: `ui`) |
| `desktop_applaunchname` | string | No | v1.2.3 | Entry ID for App Center card (when multiple entries exist) |
| `service_port` | string | No | v1.2.3 | App service port |
| `checkport` | bool | No | v1.2.3 | Port check before start (default: `true`) |
| `disable_authorization_path` | bool | No | v1.2.3 | `true`=hide auth directory setting |
| `changelog` | string | No | v1.2.3 | Release notes for updates |

## Environment Variables *(v1.2.3)*

### App Info
| Variable | Source | Since |
|---|---|---|
| `TRIM_APPNAME` | `manifest.appname` | v1.2.3 |
| `TRIM_APPVER` | Current version | v1.2.3 |
| `TRIM_OLD_APPVER` | Previous version (upgrade only) | v1.2.3 |
| `TRIM_APP_STATUS` | `INSTALL`, `START`, `UPGRADE`, `UNINSTALL`, `STOP`, `CONFIG` | v1.2.3 |

### App Paths
| Variable | Description | Since |
|---|---|---|
| `TRIM_APPDEST` | Installed target directory | v1.2.3 |
| `TRIM_PKGETC` | App config directory | v1.2.3 |
| `TRIM_PKGVAR` | Runtime data directory (persists across restarts) | v1.2.3 |
| `TRIM_PKGTMP` | Temp directory | v1.2.3 |
| `TRIM_PKGHOME` | App user data directory | v1.2.3 |
| `TRIM_PKGMETA` | Metadata directory | v1.2.3 |
| `TRIM_APPDEST_VOL` | Storage volume path | v1.2.3 |

### User/Permission Context
| Variable | Description | Since |
|---|---|---|
| `TRIM_USERNAME` | Dedicated app user | v1.2.3 |
| `TRIM_GROUPNAME` | Dedicated app group | v1.2.3 |
| `TRIM_UID` / `TRIM_GID` | App user/group IDs | v1.2.3 |
| `TRIM_RUN_USERNAME` / `TRIM_RUN_GROUPNAME` | Current script executor (may differ in privileged ops) | v1.2.3 |
| `TRIM_RUN_UID` / `TRIM_RUN_GID` | Current executor IDs | v1.2.3 |

### Network & Resources
| Variable | Description | Since |
|---|---|---|
| `TRIM_SERVICE_PORT` | `manifest.service_port` | v1.2.3 |
| `TRIM_DATA_SHARE_PATHS` | Shared paths from `config/resource`, `:` separated | v1.2.3 |
| `TRIM_DATA_ACCESSIBLE_PATHS` | User-authorized accessible paths, `:` separated | v1.2.3 |

### Logs & Temp
| Variable | Description | Since |
|---|---|---|
| `TRIM_TEMP_LOGFILE` | Write user-visible errors here before non-zero exit | v1.2.3 |
| `TRIM_TEMP_UPGRADE_FOLDER` | Upgrade temp dir | v1.2.3 |
| `TRIM_PKGINST_TEMP_DIR` | Install extraction temp dir | v1.2.3 |
| `TRIM_TEMP_TPKFILE` | Package extraction dir | v1.2.3 |

### System Context
| Variable | Description | Since |
|---|---|---|
| `TRIM_SYS_VERSION` | Full fnOS version | v1.2.3 |
| `TRIM_SYS_VERSION_MAJOR` / `_MINOR` / `_BUILD` | Version parts | v1.2.3 |
| `TRIM_SYS_ARCH` | CPU architecture | v1.2.3 |
| `TRIM_KERNEL_VERSION` | Kernel version | v1.2.3 |
| `TRIM_SYS_MACHINE_ID` | Device unique ID | v1.2.3 |
| `TRIM_SYS_LANGUAGE` | System language | v1.2.3 |

## Privilege Config Reference *(v1.2.3)*

### `run-as: "package"` (Recommended) — v1.2.3
```json
{
  "defaults": { "run-as": "package" },
  "username": "myapp_user",
  "groupname": "myapp_group"
}
```
- `run-as` *(v1.2.3)* — running identity, `package` = dedicated app user
- `username` / `groupname` *(v1.2.3)* — optional, auto-generated from `manifest.appname`
- Isolates app from system-level permissions

### `run-as: "package"` with `join-groups` — v1.2.3
```json
{
  "defaults": { "run-as": "package" },
  "username": "media_app",
  "join-groups": ["video", "render"]
}
```
- `join-groups` *(v1.2.3)* — adds app user to system groups (GPU, video devices, etc.)
- Only add groups the app actually needs

### `run-as: "root"` (Limited use) — v1.2.3
```json
{
  "defaults": { "run-as": "root" },
  "username": "myapp_user",
  "groupname": "myapp_group"
}
```
- Only for privileged setup in lifecycle scripts
- Drop privileges before running services:
  ```bash
  runuser -u "$TRIM_USERNAME" -- /var/apps/myapp/target/server/myapp
  ```

## Resource Config Reference *(v1.2.3)*

### `data-share` — Shared directories *(v1.2.3)*
```json
{
  "data-share": {
    "shares": [
      { "name": "myapp/documents" }
    ]
  }
}
```
- `data-share` *(v1.2.3)* — declares user-accessible shared directories
- `name` *(v1.2.3)* — share path, use `appname/` as top-level prefix
- `permission` *(v1.2.3)* — optional: `{ "rw": ["user1"], "ro": ["user2"] }`
- Uses Windows ACL (not POSIX ACL)
- Access via `$TRIM_DATA_SHARE_PATHS` or `/var/apps/<appname>/share/`

### `usr-local-linker` — System link exposure *(v1.2.3)*
```json
{
  "usr-local-linker": {
    "bin": ["bin/myapp-cli"],
    "lib": ["lib/mylib.so"],
    "etc": ["etc/myapp.conf"]
  }
}
```
- `usr-local-linker` *(v1.2.3)* — expose files to standard system paths
- `bin` *(v1.2.3)* → `/usr/local/bin/`
- `lib` *(v1.2.3)* → `/usr/local/lib/`
- `etc` *(v1.2.3)* → `/usr/local/etc/`
- Use app-prefixed names (e.g., `myapp-cli`, not `cli`)

### `docker-project` — Docker Compose *(v1.2.3)*
```json
{
  "docker-project": {
    "projects": [{
      "name": "myapp-stack",
      "path": "docker"
    }]
  }
}
```
- `docker-project` *(v1.2.3)* — declares Docker Compose based app
- `name` *(v1.2.3)* — Docker Compose project name
- `path` *(v1.2.3)* — relative to `app/` dir, must contain `docker-compose.yaml`

## Wizard Field Types *(v1.2.3)*

| Type | Use | Config | Since |
|---|---|---|---|
| `text` | Short text, port, path, username | `initValue`, `rules` | v1.2.3 |
| `password` | Keys/tokens (masked display) | `initValue`, `rules` | v1.2.3 |
| `radio` | Few mutually exclusive options | `options: [{label, value}]` | v1.2.3 |
| `checkbox` | Multiple selections | `options: [{label, value}]` | v1.2.3 |
| `select` | Long list of mutually exclusive options | `options: [{label, value}]` | v1.2.3 |
| `switch` | Boolean toggle | `initValue` | v1.2.3 |
| `tips` | Read-only hint text | `helpText` | v1.2.3 |

### Validation Rules *(v1.2.3)*
```json
{ "required": true, "message": "Field required" }
{ "min": 1, "message": "Minimum 1" }
{ "max": 65535, "message": "Maximum 65535" }
{ "len": 8, "message": "Must be 8 chars" }
{ "pattern": "^[0-9]+$", "message": "Numbers only" }
```
- `required` *(v1.2.3)*, `min` *(v1.2.3)*, `max` *(v1.2.3)*, `len` *(v1.2.3)*, `pattern` *(v1.2.3)*

## App Entry Config Reference *(v1.2.3)*

`app/ui/config` JSON:

```json
{
  ".url": {
    "AppName.EntryID": {
      "title": "Display Name",
      "icon": "images/icon_{0}.png",
      "type": "iframe",
      "protocol": "http",
      "port": "8080",
      "url": "/",
      "allUsers": true,
      "fileTypes": ["txt", "md"],
      "noDisplay": false,
      "control": {
        "accessPerm": "editable"
      }
    }
  }
}
```

### Entry Fields *(v1.2.3)*

| Field | Description | Since |
|---|---|---|
| `.url` | Entry definitions root key | v1.2.3 |
| `title` | User-facing entry name | v1.2.3 |
| `icon` | Icon path relative to UI dir. `{0}` replaced with 64/256 | v1.2.3 |
| `type` | `iframe` (in fnOS desktop) or `url` (external browser) | v1.2.3 |
| `protocol` | `http`, `https`, or empty (auto-detect) | v1.2.3 |
| `port` | Service port. Use `${wizard_port}` for wizard values | v1.2.3 |
| `url` | Entry path. Use `${wizard_path}` for wizard values | v1.2.3 |
| `allUsers` | `true`=all users, `false` or omit=variable visibility | v1.2.3 |
| `fileTypes` | Array of extensions for file associations | v1.2.3 |
| `noDisplay` | `true`=hide desktop icon, keep file association only | v1.2.3 |
| `control.accessPerm` | `editable` (user can edit), `readonly` (view-only), `hidden` | v1.2.3 |
| `gatewayPrefix` | Gateway route path (e.g. `/app/myapp`) | v1.2.3 |
| `gatewaySocket` | Unix socket filename (e.g. `app.sock`) | v1.2.3 |

## CLI Commands *(v1.2.3)*

| Command | Flags | Since | Description |
|---|---|---|---|
| `fnpack create <name>` | — | v1.2.3 | Standard app with desktop entry |
| `fnpack create <name>` | `--template docker` | v1.2.3 | Docker Compose app |
| `fnpack create <name>` | `--without-ui true` | v1.2.3 | Service only, no desktop icon |
| `fnpack build` | — | v1.2.3 | Build .fpk in current dir |
| `fnpack build` | `--directory <path>` | v1.2.3 | Build .fpk from specified dir |

### GitHub Actions *(v1.2.3)*

Use [`mengzhuo/setup-fnpack@v1`](https://github.com/mengzhuo/setup-fnpack) to install fnpack in CI:

```yaml
- uses: mengzhuo/setup-fnpack@v1
- run: fnpack build
```

## CGI (index.cgi) *(v1.2.3)*

`app/ui/index.cgi` is a lightweight CGI-based entry point. When users open a CGI URL, fnOS invokes the executable CGI script. `protocol` and `port` are ignored for CGI routing.

**Best for**: simple static pages. **Not suitable for**: WebSocket, long-running services, streaming.

### Security Requirements *(v1.2.3)*
- Block `..` path traversal
- Only serve from expected app directories
- Never execute commands from request parameters

### Entry Config *(v1.2.3)*

```json
{ ".url": { "AppName.Main": {
    "title": "My App", "icon": "images/icon_{0}.png",
    "type": "iframe", "url": "/cgi/ThirdParty/appname/index.cgi/",
    "allUsers": true
} } }
```

URL format: `/cgi/ThirdParty/{appname}/index.cgi/` *(v1.2.3)*

### Static File Serving Template *(v1.2.3)*

```bash
#!/bin/bash
BASE_PATH="/var/apps/APPNAME/target/www"
URI_NO_QUERY="${REQUEST_URI%%\?*}"
REL_PATH="/"
case "$URI_NO_QUERY" in
  *index.cgi*) REL_PATH="${URI_NO_QUERY#*index.cgi}" ;;
esac
[ -z "$REL_PATH" ] || [ "$REL_PATH" = "/" ] && REL_PATH="/index.html"
TARGET_FILE="${BASE_PATH}${REL_PATH}"

# Block path traversal
if echo "$TARGET_FILE" | grep -q '\.\.'; then
  echo "Status: 400 Bad Request"
  echo "Content-Type: text/plain; charset=utf-8"
  echo ""; echo "Bad Request"; exit 0
fi

if [ ! -f "$TARGET_FILE" ]; then
  echo "Status: 404 Not Found"
  echo "Content-Type: text/plain; charset=utf-8"
  echo ""; echo "404 Not Found"; exit 0
fi

# MIME detection
case "${TARGET_FILE##*.}" in
  html|htm) mime="text/html; charset=utf-8" ;;
  css) mime="text/css; charset=utf-8" ;;
  js) mime="application/javascript; charset=utf-8" ;;
  png) mime="image/png" ;;
  jpg|jpeg) mime="image/jpeg" ;;
  svg) mime="image/svg+xml" ;;
  *) mime="application/octet-stream" ;;
esac
echo "Content-Type: $mime"
echo ""; cat "$TARGET_FILE"
```

## Unified Gateway *(v1.2.3)*

Requests authenticated by fnOS, forwarded to app's Unix Socket. Supports WebSocket.

### Access Model Comparison

| Capability | CGI | Gateway | Since |
|---|---|---|---|
| Static pages | Good | Supported | v1.2.3 |
| Persistent services | Not recommended | Good | v1.2.3 |
| WebSocket | No | Yes | v1.2.3 |
| NAS auth | Before CGI call | Before forward + user Headers | v1.2.3 |
| Performance | New process per request | Forwarded to running service | v1.2.3 |
| Common path | `/cgi/ThirdParty/{app}/index.cgi/` | `/app/{appname}` | v1.2.3 |

### Entry Config *(v1.2.3)*

```json
{ ".url": { "myapp.main": {
    "title": "My App", "icon": "images/icon_{0}.png",
    "type": "iframe", "protocol": "",
    "gatewayPrefix": "/app/myapp",
    "gatewaySocket": "app.sock",
    "url": "/app/myapp", "allUsers": true
} } }
```

- `gatewayPrefix` *(v1.2.3)* — `/app/{appname}` or `/app/{appname}/{customPath}`. No dots in path.
- `gatewaySocket` *(v1.2.3)* — socket filename (e.g. `app.sock`), placed in `$TRIM_APPDEST`.

### Auth Headers *(v1.2.3)*

| Header | Example | Description | Since |
|---|---|---|---|
| `X-Trim-Userid` | `1000` | Current user UID | v1.2.3 |
| `X-Trim-Isadmin` | `true` / `false` | Is admin | v1.2.3 |
| `X-Trim-Username` | `admin` | Current username | v1.2.3 |

Node.js:
```js
function getGatewayUser(req) {
  return {
    uid: req.headers["x-trim-userid"],
    isAdmin: req.headers["x-trim-isadmin"] === "true",
    username: req.headers["x-trim-username"]
  };
}
```

### WebSocket *(v1.2.3)*
Use sub-paths like `/app/myapp/ws`. Gateway forwards identity context on connect.

```js
const wsProtocol = location.protocol === "https:" ? "wss:" : "ws:";
const wsUrl = `${wsProtocol}//${location.host}/app/myapp/ws`;
const socket = new WebSocket(wsUrl);
```

## Application Dependencies *(v1.2.3)*

```
install_dep_apps=database:cache
```
- `install_dep_apps` *(v1.2.3)* — colon-separated deps, right-to-left install order
- Version requirement *(v1.2.3)*: `install_dep_apps=database>2.2.2:cache`
- Not recursive — declare all direct deps explicitly

## Middleware Services *(v1.2.3)*

Available: `redis`, `minio`, `rabbitmq`. Declared via `install_dep_apps`.

### Redis *(v1.2.3)*
```
install_dep_apps=redis
```
Connect at `127.0.0.1:6379`. Use separate DB number or key prefix.

### MinIO *(v1.2.3)*
```
install_dep_apps=minio
```
Connect at `127.0.0.1:9000`. Never hardcode credentials.

### RabbitMQ *(v1.2.3)*
```
install_dep_apps=rabbitmq
```
Connect at `127.0.0.1:5672`. Use app-specific queues/routing keys.

## Runtime Environments *(v1.2.3)*

Declared via `install_dep_apps`. Add `bin` to `PATH` before use.

| Runtime | Dependency | Binary Path | Since |
|---|---|---|---|
| Python 3.12 *(v1.2.3)* | `python312` | `/var/apps/python312/target/bin` | v1.2.3 |
| Node.js v22 *(v1.2.3)* | `nodejs_v22` | `/var/apps/nodejs_v22/target/bin` | v1.2.3 |
| Java 21 *(v1.2.3)* | `java-21-openjdk` | `/var/apps/java-21-openjdk/target/bin` | v1.2.3 |

## Build Validation Checklist *(v1.2.3)*

`fnpack build` validates before generating `.fpk`:

| Path | Type | Requirement | Since |
|---|---|---|---|
| `manifest` | File | Must exist with required fields | v1.2.3 |
| `config/privilege` | File | Must exist, valid JSON | v1.2.3 |
| `config/resource` | File | Must exist, valid JSON | v1.2.3 |
| `ICON.PNG` | File | Must exist | v1.2.3 |
| `ICON_256.PNG` | File | Must exist | v1.2.3 |
| `app/` | Dir | Must exist | v1.2.3 |
| `cmd/` | Dir | Must exist | v1.2.3 |
| `wizard/` | Dir | Must exist | v1.2.3 |
| `app/{desktop_uidir}/` | Dir | Must exist if `desktop_uidir` declared | v1.2.3 |

## Installed Directory Structure *(v1.2.3)*

After install under `/var/apps/{appname}/`:

| Path | Purpose | Since |
|---|---|---|
| `target -> /vol{n}/@appcenter/{appname}` | Installed app files | v1.2.3 |
| `etc -> /vol{n}/@appconf/{appname}` | App config (persist across upgrades) | v1.2.3 |
| `var -> /vol{n}/@appdata/{appname}` | Runtime data (persist across restarts) | v1.2.3 |
| `tmp -> /vol{n}/@apptemp/{appname}` | Temp files | v1.2.3 |
| `home -> /vol{n}/@apphome/{appname}` | User data | v1.2.3 |
| `shares/` | Shared dirs from `config/resource` | v1.2.3 |
| `meta/` | Metadata | v1.2.3 |
| `cmd/` | Lifecycle scripts | v1.2.3 |
| `wizard/` | Wizard forms | v1.2.3 |

## Lifecycle Script Execution Order *(v1.2.3)*

| Phase | Scripts (in order) | Since |
|---|---|---|
| Install | `install_init` → copy files → `install_callback` | v1.2.3 |
| Start | `main start` | v1.2.3 |
| Stop | `main stop` | v1.2.3 |
| Upgrade | stop → `upgrade_init` → copy → `upgrade_callback` → start | v1.2.3 |
| Uninstall | `uninstall_init` → cleanup → `uninstall_callback` | v1.2.3 |
| Config | `config_init` → apply → `config_callback` | v1.2.3 |

Scripts must be idempotent — install/upgrade/config may be re-executed. *(v1.2.3)*
