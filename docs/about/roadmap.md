# Roadmap

We're tracking ongoing development in a [public
repository](https://github.com/kubinity-com/issues/issues). Got feature requests
or suggestions? Feel free to open an issue!

## Milestone 1: MVP (Reached! 🎉)

- [x] Admins can onboard new users via a simple command
- [x] Users are able to authenticate against the cluster to view, deploy and delete resources
- [x] A user can only access resources in their namespace
- [x] Users can consult the documentation to see how to access the cluster and deploy services

## Milestone 2: Service Abstraction (Reached! 🎉)

- [x] Users can leverage preexisting tools like ingress and storage

## Milestone 3: User Autonomy

- [ ] Users can create an account via a web service to access the cluster
- [ ] Users can create multiple namespaces
- [ ] Users can fetch their access token, or, if they wish, the entire kubeconfig file
- [ ] Users can regenerate their access token
- [ ] Users can delete their account, whereby the namespace is deleted

## Milestone 4: Web presence

- [ ] A landing page advertising the service, with links to docs and contact
- [ ] A blog with news to the platform

## Milestone 5: Open Ecosystem

- [ ] Factor out the logic of the management console to a public API
- [ ] Make console use the new API
- [ ] Open source console

## Milestone 6: Pay-as-you-go

- [ ] A new "pay as you go" tier for namespaces, billed monthly
