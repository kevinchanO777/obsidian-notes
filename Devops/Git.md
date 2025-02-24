---
tags:
  - git
---

#### 1. Git pager
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




## Advance topic

---

#### 1. Git garbage collect

*Executing `git gc` is literally telling Git to clean up the mess it's made in the current repository - [Atlassian](https://www.atlassian.com/git/tutorials/git-gc)*

!

```sh
git gc
```

#### 2. Git prune

*The `git prune` command is intended to be invoked as a child command to `git gc`. It is highly unlikely you will ever need to invoke `git prune` in a day to day software engineering capacity. - [Atlassian](https://www.atlassian.com/git/tutorials/git-prune#:~:text=git%20fetch%20%2D%2Dprune%20is,use%20on%20the%20remote%20repository.)*

```sh
# Don't execute the prune. Just show an output of what it will do
git prune --dry-run
```