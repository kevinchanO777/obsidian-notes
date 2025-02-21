
`.devcontainer/devcontainer.json`
```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/go
{
	"name": "Go",
	// Or use a Dockerfile or Docker Compose file. More info: https://containers.dev/guide/dockerfile
	// "image": "mcr.microsoft.com/devcontainers/go:1-1.23-bookworm"
	"dockerFile": "Dockerfile",
	// Features to add to the dev container. More info: https://containers.dev/features.
	"features": {
		"ghcr.io/guiyomh/features/golangci-lint:0": {"version": "latest"}
	},
}

```

`.devcontainer/Dockerfile`
```Dockerfile
FROM mcr.microsoft.com/devcontainers/go:1-1.23-bookworm

# Install additional tools
RUN apt-get update && apt-get -y install \
    tig \
    silversearcher-ag

```