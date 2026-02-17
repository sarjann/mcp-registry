# mcp-registry

The official package registry for [mcper](https://github.com/sarjann/mcper) — a Homebrew-style MCP package manager for AI coding clients.

## Available packages

| Package | Description | Transport |
|---------|-------------|-----------|
| [brave-search](packages/brave-search) | Brave Search for web and local search | stdio |
| [cloudflare](packages/cloudflare) | Cloudflare Workers, KV, R2, D1, and more | stdio |
| [context7](packages/context7) | Up-to-date library documentation and code examples | stdio |
| [convex](packages/convex) | Official Convex MCP server | stdio |
| [everything](packages/everything) | MCP reference server (all features) | stdio |
| [filesystem](packages/filesystem) | Local filesystem access | stdio |
| [firecrawl](packages/firecrawl) | Web scraping, crawling, and content extraction | stdio |
| [github](packages/github) | Official GitHub MCP server | http |
| [gitlab](packages/gitlab) | GitLab repos, issues, merge requests, pipelines | stdio |
| [google-maps](packages/google-maps) | Geocoding, directions, places, elevation | stdio |
| [linear](packages/linear) | Issue tracking and project management | stdio |
| [memory](packages/memory) | Knowledge graph memory | stdio |
| [notion](packages/notion) | Notion pages and databases | stdio |
| [playwright](packages/playwright) | Browser automation with Playwright | stdio |
| [postgres](packages/postgres) | PostgreSQL query and schema inspection | stdio |
| [puppeteer](packages/puppeteer) | Browser automation with Puppeteer | stdio |
| [sentry](packages/sentry) | Error tracking and performance monitoring | stdio |
| [sequential-thinking](packages/sequential-thinking) | Dynamic, reflective problem-solving | stdio |
| [slack](packages/slack) | Slack channels, messages, and workflows | stdio |
| [stripe](packages/stripe) | Payments, subscriptions, and billing | stdio |
| [supabase](packages/supabase) | Database, auth, storage, and edge functions | stdio |
| [tavily](packages/tavily) | AI-optimized web search and extraction | stdio |
| [upstash](packages/upstash) | Redis, Kafka, QStash, and Vector databases | stdio |
| [vercel](packages/vercel) | Official Vercel MCP server | http |

## Usage

This registry is the default `official` tap that ships with mcper. No setup needed.

```bash
# Search for packages
mcper search memory

# Install a package (auto-detects your AI clients)
mcper install convex

# Install a specific version
mcper install convex@1.31.7

# Install to specific clients only
mcper install convex --target claude,cursor
```

See the [mcper README](https://github.com/sarjann/mcper) for full CLI usage.

## Registry structure

```
index.json                              # Package index with SHA-256 hashes
packages/
  <package-name>/
    <version>/
      manifest.json                     # Full package manifest
```

For the complete schema reference, see the [registry format docs](https://github.com/sarjann/mcper/blob/master/docs/registry.md).

## Contributing

Contributions are welcome! To add or update a package, open a pull request following the guidelines below.

### Adding a new package

1. Create the directory structure: `packages/<name>/<version>/manifest.json`
2. Add an entry to `index.json` with the manifest path and SHA-256 hash
3. Ensure the manifest passes all [validation rules](#validation-rules)

### Manifest requirements

Every manifest must include at minimum:

```json
{
  "schema_version": 1,
  "name": "my-package",
  "version": "1.0.0",
  "description": "Short description of what this server does",
  "mcp_servers": {
    "server-name": {
      "transport": "stdio",
      "command": "npx",
      "args": ["-y", "my-package@1.0.0"]
    }
  }
}
```

Optional fields that are encouraged where applicable:

- **`env_required`** — List env vars the server needs at runtime (used by `mcper doctor`)
- **`setup_commands`** — Interactive post-install commands to help users obtain API tokens
- **`compatibility`** — Platform constraints (`os`, `arch`)

### Generating the SHA-256 hash

The `sha256` field in `index.json` must match the manifest file. Generate it with:

```bash
sha256sum packages/<name>/<version>/manifest.json
```

### Validation rules

mcper validates every manifest on load. Your manifest will be rejected if:

- `name` or `version` is empty
- `mcp_servers` is empty
- A server name is empty
- A `stdio` server is missing `command`
- An `http` server is missing `url`
- Transport is not `stdio` or `http`
- A `setup_commands` entry has an empty `run`
- A `setup_commands` entry has a `pattern` that doesn't compile as valid regex

### Contribution rules

**General:**

- One package per pull request — keep changes focused and reviewable
- Pin versions explicitly in command args (e.g., `my-package@1.0.0`, not just `my-package`)
- Use the package's official/canonical name
- Write a clear, concise `description`

**Security:**

- For `stdio` servers, use well-known package runners (`npx`, `uvx`, `go run`, etc.) with pinned versions from trusted registries (npm, PyPI, crates.io, etc.)
- Avoid external scripts like `curl https://example.com | sh` in commands or setup — these are opaque, unversioned, and will likely be rejected unless from a highly trusted source (e.g., an official vendor CLI installer)
- `setup_commands` must use argv arrays (no shell execution). The `run` field takes `["binary", "arg1", "arg2"]`, not a shell string
- Do not hardcode secrets, tokens, or credentials in manifests. Use `env_required` and `setup_commands` to handle secrets properly through mcper's keyring integration
- If your package requires authentication, provide a `setup_commands` entry so users can obtain tokens interactively

**Quality:**

- Test your manifest locally before submitting: `mcper install-url ./packages/<name>/<version>/manifest.json`
- Verify the server actually starts and responds to MCP requests
- For version updates, add a new version directory — do not overwrite existing versions
- Keep manifests minimal — only include fields that are actually needed
