# AWS Dev Days: Hands-on workshop: <br> Implementing container security on EKS

## Welcome

Enterprises developing compliant cloud-native applications have two primary needs. First, they must secure and govern access to containerized workloads and the Kubernetes environment. Second, they need to simplify audit logging and compliance reporting. The Kubernetes environment is dynamic and distributed, and workloads are ephemeral, making it difficult to enforce compliance controls and provide continuous reporting.

This repo intends to guide you step-by-step on creating an AWS EKS cluster, registering the cluster on Calico Cloud and securing your cloud-native applications. Although Calico Cloud has many functionalities and security components,  this workshop will explore only a few security features used to protect your workload in runtime and deployment time.

## Time Requeriments

The estimated time to complete this workshop is 90-120 minutes.

## Target Audience

- Cloud Professionals
- DevSecOps Professional
- Site Reliability Engineers (SRE)
- Solutions Architects
- Anyone interested in Calico Cloud :)

## Learning Objectives

Learn how to:
- **Scan container images** and **block deployment** based on your security criteria during build time.
- Preview and **enforce security policies** to protect vulnerable workloads.
- Implement **zero-trust access controls** to prevent egress and lateral movement during runtime.
- Implement **runtime security** with IDS/IPS, WAF, and malware detection.
- Get **visibility** into Kubernetes cluster traffic to **troubleshoot** and **improve security**.

## Modules

This workshop is organized in sequencial modules. One module will build up on top of the previous module, so please, follow the order as proposed below.

Module 1 - [Prerequisites](./modules/module-1-prereq.md)  
Module 2 - [Create an EKS cluster and Connect it to Calico Cloud](./modules/module-2-create-eks.md)
Module 4 - Connect the AWS EKS cluster to Calico Cloud  
Module 5 - Create the test environment  
Module 6 - Enable egress gateway support  
Module 7 - Deploy Egress Gateway for a per pod selection  
Module 8 - Deploy Egress Gateway for a per namespace selector  
Module 9 - Deploy Egress Gateway with an AWS elastic IP  
Module 10 - Clean up  

> **Note**: The examples and sample code provided in this workshop are intended to be consumed as instructional content. These will help you understand how Calico Cloud can be configured to build a functional solution. These examples are not intended for use in production environments.


