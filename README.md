### earthly-kubernetes

This is a short proof-of-concept for deploying [Earthly remote buildkit][erb]
on Kubernetes, with the goals of exploring multi-node buildkit deployment (via
DaemonSet to ensure each available node has a buildkit) and employing
[cert-manager][cm] to create a PKI for mTLS between buildkit and earthly
clients.

[erb]: https://docs.earthly.dev/ci-integration/remote-buildkit
[cm]: https://cert-manager.io

##### Quick start

This repository is _not_ meant to be cloned directly, unless for contributions.

You will need both [kubectl and kustomize][kubectl].

[kubectl]: https://kubectl.docs.kubernetes.io/

To get started with a basic workload with no mTLS configured, please do:

```sh
kustomize localize "https://github.com/zakame/earthly-kubernetes?ref=master" earthly-kubernetes
cd earthly-kubernetes
kubectl apply -k .
```

This deploys a DaemonSet of earthly-buildkitd for each node on the cluster, barring taints.

Once ready, test the DaemonSet by loading some jobs:

```sh
kubectl apply -f https://raw.githubusercontent.com/zakame/earthly-kubernetes/master/jobs.yaml
```

Then inspect logs for all jobs via:

```sh
kubectl logs -n earthly -f -l job=earthly
```

To add mTLS, first get the Kustomize component:

```sh
kustomize localize "https://github.com/zakame/earthly-kubernetes//mtls?ref=master" components
```

Then add this component and redeploy:

```sh
kustomize edit add component components/mtls
kubectl apply -k .
```

Then re-test with updated jobs:

```sh
kubectl delete jobs -n earthly -l job=earthly
kubectl apply -f https://raw.githubusercontent.com/zakame/earthly-kubernetes/master/mtls/jobs.yaml
```

Then reinspect with the same `kubectl logs` command as above.

Finally, for clean-up:

```sh
kubectl delete -k .
```