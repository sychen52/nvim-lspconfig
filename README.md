# nvim-lspconfig

A [collection of common configurations](https://github.com/neovim/nvim-lspconfig/blob/master/CONFIG.md) for Neovim's built-in [language server client](https://neovim.io/doc/user/lsp.html).

This plugin allows for declaratively configuring, launching, and initializing language servers you have installed on your system. Language server configurations are community-maintained.

# LSP overview

Neovim supports the Language Server Protocol (LSP), which means it acts as a client to language servers and includes a Lua framework `vim.lsp` for building enhanced LSP tools. LSP facilitates features like:

- go-to-definition
- find-references
- hover
- completion
- rename
- format
- refactor

Neovim provides an interface for all of these features, and the language server client is designed to be highly extensible to allow plugins to integrate language server features which are not yet present in Neovim core such as [**auto**-completion](https://github.com/neovim/nvim-lspconfig/wiki/Autocompletion) (as opposed to manual completion with omnifunc) and [snippet integration](https://github.com/neovim/nvim-lspconfig/wiki/Snippets).

These features are not implemented in this repo, but in Neovim core. See `:help lsp` for more details.

## Install

* Requires [Neovim v0.5.0](https://github.com/neovim/neovim/releases/tag/v0.5.0) or [Nightly](https://github.com/neovim/neovim/releases/tag/nightly). Update Neovim and nvim-lspconfig before reporting an issue.

* Install nvim-lspconfig like any other Vim plugin, e.g. with [vim-plug](https://github.com/junegunn/vim-plug):

    ```vim
    Plug 'neovim/nvim-lspconfig'
    ```

## Quickstart

1. Install a language server, e.g. [pyright](CONFIG.md#pyright)

    ```bash
    npm i -g pyright
    ```

2. Add the language server setup to your init.vim. The server name must match those found in the table of contents in [CONFIG.md](CONFIG.md)

    ```lua
    lua << EOF
    require'lspconfig'.pyright.setup{}
    EOF
    ```

3. Create a project, this project must contain a file matching the root directory trigger. See [Automatically launching language servers](#Automatically-launching-language-servers) for additional info.

    ```bash
    mkdir test_python_project
    cd test_python_project
    git init
    touch main.py
    ```

4. Launch neovim, the language server will now be attached and providing diagnostics (see `:LspInfo`)

    ```
    nvim main.py
    ```

5. See [Keybindings and completion](#Keybindings-and-completion) for mapping useful functions and enabling omnifunc completion

## Automatically launching language servers

In order to automatically launch a language server, lspconfig searches up the directory tree from your current buffer to find a file matching the `root_dir` pattern defined in each server's configuration.  For [pyright](CONFIG.md#pyright), this is any directory containing ".git", "setup.py",  "setup.cfg", "pyproject.toml", or "requirements.txt").

Language servers require each project to have a `root` in order to provide completion and search across symbols that may not be defined in your current file, and to avoid having to index your entire filesystem on each startup.

## Enabling additional language servers

Enabling most language servers is as easy as installing the language server, ensuring it is on your PATH, and adding the following to your config:

```vim
lua << EOF
require'lspconfig'.rust_analyzer.setup{}
EOF
```

For a full list of servers, see [CONFIG.md](CONFIG.md). This document contains installation instructions and additional, optional customization suggestions for each language server. For some servers that are not on your system path (jdtls, elixirls) you will be required to manually add `cmd` as an entry in the table passed to setup. Most language servers can be installed in less than a minute.

## Keybindings and completion

nvim-lspconfig does not map keybindings or enable completion by default. Manual, triggered completion can be provided by neovim's built-in omnifunc. For autocompletion, a general purpose [autocompletion plugin](https://github.com/neovim/nvim-lspconfig/wiki/Autocompletion) is required. The following example configuration provides suggested keymaps for the most commonly used language server functions, and manually triggered completion with omnifunc (\<c-x\>\<c-o\>).
Note: **you must pass the defined `on_attach` as an argument to every `setup {}` call** and **the keybindings in `on_attach` only take effect after the language server has started (attached to the current buffer)**. 

```lua
lua << EOF
local nvim_lsp = require('lspconfig')

-- Use an on_attach function to only map the following keys
-- after the language server attaches to the current buffer
local on_attach = function(client, bufnr)
  local function buf_set_keymap(...) vim.api.nvim_buf_set_keymap(bufnr, ...) end
  local function buf_set_option(...) vim.api.nvim_buf_set_option(bufnr, ...) end

  -- Enable completion triggered by <c-x><c-o>
  buf_set_option('omnifunc', 'v:lua.vim.lsp.omnifunc')

  -- Mappings.
  local opts = { noremap=true, silent=true }

  -- See `:help vim.lsp.*` for documentation on any of the below functions
  buf_set_keymap('n', 'gD', '<cmd>lua vim.lsp.buf.declaration()<CR>', opts)
  buf_set_keymap('n', 'gd', '<cmd>lua vim.lsp.buf.definition()<CR>', opts)
  buf_set_keymap('n', 'K', '<cmd>lua vim.lsp.buf.hover()<CR>', opts)
  buf_set_keymap('n', 'gi', '<cmd>lua vim.lsp.buf.implementation()<CR>', opts)
  buf_set_keymap('i', '<C-k>', '<cmd>lua vim.lsp.buf.signature_help()<CR>', opts)
  buf_set_keymap('n', '<space>wa', '<cmd>lua vim.lsp.buf.add_workspace_folder()<CR>', opts)
  buf_set_keymap('n', '<space>wr', '<cmd>lua vim.lsp.buf.remove_workspace_folder()<CR>', opts)
  buf_set_keymap('n', '<space>wl', '<cmd>lua print(vim.inspect(vim.lsp.buf.list_workspace_folders()))<CR>', opts)
  buf_set_keymap('n', '<space>D', '<cmd>lua vim.lsp.buf.type_definition()<CR>', opts)
  buf_set_keymap('n', '<space>rn', '<cmd>lua vim.lsp.buf.rename()<CR>', opts)
  buf_set_keymap('n', '<space>ca', '<cmd>lua vim.lsp.buf.code_action()<CR>', opts)
  buf_set_keymap('n', 'gr', '<cmd>lua vim.lsp.buf.references()<CR>', opts)
  buf_set_keymap('n', '<space>e', '<cmd>lua vim.lsp.diagnostic.show_line_diagnostics()<CR>', opts)
  buf_set_keymap('n', '[d', '<cmd>lua vim.lsp.diagnostic.goto_prev()<CR>', opts)
  buf_set_keymap('n', ']d', '<cmd>lua vim.lsp.diagnostic.goto_next()<CR>', opts)
  buf_set_keymap('n', '<space>q', '<cmd>lua vim.lsp.diagnostic.set_loclist()<CR>', opts)
  buf_set_keymap('n', '<space>f', '<cmd>lua vim.lsp.buf.formatting()<CR>', opts)

end

-- Use a loop to conveniently call 'setup' on multiple servers and
-- map buffer local keybindings when the language server attaches
local servers = { 'pyright', 'rust_analyzer', 'tsserver' }
for _, lsp in ipairs(servers) do
  nvim_lsp[lsp].setup {
    on_attach = on_attach,
    flags = {
      debounce_text_changes = 150,
    }
  }
end
EOF
```

The `on_attach` hook is used to only activate the bindings after the language server attaches to the current buffer.

## Debugging

If you have an issue with lspconfig, the first step is to reproduce with a [minimal configuration](https://github.com/neovim/nvim-lspconfig/blob/master/test/minimal_init.lua).

The most common reasons a language server does not start or attach are:

1. The language server is not installed. nvim-lspconfig does not install language servers for you. You should be able to run the `cmd` defined in each server's lua module from the command line and see that the language server starts. If the `cmd` is an executable name, ensure it is on your path.

2. Not triggering root detection. The language server will only start if it is opened in a directory, or child directory, containing a file which signals the *root* of the project. Most of the time, this is a `.git` folder, but each server defines the root config in the lua file. See [CONFIG.md](CONFIG.md) or the source for the list of root directories.

3. Misconfiguration. You must pass `on_attach` and `capabilities` for **each** `setup {}` if you want these to take effect. You must also **not call `setup {}` twice for the same server**. The second call to `setup {}` will overwrite the first.

:LspInfo provides a handy overview of your active and configured language servers.  Note, that it will not report any configuration changes applied in `on_new_config`.

Before reporting a bug, check your logs and the output of `:LspInfo`. Add the following to your init.vim to enable logging:

```lua
lua << EOF
vim.lsp.set_log_level("debug")
EOF
```
Attempt to run the language server, and open the log with:
```
:lua vim.cmd('e'..vim.lsp.get_log_path())
```
Most of the time, the reason for failure is present in the logs.

## Built-in commands

* `:LspInfo` shows the status of active and configured language servers.

The following support tab-completion for all arguments:

* `:LspStart <config_name>` Start the requested server name. Will only successfully start if the command detects a root directory matching the current config. Pass `autostart = false` to your `.setup{}` call for a language server if you would like to launch clients solely with this command. Defaults to all servers matching current buffer filetype.
* `:LspStop <client_id>` Defaults to stopping all buffer clients.
* `:LspRestart <client_id>` Defaults to restarting all buffer clients.

## The wiki

Please see the [wiki](https://github.com/neovim/nvim-lspconfig/wiki) for additional topics, including:

* [Installing language servers automatically](https://github.com/neovim/nvim-lspconfig/wiki/Installing-language-servers-automatically)
* [Snippets support](https://github.com/neovim/nvim-lspconfig/wiki/Snippets)
* [Project local settings](https://github.com/neovim/nvim-lspconfig/wiki/Project-local-settings)
* [Recommended plugins for enhanced language server features](https://github.com/neovim/nvim-lspconfig/wiki/Language-specific-plugins)

## Windows

In order for neovim to launch certain executables on Windows, it must append `.cmd` to the command name. To work around this, manually append `.cmd` to the entry `cmd` in a given plugin's setup{} call.

## Contributions

If you are missing a language server on the list in [CONFIG.md](CONFIG.md), contributing
a new configuration for it would be appreciated. You can follow these steps:

1. Read [CONTRIBUTING.md](CONTRIBUTING.md).

2. Choose a language from [the coc.nvim wiki](https://github.com/neoclide/coc.nvim/wiki/Language-servers) or
[emacs-lsp](https://github.com/emacs-lsp/lsp-mode#supported-languages).

3. Create a new file at `lua/lspconfig/SERVER_NAME.lua`.

    - Copy an [existing config](https://github.com/neovim/nvim-lspconfig/blob/master/lua/lspconfig/)
      to get started. Most configs are simple. For an extensive example see
      [texlab.lua](https://github.com/neovim/nvim-lspconfig/blob/master/lua/lspconfig/texlab.lua).

4. Ask questions on our [Discourse](https://neovim.discourse.group/c/7-category/7) or in the [Neovim Gitter](https://gitter.im/neovim/neovim).
