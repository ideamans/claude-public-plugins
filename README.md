# ideamans/claude-public-plugins

Claude Code plugin marketplace maintained by [Ideamans Inc.](https://ideamans.com).

Each plugin is an [Agent Skills](https://agentskills.io/) bundle, so the underlying `SKILL.md` files also work with `gh skill install`, Cursor, Gemini CLI, and other hosts that speak the open standard.

## Quick install (Claude Code)

```
/plugin marketplace add ideamans/claude-public-plugins
/plugin install gridgram@ideamans-plugins
/plugin install chartjs2img@ideamans-plugins
```

Then verify with:

```
/gg-render --help            # gridgram skill is live when /gg-* autocompletes
/chartjs2img-render --help   # chartjs2img skill is live when /chartjs2img-* autocompletes
```

## Available plugins

### gridgram

Grid-based diagram rendering. Three skills:

- **`/gg-render`** — render a `.gg` file to SVG / PNG / JSON; surface diagnostics
- **`/gg-icons`** — search 6,000+ built-in Tabler icons semantically
- **`/gg-author`** — compose a new diagram from a description, validate, render

Runtime dependency: the `gg` CLI must be on `PATH`. Install from
[ideamans/gridgram releases](https://github.com/ideamans/gridgram/releases)
or (once published) `bun install -g gridgram`.

Source: [ideamans/gridgram](https://github.com/ideamans/gridgram) → `plugins/gridgram/`

### chartjs2img

Server-side Chart.js rendering — turn a Chart.js config JSON into a PNG / JPEG using headless Chromium. Three skills:

- **`/chartjs2img-render`** — render a Chart.js config JSON to an image; echo `X-Chart-Messages` / stderr warnings
- **`/chartjs2img-author`** — compose a new Chart.js config from a description, validate via render-and-iterate
- **`/chartjs2img-install`** — install / update the `chartjs2img` CLI from GitHub Releases, cross-platform

Runtime dependency: the `chartjs2img` CLI must be on `PATH`. Install via
`/chartjs2img-install` or from
[ideamans/chartjs2img releases](https://github.com/ideamans/chartjs2img/releases).
Chromium is auto-downloaded on first render.

Source: [ideamans/chartjs2img](https://github.com/ideamans/chartjs2img) → `plugins/chartjs2img/`

## Structure

```
claude-public-plugins/
├── .claude-plugin/
│   └── marketplace.json   # catalog consumed by /plugin marketplace add
└── README.md
```

The marketplace is intentionally thin: every plugin entry points at
`git-subdir` source in its home repo. That way plugin code lives next to
the library it wraps, versions stay in sync, and this repo is only ever
a catalog.

## Local development (before GitHub push)

The production `marketplace.json` uses `git-subdir` sources pointing at
`ideamans/*` repositories on GitHub. When iterating on plugin content
before pushing, use the local variant instead:

```
.claude-plugin/marketplace.local.json   # relative-path sources
gridgram                                # symlink → ../gridgram/plugins/gridgram
chartjs2img                             # symlink → ../chartjs2img/plugins/chartjs2img
```

Workflow:

```bash
# 1. Point the symlinks at your checkouts.
ln -sfn ~/dev/gridgram/plugins/gridgram ./gridgram
ln -sfn ~/dev/chartjs2img/plugins/chartjs2img ./chartjs2img

# 2. Validate the schema and every listed plugin.
claude plugin validate .                  # checks marketplace.json (GitHub sources)
claude plugin validate ./gridgram         # checks the plugin directly
claude plugin validate ./chartjs2img

# 3. Register the local marketplace in a test Claude Code session.
#    Swap marketplace.json with the local variant just for this test.
cp .claude-plugin/marketplace.local.json /tmp/mp/.claude-plugin/marketplace.json
# …or run Claude Code with CLAUDE_CODE_PLUGIN_CACHE_DIR=$(mktemp -d) and
# /plugin marketplace add /abs/path/to/claude-public-plugins
```

The symlinks are stable across checkouts because this repo's `.gitignore`
excludes `gridgram/` and `chartjs2img/` (they resolve at test time).

## Adding a new plugin

1. Build the plugin in its own repository under a `plugins/<name>/` directory.
   It must contain `.claude-plugin/plugin.json` and at least one
   `skills/<skill-name>/SKILL.md`.
2. Append an entry to `plugins: []` in `.claude-plugin/marketplace.json`:

   ```json
   {
     "name": "my-plugin",
     "source": {
       "source": "git-subdir",
       "url": "https://github.com/ideamans/<repo>.git",
       "path": "plugins/<name>"
     },
     "description": "…"
   }
   ```

3. Open a PR. When it merges, users get the new plugin the next time they
   run `/plugin marketplace update`.

Run `claude plugin validate .` before pushing to catch JSON / schema issues
early.

## License

MIT — see [LICENSE](LICENSE).
