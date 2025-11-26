# Neovim

Tips and tricks with [LazyVim](https://www.lazyvim.org/)

<!--toc:start-->

- [Neovim](#neovim)
  - [General](#general)
    - [Search Specific Keymaps/Keybindings](#search-specific-keymapskeybindings)
    - [Command History](#command-history)
  - [Files](#files)
    - [Diff Files](#diff-files)
    - [Show Inline Error Message (Diagnostics)](#show-inline-error-message-diagnostics)
  - [Markdown](#markdown)
    - [Add Table of Content (`marksman`)](#add-table-of-content-marksman)

<!--toc:end-->

## General

#### Search Specific Keymaps/Keybindings

#### Yank Relative File Path

```vim
:let @+ = expand("%")
```

#### Command History

#### Yank History

#### Undo Tree

#### Search and Replace

Vim:

```vim
:%s/old/new/gc
```

LazyVim built-in:

**\<leader\>sr**

#### Show All Man Pages

## Files

#### Diff Files

When you have 2 files opened in the same window:

```vim
:windo diffthis

:windo diffoff
```

#### Show Error Messages (Diagnostics)

1. Floating Window

_\<leader\>cd_

Hover: The default keybinding for "Hover" is K. If you have configured your LSP
settings to show diagnostics on hover, pressing K on a line with a diagnostic
might display the message.

2. Diagnostics List

Opened buffer: _\<leader\>xx_

Current buffer: _\<leader\>x**X**_

#### Wrapping Texts (Format)

_gww_

goto -> Format -> Next word

## Markdown

Anything .md files

#### Add/Update Table of Content (`marksman`)

_\<leader\>ca_:

Code Actions -> Create Table of Content
