+++
categories = []
date = "2020-02-22"
description = ""
slug = ""
tags = []
title = "Read-only root file system"
draft = false
toc = true
weight = 10
+++

### Introduction
In this exercise, we cover:

 - How to mark the root of your container read-only
 - How to make certain paths writable if you need to

### Setup
We'll use the same image as before, because app vulnerabilities
are useful to demonstrate attacks!

We'll start with a writable file system and show how useful
that is for an adversary.

_This is the same as the [previous exercise](../01-streamline-images).
Skip ahead to the [Countermeasure](#countermeasure) if you just did that one._

Now, deploy the application:

```
kubectl apply -f https://securek8s.dev/struts/base.yaml
```

Wait for it to deploy:

```
kubectl get pod -n struts -w
```

### Attack

```
apps/struts/attack struts "$(./utils/get-node-extip):30003"
```

### Countermeasure
We'll update to a new deployment that asks for a read-only root file system,
with an image that has a small modification to account for that.

Check out what's different in the Dockerfile:

```
diff apps/struts/Dockerfile apps/struts/Dockerfile-ro
```

Not too much! Just a path we want to make writable.

What about the deployment? We can use a neat command, `kubectl diff`, to compare our new YAML with the currently-running app.

```
kubectl diff -f https://securek8s.dev/struts/ro.yaml
```

Now that we see the difference, let's deploy the read-only app:

```
kubectl apply -f https://securek8s.dev/struts/ro.yaml
```

Wait for it to deploy:

```
kubectl get pod -n struts -w
```

Then we'll attack it:

```
apps/struts/attack struts "$(./utils/get-node-extip):30003"
```

👍

### Attack effects after patching
The attack fails to modify the running container.
While this is still a problem--you don't want them in your
container anyway!--at least they are more constrained and
have to find other ways to accomplish their goals.

### How to use it yourself
Use the [`SecurityContext`](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#securitycontext-v1-core)
field called `readOnlyRootFilesystem`.

If you need a writable path, use a `VOLUME` instruction
(if you deploy only on clusters or machines running Docker)
or mount a Kubernetes `emptyDir` if you want a solution that
works on other runtimes (especially CRI-O, which by default simply
[ignores](https://medium.com/cri-o/cri-o-configurable-image-volume-support-dda7b54f4bda)
your `VOLUME` declarations).

### Next up
We'll explore host mounts, and their read-only bit, in the next exercise:

[**Read-only host mounts**](../15-ro-mount)
