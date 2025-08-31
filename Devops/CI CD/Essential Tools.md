
> [!info]
> ***This following tools are consider to be essential or at least useful in DevOps.***

Infrastructure as code (IaC)
- Terraform

Configuration management
- Ansible
- Puppet
- Cloud specific solutions
	- AWS CloudFormation

Conventional commits
- commit-lint
- commitizen

CI/CD
- Github actions
- GitLab
- CircleCI
- Gitea for self-hosting

Monitoring, observability and telemetry
- Grafana stack
	- Prometheus - *TSDB*
	- Loki - *Log aggregator*
- Cloud-native solutions
	- AWS CloudWatch
	- AWS CloudTrail

Kubernetes:
- ArgoCD -> Deployment automation, monitoring
- kubeadm -> Deploy production ready k8s cluster
- minikube -> Local k8s cluster with ease
- Tilt -> Development tool kits for k8s, CRI images (e.g. auto build image and reload pod, better UI for logs)

Terminal UI
- lazygit
- lazydocker
- [k9s](https://github.com/derailed/k9s)(TUI for managing kubernentes)
- [popeye](https://github.com/derailed/popeye) (can be integrated in k9s)
- stern (prints logs for multiple pods at the same time)

tmux
- [tpm](https://github.com/tmux-plugins/tpm)
