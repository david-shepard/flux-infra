# Flux K3s Example (Oracle free tier)

This repo is used to illustrate a flux-managed Kubernetes cluster (k3s) on Oracle Cloud free tier.

- [x] Implement UI [(gimlet/capacitor-ui)](https://github.com/gimlet-io/capacitor)
- [ ] Implement pod [podinfo](https://github.com/stefanprodan/podinfo) microservice application
- [ ] Add requesite infrastructure for Tailscale [(TailScale Kubernetes operator?)](https://tailscale.com/kb/1236/kubernetes-operator)

## Configuration
- [`clusters/dev`](./clusters/dev/) points to the oracle cluster
- apps base configuration in [`apps/base`](./apps/base/)
- apps override configuration in [`apps/dev`](./apps/dev/)