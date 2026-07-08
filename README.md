# fnpack-dev — 飞牛 fnOS 应用开发 Skill

AI 辅助开发飞牛 fnOS 应用的 Agent Skill，基于 [fnpack](https://developer.fnnas.com/docs/cli/fnpack/) 文档构建。

## Install

```bash
mkdir -p ~/.config/opencode/skills
ln -s "$(pwd)/fnpack-dev" ~/.config/opencode/skills/fnpack-dev
```

Or clone directly:

```bash
git clone https://github.com/mengzhuo/skill-fnpack ~/.config/opencode/skills/fnpack-dev
```

## Usage

The skill auto-activates when the agent encounters fnpack/fnOS related tasks. It covers:

- **Project scaffolding** — `fnpack create` for standard, Docker, and service-only apps
- **Manifest config** — all key=value fields with descriptions
- **Lifecycle scripts** — `cmd/main` (start/stop/status), install/upgrade/uninstall/config callbacks
- **App entries** — desktop icons, file associations, CGI, unified gateway
- **Wizards** — install/upgrade/uninstall/config forms with validation
- **Resources** — shared dirs, system links, Docker projects
- **Environment variables** — complete `$TRIM_*` reference
- **App patterns** — Static CGI, Service, Docker, **Notepad (full-stack: Node.js + React + Gateway)**
- **Middleware & runtimes** — Redis, MinIO, RabbitMQ, Python, Node.js, Java
- **CI/CD** — `mengzhuo/setup-fnpack@v1` for GitHub Actions

## Structure

```
.
├── SKILL.md       # Core workflow (loaded by agent)
└── REFERENCE.md   # Full field specs, env vars, CGI/gateway templates
```

## License

MIT
