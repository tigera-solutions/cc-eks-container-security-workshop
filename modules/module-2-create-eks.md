# Create an EKS cluster and Connect it to Calico Cloud

Calico can be used as a CNI, or you can decide to use AWS VPC networking and have Calico only as plugin for the security policies. 

We will use the second approach during this workshop. Below an example on how to create a two nodes cluster with an smaller footprint, but feel free to create your EKS cluster with the parameters you prefer. Do not forget to include the region if different than the default on your account.

```bash
export CLUSTERNAME=tigera-demo
export REGION=ca-central-1
eksctl create cluster --name $CLUSTERNAME --version 1.22  --region $REGION --node-type m5.xlarge
```

- View EKS cluster.

  Once cluster is created you can list it using eksctl.
  
  ```bash
  eksctl get cluster tigera-workshop
  ```

- Test access to EKS cluster with kubectl

  Once the EKS cluster is provisioned with eksctl tool, the kubeconfig file would be placed into ~/.kube/config path. The kubectl CLI looks for kubeconfig at ~/.kube/config path or into KUBECONFIG env var.

  ```bash
  # verify kubeconfig file path
  ls ~/.kube/config
  # test cluster connection
  kubectl get nodes
  ```
  
## Connect your cluster to Calico Cloud

Subscribe to the free Calico Cloud trial on the link below:

https://www.calicocloud.io/home

Once you are able to login to Calico Cloud UI, go to the "Managed clusters" section, and click on the "Connect Cluster" button, then leave "Amazon EKS" selected, and give a name to your cluster, and click "Next". Read the cluster requirements in teh next section, and click "Next". Finally, copy the kubectl command you must run in order to connect your cluster to the management cluster for your Calico Cloud instance.

![managed-clusters](https://user-images.githubusercontent.com/104035488/206290672-8af9c13f-314a-4752-8cb8-ff1b892484d6.png)

---

## Enviroment Preparation

### Clone this repository

```bash
git clone https://github.com/tigera-solutions/cc-eks-container-security-workshop
```

### Decrease the time to collect flow logs

By default, flow logs are collected every 5 minutes. We will decrease that time to 15 seconds, which will increase the amount of information we must store, and while that is not recommended for production environments, it will help to speed up the time in which events are seen within Calico observability features.

```bash
kubectl patch felixconfiguration default -p '{"spec":{"flowLogsFlushInterval":"15s"}}'
kubectl patch felixconfiguration default -p '{"spec":{"dnsLogsFlushInterval":"15s"}}'
kubectl patch felixconfiguration default -p '{"spec":{"flowLogsFileAggregationKindForAllowed":1}}'
kubectl patch felixconfiguration default -p '{"spec":{"flowLogsFileAggregationKindForDenied":0}}'
kubectl patch felixconfiguration default -p '{"spec":{"dnsLogsFileAggregationKind":0}}'
```

Configure Felix to collect TCP stats - this uses eBPF TC program and requires miniumum Kernel version of v5.3.0/v4.18.0-193. Further documentation.

```bash
kubectl patch felixconfiguration default -p '{"spec":{"flowLogsCollectTcpStats":true}}'
```

### Install demo applications

Deploy the dev app stack

```bash
kubectl apply -f ./manifests/dev-app-manifest.yaml
```
 
Deploy the Online Boutique app stack

```bash
kubectl apply -f ./manifests/kubernetes-manifests.yaml
```

--- 

[:arrow_right: Module 1 - Prerequisites](./modules/module-1-prereq.md) <br>

[:arrow_left: Module 3 - ](/modules/module-3-deploy-eks.md)  
[:leftwards_arrow_with_hook: Back to Main](/README.md)  