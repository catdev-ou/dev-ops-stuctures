# Overview on our Dev Ops Stuctures

The documents in this repository aim to explain how we use microk8s to automate our development workflows.
Kubernetes as a hugely powerful open source container orchestration engine for automating deployment, scaling, and management of containerized applications, becomes maintainable with products like k3s or minikube or microk8s. Microk8s as flavor running on ubuntu is what we decided to take as it's easy to manage and operate, does not need etcd and the serparation of nodes in worker and master roles and follows a setup, start and forget approach. This is exactly what small companies and one man show developers need for building software and following devops paradigmas.

The docs will provide an step by step tutorial on how to setup microk8s with terraform and ansible in hcloud as single node or ha cluster.
On this cluster, a gitlab instance automates the builds of our software projects and deploys them into staging and production environments which are availible for our customers.
A private container registry, the use of industry standard tools like skopeo for container operations or clair for automated vulnerability scanning will be explained, How we use them and what we achieve by utilising them.

There are many providers like Google (GKE), Amazon Web Services (AKS) or Microsoft, to name the most important, that offer managed kubernetes but when you scale out, it quickliy becomes a huge costfactor. The using of microk8s can keep this cost much lower but still provides the possibility to scale out.

This article series is quite lengthy and in-depth and is designed for our company requirements:

The following topics are covered:
* Installation of microk8s on Ubuntu 20.04 LTS in Hcloud
* Automation of the installation with Terraform and Ansible
* Firewall and networking configuration
* SSL selfsigned or by signed letsencrypt
* Multiple domains pointing to applications in the same Kubernetes cluster
* Ingress
* Basic Auth for certain routes
* Running a private container registry
* Seting up a mac to higly integrate this environment

# Goal

A kubernetes environment, that serves all of our customer projects. The web apps are served over https. Any domain name can be configured to point to the services in the cluster. Customers can browse the apps by a given domain and credentials to see always the most recent builds of their product. Gitlab, builds the commited code into oci compliant containers and deploys them straight after the code was commited.

Most of the projects have three to four environments in which they need to be deployed depending on the customers needs. DEV, UAT, INTEG, and PROD are typical cases. When a managed kubernetes or Openshift is used, cost rise up very very fast.
Especially if steps like

* build app
* test app
* unit test app
* build artefact management
* Deploy activities

are part of the pipelines.

The goal is to increase operational efficiency, increase product deployment velocity and increase product quality.

# microk8s

MicroK8s is a single package that enables developers to get a fully featured, conformant and secure Kubernetes system running. Designed for local development, IoT appliances, CI/CD, and use at the edge, MicroK8s is available as a snap package eveloped from canonical.

## Why MicroK8s?

Some advantages of microk8s:

* Reduces cloud vender lock-in as it runs on any Ubuntu box
* Lower cost than a mature kubernetes cluster or products like Openshift
* Mature, easy to install, well supported
* Can scale since 1.19 which other lightweight distributiuons like k3s, minikube, minishift dont do

Some disadvantages of microk8s:

* Not all Kubernetes features are supported (e.g. load balancing)
* Non production deployment configuration may be slightly different to production deployment configuration
* Management of the microk8s installation (patching, disk space, security, etc.)
* Not enterprise grade

For our needs with several clients, small budget and only a couple of applications, the advantages of using microk8s for our dev ops structures and deployments, considerably outweighs any disadvantages.

# Table of contents

1. [Setup of microk8s cluster on hcloud as single node or ha cluster](01-Setup-microk8s.md)

2. [Usage of microk8s cluster for development](02-Using-microk8s.md)