---
title: Custom image creation
description: Learn how to write custom images for use with Coder.
---

Custom images allow you to define workspaces that include the dependencies,
scripts, and user preferences helpful for your project.

This guide assumes that you're familiar with:

- [Dockerfiles](https://docs.docker.com/engine/reference/builder/)
- [docker login](https://docs.docker.com/engine/reference/commandline/login/)
- [docker build](https://docs.docker.com/engine/reference/commandline/build/)
- [docker push](https://docs.docker.com/engine/reference/commandline/push/)

## Resources

For ideas on what you can include in your images, see:

- [Sample Coder images](https://github.com/coder/enterprise-images)
- [Guide: Node.js image for Coder](../guides/customization/node)

## Creating a custom image

Instead of starting from scratch, we recommend extending one of our
[sample images](https://github.com/coder/enterprise-images):

```Dockerfile
# Dockerfile
FROM codercom/enterprise-base:ubuntu

USER root

# Add software, files, dev tools, and dependencies here
RUN apt-get install -y ...
COPY file ./

USER coder

...
```

Please note:

- Coder workspaces mount a
  [home volume](../workspaces/personalization#persistent-home). Any files in the
  image's home directory will be replaced by this persistent volume. If you have
  install scripts (e.g., those for Rust), you must configure them to install
  software in another directory.

- If you're using a different base image, see our
  [image minimum requirements](https://github.com/coder/enterprise-images/#image-minimums)
  to make sure that your image will work with all of Coder's features.

- You can build images inside a
  [CVM](../admin/workspace-management/cvms.md)-enabled Coder workspace with
  Docker installed (see our
  [base image](https://github.com/coder/enterprise-images/tree/main/images/base)
  for an example of how you can do this).

- If you're using CVM-only features during an image's build time (e.g., you're
  [pre-loading images](https://github.com/nestybox/sysbox/blob/master/docs/quickstart/images.md#building-a-system-container-that-includes-inner-container-images--v012-)
  in workspaces), you may need to install the
  [sysbox runtime](https://github.com/nestybox/sysbox) onto your local machine
  and build your images locally. Note that this isn't usually necessary, even if
  your image installs and enables Docker.

## Example: Installing an IntelliJ IDE

This snippet shows you how to install an IntelliJ IDE onto your image so that
you can use it in your Coder workspace:

```Dockerfile
# Dockerfile
FROM ...

# Install IDEs as root
USER root

RUN mkdir -p /opt/[IDE]
RUN curl -L \
"https://download.jetbrains.com/product?code=[CODE]&latest&distribution=linux" \
| tar -C /opt/[IDE] --strip-components 1 -xzvf -

# Add a binary to the PATH that points to the IDE startup script.
RUN ln -s /opt/[IDE]/bin/idea.sh /usr/bin/[IDE]
```

Make sure that you replace `[IDE]` with the name of the IDE in lowercase and
provide its
[corresponding `[CODE]`](https://plugins.jetbrains.com/docs/marketplace/product-codes.html).

More specifically, here's how to install the IntelliJ IDE onto your image:

```Dockerfile
# Dockerfile
FROM ...

USER root

# Install intellij
RUN mkdir -p /opt/idea
RUN curl -L "https://download.jetbrains.com/product?code=IIC&latest&distribution=linux" \
| tar -C /opt/idea --strip-components 1 -xzvf -

# Add a binary to the PATH that points to the Intellij startup script.
RUN ln -s /opt/idea/bin/idea.sh /usr/bin/intellij-idea-community
```
