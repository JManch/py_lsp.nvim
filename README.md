# py_lsp.nvim

## What is py_lsp?

`py_lsp.nvim` is a neovim plugin that helps with using the [lsp](https://neovim.io/doc/user/lsp.html) feature for python development.

It tackles the problem about the activation and usage of python virtual environments
for the nvim lsp.

## Installation

Using [vim-plug](https://github.com/junegunn/vim-plug):

```viml
Plug 'HallerPatrick/py_lsp.nvim'
```

Using [packer.nvim](https://github.com/wbthomason/packer.nvim):

```lua
use {'HallerPatrick/py_lsp.nvim'}
```

## Usage

Instead of initializing the server on your own, like in [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig#quickstart),
`py_lsp` is doing that for you.

Put this in your `init.lua` (or wrap in lua call for your `init.vim`)

```lua
require'py_lsp'.setup {
  -- This is optional, but allows to create virtual envs from nvim
  host_python = "/path/to/python/bin"
}
```

This minimal setup will automatically pass a python virtual environment path
to the LSP client for completion/linting.

WARNING: `py_lsp` is currently not agnostic against other python lsp servers that are starting or attaching.
Please be aware of other plugins, autostarting lsp servers.

### Features

`py_lsp` exposes several commands that help with a virtual env workflow.

| Command              | Parameter | Usage                                                                                                                     |
| -------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------- |
| `:PyLspCurrentVenv`  | No        | Prints the currently used python venv                                                                                     |
| `:PyLspDeactiveVenv` | No        | Shuts down the current LSP client                                                                                         |
| `:PyLspReload`       | No        | Reload LSP client with current python venv                                                                                |
| `:PyLspActivateVenv` | venv name | Activates a virtual env with given name (default: 'venv'). This venv should lie in project root                           |
| `:PyLspCreateVenv`   | venv name | Creates a virtual env with given name (default: 'venv'). Requires `host_python` to be set and have `virtualenv` installed |
| `:PyRun`             | command   | Run files and modules from current virtuale env                                                                           |

Most of these commands can be also run over a popup menu with `:PyLspPopup`.

#### Example Workflow

You open up your python project. Because there is no python virtual env confirgured, the LSP is not starting.

1. You run `:PyLspCreateVenv venv` to create a new virtual env from `host_python`.
2. You run `:PyLspCurrentVenv` to check if the LSP client is using your new venv.
3. You run `:PyRun -m pip install -r requirements.txt` to install project dependencies.
4. You run `:PyLspReload` so that the LSP client also find your new site-packages for autocompletion and correct linting.

You start programming!

### Extras

The virtual environment path and name can be retrieved with `client.config.settings.python.pythonPath` and `client.config.settings.python.venv_name`. This can for example be used in your statuslines.

Example provider for [feline](https://github.com/famiu/feline.nvim):

```lua
local function lsp_provider(component)

    local clients = {}
    local icon = component.icon or ' '

    for _, client in pairs(vim.lsp.buf_get_clients()) do
        if client.name == "pyright" then
          -- Check if lsp was initialized with py_lsp
          if client.config.settings.python["pythonPath"] ~= nil then
            local venv_name = client.config.settings.python.venv_name
            clients[#clients+1] = icon .. client.name .. '('.. venv_name .. ')'
          end
        else
          clients[#clients+1] = icon .. client.name
        end
    end

    return table.concat(clients, ' ')
end
```

This will give you a VSCode like status:

![Statusline with LSP server and venv name](./statusline_venv_name.png)

### Configuration

The configurations are not sensible yet, and are suiting my setup. This will change.

Default:

```
Default Values:
    auto_source = true,
    language_server = "pyright",
    on_attach = nil,
    source_strategies = {"default", "poetry", "system"},
    capabilities = nil,
    host_python = nil
```

## Other Language servers

I am using `pyright` and therefore works well with `pyright`. When using a not registered language server
like `jedi-language-server` you can register it like following:

```lua
local nvim_lsp_configs = require "lspconfig.configs"

nvim_lsp_configs["jedi_language_server"] = {
    default_config = {
        cmd = {"jedi-language-server"},

        -- Depending on your environment
        root_dir = nvim_lsp.util.root_pattern(".git", "setup.py",
                                              "pyproject.toml"),
        filetypes = {"python"}
    }
}

local capabilities = require('cmp_nvim_lsp').update_capabilities(vim.lsp
                                                                     .protocol
                                                                     .make_client_capabilities())

require("py_lsp").setup({
    language_server = "jedi_language_server",
    capabilities = capabilities
})


```

## Todo

- Support for different environment systems:
  - virtualenvwrapper
  - Conda
  - Pipenv
- Agnostic against other python lsp server

## Limitations

- All features are currently only available with `pyright` and `jedi-language-server`. `pylsp` is weird. It will still be started,
  but all features are run with a 'pyright' server or not at all.
- `py_lsp` expects to find virtualenv in the `cwd`, please check for that

## Note

This plugin is created due to following [Issue](https://github.com/neovim/nvim-lspconfig/issues/500#issuecomment-877305226).

This plugin currently includes a utility to automatically pass a virtualenv to
the pyright lsp server before initialization also take from the [Issue](https://github.com/neovim/nvim-lspconfig/issues/500#issuecomment-851247107).
(Thanks [lithammer](https://github.com/lithammer) and others).
