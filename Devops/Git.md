---
tags:
  - git
---

<!--toc:start-->

- [Git](#git)
  - [Basic Topic](#basic-topic)
    - [1. Git pager](#1-git-pager)
    - [2. Git show remote](#2-git-show-remote)
    - [3. Git delete branch](#3-git-delete-branch)
    - [4.Git built-in log tree](#4git-built-in-log-tree)
    - [5. Git pickaxe](#5-git-pickaxe)
  - [Advance topic](#advance-topic)
    - [1. Git garbage collect](#1-git-garbage-collect)
    - [2. Git prune](#2-git-prune)

<!--toc:end-->

# Git

Tips and tricks with git

## Basic Topic

#### 1. Git pager

To use a different pager for next git command:

```sh
GIT_PAGER=cat git show
```

#### 2. Git show remote

```sh
# Show remotes
git remote -v

# Set remotes
git remote set-url origin <NEW_URL>
```

#### 3. Git delete branch

```sh
# Show all branches
git branch -a

# Delete local branch
git branch -d <branch>

# Delete remote branch
git push origin --delete <branch>

# Delete the refs to branches that don't exist on the remote
git fetch --prune

```

#### 4.Git built-in log tree

```sh
git log --graph --oneline --all
```

#### 5. Git pickaxe

Find commits where a specific string or regular expression was **introduced**, **removed**, or **changed**.

```sh
git log -p -S "<string_to_search"
```

## Advance topic

#### 1. Git garbage collect

_Executing `git gc` is literally telling Git to clean up the mess it's made in the current repository - [Atlassian](https://www.atlassian.com/git/tutorials/git-gc)_

<mark style="background: #FFF3A3A6;">You don't want to do it too often, sometimes those "**garbage**" may save you</mark>

```sh
git gc
```

#### 2. Git prune

_The `git prune` command is intended to be invoked as a child command to `git gc`. It is highly unlikely you will ever need to invoke `git prune` in a day to day software engineering capacity. - [Atlassian](https://www.atlassian.com/git/tutorials/git-prune#:~:text=git%20fetch%20%2D%2Dprune%20is,use%20on%20the%20remote%20repository.)_

```sh
# Don't execute the prune. Just show an output of what it will do
git prune --dry-run
```
