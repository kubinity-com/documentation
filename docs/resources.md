# Namespace Resources

Every namespace has a dedicated amount of resources. During the initial testing
phase of Kubinity, you will have a quarter of a CPU core, 250 MB RAM and 2
GB of storage available for scheduling. Once Kubinity reaches enough
maturity, it will be possible to allocate more resources in the cluster.

By default, a scheduled pod on Kubinity reserves 50m CPU and 50 MB RAM. This
means that you can deploy 5 pods in total. However, the resources requested by a
pod can be updated using its spec. Here's an example for `nginx`, which runs
fine with way less resources:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: nginx:latest
    resources:
      limits:
        memory: "25Mi"
        cpu: "25m"
```

In theory, you could deploy 10 of these pods to your namespace.

To learn more about resources, check out the [official
documentation](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
of Kubernetes.
