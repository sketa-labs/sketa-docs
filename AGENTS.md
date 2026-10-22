# Sketa Documentation

## Agent instructions

`AGENTS.md` is the single source of repository instructions for all coding agents. Read and follow this file before working on the documentation.

- Mintlify site (docs.sketa.app): pages are MDX with YAML frontmatter; navigation, `openapi` and `redirects` live in `docs.json`.
- Preview with `npx mint dev`; check with `npx mint validate` and `npx mint broken-links`.
- Every moved or removed slug gets a `docs.json` redirect.
- Every factual claim must trace to the app at `~/Herd/sketa` (cite `file:line` in the PR).
- Use active voice, second person, sentence-case headings, bold for UI labels exactly as in the app's `lang/en-US/*.php`.

## Project context

Sketa is a hand-drawn infinite whiteboard (an Excalidraw / Excalidraw+ alternative). Anyone can draw at `https://sketa.app` without an account; signing in saves scenes to the cloud; **Sketa Plus** workspaces (US$3 per seat/month or US$30 per seat/year) add collections, members, guests, teams, public links and embeds, slides, comments, version history, PDF/PPTX exports, the server library, the REST API and the MCP server.

Sketa is a hosted product only: it is **not open source and not self-hostable**. Never write self-hosting instructions, environment variables, install steps or "open-source" claims, and never link to the app's source code.

The app lives at `~/Herd/sketa` (Laravel + Inertia + Vue). Product spec: `docs/superpowers/specs/2026-10-21-sketa-product-design.md` there. This repo (`sketa-docs`) is the documentation site only.

## URLs

Everything is path-based on one domain.

| Service | URL |
|---------|-----|
| Editor (no account) | `https://sketa.app` |
| Dashboard | `https://sketa.app/dashboard` |
| Register / sign in | `https://sketa.app/register`, `https://sketa.app/login` |
| REST API | `https://sketa.app/api/v1` |
| MCP server | `https://sketa.app/mcp` |
| Public links | `https://sketa.app/l/{token}`, `/embed/{token}`, `/slides/{token}` |
| Scene content schema | `https://sketa.app/schemas/scene-content.v1.json` |
| Abuse reports | `https://sketa.app/abuse` |

There is no `api.sketa.app`, `mcp.sketa.app` or `app.sketa.app`.

## Documentation structure

4 tabs in `docs.json` (`navigation.tabs`):

| Tab | Icon | Purpose |
|-----|------|---------|
| **Documentation** | `book-open` | Home (`index`), `getting-started/` (quickstart, drawing without an account, workspaces/members/guests), `support` |
| **Knowledge Base** | `book` | How the app works: `knowledge-base/editor/`, `scenes/`, `ai`, `workspace/`, `account/`, `developers/`. Page titles and labels use the on-screen strings from `lang/en-US/*.php` |
| **API Reference** | `code` | REST API. Operation pages are generated from **`openapi.json`** at the repo root, written from the app's `routes/api.php`, `app/Http/Requests/Api/**`, `app/Http/Resources/Api/**` and `app/Http/Controllers/Api/**`. Prose lives in `api-reference/introduction.mdx` (auth, abilities, responses, pagination, errors, rate limits) and `api-reference/scene-content.mdx` (element format, PUT vs PATCH) |
| **Build with AI** | `microchip-ai` | MCP intro (`ai/introduction`), `ai/tools-reference`, `ai/format-guides`, one setup guide per client |

## Writing rules

- Describe the current app only. No roadmap promises beyond what the app itself shows (voice and screen sharing are "coming later").
- Mark Plus-only features with a `<Note>` at the top of the page or section ("… needs Sketa Plus.").
- Use the UI labels from `lang/en-US/*.php` in bold. Settings live in **Settings → Account** (Profile, Security, Preferences) and **Settings → Workspace** (General, Billing, Teams, Audit log, API keys, Workspaces).
- Locales are `en-US` and `pt-BR` only. Docs are written in English only.
- Don't invent screenshots. Images live in `images/`; only add real captures of the app.

### URLs in examples

- API curl examples: `https://sketa.app/api/v1/...` with `Authorization: Bearer sketa_YOUR_API_KEY`.
- MCP configs: `https://sketa.app/mcp`, server name `sketa`.

## API reference

- `openapi.json` is the source of truth for every operation, field and error. Public API = the `/api/v1` routes (`php artisan route:list --path=api/v1`), 35 operations today. The anonymous `api/snapshots` and `api/rooms` routes are internal to the editor; never document them.
- Auth: `Authorization: Bearer sketa_...`. Keys are Passport personal-access tokens bound to one workspace (`App\Models\AccessToken`), **personal** (act as the member) or **workspace** (act as the workspace, shared collections only, admin-equivalent). Abilities come from `App\Enums\Api\Ability`; each route declares one with `abilities:` middleware.
- `ResolveApiActor` errors: `401 Unauthenticated.` / `API key expired.` / `API key revoked.`, `402 Sketa Plus required.`, `403 Personal API keys are turned off in this workspace.`, `403 Workspace is read-only.` (writes only).
- Error shape is always `{statusCode, error, message}` plus `errors` on 422 (`App\Support\Api\ApiErrorResponse`).
- Single resources are wrapped in `data`; lists use the Laravel envelope `{data, links, meta}` with `config('app.pagination.default')` (25). Always tell readers to read `meta.per_page`. Scene content responses are not wrapped.
- Rate limits: 120/min per key, 600/min per workspace (`config/sketa.php` `api`), 429 with `Retry-After`.
- Enum values (audit actions, roles, element types) are copied from the app's enums and `public/schemas/scene-content.v1.json`, never from memory.
- When the API changes, regenerate or edit `openapi.json` and update `docs.json` navigation (`"METHOD /path"` entries).

## MCP server

- The tool list is exactly `SketaServer::$tools` (`app/Mcp/Servers/SketaServer.php`), 42 tools today; `ai/tools-reference.mdx` documents every one with its inputs (from each tool's `schema()`), its ability (`asActor(...)`) and its annotations (`#[IsReadOnly]`, `#[IsDestructive]`, `#[IsOpenWorld]`).
- Route: `Mcp::web('/mcp', SketaServer::class)` behind `api.key`, `auth:api`, `LoadMcpActor` and `throttle:mcp` (120/min per token). It accepts an OAuth grant (`mcp:use`, consent screen picks one Plus workspace, acts as the user with every ability) **or** an API key in the `Authorization` header.
- `ai/format-guides.mdx` summarizes `resources/views/prompts/mcp/guides/*.blade.php`; keep it in sync.

## Plans and limits (from `config/sketa.php`)

- Free: 1 cloud scene (`scenes.free_scene_limit`), no members.
- AI per UTC day: Free 10, Plus member 100, anonymous 10 per device and 30 per IP; text-to-diagram costs 1, wireframe-to-code 5, scene naming 0 (`App\Enums\AiUsage\Kind::quotaCost()`).
- Trash keeps items 30 days; payment grace period 7 days, then read-only; up to 20 public links per scene; invites expire after 3 days; images 4 MB (16 MB on Plus).

## Running docs locally

```bash
npx mint dev
```
