---
tags:
  - semantic-versioning
  - husky
  - lint-staged
  - commitlint
---

### Example: 

###### 1. golangci-lint versioning policy - [link](https://golangci-lint.run/product/roadmap/)

###### 2. obsidian-iconize husky hooks - [link](https://github.com/FlorianWoelki/obsidian-iconize/tree/main/.husky) 

*`.husky/pre-commit`*
```sh
#!/usr/bin/env sh
npx lint-staged
```

*`.husky/commit-msg`*
```sh
#!/bin/sh
npx --no -- commitlint --edit ${1}
```
