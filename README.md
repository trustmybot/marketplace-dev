# TMB plugin marketplace — DEV

Catalog for the **dev channel** of the [TMB plugin](https://github.com/trustmybot/plugin). Tracks the `dev` branch of `trustmybot/plugin` — what's about to ship in the next rc. Use this when you want to try the latest doctrine + features before they reach the rc or stable channels.

## Install

```
/plugin marketplace add trustmybot/marketplace-dev
/plugin install tmb@trustmybot-dev
```

The dev channel installs as `tmb` (same name as the released plugin), so only one can be enabled at a time.

### Disable any existing `tmb` install first

```
/plugin disable tmb
```

Use `/plugin disable` rather than `/plugin uninstall` so you can flip back to the released channel with `/plugin enable tmb` when done.

### When CC asks for scope, pick *local*

`/plugin marketplace add` will prompt:

```
  Install for all collaborators on this repository (project scope)
> Install for you, in this repo only (local scope)
```

**Pick local scope.** The dev channel is a per-developer choice; teammates may each be on a different channel (stable / rc / dev). Project scope is right only when a team wants everyone pinned to the same channel.

## Pull the latest dev tip

The marketplace ref is `dev`, but CC caches a clone — newer commits on `dev` don't reach your install until you refresh:

```
/plugin marketplace update trustmybot-dev
/plugin update tmb@trustmybot-dev
```

`/reload-plugins` only re-reads what CC already has on disk; you need `marketplace update` + `update` to fetch new commits from the remote `dev` branch.

## Test *your own* plugin branch via this catalog

The catalog format is reusable — any branch (yours or someone else's) becomes installable by editing one line. Workflow:

1. Push your plugin branch to `trustmybot/plugin` (or your fork).
2. Clone this catalog locally:
   ```bash
   git clone https://github.com/trustmybot/marketplace-dev.git
   ```
3. Edit `.claude-plugin/marketplace.json` — change `ref` to your branch (or change `repo` to point at your fork):
   ```json
   "source": {
     "source": "github",
     "repo": "<your-owner>/plugin",
     "ref": "<your-branch>"
   }
   ```
   No need to push this edit. The local copy is enough — CC reads the catalog from your filesystem.
4. Add the marketplace from the local clone path:
   ```
   /plugin marketplace add <path-to>/marketplace-dev
   /plugin install tmb@trustmybot-dev
   ```

CC reads the catalog from your local disk; the plugin source still resolves via github (so your branch needs to be pushed somewhere CC can clone). When you update your branch and want CC to pull: `/plugin marketplace update trustmybot-dev` + `/plugin update tmb@trustmybot-dev`.

This pattern is how the dev channel itself was built — `trustmybot/marketplace-dev` is just a `trustmybot/plugin@dev` instance of the same template.

## Test *uncommitted* edits (not pushed anywhere)

Marketplace plugins must be pulled from a remote (github / url). For edits that aren't pushed anywhere yet, skip the marketplace entirely and point CC at the working tree directly:

```bash
claude --plugin-dir <path-to>/TMB/plugin --debug api,hooks --debug-file .claude/logs/tmb.log
```

`--plugin-dir` forces `clearPluginCache`; every boot re-resolves from disk. The `--debug api,hooks --debug-file` capture is optional but useful for L5/L6 runs and MCP-server crash repros.

## Flip back to the released channel when done

```
/plugin uninstall tmb@trustmybot-dev
/plugin enable tmb
```

## DB-path overlap

`plugin.json` declares `"name": "tmb"`, so the trajectory DB resolves to `.claude/tmb/trajectory.db` for both the released and the dev install — whichever is enabled at the moment writes to the same path per project. Two ways to keep state separated when you want it:

1. **Test in a scratch project dir** — the DB is per-project; a fresh repo gives the dev install its own DB.
2. **Override per-session**:
   ```bash
   TRAJECTORY_DB_PATH=<path-to>/scratch/tmb-dev.db claude
   ```

## Channel routing

- This catalog: **DEV channel** (`trustmybot/plugin@dev` — bleeding edge)
- [trustmybot/marketplace-rc](https://github.com/trustmybot/marketplace-rc) — pre-release builds (rc tags)
- [trustmybot/marketplace](https://github.com/trustmybot/marketplace) — production (stable tags from main)

## Source

Plugin code lives at [trustmybot/plugin](https://github.com/trustmybot/plugin). This repo contains only the marketplace catalog (`.claude-plugin/marketplace.json`).
