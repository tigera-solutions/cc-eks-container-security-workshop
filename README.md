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

Module 1 - [Prerequisites](/modules/module-1-prereq.md)  
Module 2 - [Create an EKS cluster and Connect it to Calico Cloud](/modules/module-2-create-eks.md)  
Module 3 - [Scan Container Images](/modules/module-3-scan-images.md)  
Module 4 - [Calico Cloud Admission Controller](/modules/module-4-admission-controller.md)  
Module 5 - [Zero-trust access control using identity-aware microsegmentation](/modules/module-5-zero-trust.md)  
Module 6 - [Runtime security with IDS/IPS using Deep Packet Inspection](/modules/module-6-runtime-sec.md)  
Module 7 - [Traffic visualization inside your Kubernetes Cluster](/modules/module-7-visibility.md)  
Module 8 - [Clean up](/modules/module-8-clean-up.md)  

--- 

### Useful links

- [Project Calico](https://www.tigera.io/project-calico/)
- [Calico Academy - Get Calico Certified!](https://academy.tigera.io/)
- [O’REILLY EBOOK: Kubernetes security and observability](https://www.tigera.io/lp/kubernetes-security-and-observability-ebook)
- [Calico Users - Slack](https://slack.projectcalico.org/)

**Follow us on social media**

- [LinkedIn](https://www.linkedin.com/company/tigera/)
- [Twitter](https://twitter.com/tigeraio)
- [YouTube](https://www.youtube.com/channel/UC8uN3yhpeBeerGNwDiQbcgw/)
- [Slack](https://calicousers.slack.com/)
- [Github](https://github.com/tigera-solutions/)
- [Discuss](https://discuss.projectcalico.tigera.io/)

> **Note**: The examples and sample code provided in this workshop are intended to be consumed as instructional content. These will help you understand how Calico Cloud can be configured to build a functional solution. These examples are not intended for use in production environments.


