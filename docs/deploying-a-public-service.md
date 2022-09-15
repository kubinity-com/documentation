---
sidebar_position: 2
---

# Deploying a public service

This guide will walk you through deploying an instance of the `nginx` and expose
it to the public.

## Preparing your domain

In order to deploy a public service to the cluster, you will need a valid
domain. Once you own a domain, you will need to point your domain to the
cluster. There are two methods to achieve this.

### CNAME record (preferred)

A CNAME record can be thought of as an "alias" for another domain. Head over to
the DNS settings of your domain registrar and add a CNAME record to the
canonical domain of the cluster:

```
kube01.kubealliance.com
```

### A and AAAA records

As an alternative to a CNAME record, you can create an A record for each of
these IP addresses:

```
78.46.201.114
49.12.219.73
168.119.170.212
```

If you want to use IPv6 addresses, make sure to create three AAAA records
pointing to these domains:

```
2a01:4f8:c2c:2d76::1
2a01:4f8:c17:44e2::1
2a01:4f8:1c1c:8523::1
```

> **Note**: It's advised to create one record for each IP to ensure a higher
> availability.

> **Todo**: These IP addresses are currently subject to change. There should be
> a load balancer in front of these addresses instead.

### Deploying an application

With your domain set up, we're ready to get our hands dirty! The way you usually
deploy an application to a Kubernetes cluster is by creating so called "resource
definitions", also commonly called "manifests".

To make your life easier, it's advised that you keep a directory containing all
your manifests. This makes it easier to re-deploy a resource later in case it is
accidentally deleted. For inspiration about how this might look, you can take a
look at the [manifests of this cluster](https://github.com/garritfra/infra).

We already talked about creating a
[`Deployment`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
resource in our [Getting started](/docs/getting-started.md) guide. Here's the
manifest, in case you haven't yet deployed it:

```yml
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

Notice that in the `template` section, we assigned the deployment the label
`app` with a value of `nginx`. This will become relevant in just a bit.

Next up, you will need to create a
[`Service`](https://kubernetes.io/docs/concepts/services-networking/service/)
that sits in front of that deployment. In a more complex application, services
will handle load balancing and other fancy stuff. For us, a service just defines
what ports we want to expose to other parts of the cluster.

```yml
# service.yml

apiVersion: v1
kind: Service
metadata:
  name: hello-world
spec:
  selector:
    app: nginx
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 80
```

Here, we create a Service called "hello-world" and tell Kubernetes to forward
incoming traffic to every pod where the label `app` matches the name `nginx`.
Since we have a `Deployment` with such a label, every pod deployed by that
`Deployment` will be matched by our service. We also say that traffic hitting
our service on port `80` should be forwarded to port `80` of the container.

The last piece of the puzzle is the
[`Ingress`](https://kubernetes.io/docs/concepts/services-networking/ingress/).
Ingresses make up the "outer layer" of a cluster. They match incoming traffic
from outside the cluster to services running in the cluster.

```yml
# ingress.yml

kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: hello-world
  annotations:
    kubernetes.io/ingress.class: nginx # Required

spec:
  rules:
    - host: foo.bar # Your domain
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: hello-world
                port:
                  number: 80
```

The KubeAlliance cluster runs
[NGINX](https://kubernetes.github.io/ingress-nginx/) as a
public Ingress controller. Every request from and to the cluster goes through
this controller. By creating an `Ingress`, we tell NGINX that we want to
handle the traffic for a specific hostname, which is the domain we prepared
earlier. Note that there can be many ingresses in a cluster at the same time.

After deploying all these manifests, you should be able to open your browser and
navigate to your domain. If you have any questions, please let us know.

> **TODO**: Link to contact information

## Securing your site with TLS

To make securing your site via HTTPS as convenient as possible, you can make use
of our instance of [`cert-manager`](https://cert-manager.io/), which is running
in the cluster.

To issue certificates, you first need to deploy an
[Issuer](https://cert-manager.io/docs/tutorials/acme/http-validation/) to your
namespace. As the name implies, an issuer is responsible to issue TLS
certificates for your domain. Here's an example for an issuer that uses
[LetsEncrypt](https://letsencrypt.org/) to fetch free certificates:

```yml
# issuer.yml

apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: example@yourdomain.com
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource used to store the account's private key.
      name: letsencrypt-staging-account-key
    solvers:
      - http01:
          ingress:
            class: nginx
```

Notice that this issuer uses the [staging
environment](https://letsencrypt.org/docs/staging-environment/) of LetsEncrypt.
The certificates issued by this environment are not considered secure, but they
are great for testing the waters before generating a "real" certificate. If you
directly use the production environment without testing your setup first, your
account might run into a [rate
limit](https://letsencrypt.org/docs/rate-limits/)! To start issuing real
certificates, either create a new Issuer or override the existing one with the
URL of the LetsEncrypt production environment:

```
https://acme-staging-v02.api.letsencrypt.org/directory
```

Once you deployed the issuer, you should be able to generate certificates. You
can now configure TLS on your existing ingress by adding the `tls` section like
this:

```yml
# ingress.yml

kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: hello-world
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/issuer: letsencrypt-staging

spec:
  rules:
    - host: mydomain.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: hello-world
                port:
                  number: 80
  tls:
  - hosts:
    - mydomain.com
    secretName: mydomain.com-tls # The certificate will be stored in this secret
```

Wait for a minute while cert-manager fetches your certificate, and you should
see a `Secret` resource in your namespace, containing the certificate:

```
> kubectl get secrets

NAME                             TYPE                                  DATA   AGE
hello-world-issuer-account-key   Opaque                                1      6m
mydomain.com-tls                 kubernetes.io/tls                     2      1m
```

And if everything went well, you can now access your site via HTTPS!
