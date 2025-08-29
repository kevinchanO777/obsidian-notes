---
tags:
  - obsidian
  - git
  - plugin
---
> [!WARNING] Git plugin is FUCKING BROKEN. Make sure open Obsidian from terminal !!!!!!!!!!


-> Seems like the git plugin uses the incorrect ssh agent/socket (i.e. the `macos launchd ssh socket`) when we open Obsidian using *finder*


-> As we add our personal key and create a user ssh agent during zsh startup, we can 'force' Obsidian to use the correct one when we open Obsidian from the terminal: `cd /Application; open Obsidian`





# Using Obsidian
###### 1. Press `cmd` and hover to preview note - [doc](https://help.obsidian.md/Plugins/Page+preview)

###### 2. Callouts - [doc](https://help.obsidian.md/Editing+and+formatting/Callouts)

> [!info] Custom Title
> 

###### 3. Hide tags in preview - [reddit](https://www.reddit.com/r/ObsidianMD/comments/nm1zl6/hide_tags/)

%% Use comment syntax to wrap around a tag %%

###### 4. Obsidian properties - [doc](https://help.obsidian.md/Editing+and+formatting/Properties)
	Useful for adding tags at the begining of each note

###### 5. Bring color to obsidianmd - [forum](https://forum.obsidian.md/t/coloured-text/18031/2)
	 Use CSS snippet ✅

###### 6. Obsidian CSS snippets - [github](https://github.com/Dmytro-Shulha/obsidian-css-snippets)
	Community maintained snippets
