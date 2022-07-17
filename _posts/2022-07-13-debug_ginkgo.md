---
layout: post
title: How to debug ginkgo code remotely using goland
tags: [go, ginkgo, goland]
---

I want to debug ginkgo code on a disconnected environment, which means I want to code my code to a remote host and run it from there.

My laptop runs IPv4 and I can ssh to the hub cluster which has a IPv6 connection to the open shift cluster I want to test with the Ginkgo code.

I will copy my code to the hub cluster and execute it from there on the cluster,

## Passwordless ssh to host
First I need to ssh from my laptop to the hub cluster and establish a passwordless ssh to it: https://duckduckgo.com/?q=passwordless+ssh+linux&ia=web

## Rsync
In my case the cnf-gotest repository has many files, and I do not want to copy all these files when ever I change the code, to be able to run them remotly.

So I use rsync, to compare the local files and remote and only copy the different files.

On that host used to run the test, you need to have rsync installed.

## Go
I downloaded the latest go from https://go.dev/dl/ and opened the content at the user home dir: /home/kni/go

## KUBECONFIG
The hub cluster was the server used to deploy the cluster I want to test, so it has the kubeconfig in the filesystem.

In my case I will define this environment var:

`export KUBECONFIG=~/server-0/auth/kubeconfig`
I may also need to set other environment variables for the test on that host.

## Define run/debug configuration
![debug_ginkgo1](/assets/posts/debug_ginkgo/debug_ginkgo1.png)

Use the go test for ginkgo.

You must choose package to be able to debug

First click the Working directory: and choose the location for the code on your local laptop. Only then click the Package path and start writing the package name, goland will auto complete it to the full import name:

![debug_ginkgo2](/assets/posts/debug_ginkgo/debug_ginkgo2.png)

As the hub cluster is stronger, use it to build your code.

In Environment, I added the location of the remote kubeconfig, see above.

Define the target in goland
Near the run on click the Manage targets...

![debug_ginkgo3](/assets/posts/debug_ginkgo/debug_ginkgo3.png)

Note: In order to use rsync, you have to install goland in your laptop and can not use the flatpak issue #38

## Debugging
Now choose the config and click the debug button on the right:


