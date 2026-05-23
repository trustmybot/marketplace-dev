# TMB plugin marketplace — LOCAL DEV

Catalog for the local-dev channel of the [TMB plugin](https://github.com/trustmybot/plugin). Points at the in-tree `plugin/` checkout — for contributors who want to test local edits against a real Claude Code session before promoting to rc.

## 1. Set up the TMB workspace

The workspace is a parent directory holding the plugin source, the marketplace catalogs, and any dev artifacts (logs, scratch DBs, notes). Clone the plugin into it; the marketplaces are siblings.

```bash
mkdir -p TMB && cd TMB
git clone https://github.com/trustmybot/plugin.git
git clone https://github.com/trustmybot/marketplace-local.git
# optional siblings:
# git clone https://github.com/trustmybot/marketplace.git
# git clone https://github.com/trustmybot/marketplace-rc.git
```

Final layout:

```
TMB/
├── plugin/                  ← the plugin source you edit + test
└── marketplace-local/       ← this catalog
    └── .claude-plugin/
        └── marketplace.json ← source: ../../plugin  (resolves to TMB/plugin)
```

## 2. Switch any existing `tmb` install out of the way

The local plugin installs as `tmb` (same name as the released plugin) so the DB path / hook namespace matches production. Only one `tmb` can be enabled at a time. Before installing local:

```
/plugin disable tmb
```

(Use `/plugin disable` rather than `/plugin uninstall` so you can flip back to the released channel with `/plugin enable tmb` when done.)

## 3. Install the local-dev plugin

```
/plugin marketplace add <path-to>/TMB/marketplace-local
/plugin install tmb@trustmybot-local
```

The plugin is now loaded from `TMB/plugin/` on disk.

### When CC asks for scope, pick *local*

`/plugin marketplace add` will prompt:

```
  Install for all collaborators on this repository (project scope)
> Install for you, in this repo only (local scope)
```

**Pick local scope.** The `<path-to>/TMB/marketplace-local` path is developer-specific — committing it to project scope would be wrong for anyone else who clones the repo. Local-dev is always a per-developer choice; teammates may each be on a different channel (stable / rc / local) and that's fine.

(Project scope is the right answer for the released marketplaces — `trustmybot` or `trustmybot-rc` — when a team wants everyone pinned to the same channel.)

## 4. Use it

### Reflect the latest on-disk state

Edit files in `TMB/plugin/`, then in any active CC session:

```
/reload-plugins
```

`/reload-plugins` re-resolves hooks + skills + MCP server registrations from the source tree. No reinstall cycle — your next prompt sees the new state.

### Reflect a specific version (tag, branch, commit)

The `file` source tracks whatever's at the on-disk path, so version pinning happens in git, not in the marketplace:

```bash
cd TMB/plugin
git fetch --tags origin
git checkout v0.7.0-rc.4              # or any branch / commit SHA
# back in CC:
/reload-plugins
```

To return to the latest:

```bash
cd TMB/plugin && git switch dev && git pull --ff-only
# back in CC:
/reload-plugins
```

## 5. Fresh-session boot with debug logs

When you need a clean session (no cached MCP server, no inherited state) and want to capture API + hook traffic to disk, skip the marketplace add and point CC at the checkout directly:

```bash
claude --plugin-dir <path-to>/TMB/plugin --debug api,hooks --debug-file .claude/logs/tmb.log
```

`--plugin-dir` forces `clearPluginCache`; the new session re-resolves everything from disk on boot.

## 6. Done? Flip back to the released channel

```
/plugin uninstall tmb@trustmybot-local
/plugin enable tmb
```

## DB-path overlap

`plugin.json` declares `"name": "tmb"`, so the trajectory DB resolves to `.claude/tmb/trajectory.db` for both the released and the local install — whichever is enabled at the moment writes to the same path per project. Two clean ways to keep state separated when you want it:

1. **Test in a scratch project dir** — the DB is per-project; a fresh repo gives the dev install its own DB.
2. **Override per-session**:
   ```bash
   TRAJECTORY_DB_PATH=<path-to>/scratch/tmb-dev.db claude
   ```

## Channel routing

- This dir: LOCAL DEV channel (in-tree `plugin/` checkout)
- RC: [trustmybot/marketplace-rc](https://github.com/trustmybot/marketplace-rc) — pre-release builds
- Stable: [trustmybot/marketplace](https://github.com/trustmybot/marketplace) — production

## Source

Plugin code lives in `../plugin`. This dir contains only the marketplace catalog (`.claude-plugin/marketplace.json`).
