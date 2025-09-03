
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
 - Provisioning Production Ready k8s Cluster (Recommendation in order)
	 - Provider Managed: AWS EKS, AKS, etc...
	 - [Rancher Managed](https://www.rancher.com/)
	 - kops
	 - kubeadm
	 - kubespray
	 - Other tools

- minikube -> Local k8s cluster with ease
- ArgoCD -> Deployment automation, monitoring
- Tilt -> Development tool kits for k8s, CRI images (e.g. auto build image and reload pod, better UI for logs)
- [k9s](https://github.com/derailed/k9s)(TUI for managing kubernentes)
- [Cillium + Hubble](https://docs.cilium.io/en/stable/overview/intro/#what-is-hubble) (distributed networking and security observability platform)
- [Kompose](https://github.com/kubernetes/kompose)(Convert docker-compose.yaml to k8s resources, *not really necessary*)

Honorable mention:
- [Kubernetes test infra](https://github.com/kubernetes/test-infra)
	- Check out how they CI/CD Kubernetes itself
- [Kubernetes application example tutorials](https://github.com/kubernetes/examples)

Terminal UI
- lazygit
- lazydocker
- [popeye](https://github.com/derailed/popeye) (can be integrated in k9s)
- stern (prints logs for multiple pods at the same time)

tmux
- [tpm](https://github.com/tmux-plugins/tpm)

stress-testing
- k6