# Release announcement

## Where to publish

| Channel | What | Why |
| --- | --- | --- |
| `r/neovim` on Reddit | short text post with bullet highlights | highest discoverability for new plugin users |
| `neovim` Discourse (`community.neovim.io`) | one announcement topic | the canonical Neovim forum; long-form works well |
| Neovim Matrix room `#neovim:matrix.org` | short note with link | real-time traffic from plugin authors and power users |
| `@neovim` on Fediverse (Mastodon) | mirror of the Reddit post | federated audience; tag `neovim` |
| GitHub Discussions on `piqusy/murmur.nvim` | the canonical "Release v0.3.0" discussion | links from the GitHub release page |
| Twitter/X | one tweet with tag `@neovim` | short reach, but the Neovim account amplifies |
| `#lazy.nvim` topic and Neovim subreddit wiki page (add your plugin) | secondary indexing | sticky references help newcomers |

## What to write

A short announcement (3-5 short paragraphs) plus the changelog. The README + CHANGELOG already exist, so the post can be terse.

## Draft

> **Murmur.nvim 0.3.0 released** — batch read, project scan, and edit preflight for AI harness sidecars
>
> Murmur now exposes `read_murmurs` and `scan_murmurs` to OMP and OpenCode, returns per-file status (`annotated` | `clear` | `invalid_sidecar` | `missing_source`), and the OMP extension auto-injects only the sidecars that changed since session start before every `edit` / `write` / `multiedit` / `read` tool call (multi-file hashline edits included). No more "did I remember to call read_murmur first" — the preflight does it.
>
> The shared read core (`integrations/shared/murmur-core.ts`) deduplicates parsing/scanning across OMP and OpenCode.
>
> Update with your favourite plugin manager — for example:
> ```vim
> :Lazy update murmur
> ```
> or `:PackerUpdate`, `:PlugUpdate murmur`, or `minpac#update` with the new tag.
>
> Full changelog and source: <https://github.com/piqusy/murmur.nvim/releases/tag/v0.3.0>

## Manual actions

1. Open the GitHub release on `piqusy/murmur.nvim` and publish a release body built from `CHANGELOG.md` via `.github/scripts/build-release-body.sh` — or copy the upstream `murmur` repo's body and adapt.
2. Post the draft above to r/neovim, neovim Discourse, and Matrix.
3. Update the Neovim subreddit wiki's "List of plugins" entry if it currently exists.

## How to keep this automated

Add a GitHub Action in the `murmur` repo that runs on `push: tags: ["v*"]` to:

- mirror the `lua/` subtree into `piqusy/murmur.nvim`,
- push the same tag there with a generated commit message, and
- post the rendered changelog into a GitHub Discussion (via API).

Then every `v*` tag on `murmur` self-propagates to the Neovim mirror.
