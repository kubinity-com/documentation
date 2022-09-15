# Getting started

This document describes how to get started with KubeAlliance.

## Prerequisites

To access the cluster and deploy resources, you need KubeCTL. If you haven't
already, check out [the official installation
guide](https://kubernetes.io/de/docs/tasks/tools/install-kubectl/) on the
Kubernetes documentation website.

## Creating an account

Kube Alliance is still being developed. There is no user management yet, so
administrative actions have to be done by a human. To request an account, please
head over to [my website](https://garrit.xyz/contact) and ask me about access to
the cluster.

After your account has been created, you will be provided with a
configuration-file that you need to copy to your home directory. KubeCTL usually
expects this file to be at `~/.kube/config`.

> **Note**: Keep in mind that this file contains secrets, so please
handle it with care!

And that's it! Just like that, you have access to your very own space on the
cluster.

To test your access, enter this command:

```
$ kubectl get resourcequota
```

If everything went fine, you should see your current resource usage on the
system.

And just like that, you have access to your very own space on the cluster.
Welcome aboard!

## Deploying a resource

To deploy a resource to a Kubernetes, you will need to create a manifest file
describing that resource. Here is a very simple example of a resource (in this
case a
[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/))
that will spin up a bare-bones NGINX server:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: 10Mi
            cpu: 10m
          limits:
            memory: 20Mi
            cpu: 20m
```

After saving this manifest to a file (e.g. `deployment.yml`), you can deploy it
to the cluster using KubeCTL:

```
> kubectl apply -f deployment.yml

deployment.apps/hello-world created
```

You can check if the resources have been created successfully by running:

```
> kubectl get pods

NAME                           READY   STATUS    RESTARTS   AGE
hello-world-7df56d9c98-92ngn   1/1     Running   0          3m7s
```

If you want, you can already access your application via the port-forwarding
capability of KubeCTL. The following command will forward traffic on port 8080
of your local machine to port 80 in the pod, which is the port we specified in the deployment:

```
> kubectl port-forward deployment/hello-world 8080:80

Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

You should be able to navigate to `localhost:8080` in your browser and see your
application!

### A note about resources

During the initial testing phase of KubeAlliance, I decided to keep the resource
limits fairly small. As a result, 

Once KubeAlliance reaches enough maturity, it will be possible
to allocate more resources in the cluster.

## Next steps

If you want to learn more about creating resources in Kubernetes, check out
[this](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)
amazing tutorial on their website. If you are completely new to Kubernetes, [the
concept overview](https://kubernetes.io/docs/concepts/) might help you make
sense of some things. If you're still feeling lost, don't hesitate to join our
[Matrix room](https://matrix.to/#/#kubeshare:matrix.org) and join the
conversation!
