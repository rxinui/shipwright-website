---
title: Build Controller Profiling
linkTitle: Profiling
---
<!--
Copyright The Shipwright Contributors

SPDX-License-Identifier: Apache-2.0
-->

The build controller supports a `pprof` profiling mode, which is omitted from the binary by default. To use the profiling, use the controller image that was built with `pprof` enabled.

## Enable `pprof` in the build controller

In the Kubernetes cluster, edit the `shipwright-build-controller` deployment to use the container tag with the `debug` suffix.

```bash
kubectl --namespace <namespace> set image \
  deployment/shipwright-build-controller \
  shipwright-build-controller="$(kubectl --namespace <namespace> get deployment shipwright-build-controller --output jsonpath='{.spec.template.spec.containers[].image}')-debug"
```

## Connect `go pprof` to build controller

Depending on the respective setup, there could be multiple build controller pods for high availability reasons. In this case, you have to look up the current leader first. The following command can be used to verify the currently active leader:

```bash
kubectl --namespace <namespace> get configmap shipwright-build-controller-lock --output json \
  | jq --raw-output '.metadata.annotations["control-plane.alpha.kubernetes.io/leader"]' \
  | jq --raw-output .holderIdentity
```

The `pprof` endpoint is not exposed in the cluster and can only be used from inside the container. Therefore, set-up port-forwarding to make the `pprof` port available locally.

```bash
kubectl --namespace <namespace> port-forward <controller-pod-name> 8383:8383
```

Now, you can set up a local webserver to browse through the profiling data.

```bash
go tool pprof -http localhost:8080 http://localhost:8383/debug/pprof/heap
```

_Please note:_ For it to work, you have to have `graphviz` installed on your system, for example using `brew install graphviz`, `apt-get install graphviz`, `yum install graphviz`, or similar.
