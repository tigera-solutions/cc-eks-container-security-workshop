# Module 5 - Zero-trust access control using identity-aware microsegmentation

A global default deny policy ensures that unwanted traffic (ingress and egress) is denied by default. Pods without policy (or incorrect policy) are not allowed traffic until appropriate network policy is defined. Although the staging policy tool will help you find incorrect and missing policy, a global deny helps mitigate against other lateral malicious attacks.

By default, all traffic is allowed between the pods in a cluster. First, let's test connectivity between application components and across application stacks. All of these tests should succeed as there are no policies in place.

a. Test connectivity between workloads within each namespace, use dev and default namespaces as example

   ```bash
   # test connectivity within dev namespace, the expected result is "HTTP/1.1 200 OK" 
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://nginx-svc 2>/dev/null | grep -i http'
   ```

   ```bash
   # test connectivity within default namespace in 8080 port
   kubectl exec -it $(kubectl -n default get po -l app=frontend -ojsonpath='{.items[0].metadata.name}') \
   -c server -- sh -c 'nc -zv recommendationservice 8080'
   ```

b. Test connectivity across namespaces dev/centos and default/frontend.

   ```bash
   # test connectivity from dev namespace to default namespace, the expected result is "HTTP/1.1 200 OK"
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://frontend.default 2>/dev/null | grep -i http'
   ```

c. Test connectivity from each namespace dev and default to the Internet.

   ```bash
   # test connectivity from dev namespace to the Internet, the expected result is "HTTP/1.1 200 OK"
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://www.google.com 2>/dev/null | grep -i http'
   ```

   ```bash
   # test connectivity from default namespace to the Internet, the expected result is "HTTP/1.1 200 OK"
   kubectl exec -it $(kubectl get po -l app=loadgenerator -ojsonpath='{.items[0].metadata.name}') \
   -c main -- sh -c 'curl -m3 -sI http://www.google.com 2>/dev/null | grep -i http'
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

   ```bash
   # make a request across namespaces and view Packets by Policy histogram, the expected result is "HTTP/1.1 200 OK"
   for i in {1..5}; do kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://frontend.default 2>/dev/null | grep -i http'; sleep 2; done
   ```

   The staged policy does not affect the traffic directly but allows you to view the policy impact if it were to be enforced. You can see the deny traffic in staged policy.


2. Create other network policies to individually allow the traffic shown as blocked in step 1, until no connections are denied.
  
   Apply network policies to your application with explicity allow and deny control.

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
       protocol: TCP
       destination:
         selector: app == "nginx"
     types:
       - Egress
   EOF
   ```

   Test connectivity with policies in place.

   a. The only connections between the components within namespaces dev are from centos to nginx, which should be allowed as configured by the policies.

   ```bash
   # test connectivity within dev namespace, the expected result is "HTTP/1.1 200 OK"
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://nginx-svc 2>/dev/null | grep -i http'
   ```
   
   The connections within namespace default should be allowed as usual.
   
   ```bash
   # test connectivity within default namespace in 8080 port
   kubectl exec -it $(kubectl get po -l app=frontend -ojsonpath='{.items[0].metadata.name}') \
   -c server -- sh -c 'nc -zv recommendationservice 8080'
   ``` 

   b. The connections across dev/centos pod and default/frontend pod should be blocked by the application policy.
   
   ```bash   
   # test connectivity from dev namespace to default namespace, the expected result is "command terminated with exit code 1"
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://frontend.default 2>/dev/null | grep -i http'
   ```

   c. Test connectivity from each namespace dev and default to the Internet.
   
   ```bash   
   # test connectivity from dev namespace to the Internet, the expected result is "command terminated with exit code 1"
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://www.google.com 2>/dev/null | grep -i http'
   ```
   
   ```bash
   # test connectivity from default namespace to the Internet, the expected result is "HTTP/1.1 200 OK"
   kubectl exec -it $(kubectl get po -l app=loadgenerator -ojsonpath='{.items[0].metadata.name}') \
   -c main -- sh -c 'curl -m3 -sI http://www.google.com 2>/dev/null | grep -i http'
   ```

   Implement explicitic policy to allow egress access from a workload in one namespace/pod, e.g. dev/centos, to default/frontend.
   
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

   b. Test connectivity between dev/centos pod and default/frontend service again, should be allowed now.

   ```bash   
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://frontend.default 2>/dev/null | grep -i http'
   #output is HTTP/1.1 200 OK
   ```

   Apply the policies to allow the microservices to communicate with each other.

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/regismartins/cc-aks-security-compliance-workshop/main/manifests/east-west-traffic.yaml
   ```

3. Use the Calico Cloud GUI to enforce the default-deny staged policy. After enforcing a staged policy, it takes effect immediatelly. The default-deny policy will start to actually deny traffic.
   
---

[:arrow_right: Module 6 - Runtime security with IDS/IPS using Deep Packet Inspection](/modules/module-6-runtime-sec.md)  <br>

[:arrow_left: Module 4 - Calico Cloud Admission Controller](/modules/module-4-admission-controller.md)  
[:leftwards_arrow_with_hook: Back to Main](/README.md) 