### 1. Bash parameter expansion

In Bash, you can use [parameter expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html) to set a **default** **value** for a variable. 

```bash
LENGTH=${1:-16}
pwgen -sy $LENGTH 1 | tr -d '\n' | pbcopy
```

This sets `LENGTH` to the value of `$1` *(i.e. the first argument when calling the script)* if it's provided; otherwise, it defaults to `16`.


### 2. Redirection

> [!IMPORTANT] Redirection is shell specific

| Operator       | Description                                                      |
| -------------- | ---------------------------------------------------------------- |
| `>` *OR* `1>`  | Redirects standard output (`stdout`)                             |
| `2>`           | Redirects standard error (`stderr`)                              |
| `&>` *OR* `>&` | Redirects **both** `stdout` and `stderr` to a file               |
| `2>&1`         | Redirect standard error to the same location as standard output. |
