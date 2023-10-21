# k8s-port-forward
[![Testing](https://github.com/vbem/k8s-port-forward/actions/workflows/test.yml/badge.svg)](https://github.com/vbem/k8s-port-forward/actions/workflows/test.yml)
[![Super Linter](https://github.com/vbem/k8s-port-forward/actions/workflows/linter.yml/badge.svg)](https://github.com/vbem/k8s-port-forward/actions/workflows/linter.yml)
[![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/vbem/k8s-port-forward?label=Release&logo=github)](https://github.com/vbem/k8s-port-forward/releases)
[![Marketplace](https://img.shields.io/badge/GitHub%20Actions-Marketplace-blue?logo=github)](https://github.com/marketplace/actions/kubernetes-port-forward)

## About

This action can be used to [forward](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) local ports on GitHub runners to workloads in your Kubernetes cluster.

Current implementation is to setup a `kubectl port-forward` daemon in background and immune to hangups, which make local ports on runner available to subsequent steps.

Note that this action follow [official *kubeconfig* authentication methods](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) and you need to sign in your K8s cluster via other actions such as [kubeconfig4sa](https://github.com/marketplace/actions/kubeconfig-for-service-account-sa), or manually set `KUBECONFIG` environment variable for this action in your workflows file.


## Example usage

```yaml
- name: Setup KUBECONFIG
  uses: vbem/kubeconfig4sa@v1
  with:
    server:     https://your-kubeapi-server:6443
    ca-base64:  ${{ secrets.K8S_CA_BASE64 }}
    token:      ${{ secrets.K8S_SA_TOKEN }}
    namespace:  my-namespace

# This action will forward port 8080 on runner to port 80 of your service in Kubernetes!
- name: Setup Kubernetes port-forward daemon
  uses: vbem/k8s-port-forward@v1
  with:
    workload: 'svc/mysvc'

- name: Request a service in Kubernetes
  run: curl localhost:8080
```

![Example](https://repository-images.githubusercontent.com/479920190/ce3b08cc-302b-481e-8447-5f060270d3f5)

## Inputs

ID | Type | Default | Description
--- | --- | --- | ---
`workload` | String | *Required input* | Kubernetes workload `type/name` such as `deploy/mydeploy`, `svc/mysvc` or `po/mypod`
`mappings` | String  | `8080:80` | Ports mappings `[LOCAL_PORT:]REMOTE_PORT [...[LOCAL_PORT_N:]REMOTE_PORT_N]`
`options` | String | `<empty>` | Other command-line options, such as `--namespace=myns`
`sleep` | String | `3` | Seconds to wait before action finished

## Outputs

ID | Type | Description
--- | --- | ---
`pid` | String | Process ID of port-forward daemon
`log` | String | Path to port-forward log file