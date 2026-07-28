# murmur.nvim

Inline line annotations for Neovim — leave instructions for your AI agent (or future-you) directly on source lines. Murmurs render as boxed virtual text below the anchored line, or as compact end-of-line shadow text, with a persistent sign-column indicator so you always know where annotations live.

This package is the Neovim-only mirror of the [piqusy/murmur](https://github.com/piqusy/murmur) project. The Lua plugin lives under `lua/murmur/`. Agent integration code (OMP, OpenCode, Claude Code, Codex, Antigravity) lives in the upstream `piqusy/murmur` repository and is installed separately.

## Features

- **Box mode** — closed `╭─│─╰` frame with author, message, and source line number
- **Inline mode** — compact EOL shadow text (`Author: message`)
- **Always-on sign indicator** — `◉` in the sign column whenever a murmur exists
- **Content wrapping** — long messages wrap to fit the window (box mode)
- **Line-drift tracking** — extmarks follow text edits; a content anchor re-locates murmurs after external edits
- **Sidecar storage** — `<file>.murmur.json` alongside each file (gitignored globally)
- **Pluggable picker** — snacks / telescope / fzf-lua / builtin `vim.ui.select`
- **Zero dependencies** — works on bare Neovim

## Requirements

- **Neovim ≥ 0.10**
- No plugins required. [snacks.nvim](https://github.com/folke/snacks.nvim), [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim), [fzf-lua](https://github.com/ibhagwan/fzf-lua), [dressing.nvim](https://github.com/stevearc/dressing.nvim), or [noice.nvim](https://github.com/folke/Lightning-Dev/noice.nvim) enhance the UI but are optional.

## Installation

### [lazy.nvim](https://github.com/folke/lazy.nvim)

```lua
{
  "piqusy/murmur.nvim",
  event = "VeryLazy",
  config = function()
    require("murmur").setup()
  end,
  keys = {
    { "<leader>ma", "<cmd>MurmurAdd<cr>",    desc = "Murmur Add" },
    { "<leader>md", "<cmd>MurmurDelete<cr>", desc = "Murmur Delete" },
    { "<leader>me", "<cmd>MurmurEdit<cr>",   desc = "Murmur Edit" },
    { "<leader>ml", "<cmd>MurmurList<cr>",   desc = "Murmur List" },
    { "<leader>mL", "<cmd>MurmurListAll<cr>", desc = "Murmur List All" },
    { "<leader>mt", "<cmd>MurmurToggle<cr>", desc = "Murmur Toggle" },
    { "<leader>mm", "<cmd>MurmurMode<cr>",   desc = "Murmur Mode" },
    { "<leader>mD", "<cmd>MurmurDeleteFile<cr>", desc = "Murmur Delete File" },
    { "<leader>mA", "<cmd>MurmurDeleteAll<cr>",  desc = "Murmur Delete All" },
  },
}
```

### AI integrations

The Neovim mirror is a plugin only — the harness integrations (OMP, OpenCode, Claude Code, Codex, Antigravity) ship from the upstream [piqusy/murmur](https://github.com/piqusy/murmur) repository under `integrations/`. Install whichever harness you use; the Neovim sidecar JSON contract is identical.

## Commands

| Command | Description |
|---|---|
| `:MurmurAdd` | Add a murmur on the current line |
| `:MurmurDelete` | Select and delete a single murmur |
| `:MurmurDeleteFile` | Delete all murmurs in the current file |
| `:MurmurDeleteAll` | Delete all murmurs across every open buffer (with confirm) |
| `:MurmurEdit` | Select and edit a murmey's message |
| `:MurmurList` | List and jump to a murmur (current buffer) |
| `:MurmurListAll` | List and jump to any murmur in the project (all sidecar files) |
| `:MurmurToggle` | Toggle content visibility (sign stays) |
| `:MurmurMode` | Toggle box ↔ inline render mode |
| `:MurmurClear` | Clear all murmur extmarks in the buffer (visual only) |

## Configuration

```lua
require("murmur").setup({
  render_mode = "box",        -- "box" | "inline"
  sign_text = "◉",            -- sign-column glyph
  sidecar_suffix = ".murmur.json",
  picker = "auto",            -- "auto" | "snacks" | "telescope" | "fzf" | "builtin"
  highlights = {
    user_header  = { fg = "#4dbd9f", italic = true }, -- teal
    user_sign    = { fg = "#4dbd9f" },
    agent_header = { fg = "#d3869b", italic = true }, -- purple
    agent_sign   = { fg = "#d3869b" },
    body   = { fg = "#ebdbb2" },
    border = { fg = "#928374" },
    orphan = { fg = "#fe8019", bold = true },
    foreign = { fg = "#928374", italic = true }, -- gray, dimmed (diff-view foreign revision)
  },
})
```

The render mode persists across restarts (`stdpath('data')/murmur.json`).

User and agent murmurs are visually distinct: user = teal, agent = purple
(both sign glyph and header). Author is determined by the `author` field -
`"User"` gets user styling, anything else gets agent styling.

## Diff view support

Murmur resolves diff buffers from both fugitive (`:Gdiff`, `:Gvdiffsplit`) and
gitsigns (`:Gitsigns diffthis`) to their real source file, so murmurs from the
working-tree sidecar appear on the staged/HEAD side too. Line differences are
handled by the existing anchor-based relocation — the anchor text is searched
in the target buffer and the murmur relocates automatically.

Foreign-revision buffers (staged `//0`, `HEAD`, specific commits) are
**read-only**: you can see murmurs but not add, edit, or delete them. They
render with a dimmed gray style and a `⊞ staged` / `⊞ HEAD` badge in the box
header so you always know which side you're looking at.

To pin or modify murmurs, switch to the worktree buffer (the real file).

## How it works

Murmurs are stored in a sidecar file `<original-file>.murmur.json` next to each annotated file. Add `*.murmur.json` to your global gitignore so they never get committed:

```gitignore
# ~/.gitignore_global
*.murmur.json
```

When a file is opened, the sidecar loads and extmarks are placed at each murmey's line. Extmarks move with text edits automatically. A content anchor (the line's text) lets murmurs re-locate if the file changed externally (e.g. a git pull). If the anchor can't be found within ±20 lines, the murmur is marked orphaned (⚠) for manual review. In diff views (fugitive `:Gdiff`), the buffer's pseudo-path is resolved to the real source file so the same sidecar loads on both sides; the non-worktree side is read-only.

## Development

Tests use [plenary.nvim](https://github.com/nvim-lua/plenary.nvim):

```bash
git clone --depth 1 https://github.com/nvim-lua/plenary.nvim .deps/plenary.nvim
for spec in spec/*_spec.lua; do
  nvim --headless --noplugin -u NORC \
    -c "set rtp+=$(pwd)" \
    -c "set rtp+=$(pwd)/.deps/plenary.nvim" \
    -c "runtime! plugin/plenary.vim" \
    -c "PlenaryBustedFile $spec" \
    -c "qa"
done
```

## License

MIT
