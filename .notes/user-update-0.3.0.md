# User update instructions for murmur.nvim 0.3.0

## Inside Neovim (one command per package manager)

| Manager | Command |
| --- | --- |
| lazy.nvim | `:Lazy update murmur` |
| packer.nvim | `:PackerUpdate` (or `:PackerUpdate murmur` if scoped) |
| vim-plug | `:PlugUpdate murmur` |
| minpac | `:call minpac#update('{}', {'packages': ['murmur']})` |
| paq-nvim | `:PaqUpdate` |

## Inside a terminal (no Neovim needed)

```sh
cd ~/.local/share/nvim/lazy/murmur.nvim       # lazy.nvim
cd ~/.local/share/nvim/site/pack/murmur/start/murmur.nvim   # vim-plug
git fetch --tags
git checkout v0.3.0
```

For lazy.nvim this is unnecessary; it handles the tag fetch and checkout itself.

## One-shot sh snippet for support threads

```sh
nvim --headless +'Lazy update murmur' +qa 2>&1 | tail -5
```

## Verify you are on 0.3.0

Inside Neovim:

```vim
:lua print(require("murmur").config and require("murmur").version or "no version field")
:edit /path/to/any/source-file.lua
:lua print(vim.inspect(vim.fn.getbufvar('%', 'murmur_murmurs')))
```

Or with `git`:

```sh
git -C ~/.local/share/nvim/lazy/murmur.nvim describe --tags --exact-match
# expect: v0.3.0
```

## If the update fails

1. Run the manager's install command explicitly: `:Lazy install murmur` (or equivalent).
2. Force-clean: `:Lazy clean murmur` then `:Lazy install murmur`.
3. If you self-symlinked the plugin, re-point the symlink to the v0.3.0 tag:
   ```sh
   cd ~/.local/share/nvim/lazy/murmur.nvim
   git fetch --tags
   git reset --hard v0.3.0
   ```
4. If you use the upstream `piqusy/murmur` integrations (OMP, OpenCode, Claude Code, Codex, Antigravity), update the integration symlinks in `~/.omp/agent/extensions/murmur/` to point at v0.3.0 of the upstream repo.

## Auto-update after this release

A future GitHub Action will mirror every `v*` tag on `piqusy/murmur` to `piqusy/murmur.nvim`. Once that lands, the only command you will ever need is `:Lazy update murmur`.
