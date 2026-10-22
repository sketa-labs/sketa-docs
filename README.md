# Sketa Docs

Documentation for [Sketa](https://sketa.app), the hand-drawn whiteboard, published at [sketa.app/docs](https://sketa.app/docs). Built with [Mintlify](https://mintlify.com).

## Development

Preview changes locally with the [Mintlify CLI](https://www.npmjs.com/package/mint):

```bash
npx mint dev
```

Preview at `http://localhost:3000`. Before opening a pull request, run:

```bash
npx mint validate
npx mint broken-links
```

## Publishing

Changes pushed to the `main` branch are deployed automatically by the Mintlify GitHub app.

## Structure

```
├── index.mdx, getting-started/   # Home, quickstart, no-account drawing, workspaces
├── knowledge-base/               # How the app works (editor, scenes, AI, workspace, account, API keys)
├── api-reference/ + openapi.json # REST API (operation pages generated from openapi.json)
├── ai/                           # MCP server, tools reference, format guides, client setup
├── support.mdx
├── logo/, favicon.svg, images/   # Brand assets and client icons
└── docs.json                     # Navigation, OpenAPI, branding
```

See `AGENTS.md` for the writing rules and where each fact comes from, and `CONTRIBUTING.md` to contribute.
