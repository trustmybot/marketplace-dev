# TMB plugin marketplace — DEV

Catalog for the **dev channel** of the [TMB plugin](https://github.com/trustmybot/plugin). Tracks the `dev` branch of `trustmybot/plugin` — what's about to ship in the next rc. Use this when you want to try the latest doctrine + features before they reach the rc or stable channels.

## Install

```
/plugin marketplace add trustmybot/marketplace-local
/plugin install tmb@trustmybot-local
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

The marketplace ref is `dev`, but CC caches a clone — newer commits on `dev` don't reach your install until you refresh the catalog:

```
/plugin marketplace update trustmybot-local
/plugin update tmb@trustmybot-local
```

Or simpler — `/reload-plugins` only re-reads what CC already has on disk; you need the `update` calls to fetch new commits from the remote `dev` branch.

## Test your own uncommitted edits (not the dev tip)

`marketplace-local` tracks the published `dev` branch only — your in-flight local changes in `TMB/plugin/` aren't reflected until you push them. To test uncommitted edits, skip the marketplace and point CC at the working tree directly:

```bash
claude --plugin-dir <path-to>/TMB/plugin --debug api,hooks --debug-file .claude/logs/tmb.log
```

`--plugin-dir` forces `clearPluginCache`; every boot re-resolves from disk. The `--debug api,hooks --debug-file` capture is optional but useful for L5/L6 runs and MCP-server crash repros.

## Flip back to the released channel when done

```
/plugin uninstall tmb@trustmybot-local
/plugin enable tmb
```

## DB-path overlap

`plugin.json` declares `"name": "tmb"`, so the trajectory DB resolves to `.claude/tmb/trajectory.db` for both the released and the dev install — whichever is enabled at the moment writes to the same path per project. Two clean ways to keep state separated when you want it:

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
