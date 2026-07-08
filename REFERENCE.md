# fnpack Reference

## Full Manifest Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `appname` | string | Yes | Unique app identifier |
| `version` | string | Yes | App version (e.g. `1.0.0`, `2.1.3-beta`) |
| `display_name` | string | Yes | Name shown in App Center and UI |
| `desc` | string | No | App description (HTML allowed) |
| `source` | string | Yes | `thirdparty` for 3rd-party apps |
| `platform` | string | Yes | `x86`, `arm`, or `all` (use `all` only if no arch-specific binaries) |
| `maintainer` | string | No | Developer/team name |
| `maintainer_url` | string | No | Developer website |
| `distributor` | string | No | Publisher (if different from maintainer) |
| `distributor_url` | string | No | Publisher website |
| `os_min_version` | string | No | Minimum fnOS version (e.g. `1.2.0`) |
| `os_max_version` | string | No | Maximum fnOS version |
| `ctl_stop` | bool | No | `true`=show start/stop, `false`=hide (for static/config apps) |
| `install_type` | string | No | Empty=user selects volume, `root`=system partition |
| `install_dep_apps` | string | No | Deps separated by `:`, use `>` for min version: `db>2.2.2:cache` |
| `desktop_uidir` | string | No | UI directory relative to app dir (default: `ui`) |
| `desktop_applaunchname` | string | No | Entry ID for App Center card (when multiple entries exist) |
| `service_port` | string | No | App service port |
| `checkport` | bool | No | Port check before start (default: `true`) |
| `disable_authorization_path` | bool | No | `true`=hide auth directory setting |
| `changelog` | string | No | Release notes for updates |

## Environment Variables

### App Info
| Variable | Source |
|---|---|
| `TRIM_APPNAME` | `manifest.appname` |
| `TRIM_APPVER` | Current version |
| `TRIM_OLD_APPVER` | Previous version (upgrade only) |
| `TRIM_APP_STATUS` | `INSTALL`, `START`, `UPGRADE`, `UNINSTALL`, `STOP`, `CONFIG` |

### App Paths
| Variable | Description |
|---|---|
| `TRIM_APPDEST` | Installed target directory |
| `TRIM_PKGETC` | App config directory |
| `TRIM_PKGVAR` | Runtime data directory (persists across restarts) |
| `TRIM_PKGTMP` | Temp directory |
| `TRIM_PKGHOME` | App user data directory |
| `TRIM_PKGMETA` | Metadata directory |
| `TRIM_APPDEST_VOL` | Storage volume path |

### User/Permission Context
| Variable | Description |
|---|---|
| `TRIM_USERNAME` | Dedicated app user |
| `TRIM_GROUPNAME` | Dedicated app group |
| `TRIM_UID` / `TRIM_GID` | App user/group IDs |
| `TRIM_RUN_USERNAME` / `TRIM_RUN_GROUPNAME` | Current script executor (may differ in privileged ops) |
| `TRIM_RUN_UID` / `TRIM_RUN_GID` | Current executor IDs |

### Network & Resources
| Variable | Description |
|---|---|
| `TRIM_SERVICE_PORT` | `manifest.service_port` |
| `TRIM_DATA_SHARE_PATHS` | Shared paths from `config/resource`, `:` separated |
| `TRIM_DATA_ACCESSIBLE_PATHS` | User-authorized accessible paths, `:` separated |

### Logs & Temp
| Variable | Description |
|---|---|
| `TRIM_TEMP_LOGFILE` | Write user-visible errors here before non-zero exit |
| `TRIM_TEMP_UPGRADE_FOLDER` | Upgrade temp dir |
| `TRIM_PKGINST_TEMP_DIR` | Install extraction temp dir |
| `TRIM_TEMP_TPKFILE` | Package extraction dir |

### System Context
| Variable | Description |
|---|---|
| `TRIM_SYS_VERSION` | Full fnOS version |
| `TRIM_SYS_VERSION_MAJOR` / `_MINOR` / `_BUILD` | Version parts |
| `TRIM_SYS_ARCH` | CPU architecture |
| `TRIM_KERNEL_VERSION` | Kernel version |
| `TRIM_SYS_MACHINE_ID` | Device unique ID |
| `TRIM_SYS_LANGUAGE` | System language |

## Privilege Config Reference

### `run-as: "package"` (Recommended)
```json
{
  "defaults": { "run-as": "package" },
  "username": "myapp_user",
  "groupname": "myapp_group"
}
```
- `username`/`groupname` optional — system auto-generates from `manifest.appname`
- Isolates app from system-level permissions

### `run-as: "package"` with `join-groups`
```json
{
  "defaults": { "run-as": "package" },
  "username": "media_app",
  "join-groups": ["video", "render"]
}
```
- App still runs as package user, but added to specified system groups
- Use for hardware access (GPU, video devices, etc.)
- Only add groups the app actually needs

### `run-as: "root"` (Limited use)
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

## Resource Config Reference

### `data-share` — Shared directories
```json
{
  "data-share": {
    "shares": [
      { "name": "myapp/documents" }
    ]
  }
}
```
- `name`: share path. Use `appname/` as top-level prefix.
- `permission` (optional): `{ "rw": ["user1"], "ro": ["user2"] }`
- Uses Windows ACL (not POSIX ACL)
- Access via `$TRIM_DATA_SHARE_PATHS` or `/var/apps/<appname>/share/`

### `usr-local-linker` — System link exposure
```json
{
  "usr-local-linker": {
    "bin": ["bin/myapp-cli"],
    "lib": ["lib/mylib.so"],
    "etc": ["etc/myapp.conf"]
  }
}
```
- `bin` linked to `/usr/local/bin/`
- `lib` linked to `/usr/local/lib/`
- `etc` linked to `/usr/local/etc/`
- Use app-prefixed names (e.g., `myapp-cli`, not `cli`)

### `docker-project` — Docker Compose
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
- `name`: Docker Compose project name
- `path`: relative to `app/` dir, must contain `docker-compose.yaml`

## Wizard Field Types

| Type | Use | Config |
|---|---|---|
| `text` | Short text, port, path, username | `initValue`, `rules` |
| `password` | Keys/tokens (masked display) | `initValue`, `rules` |
| `radio` | Few mutually exclusive options | `options: [{label, value}]` |
| `checkbox` | Multiple selections | `options: [{label, value}]` |
| `select` | Long list of mutually exclusive options | `options: [{label, value}]` |
| `switch` | Boolean toggle | `initValue` |
| `tips` | Read-only hint text | `helpText` |

### Validation Rules
```json
{ "required": true, "message": "Field required" }
{ "min": 1, "message": "Minimum 1" }
{ "max": 65535, "message": "Maximum 65535" }
{ "len": 8, "message": "Must be 8 chars" }
{ "pattern": "^[0-9]+$", "message": "Numbers only" }
```

## App Entry Config Reference

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

### Entry Fields

| Field | Description |
|---|---|
| `title` | User-facing entry name |
| `icon` | Icon path relative to UI dir. `{0}` replaced with 64/256 |
| `type` | `iframe` (in fnOS desktop) or `url` (external browser) |
| `protocol` | `http`, `https`, or empty (auto-detect) |
| `port` | Service port. Use `${wizard_port}` for wizard values |
| `url` | Entry path. Use `${wizard_path}` for wizard values |
| `allUsers` | `true`=all users, `false` or omit=variable visibility |
| `fileTypes` | Array of extensions for file associations |
| `noDisplay` | `true`=hide desktop icon, keep file association only |
| `control.accessPerm` | `editable` (user can edit), `readonly` (view-only), `hidden` |

### File Type Handling
When a file is opened via a file association, fnOS appends `?path=<file_path>` to the entry URL:
```
http://localhost:8080/edit?path=/vol1/Users/admin/Documents/example.txt
```
Always validate the `path` parameter before reading/writing.

### Entry Access Models
- **Port service**: Entry opens app's own service port. Independent of NAS login state.
- **CGI**: `app/ui/index.cgi` serves under system domain with NAS auth. Entry URL: `/cgi/ThirdParty/<appname>/index.cgi/`
- **Unified Gateway**: Registers route in `app/ui/config` with gateway middleware. Supports WebSocket and user context.

## CGI (index.cgi)

`index.cgi` is a lightweight CGI-based entry point. When users open a CGI URL, fnOS invokes the executable `app/ui/index.cgi`. CGI requests reuse the current access domain and route through the configured `url` path — `protocol` and `port` are ignored for CGI routing.

**Best for**: simple static pages, lightweight package compatibility. **Not suitable for**: WebSocket, long-running services, streaming, high-traffic APIs.

### Security Requirements
- Block `..` path traversal
- Only serve from expected app directories
- Return explicit status codes for missing/invalid files
- Never execute commands from request parameters
- Never concatenate request paths into shell commands

### Entry Config

```json
{ ".url": { "AppName.Main": {
    "title": "My App", "icon": "images/icon_{0}.png",
    "type": "iframe", "url": "/cgi/ThirdParty/appname/index.cgi/",
    "allUsers": true
} } }
```

URL format: `/cgi/ThirdParty/{appname}/index.cgi/`

### Static File Serving Template

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

## Unified Gateway

Unified Gateway gives apps a stable URL under the fnOS access domain. Requests are authenticated by fnOS, then forwarded to the app's Unix Socket. Supports WebSocket and persistent services.

### Access Model Comparison

| Capability | CGI | Gateway |
|---|---|---|
| Static pages | Good | Supported |
| Persistent services | Not recommended | Good |
| WebSocket | No | Yes |
| NAS auth | Before CGI call | Before forward + user Headers |
| Performance | New process per request | Forwarded to running service |
| Common path | `/cgi/ThirdParty/{app}/index.cgi/` | `/app/{appname}` |

### How It Works
1. App declares `gatewayPrefix` (public path) and `gatewaySocket` (Unix socket filename)
2. App service listens on the socket at `$TRIM_APPDEST/app.sock`
3. User opens `/app/myapp` → fnOS verifies session → forwards to socket
4. App reads forwarded user Headers for identity context

### Entry Config

```json
{ ".url": { "myapp.main": {
    "title": "My App", "icon": "images/icon_{0}.png",
    "type": "iframe", "protocol": "",
    "gatewayPrefix": "/app/myapp",
    "gatewaySocket": "app.sock",
    "url": "/app/myapp", "allUsers": true
} } }
```

- `gatewayPrefix`: `/app/{appname}` or `/app/{appname}/{customPath}`. Keep stable across versions. No dots in path.
- `gatewaySocket`: just the socket filename (e.g. `app.sock`). Socket goes in installed `target` dir.
- `protocol` and `port` are ignored for gateway routing.

### Auth Headers (forwarded by gateway)

| Header | Example | Description |
|---|---|---|
| `X-Trim-Userid` | `1000` | Current user UID |
| `X-Trim-Isadmin` | `true` / `false` | Is admin |
| `X-Trim-Username` | `admin` | Current username |

These are trusted identity context from the gateway. App still handles its own business authorization rules.

Node.js example:
```js
function getGatewayUser(req) {
  return {
    uid: req.headers["x-trim-userid"],
    isAdmin: req.headers["x-trim-isadmin"] === "true",
    username: req.headers["x-trim-username"]
  };
}
```

### WebSocket
Reuse same gateway prefix/socket. Put WebSocket routes under stable sub-paths like `/app/myapp/ws`. Gateway provides same identity context on WebSocket connect. Bind connection to `X-Trim-Userid` after establish.

```js
const wsProtocol = location.protocol === "https:" ? "wss:" : "ws:";
const wsUrl = `${wsProtocol}//${location.host}/app/myapp/ws`;
const socket = new WebSocket(wsUrl);
```

### Security Rules
- Users only access their own data
- Admin endpoints require admin check
- High-risk operations need explicit permission checks
- Validate file paths against current user
- Block `..` traversal. Never expose secrets, DB, configs, or private logs.
- OAuth callback paths stay narrow and explicit

## Application Dependencies

Declare dependencies in `manifest`:

```
install_dep_apps=database:cache
```

- System installs/enables deps right-to-left: `cache` before `database`
- Version requirement: `install_dep_apps=database>2.2.2:cache`
- Dependency resolution is NOT recursive — declare all direct deps explicitly
- Keep dependency names stable across versions
- Test on clean device to verify dep chain

## Middleware Services

Available middleware: `redis`, `minio`, `rabbitmq`. Declare via `install_dep_apps`.

### Redis
```
install_dep_apps=redis
```
Connect at `127.0.0.1:6379`. Use separate DB number or key prefix to avoid conflicts.

```python
import redis
client = redis.Redis(host="127.0.0.1", port=6379, db=1, decode_responses=True)
```

### MinIO
```
install_dep_apps=minio
```
Connect at `127.0.0.1:9000`. Never hardcode credentials in packages.

```python
from minio import Minio
client = Minio("127.0.0.1:9000", access_key="...", secret_key="...", secure=False)
```

### RabbitMQ
```
install_dep_apps=rabbitmq
```
Connect at `127.0.0.1:5672`. Use app-specific queues, exchanges, routing keys.

```python
import pika
conn = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
```

**Rules**: declare only needed services, namespace DB/bucket/queue/key names, never commit credentials.

## Runtime Environments

Packaged runtimes declared via `install_dep_apps`. Add runtime `bin` to `PATH` in lifecycle scripts before use.

| Runtime | Dependency | Binary Path |
|---|---|---|
| Python 3.12 | `python312` | `/var/apps/python312/target/bin` |
| Node.js v22 | `nodejs_v22` | `/var/apps/nodejs_v22/target/bin` |
| Java 21 OpenJDK | `java-21-openjdk` | `/var/apps/java-21-openjdk/target/bin` |

### Usage Examples

Python:
```bash
export PATH=/var/apps/python312/target/bin:$PATH
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

Node.js:
```bash
export PATH=/var/apps/nodejs_v22/target/bin:$PATH
node -v && npm -v
```

Java:
```bash
export PATH=/var/apps/java-21-openjdk/target/bin:$PATH
java --version
```

**Rules**: declare actual runtimes used, add to PATH before calling in scripts, keep app deps in app dir or virtual env, test on clean fnOS device.

## Build Validation Checklist

fnpack checks before generating `.fpk`:

| Path | Type | Requirement |
|---|---|---|
| `manifest` | File | Must exist with required fields |
| `config/privilege` | File | Must exist, valid JSON |
| `config/resource` | File | Must exist, valid JSON |
| `ICON.PNG` | File | Must exist |
| `ICON_256.PNG` | File | Must exist |
| `app/` | Dir | Must exist |
| `cmd/` | Dir | Must exist |
| `wizard/` | Dir | Must exist |
| `app/{desktop_uidir}/` | Dir | Must exist if `desktop_uidir` declared |

## Installed Directory Structure

After install, fnOS creates under `/var/apps/{appname}/`:

| Path | Purpose |
|---|---|
| `target -> /vol{n}/@appcenter/{appname}` | Installed app files |
| `etc -> /vol{n}/@appconf/{appname}` | App config (persist across upgrades) |
| `var -> /vol{n}/@appdata/{appname}` | Runtime data (persist across restarts) |
| `tmp -> /vol{n}/@apptemp/{appname}` | Temp files |
| `home -> /vol{n}/@apphome/{appname}` | User data |
| `shares/` | Shared dirs from `config/resource` |
| `meta/` | Metadata |
| `cmd/` | Lifecycle scripts |
| `wizard/` | Wizard forms |

## Lifecycle Script Execution Order

| Phase | Scripts (in order) |
|---|---|
| Install | `install_init` → copy files → `install_callback` |
| Start | `main start` |
| Stop | `main stop` |
| Upgrade | stop app → `upgrade_init` → copy files → `upgrade_callback` → start app |
| Uninstall | `uninstall_init` → cleanup → `uninstall_callback` |
| Config | `config_init` → apply → `config_callback` |

Scripts should be idempotent — install/upgrade/config may be re-executed.
