# Configurability and Security

In addition to choosing a base image, and getting a good build strategy, you need to decide how you plan on injecting configuration and secrets into your application.

- Configurability
    - What configuration items might change between deployments?
    - Does your app connect to external data-storage services?
- Security
    - If your app connects to external services, how are credentials passed in?
    - Does this final image have packages which have security flaws?

## Configurability

There are certain aspects of your application that may be external, such as a database or cache, and you will want a way to specify the connection credentials as env vars or configuration files. This may require a code change and a slight re-architecture, depending on your current build/deploy strategy.

## Security

There are at least a couple of layers to consider when talking container security. The first rule of thumb is to never bake secrets or credentials into your builds, or store them in source control, where the builds are built from. If you did this, and someone got a hold of your image, they could simply `exec` into it, and read your credentials.

The second priority should be on ensuring that packages and other plugins that you install, do not have their own security flaws. Tools like [Twistlock](https://www.twistlock.com/2016/11/07/azure-container-registry/) and [Aqua Security](https://blog.aquasec.com/image-vulnerability-scanning-in-azure-container-registry) support Azure container registry. There are also open source tools that integrate with most repositories like [Clair](https://github.com/coreos/clair).
