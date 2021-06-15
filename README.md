# TeXMagic.nvim
## 1. Introduction
This is a simple, lightweight [Neovim](https://neovim.io) plugin that facilitates LaTeX build engine selection via magic comments. It is designed with the [TexLab](https://github.com/latex-lsp/texlab) LSP server's build service in mind, which at the time of this plugin's making had to be specified in `init.lua`/`vim` and could not be set on a by-project basis.

The plugin identifies magic comments at the very top of a LaTeX document (as below) and will currently only extract magic TeX *program* comments, which specify the name of a build "engine" or toolchain.

```
%! TEX program = xelatex
```
The plugin looks in two places for the engine name requested in the magic comment:

1. In a table of custom build engines that you provide in `init.lua`/`vim` (via the `setup` function).
2. In the table of default build engines.

The plugin provides three default `latexmk` build engines: `pdflatex`, `xelatex`, and `dvipspdf`. In order to be used, the `setup` function must be called in `init.lua`/`vim`. The user can also provide any number of custom build engines (see instructions).

### 1.1 The purpose of the plugin
Currently, anyone who wishes to use TexLab's build service can only specify a global build engine, which is troublesome if you need different build engines for different projects. This plugin is mainly for people who would like to use TexLab's build service and would like to make it a little more configurable. [Vimtex](https://github.com/lervag/vimtex) is more functional than the [TexLab LSP server](https://github.com/latex-lsp/texlab), but anyone not using Vimtex for whatever reason should find [TeXMagic.nvim](https://github.com/jakewvincent/texmagic.nvim) useful.

### 1.2 Improvements
- [X] Allow for build processes to be specified by the user and given a name that the magic comment can point to
- [ ] Make documentation
- [ ] Allow plugin to be restarted and TexLab's build settings to be reloaded without closing file, e.g. in the event that a magic comment is added or changed

## 2. Instructions
### 2.1. Requirements
- Should be used alongside TexLab LSP server or similar.
- Neovim 0.5.0+ (currently untested on 0.4.4)
- [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)
- Set `g:texflavor` to `latex`.
    - Lua: `vim.g['tex_flavor'] = 'latex'`
    - Vimscript: `let g:tex_flavor = 'latex'`

### 2.2 Installation
Install using your preferred package manager. Using [paq-nvim](https://github.com/savq/paq-nvim), for instance:

```lua
require('paq-nvim').paq{'jakewvincent/texmagic.nvim'}
```

For the default build engines to be available, you must call the setup function in your `init.lua`/`vim`. Once it is called, the global variable called `TeXMagicBuildConfig` becomes available and can be used to value the `build` key in your TexLab LSP server config.

### 2.3 Configuration
#### 2.3.1 Default build engines only
You can pass nothing to the setup function, in which case the default build engines are made available and can be selected if their name matches the program name specified in a magic comment.

```lua
-- Only default build engines made available (pdflatex, xelatex, dvipspdf)
require('texmagic').setup{}
```

#### 2.3.2 Custom build engines
The user can also specify different build engines. If these have the same name as any of the default build engines, TeXMagic will prioritize those over the defaults. **There must be a key named 'engines' in your config table.**

```lua
-- Run setup and specify two custom build engines
require('texmagic').setup{
    engines = {
        pdflatex = {    -- This has the same name as a default engine but will be preferred
            executable = "latexmk",
            args = {
                "-pdflatex",
                "-interaction=nonstopmode",
                "-synctex=1",
                "-pv",
                "%f"
            },
            isContinuous = false
        },
        lualatex = {    -- This is *not* one of the defaults, but it can be called via magic comment if defined here
            executable = "latexmk"
            args = {
                "-pdflua",
                "-interaction=nonstopmode",
                "-synctex=1",
                "-pv",
                "%f"
            },
            isContinuous = false
        }
    }
}
```

#### 2.3.3 Pass the global build variable to build config
After calling the setup function, interface your [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) TeXLab setup with TeXMagic using the global variable `TeXMagicBuildConfig`. If I'm using TexLab's build service and want to allow the build engine to vary based on my magic comments, I'll use `TeXMagicBuildConfig` to value the `build` key in my TexLab config table (such as below).

```lua
require('lspconfig').texlab.setup{
    cmd = {"texlab"},
    filetypes = {"tex", "bib"},
    settings = {
        texlab = {
            rootDirectory = nil,
            --      ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓
            build = TeXMagicBuildConfig,
            --      ↑ ↑ ↑ ↑ ↑ ↑ ↑ ↑ ↑ ↑
            forwardSearch = {
                executable = "evince",
                args = {"-f"}
            }
        }
    }
}
```

### 2.4 Scenarios
1. Document is a `tex` document but has no magic comment on open ⇒ Default build engine is selected (pdflatex)
2. Document is a `tex` document and did have a magic comment on open ⇒ The named build engine is selected from the custom build engines or the default build engines (or if it is in neither of those, pdflatex is selected as the build engine)
3. A magic comment is added or changed *after* the document has been open ⇒ The previously selected build engine will remain selected until the file is closed and reopened (or else until [TeXMagic.nvim](https://github.com/jakewvincent/texmagic.nvim)'s setup function and the [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) setup function for TexLab are run again)

## 3. Troubleshooting
A few vim functions are defined that may help diagnose problems:

* `TeXMagicShowComments`: returns any magic comments found at the top of your document
* `TeXMagicCommentsFound`: returns a boolean indicating whether or not magic comments were found
* `TeXMagicSetupStatus`: returns a boolean indicating whether TeXMagic's setup function was run
* `TeXMagicSelectedProgram`: returns the name and source of the selected TeX program (if any was selected)
* `TeXMagicConfigFound`: returns a boolean indicating whether any user-provided build engines were found
* `TeXMagicLoaded`: returns the status of the plugin (if loaded, returns `1`)

## 4. Links
* [TexLab LSP server](https://github.com/latex-lsp/texlab) 
* [nvim-lspconfig: Helpful defaults for Neovim's LSP client](https://github.com/neovim/nvim-lspconfig)
* [Config for TexLab via nvim-lspconfig](https://github.com/neovim/nvim-lspconfig/blob/master/CONFIG.md#texlab)
