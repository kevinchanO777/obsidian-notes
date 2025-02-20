
### 1. Test internet speed 
```sh
curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python -
```

### 2. Checksum
```bash
md5 yourfile.ext
shasum yourfile.ext
shasum -a 256 yourfile.ext
```

### 3. Ansible Dev Container
`.devcontainer/devcontainer.json`
https://github.com/ansible/ansible-dev-tools/tree/main/.devcontainer
https://ansible.readthedocs.io/projects/dev-tools/container/

```json
{
	"name": "ansible-dev-container-docker",
	"image": "ghcr.io/ansible/community-ansible-dev-tools:latest",
	"containerUser": "root",
	"runArgs": [
		"--privileged",
		"--device",
		"/dev/fuse",
		"--hostname=ansible-dev-container"
	],
	"updateRemoteUserUID": true,
	"customizations": {
		"vscode": {
			"extensions": [
				"redhat.ansible"
			]
		}
	}
}
```