---
layout: post
title: Kubernetes simulator
tags: [kubernetes, test, simulator]
comments: true
---

I wanted to test my deployment from a previous post, and used a simulator.
To run kind on RHEL 8.6 I had to:
* [Use Iptables in firewalld](https://kind.sigs.k8s.io/docs/user/known-issues/#firewalld)
* sudo edit /etc/sysconfig/grub line "GRUB_CMDLINE_LINUX" change ipv6.disable to =0 , then grub2-mkconfig -o /boot/grub2/grub.cfg due to [Setting ipv6.disable=1 prevents both IPv4 and IPv6 socket opening for VXLAN tunnels](https://bugzilla.redhat.com/show_bug.cgi?id=1445054) - requires reboot
* trying to run rootless proved to not work due to some cgroups v2 I can not understand
* Running sudo $PWD/bin/kind create cluster -v 10 `--retain` --kubeconfig /home/nwaizer/.kube/simulator_kind , then run kind export logs and read jounrnalcrl.log 