# Module 5 - Zero-trust access control using identity-aware microsegmentation

A global default deny policy ensures that unwanted traffic (ingress and egress) is denied by default. Pods without policy (or incorrect policy) are not allowed traffic until appropriate network policy is defined. Although the staging policy tool will help you find incorrect and missing policy, a global deny helps mitigate against other lateral malicious attacks.

By default, all traffic is allowed between the pods in a cluster. First, let's test connectivity between application components and across application stacks. All of these tests should succeed as there are no policies in place.

a. Test connectivity between workloads within the same namespace. Use dev as example. The expected result is `HTTP/1.1 200 OK`.

   ```bash
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://nginx-svc 2>/dev/null | grep HTTP'
   ```

b. Test connectivity across namespaces dev/centos and default/frontend. The expected result is `HTTP/1.1 200 OK`.

   ```bash
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://frontend.default 2>/dev/null | grep HTTP'
   ```

c. Test connectivity from dev namespace to the Internet. The expected result is `HTTP/1.1 200 OK`.

   ```bash
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://www.google.com 2>/dev/null | grep HTTP'
   ```

We recommend that you create a global default deny policy after you complete writing policy for the traffic that you want to allow. Use the stage policy feature to get your allowed traffic working as expected, then lock down the cluster to block unwanted traffic.

1. Create a staged global default deny policy. It will shows all the traffic that would be blocked if it were converted into a deny.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: StagedGlobalNetworkPolicy
   metadata:
     name: default-deny
   spec:
     order: 2000
     selector: "projectcalico.org/namespace in {'dev','default'}"
     types:
     - Ingress
     - Egress
   EOF
   ```

   You should be able to view the potential affect of the staged default-deny policy if you navigate to the Dashboard view in the Enterprise Manager UI and look at the Packets by Policy histogram.

   The staged policy does not affect the traffic directly but allows you to view the policy impact if it were to be enforced. You can see the deny traffic in staged policy.


2. Create other network policies to individually allow the traffic shown as blocked in step 1, until no connections are denied.
  
   First, create the policy tiers. Tiers are a hierarchical construct used to group policies and enforce higher precedence policies that cannot be circumvented by other teams. As you will learn in this tutorial, tiers have built-in features that support workload microsegmentation.

   ```yaml
   kubectl apply -f - <<-EOF   
   apiVersion: projectcalico.org/v3
   kind: Tier
   metadata:
     name: security
   spec:
     order: 300
   ---
   apiVersion: projectcalico.org/v3
   kind: Tier
   metadata:
     name: platform
   spec:
     order: 400
   EOF
   ```

   Now Apply network policies to your application with explicity allow and deny control.

   ```yaml
   kubectl apply -f - <<-EOF   
   apiVersion: projectcalico.org/v3
   kind: NetworkPolicy
   metadata:
     name: default.centos
     namespace: dev
   spec:
     tier: default
     order: 800
     selector: app == "centos"
     egress:
     - action: Allow
       protocol: UDP
       destination:
         selector: k8s-app == "kube-dns"
         namespaceSelector: kubernetes.io/metadata.name == "kube-system" 
         ports:
         - '53'
     - action: Allow
       protocol: TCP
       destination:
         selector: app == "nginx"
     types:
       - Egress
   EOF
   ```

   Test connectivity with policies in place.

   a. The only connections between the components within namespaces dev are from centos to nginx, which should be allowed as configured by the policy. The expected result is `HTTP/1.1 200 OK`.

   ```bash
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://nginx-svc 2>/dev/null | grep HTTP'
   ```

   b. The connections across dev/centos pod and default/frontend pod should be blocked by the policy, as it is not explictly allowing this egress traffic. The expected result is `command terminated with exit code 1`.
   
   ```bash   
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://frontend.default 2>/dev/null | grep HTTP'
   ```

   c. Test connectivity from each namespace dev and default to the Internet. The expected result is `command terminated with exit code 1`.
   
   ```bash   
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://www.google.com 2>/dev/null | grep HTTP'
   ```

   Implement a policy to explicit allow egress access from a workload dev/centos to default/frontend.
   
   a. Deploy egress policy between two namespaces dev and default.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: NetworkPolicy
   metadata:
     name: platform.centos-to-frontend
     namespace: dev
   spec:
     tier: platform
     order: 100
     selector: app == "centos"
     egress:
       - action: Allow
         protocol: TCP
         source: {}
         destination:
           selector: app == "frontend"
           namespaceSelector: projectcalico.org/name == "default"
       - action: Pass
     types:
       - Egress
   EOF
   ```

   b. Test connectivity between dev/centos pod and default/frontend service again, should be allowed now. The output should be `HTTP/1.1 200 OK`.

   ```bash   
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://frontend.default 2>/dev/null | grep HTTP'
   ```

   **c.** Deploy the following security policy in the security tier for allowing DNS access to all endpoints. In this way you don't need to create a rule allowing DNS traffic for every policy.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: GlobalNetworkPolicy
   metadata:
     name: security.allow-kube-dns
   spec:
     tier: security
     order: 200
     selector: all()
     types:
     - Egress    
     egress:
       - action: Allow
         protocol: UDP
         source: {}
         destination:
           selector: k8s-app == "kube-dns"
           ports:
           - '53'
       - action: Pass
   EOF
   ```

   Apply the policies to allow the microservices to communicate with each other.

   ```bash
   kubectl apply -f manifests/east-west-traffic.yaml
   ```

3. Use the Calico Cloud GUI to enforce the default-deny staged policy. After enforcing a staged policy, it takes effect immediatelly. The default-deny policy will start to actually deny traffic.
   
---

[:arrow_right: Module 6 - Runtime security with IDS/IPS using Deep Packet Inspection](/modules/module-6-runtime-sec.md)  <br>

[:arrow_left: Module 4 - Calico Cloud Admission Controller](/modules/module-4-admission-controller.md)  
[:leftwards_arrow_with_hook: Back to Main](/README.md) 