<div align="center">

# NovaHunter

### Self-hosted, AI-driven offensive-security control plane.

NovaHunter turns autonomous hacker agents into a team you can run on your
own VPS — with a full web dashboard, a REST + websocket API, and a single
bash installer that stands up the whole stack on a bare Ubuntu box.

<br/>

<a href="https://github.com/MaramHarsha/NovaHunter"><img src="https://img.shields.io/github/stars/MaramHarsha/NovaHunter?style=flat-square" alt="GitHub Stars"></a>
<a href="LICENSE"><img src="https://img.shields.io/badge/License-Apache%202.0-3b82f6?style=flat-square" alt="License"></a>
<a href="docs/INSTALL_UBUNTU.md"><img src="https://img.shields.io/badge/install-Ubuntu%20one--liner-2b9246?style=flat-square&logo=ubuntu&logoColor=white" alt="Ubuntu installer"></a>
<img src="https://img.shields.io/badge/Next.js-16-000000?style=flat-square&logo=next.js" alt="Next.js 16">
<img src="https://img.shields.io/badge/React-19.2-149eca?style=flat-square&logo=react" alt="React 19.2">
<img src="https://img.shields.io/badge/FastAPI-backend-009688?style=flat-square&logo=fastapi&logoColor=white" alt="FastAPI">
<img src="https://img.shields.io/badge/Docker-compose-2496ed?style=flat-square&logo=docker&logoColor=white" alt="Docker Compose">

</div>

---

## Contributing

Contributions are welcome. Please open [issues](https://github.com/MaramHarsha/NovaHunter/issues) for bugs or ideas, or submit a [pull request](https://github.com/MaramHarsha/NovaHunter/pulls) with a short description of the change and how you tested it.

---

## Credit & upstream

NovaHunter is a **fork of [Strix](https://github.com/usestrix/strix)**
(Apache-2.0 licensed) from the [UseStrix](https://strix.ai) team. All the
heavy lifting — the agent graph, hacker toolkit, HTTP proxy, browser
automation, vulnerability reasoning, LiteLLM-backed model adapters, run
artifact format — is theirs. Huge thanks to the Strix maintainers and
contributors for open-sourcing a genuinely remarkable project. If you like
what you see here, please also ⭐ the upstream repository.

NovaHunter builds **on top of** Strix with the following additions:

- A full **Next.js 16 web dashboard** (`frontend/`) — runs, findings,
  reports, analytics, admin panels, live run view, API docs, command palette.
- A **FastAPI backend** (`strix/api/`) exposing a REST + websocket API over
  the Strix agent core, with Postgres-backed metadata, Redis-backed rate
  limits, Clerk auth hooks, and an admin audit log.
- A production `docker compose` stack (`deploy/`) that wires the dashboard,
  the backend, Postgres, Redis, and a Caddy reverse proxy into one deploy.
- A **zero-prompt Ubuntu installer** (`scripts/setup.sh`) that stands up
  the whole stack on a fresh VPS with just a public IP.

The original Strix CLI (`strix --target …`) is still here and still works
the same way — NovaHunter extends the project, it does not replace it.

---

## What's included

| Tier        | Tech                                  | Lives in           |
| ----------- | ------------------------------------- | ------------------ |
| Dashboard   | Next.js 16 · React 19.2 · Tailwind    | `frontend/`        |
| API server  | FastAPI · SQLAlchemy · Redis          | `strix/api/`       |
| Agent core  | **Strix agent graph + hacker toolkit** | `strix/`           |
| Deploy      | docker compose · Caddy · Postgres 16 · Redis 7 | `deploy/` |
| Installer   | Pure bash, idempotent, non-interactive | `scripts/setup.sh` |
| Docs        | Ubuntu install guide · API reference   | `docs/`            |

---

## What's new in this build

- **Run control plane**: mid-run `pause` / `resume` / `restart` / `kill` + per-run budget caps.
- **Operator cockpit**: run-level **Terminals** tab (xterm), **Live Browser** tab (noVNC), and **Burp** tab.
- **Burp + Caido coexistence**: both available; agents can choose tooling per task.
- **Persistent shell + netcat workflows**: spawn/read/write/close shells and listener management APIs/UI.
- **LLM role router**: role-scoped model routes, per-run overrides, role usage/cost telemetry.
- **Encrypted secret store**: AES-GCM-backed secret references for provider/integration credentials.
- **Reporting upgrades**: canonical finding schema and exports in `md`, `txt`, `html`, `pdf`, `json`, `sarif`, `csv`.
- **Program governance**: finding dedup fingerprints, triage lifecycle, retest endpoint, scheduled scans.
- **Outbound integrations**: webhook, Slack, Discord, Jira, GitHub Issues.
- **Storage/abuse controls**: retention sweeps + evidence cap trimming + per-host politeness/rate limits.
- **MCP support**: NovaHunter as MCP server (`stdio`, `HTTP+SSE`) and MCP client (gallery + custom endpoints).

---

## Latest additions (2026-04-25)

- Added **Burp route hardening** so API Burp endpoints execute real Burp tool actions instead of static stubs.
- Added **durable MCP persistence**:
  - Postgres tables for MCP servers/tokens when DB is enabled.
  - File-backed fallback at `STRIX_RUNS_DIR/.config/mcp_registry.json` when Postgres is unavailable.
- Added **MCP Settings entry** in Settings for easier access to client-side MCP configuration.
- Added **new test scaffolding**:
  - `tests/api/test_mcp_routes.py`
  - `tests/api/test_burp_routes.py`
  - `tests/tools/test_capabilities_tool.py`
- Added comprehensive implementation history in `changelogs.md`.

## Latest additions (2026-05-01)

- Added **run restart** support in the control plane:
  - New `restart` action in run controls and command palette.
  - Restart reuses the previous run's saved target + scan configuration and starts a fresh run id.
- Added **NVIDIA NIM model normalization** for LiteLLM compatibility:
  - Models entered as `mistralai/...` against NIM's OpenAI-compatible endpoint are normalized automatically.
  - Prevents `LLM Provider NOT provided` errors on NIM setups.
- Hardened **run shell APIs**:
  - Shell sessions are auto-created on read/write paths so `default` shell polling does not 500 before explicit spawn.
- Improved **Runs mobile UX**:
  - Run detail tabs use horizontal scrolling with non-wrapping tab chips on small screens.
- Hardened **LLM usage telemetry endpoint**:
  - Defensive numeric parsing for token/cost aggregation to avoid malformed-event 500s.

---

## Quick start — self-host on Ubuntu in one command

Fresh Ubuntu 22.04 / 24.04 (or Debian 12+) VPS with a public IP is all you
need. No domain, no TLS cert, no LLM key required up front.

```bash
curl -fsSL https://raw.githubusercontent.com/MaramHarsha/NovaHunter/main/scripts/setup.sh | sudo bash
```

What happens in ~5 minutes:

1. Docker Engine + Compose plugin are installed if missing.
2. `deploy/.env` is generated with a random Postgres password, `STRIX_MASTER_KEY`, and safe defaults (Clerk keys left empty until you add your own — see [Environment & setup](#environment--setup) below).
3. Preflight checks validate required ports and runtime dependencies.
4. `docker compose up -d --build` launches **frontend + backend + Postgres + Redis + Caddy**.
5. Every container is waited on until `healthy`.
6. The installer prints:
   ```
   NovaHunter is live!
       Dashboard : http://<your-vps-ip>/
       API       : http://<your-vps-ip>/api/
       Health    : http://<your-vps-ip>/api/health
   ```

Browse to `http://<your-vps-ip>/`, sign in, and configure models in
**Settings → LLM** or **Settings → Advanced LLM**.

> The setup script expects to run from an existing checkout (it does not clone the repo for you).
> ```bash
> git clone https://github.com/MaramHarsha/NovaHunter.git
> cd NovaHunter
> sudo bash scripts/setup.sh
> ```

See **[docs/INSTALL_UBUNTU.md](docs/INSTALL_UBUNTU.md)** for the full guide
(manual install, HTTPS enablement, backups, `ufw`, troubleshooting).

---

## Web dashboard — what you get

- **Overview** — live status of active scans, most recent findings, severity mix.
- **Runs** — searchable history of every scan, live run view with agent
  tree / timeline / findings pane / log stream / run controls.
- **Run workbench** — **Terminals**, **Live Browser**, **Listeners**, and **Burp** tabs.
- **Findings** — severity-indexed list, detail page with CVSS / CWE / CVE
  metadata, reproduction PoC, affected code locations, remediation, status lifecycle.
- **Reports** — run artifact export in `md`, `txt`, `html`, `pdf`, `json`, `sarif`, `csv`.
- **Analytics** — trends, top targets, remediation velocity.
- **Admin** — organizations, LLM role routing, integrations, audit trail, platform settings.
- **API docs** — in-dashboard OpenAPI explorer with a "Try it" runner.
- **Settings** — provider config, role matrix, route tests, MCP settings, and environment controls.

### Live Browser (noVNC/VNC via iframe)

- The run details page includes a **Live Browser** tab that embeds noVNC in an iframe.
- Frontend requests run sidechannels from the API and renders the signed VNC URL.
- Sidechannels are routed through Caddy with run-scoped paths:
  - `/runs/{run_id}/vnc/...`
  - `/runs/{run_id}/shell/...`
  - `/runs/{run_id}/burp/...`
  - `/runs/{run_id}/ovpn/...`
  - `/runs/{run_id}/listeners/...`
- VNC/noVNC services run inside the sandbox container and are managed by `supervisord`.
- Access uses short-lived signed links issued by the sidechannels API.
- Caddy keeps strict security headers globally, but sidechannel routes remove `X-Frame-Options` so noVNC can render inside the dashboard iframe.

Fully responsive — the [previous mobile optimisation pass](#) adapted every
view to work on phones (hamburger drawer, stacked card tables, scalable
panels for the live view).

---

## CLI — original Strix agent (unchanged)

The standalone Strix CLI is preserved end-to-end. You can still use it on
its own, without running the dashboard.

**Prerequisites**

- Docker daemon running
- An LLM API key from any
  [supported provider](https://docs.strix.ai/llm-providers/overview)
  (OpenAI, Anthropic, OpenRouter, Google Vertex, Bedrock, Azure, …)

**Install the CLI**

```bash
curl -sSL https://strix.ai/install | bash     # upstream installer
export STRIX_LLM="openai/gpt-5.4"
export LLM_API_KEY="your-api-key"
```

**Run a scan**

```bash
# Local codebase
strix --target ./app-directory

# GitHub repository
strix --target https://github.com/org/repo

# Black-box web app
strix --target https://your-app.com
```

**Advanced usage**

```bash
# Grey-box authenticated test
strix --target https://your-app.com \
      --instruction "Authenticated testing using credentials: user:pass"

# Source + deployed, multi-target
strix -t https://github.com/org/app -t https://your-app.com

# White-box source-aware quick scan
strix --target ./app-directory --scan-mode standard

# Focused instruction via file
strix --target api.your-app.com --instruction-file ./instruction.md

# PR-diff scope in CI
strix -n --target ./ --scan-mode quick \
      --scope-mode diff --diff-base origin/main
```

**Headless mode** (`-n` / `--non-interactive`) prints findings in real time
and exits non-zero when vulnerabilities are found — drop it into any CI
pipeline:

```yaml
name: nova-pentest
on: [pull_request]
jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with: { fetch-depth: 0 }
      - name: Install Strix CLI
        run: curl -sSL https://strix.ai/install | bash
      - name: Scan
        env:
          STRIX_LLM: ${{ secrets.STRIX_LLM }}
          LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
        run: strix -n -t ./ --scan-mode quick
```

---

## Capabilities (inherited from Strix)

### Agentic security toolkit

- **Full HTTP proxy** — request/response manipulation and analysis
- **Browser automation** — multi-tab browser for XSS, CSRF, auth-flow testing
- **Terminal environments** — interactive shells for command execution
- **Python runtime** — custom exploit development & validation
- **Reconnaissance** — automated OSINT and attack-surface mapping
- **Code analysis** — static and dynamic capabilities
- **Knowledge management** — structured findings and attack documentation

### Vulnerability classes

- Access control — IDOR, privilege escalation, auth bypass
- Injection — SQL, NoSQL, command injection
- Server-side — SSRF, XXE, deserialization flaws
- Client-side — XSS, prototype pollution, DOM sinks
- Business logic — race conditions, workflow manipulation
- Authentication — JWT flaws, session management
- Infrastructure — misconfigurations, exposed services

### Graph-of-agents

- Distributed workflows — specialist agents per attack class or asset
- Parallel execution for broad, fast coverage
- Dynamic coordination — agents share discoveries in-flight

---

## Architecture

```
                       ┌──────────────────────────┐
 Internet ───────────► │  Caddy  (port 80 / 443)  │
                       └──────────┬───────────────┘
                                  │ strix_net (bridge)
                  ┌───────────────┼───────────────┐
                  │               │               │
           frontend (3000)   api (8000)    postgres / redis
           Next.js 16         FastAPI        state + cache
                                  │
                          ┌───────┴────────┐
                          │ Strix/NovaHunter │
                          │ agent core + tools│
                          └────────────────┘
```

- Only Caddy binds a host port. Postgres and Redis are unreachable from
  outside the Docker network.
- API boots in `development` mode by default so a fresh install is usable
  immediately; flip to `STRIX_ENV=production` once Clerk is configured.
- All service-to-service traffic stays on the internal bridge network.
- Sidechannel access (`/runs/{id}/vnc|shell|burp|ovpn|listeners`) is reverse-proxied through Caddy and signed by API-issued short-lived tokens.

---

## Project layout

```
NovaHunter/
├─ frontend/             # Next.js 16 dashboard (React 19.2)
├─ strix/                # Strix agent core (upstream, preserved)
│  └─ api/               # FastAPI backend + Dockerfile
├─ deploy/               # docker-compose, Caddyfile, .env.example
├─ scripts/
│  ├─ setup.sh           # one-shot Ubuntu installer  ← primary
│  ├─ web-install.sh     # legacy interactive installer
│  ├─ install.sh         # Strix CLI installer (upstream)
│  ├─ build.sh
│  └─ docker.sh
├─ docs/
│  ├─ INSTALL_UBUNTU.md  # full install / ops guide
│  └─ PARITY.md          # feature parity with upstream
├─ benchmarks/ · tests/ · containers/
├─ README.md             # ← this file
├─ LICENSE
└─ pyproject.toml        # agent / CLI dependencies
```

---

## Environment & setup

**Do not commit secrets.** The repo ignores `deploy/.env` and generic `.env` files. Only ever commit the templates (`deploy/.env.example`, `frontend/.env.example`).

### Full stack (Docker on Linux / VPS)

1. Clone the repository and `cd` into it.
2. **Create `deploy/.env`.** Either run `sudo bash scripts/setup.sh` (generates the file) or copy the template manually:
   ```bash
   cp deploy/.env.example deploy/.env
   ```
   Edit `deploy/.env`: set `POSTGRES_PASSWORD` to a long random string (unless the installer already generated one), and optionally fill [Clerk](https://dashboard.clerk.com) variables (`NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, `CLERK_ISSUER`, `CLERK_JWKS_URL`) plus `STRIX_ADMIN_EMAILS` / `STRIX_ADMIN_USER_IDS` for platform-admin access.
3. With `STRIX_ENV=development` and Clerk vars **empty**, the API still starts and uses a **demo principal** (no real JWT auth). For a normal dashboard login flow, configure Clerk and **rebuild** the frontend image so `NEXT_PUBLIC_*` keys are baked in: `cd deploy && docker compose up -d --build`.
4. Start the stack: `cd deploy && docker compose up -d --build` (or use the installer, which runs compose for you).

Variable reference: see [Configuration reference](#configuration-reference) and inline comments in [`deploy/.env.example`](deploy/.env.example).

### Frontend only (local Next.js)

```bash
cd frontend
cp .env.example .env.local
# Edit .env.local: set NEXT_PUBLIC_DEMO=true for standalone UI with fake data,
# or NEXT_PUBLIC_DEMO=false and NEXT_PUBLIC_API_BASE_URL to your API URL.
npm install
npm run dev
```

### LLM keys for scans

Scans need a model and provider key. Set `STRIX_LLM` and `LLM_API_KEY` in `deploy/.env`, or configure providers in the dashboard (**Settings → LLM**) once the app is running.

---

## Configuration reference

All runtime config lives in `deploy/.env`. The installer generates a
minimal file for you; extra knobs are listed in `deploy/.env.example` and
include:

| Variable                               | Purpose                                   |
| -------------------------------------- | ----------------------------------------- |
| `STRIX_ENV`                            | `development` (default) or `production`   |
| `POSTGRES_PASSWORD`                    | Auto-generated by the installer           |
| `NEXT_PUBLIC_API_BASE_URL`             | `/api` when behind Caddy                  |
| `NEXT_PUBLIC_DEMO`                     | `true` runs the dashboard with fake data  |
| `CLERK_ISSUER` / `_AUDIENCE` / `_JWKS_URL` | Optional auth (enforced in production) |
| `LLM_MODEL` / `LLM_API_KEY`            | LLM provider — or set from the admin UI    |
| `STRIX_MASTER_KEY`                     | AES-GCM key for encrypted secret storage    |
| `STRIX_IMAGE`                          | Sandbox image (default GHCR image)          |
| `STRIX_POLICY_MAX_RPS_PER_HOST`        | Per-target politeness rate limit            |
| `STRIX_POLICY_MAX_CONCURRENCY_PER_HOST`| Per-target concurrency cap                  |
| `STRIX_RETENTION_DAYS`                 | Run retention sweep window                  |
| `STRIX_RUN_MAX_EVIDENCE_BYTES`         | Per-run evidence size trimming cap          |
| `STRIX_ADMIN_EMAILS`                   | Comma-separated admin emails               |
| `STRIX_TRUSTED_HOSTS`                  | Allowed `Host:` values (default `*`)      |
| `STRIX_DOMAIN` / `STRIX_TLS_EMAIL`     | Populate to enable HTTPS via Caddy + Let's Encrypt |

---

## Development

```bash
# Frontend (Next.js 16, Turbopack)
cd frontend
npm install
npm run dev            # http://localhost:3000

# Type-check + lint
npm run type-check
npm run lint

# Production build
npm run build && npm run start
```

```bash
# Backend (FastAPI, served by uvicorn inside Docker)
cd deploy
docker compose up -d postgres redis api
docker compose logs -f api
```

The dashboard and the API can also be developed independently of the
Strix CLI. The CLI has its own install flow documented above.

---

## Documentation map

- `docs/INSTALL_UBUNTU.md` — deployment and operations bootstrap.
- `docs/integrations/outbound.mdx` — webhook/Slack/Discord/Jira/GitHub issue integrations.
- `docs/integrations/mcp.mdx` — MCP server/client config examples for Cursor/Claude Desktop.
- `docs/operations/retention.mdx` — retention and storage hygiene behavior.
- `docs/report-templates/` — canonical report templates and sections.

---

## Support / issues

For NovaHunter-specific issues (dashboard, API, installer, Ubuntu deploy)
file them against this fork:

- https://github.com/MaramHarsha/NovaHunter/issues

For questions about the Strix agent itself, the LLM providers, the hacker
toolkit, or the CLI, please use the upstream resources:

- Upstream repo: https://github.com/usestrix/strix
- Upstream docs: https://docs.strix.ai
- Discord: https://discord.gg/strix-ai

---

## License

Apache License 2.0 — see [LICENSE](LICENSE).

NovaHunter retains the upstream Strix copyright and license notices. Any
code written specifically for this fork (the `frontend/`, `strix/api/`,
`deploy/`, and `scripts/setup.sh`) is contributed under the same
Apache-2.0 license.

## Acknowledgements

- The **[Strix](https://github.com/usestrix/strix) team** — for building
  and open-sourcing the agent platform this project is built on.
- [LiteLLM](https://github.com/BerriAI/litellm),
  [Caido](https://github.com/caido/caido),
  [Nuclei](https://github.com/projectdiscovery/nuclei),
  [Playwright](https://github.com/microsoft/playwright), and
  [Textual](https://github.com/Textualize/textual) — foundations that
  Strix (and therefore NovaHunter) stands on.
- Vercel for Next.js 16, the React team for React 19.2, and the FastAPI
  maintainers — the web stack under the dashboard.

---

> [!WARNING]
> NovaHunter (and Strix) are offensive-security tools. Only test
> applications, infrastructure and domains **you own or have written
> permission to test**. You are responsible for using this project
> ethically and legally.
