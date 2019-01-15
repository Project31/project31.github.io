---
layout: page
title: Project31
permalink: /
---
# Anything running Kubernetes on Aarch64/ARM64

Project31 aims to deploy the Fabric8 applications on Aarch64 (ARM64) architectures, in particular a cluster of RaspberryPi3, Pine64, Rock64, Rock64Pro

## 1. Install a base Centos-7 on your ARM64 hardware of choice

* Install [Centos-7 Aarch64 on Pine64, Rock64 or Rock64Pro](/pine64/)

Repeat step one until all nodes are up and running.

## 2. Install Docker and OpenShift on your Centos-7 Cluster

Pick a distro in Step 1 that already includes Docker and OpenShift. Instructions will follow which Ansible scripts to run to set up your local cluster
