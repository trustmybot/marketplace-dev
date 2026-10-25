# TMB plugin marketplace — DEV

Catalog for the **dev channel** of the [TMB plugin](https://github.com/trustmybot/plugin). It tracks `trustmybot/plugin@dev-build`: a copy of the `dev` branch that CI stamps with a version on every push. Use it to run what will ship in the next rc.

| Channel | Catalog | Plugin source | Version |
|---|---|---|---|
| **dev** (this one) | `trustmybot-dev` | `trustmybot/plugin@dev-build` | `X.Y.Z-dev.<n>.gh<PR>` |
| rc | [`trustmybot-rc`](https://github.com/trustmybot/marketplace-rc) | a `vX.Y.Z-rc.N` tag | `X.Y.Z-rc.N` |
| stable | [`trustmybot`](https://github.com/trustmybot/marketplace) | `trustmybot/plugin@main` | `X.Y.Z` |

Every channel installs the plugin as `tmb`, so one `tmb` is enabled per project at a time.

## How a dev build is made

There is no GitHub release for the dev channel. The plugin's [`dev-build` workflow](https://github.com/trustmybot/plugin/blob/dev/.github/workflows/dev-build.yml) runs on every push to `dev`:

1. Resolves the next release version from the lowest open `vX.Y.Z` milestone above the last stable tag.
2. Stamps the commit as `X.Y.Z-dev.<n>.gh<PR>` (`n` counts dev builds since the last stable tag; `gh<PR>` is the PR that landed).
3. Force-pushes the stamped commit to `refs/heads/dev-build` and comments the version on the PR.

The version is in `.claude-plugin/plugin.json` on `dev-build` and in the installed plugin's cache directory name. Check what the channel currently serves:

```bash
gh api repos/trustmybot/plugin/contents/.claude-plugin/plugin.json?ref=dev-build --jq .content | base64 -d | jq -r .version
```

## Install

Pick a scope. TMB keeps its trajectory DB and hooks per project, so install it **into the project**, never at user scope:

| Scope | Written to | Use when |
|---|---|---|
| `local` | `.claude/settings.local.json` (git-ignored) | Your own choice of channel; teammates may be on another one. The default for dev. |
| `project` | `.claude/settings.json` (committed) | The whole team pins the same channel. |

Run from the project root.

```bash
claude plugin marketplace add trustmybot/marketplace-dev --scope local
claude plugin install tmb@trustmybot-dev --scope local
```

Replace both `--scope local` with `--scope project` for a team install.

Then fully quit and relaunch Claude Code, and run `/tmb:onboard` in the project.

### Switching from another channel

Disable the `tmb` you have before installing this one, at the scope it was installed in. `disable` keeps it so you can `enable` it again later; `uninstall` removes it.

```bash
claude plugin disable tmb@trustmybot --scope local
```

Then install this channel with the commands above, and fully quit and relaunch Claude Code once.

## Upgrade to the newest dev build

Claude Code caches a clone of `dev-build`. New builds reach your install only after a refresh, run from the project root:

```bash
claude plugin marketplace update trustmybot-dev
claude plugin update tmb@trustmybot-dev --scope local
```

Then fully quit and relaunch Claude Code. Every merged PR on `trustmybot/plugin` carries a comment with these two commands and the build it produced.

## Back to the stable channel

Run each at the scope it was installed in.

```bash
claude plugin uninstall tmb@trustmybot-dev --scope local
claude plugin enable tmb@trustmybot --scope local
```

Fully quit and relaunch Claude Code afterwards.

## Test your own plugin branch

The catalog is a template: one line points it at any pushed branch.

1. Push your branch to `trustmybot/plugin` or a fork.
2. Clone this repo and edit `.claude-plugin/marketplace.json`: set the top-level `name` to something distinctive (so it does not collide with the published `trustmybot-dev`) and `plugins[0].source.ref` to your branch (`source.repo` to your fork). Do not push the edit; Claude Code reads the catalog from disk.
3. Add the catalog by path and install from it:

```bash
claude plugin marketplace add <path-to>/marketplace-dev --scope local
claude plugin install tmb@<your-catalog-name> --scope local
```

Then fully quit and relaunch Claude Code.

New commits on your branch arrive with `claude plugin marketplace update <your-catalog-name>` followed by `claude plugin update tmb@<your-catalog-name> --scope local`, then fully quit and relaunch Claude Code.

## Source

Plugin code lives at [trustmybot/plugin](https://github.com/trustmybot/plugin). This repo holds only the catalog, `.claude-plugin/marketplace.json`.
