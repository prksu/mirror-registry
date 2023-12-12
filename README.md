# Mirror Registry

This repo contains my expirement to use mirror container image registry for Kubernetes container-runtime.

## How to

1. Create Kind Kubernetes cluster with 3 nodes (1 control-plane, 2 nodes) with `extraMounts` to containerd registry mirror configuration.

```
kind create cluster --name cluster -f kind.yaml
```

2. Since Kubernetes host unable to resolve Cluster DNS (even possible, it's not a best practice) so we need metallb to create external IP for the registry.

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
```

3. Provide metallb `IPAddressPool`. Adjust to the Kind cluster network (in my case, Kind cluster use network with IP addresses `172.18.0.0/16` )

```
kubectl apply -f metallb-config.yaml
```

4. Deploy the registry.

```
kubectl apply -f registry.yaml
```

5. Test with sample-app

```
kubectl apply -f sample-app.yaml
```

The sample-app has `podAntiAffinity` so it would be spread across the nodes. The first pull image Kubernetes container-runtime will pull the image from upstream (`docker.io`) registry and then stored in the mirror (`registry`). If the image exists on the mirror registry, Kubernetes container-runtime will be pull the image from the mirror registry instead of upstream. You can make a test by upscale the `sample-app` replicas.
