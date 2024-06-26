# `Fzf`

<!-- plugins:start -->

:::info
You can enable the extra with the `:LazyExtras` command.
Plugins marked as optional will only be configured if they are installed.
:::

<details>
<summary>Alternatively, you can add it to your <code>lazy.nvim</code> imports</summary>

```lua title="lua/config/lazy.lua" {4}
require("lazy").setup({
  spec = {
    { "LazyVim/LazyVim", import = "lazyvim.plugins" },
    { import = "lazyvim.plugins.extras.editor.fzf" },
    { import = "plugins" },
  },
})
```

</details>

Below you can find a list of included plugins and their default settings.

:::caution
You don't need to copy the default settings to your config.
They are only shown here for reference.
:::

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## [fzf-lua](https://github.com/ibhagwan/fzf-lua)

<Tabs>

<TabItem value="opts" label="Options">

```lua
opts = function(_, opts)
  local config = require("fzf-lua.config")
  local actions = require("fzf-lua.actions")

  -- Quickfix
  config.defaults.keymap.fzf["ctrl-q"] = "select-all+accept"
  config.defaults.keymap.builtin["<c-f>"] = "preview-page-down"
  config.defaults.keymap.builtin["<c-b>"] = "preview-page-up"

  -- Trouble
  config.defaults.actions.files["ctrl-t"] = require("trouble.sources.fzf").actions.open

  -- Toggle root dir / cwd
  config.defaults.actions.files["ctrl-r"] = function(_, ctx)
    local o = vim.deepcopy(ctx.__call_opts)
    o.root = o.root == false
    o.cwd = nil
    o.buf = ctx.__CTX.bufnr
    LazyVim.pick.open(ctx.__INFO.cmd, o)
  end
  config.defaults.actions.files["alt-c"] = config.defaults.actions.files["ctrl-r"]

  -- use the same prompt for all
  local defaults = require("fzf-lua.profiles.default-title")
  local function fix(t)
    t.prompt = t.prompt ~= nil and " " or nil
    for _, v in pairs(t) do
      if type(v) == "table" then
        fix(v)
      end
    end
  end
  fix(defaults)

  vim.api.nvim_set_hl(0, "FzfLuaPath", { link = "Directory", default = true })

  return vim.tbl_deep_extend("force", opts, defaults, {
    fzf_colors = true,
    files = {
      cwd_prompt = false,
      actions = {
        ["alt-i"] = { actions.toggle_ignore },
        ["alt-h"] = { actions.toggle_hidden },
      },
    },
    grep = {
      formatter = "path.hl",
      actions = {
        ["alt-i"] = { actions.toggle_ignore },
        ["alt-h"] = { actions.toggle_hidden },
      },
    },
    formatters = {
      path = {
        hl = {
          _to = function()
            local _, escseq = require("fzf-lua.utils").ansi_from_hl("FzfLuaPath", "foo")
            return [[
                return function(s, _, m)
                  return "]] .. (escseq or "") .. [["
                    .. s .. m.utils.ansi_escseq.clear
                end
              ]]
          end,
        },
      },
    },
  })
end
```

</TabItem>


<TabItem value="code" label="Full Spec">

```lua
{
  "ibhagwan/fzf-lua",
  event = "VeryLazy",
  opts = function(_, opts)
    local config = require("fzf-lua.config")
    local actions = require("fzf-lua.actions")

    -- Quickfix
    config.defaults.keymap.fzf["ctrl-q"] = "select-all+accept"
    config.defaults.keymap.builtin["<c-f>"] = "preview-page-down"
    config.defaults.keymap.builtin["<c-b>"] = "preview-page-up"

    -- Trouble
    config.defaults.actions.files["ctrl-t"] = require("trouble.sources.fzf").actions.open

    -- Toggle root dir / cwd
    config.defaults.actions.files["ctrl-r"] = function(_, ctx)
      local o = vim.deepcopy(ctx.__call_opts)
      o.root = o.root == false
      o.cwd = nil
      o.buf = ctx.__CTX.bufnr
      LazyVim.pick.open(ctx.__INFO.cmd, o)
    end
    config.defaults.actions.files["alt-c"] = config.defaults.actions.files["ctrl-r"]

    -- use the same prompt for all
    local defaults = require("fzf-lua.profiles.default-title")
    local function fix(t)
      t.prompt = t.prompt ~= nil and " " or nil
      for _, v in pairs(t) do
        if type(v) == "table" then
          fix(v)
        end
      end
    end
    fix(defaults)

    vim.api.nvim_set_hl(0, "FzfLuaPath", { link = "Directory", default = true })

    return vim.tbl_deep_extend("force", opts, defaults, {
      fzf_colors = true,
      files = {
        cwd_prompt = false,
        actions = {
          ["alt-i"] = { actions.toggle_ignore },
          ["alt-h"] = { actions.toggle_hidden },
        },
      },
      grep = {
        formatter = "path.hl",
        actions = {
          ["alt-i"] = { actions.toggle_ignore },
          ["alt-h"] = { actions.toggle_hidden },
        },
      },
      formatters = {
        path = {
          hl = {
            _to = function()
              local _, escseq = require("fzf-lua.utils").ansi_from_hl("FzfLuaPath", "foo")
              return [[
                  return function(s, _, m)
                    return "]] .. (escseq or "") .. [["
                      .. s .. m.utils.ansi_escseq.clear
                  end
                ]]
            end,
          },
        },
      },
    })
  end,
  keys = {
    { "<esc>", "<cmd>close<cr>", ft = "fzf", mode = "t", nowait = true },
    { "<c-j>", "<Down>", ft = "fzf", mode = "t", nowait = true },
    { "<c-k>", "<Up>", ft = "fzf", mode = "t", nowait = true },
    {
      "<leader>,",
      "<cmd>FzfLua buffers sort_mru=true sort_lastused=true<cr>",
      desc = "Switch Buffer",
    },
    { "<leader>/", LazyVim.pick("live_grep"), desc = "Grep (Root Dir)" },
    { "<leader>:", "<cmd>FzfLua command_history<cr>", desc = "Command History" },
    { "<leader><space>", LazyVim.pick("auto"), desc = "Find Files (Root Dir)" },
    -- find
    { "<leader>fb", "<cmd>FzfLua buffers sort_mru=true sort_lastused=true<cr>", desc = "Buffers" },
    { "<leader>fc", LazyVim.pick.config_files(), desc = "Find Config File" },
    { "<leader>ff", LazyVim.pick("auto"), desc = "Find Files (Root Dir)" },
    { "<leader>fF", LazyVim.pick("auto", { root = false }), desc = "Find Files (cwd)" },
    { "<leader>fg", "<cmd>FzfLua git_files<cr>", desc = "Find Files (git-files)" },
    { "<leader>fr", "<cmd>FzfLua oldfiles<cr>", desc = "Recent" },
    { "<leader>fR", LazyVim.pick("oldfiles", { root = false }), desc = "Recent (cwd)" },
    -- git
    { "<leader>gc", "<cmd>FzfLua git_commits<CR>", desc = "Commits" },
    { "<leader>gs", "<cmd>FzfLua git_status<CR>", desc = "Status" },
    -- search
    { '<leader>s"', "<cmd>FzfLua registers<cr>", desc = "Registers" },
    { "<leader>sa", "<cmd>FzfLua autocmds<cr>", desc = "Auto Commands" },
    { "<leader>sb", "<cmd>FzfLua grep_curbuf<cr>", desc = "Buffer" },
    { "<leader>sc", "<cmd>FzfLua command_history<cr>", desc = "Command History" },
    { "<leader>sC", "<cmd>FzfLua commands<cr>", desc = "Commands" },
    { "<leader>sd", "<cmd>FzfLua diagnostics_document<cr>", desc = "Document Diagnostics" },
    { "<leader>sD", "<cmd>FzfLua diagnostics_workspace<cr>", desc = "Workspace Diagnostics" },
    { "<leader>sg", LazyVim.pick("live_grep"), desc = "Grep (Root Dir)" },
    { "<leader>sG", LazyVim.pick("live_grep", { root = false }), desc = "Grep (cwd)" },
    { "<leader>sh", "<cmd>FzfLua help_tags<cr>", desc = "Help Pages" },
    { "<leader>sH", "<cmd>FzfLua highlights<cr>", desc = "Search Highlight Groups" },
    { "<leader>sj", "<cmd>FzfLua jumps<cr>", desc = "Jumplist" },
    { "<leader>sk", "<cmd>FzfLua keymaps<cr>", desc = "Key Maps" },
    { "<leader>sl", "<cmd>FzfLua loclist<cr>", desc = "Location List" },
    { "<leader>sM", "<cmd>FzfLua man_pages<cr>", desc = "Man Pages" },
    { "<leader>sm", "<cmd>FzfLua marks<cr>", desc = "Jump to Mark" },
    { "<leader>sR", "<cmd>FzfLua resume<cr>", desc = "Resume" },
    { "<leader>sq", "<cmd>FzfLua quickfix<cr>", desc = "Quickfix List" },
    { "<leader>sw", LazyVim.pick("grep_cword"), desc = "Word (Root Dir)" },
    { "<leader>sW", LazyVim.pick("grep_cword", { root = false }), desc = "Word (cwd)" },
    { "<leader>sw", LazyVim.pick("grep_visual"), mode = "v", desc = "Selection (Root Dir)" },
    { "<leader>sW", LazyVim.pick("grep_visual", { root = false }), mode = "v", desc = "Selection (cwd)" },
    { "<leader>uC", LazyVim.pick("colorschemes"), desc = "Colorscheme with Preview" },
    {
      "<leader>ss",
      function()
        require("fzf-lua").lsp_document_symbols({
          regex_filter = symbols_filter,
        })
      end,
      desc = "Goto Symbol",
    },
    {
      "<leader>sS",
      function()
        require("fzf-lua").lsp_dynamic_workspace_symbols({
          regex_filter = symbols_filter,
        })
      end,
      desc = "Goto Symbol (Workspace)",
    },
  },
}
```

</TabItem>

</Tabs>

## [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)

<Tabs>

<TabItem value="opts" label="Options">

```lua
opts = function()
  local Keys = require("lazyvim.plugins.lsp.keymaps").get()
  vim.list_extend(Keys, {
    {
      "gd",
      "<cmd>FzfLua lsp_definitions     jump_to_single_result=true<cr>",
      desc = "Goto Definition",
      has = "definition",
    },
    { "gr", "<cmd>FzfLua lsp_references      jump_to_single_result=true<cr>", desc = "References", nowait = true },
    { "gI", "<cmd>FzfLua lsp_implementations jump_to_single_result=true<cr>", desc = "Goto Implementation" },
    { "gy", "<cmd>FzfLua lsp_typedefs        jump_to_single_result=true<cr>", desc = "Goto T[y]pe Definition" },
  })
end
```

</TabItem>


<TabItem value="code" label="Full Spec">

```lua
{
  "neovim/nvim-lspconfig",
  opts = function()
    local Keys = require("lazyvim.plugins.lsp.keymaps").get()
    vim.list_extend(Keys, {
      {
        "gd",
        "<cmd>FzfLua lsp_definitions     jump_to_single_result=true<cr>",
        desc = "Goto Definition",
        has = "definition",
      },
      { "gr", "<cmd>FzfLua lsp_references      jump_to_single_result=true<cr>", desc = "References", nowait = true },
      { "gI", "<cmd>FzfLua lsp_implementations jump_to_single_result=true<cr>", desc = "Goto Implementation" },
      { "gy", "<cmd>FzfLua lsp_typedefs        jump_to_single_result=true<cr>", desc = "Goto T[y]pe Definition" },
    })
  end,
}
```

</TabItem>

</Tabs>

## [todo-comments.nvim](https://github.com/folke/todo-comments.nvim) _(optional)_

<Tabs>

<TabItem value="opts" label="Options">

```lua
opts = nil
```

</TabItem>


<TabItem value="code" label="Full Spec">

```lua
{
  "folke/todo-comments.nvim",
  optional = true,
  -- stylua: ignore
  keys = {
    { "<leader>st", function() require("todo-comments.fzf").todo() end, desc = "Todo" },
    { "<leader>sT", function () require("todo-comments.fzf").todo({ keywords = { "TODO", "FIX", "FIXME" } }) end, desc = "Todo/Fix/Fixme" },
  },
}
```

</TabItem>

</Tabs>

<!-- plugins:end -->
