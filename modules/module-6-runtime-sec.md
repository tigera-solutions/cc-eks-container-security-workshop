# Module 6 - Runtime security with IDS/IPS using Deep Packet Inspection

Calico pinpoints the source of malicious activity, uses machine learning to identify anomalies, creates a security moat
around critical workloads, deploys honeypods to capture zero-day attacks, and automatically quarantines potentially
malicious workloads to thwart an attack. It monitors inbound and outbound traffic (north-south) and east-west traffic
that is traversing the cluster environment. Calico provides threat feed integration and custom alerts, and can be
configured to trigger automatic remediation.

### IDS/IPS with Deep Packet Inspection

1. Create the DPI and the Intrusion Detection for the dev/nginx service.
   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: DeepPacketInspection
   metadata:
     name: dpi-nginx
     namespace: dev
   spec:
     selector: app == "nginx"
   ---
   apiVersion: operator.tigera.io/v1
   kind: IntrusionDetection
   metadata:
     name: tigera-secure
   spec:
     componentResources:
     - componentName: DeepPacketInspection
       resourceRequirements:
         limits:
           cpu: "1"
           memory: 1Gi
         requests:
           cpu: 100m
           memory: 100Mi
   EOF
   ```

2. Attack the nginx service

   [Sid 1-21562 - MALWARE-CNC Win.Trojan.Bredolab variant outbound connection](https://www.snort.org/rule_docs/1-21562) 
   ```bash
   kubectl -n dev exec -t netshoot -- sh -c "curl -m2 http://nginx-svc/ -H 'User-Agent: Mozilla/4.0' -XPOST --data-raw 'smk=1234'"
   ```

   [Sid 1-57461 - MALWARE-BACKDOOR Perl.Backdoor.PULSECHECK variant cnc connection](https://www.snort.org/rule_docs/1-57461)
   ```bash
   kubectl -n dev exec -t netshoot -- sh -c "curl -m2 http://nginx-svc/secid_canceltoken.cgi -H 'X-CMD: Test' -H 'X-KEY: Test' -XPOST"
   ```

   [Sid 1-1002 - SERVER-IIS cmd.exe access](https://www.snort.org/rule_docs/1-1002)
   ```bash
   kubectl -n dev exec -t netshoot -- sh -c "curl -m2 http://nginx-svc/cmd.exe"
   ```
   
   [Sid 1-2585 - SERVER-WEBAPP nessus 2.x 404 probe](https://www.snort.org/rule_docs/1-2585)  
   ```bash
   kubectl -n dev exec -t netshoot -- sh -c "curl -m2 http://nginx-svc/NessusTest"
   ```
   
   [Check the Snort ID here!](https://www.snort.org/search)

---

[:arrow_right: Module 7 - Traffic visualization inside your Kubernetes Cluster](/modules/module-7-visibility.md) <br>

[:arrow_left: Module 5 - Zero-trust access control using identity-aware microsegmentation](/modules/module-5-zero-trust.md)  
[:leftwards_arrow_with_hook: Back to Main](/README.md) 