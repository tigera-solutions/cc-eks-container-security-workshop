# Module 4 - Calico Cloud Admission Controller

1. Upgrade the OpenSSL from 1.0 to 1.1

   ```bash
   sudo yum -y update
   sudo yum install -y make gcc perl-core pcre-devel wget zlib-devel
   wget https://ftp.openssl.org/source/openssl-1.1.1k.tar.gz
   sudo tar -xzvf openssl-1.1.1k.tar.gz
   cd openssl-1.1.1k
   sudo ./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib no-shared zlib-dynamic
   sudo make
   sudo make install
   openssl version && cd ..
   ```

2. Configure the Admission Controller.

   Calico Cloud uses the Admission Controller to accept or reject resources that create pods based on configured ContainerAdmissionPolicies    rules. For more information refer to Calico Cloud Admission Controller documentation.
   
   Configure the Admission Controller.
   
   ```bash
   # create workdir
   mkdir admission-controller-install && cd admission-controller-install
   # generate certs
   export URL="https://installer.calicocloud.io/manifests/v3.14.1-9/manifests" && curl ${URL}/generate-open-ssl-key-cert-pair.sh | bash
   # generate admission controller manifest
   export URL="https://installer.calicocloud.io/manifests/v3.14.1-9/manifests" && \
   export IN_NAMESPACE_SELECTOR_KEY="apply-container-policies" && \
   export IN_NAMESPACE_SELECTOR_VALUES="true" && \
   curl ${URL}/install-ia-admission-controller.sh | bash
   # install admission controller
   kubectl apply -f ./tigera-image-assurance-admission-controller-deploy.yaml && cd ..
   ```

   > The Admission Controller only watches the namespaces it is configured to track. You can configure namespace label via    `IN_NAMESPACE_SELECTOR_KEY` and `IN_NAMESPACE_SELECTOR_VALUES` variables used in the commands above. Explore    `tigera-image-assurance-admission-controller-deploy.yaml` manifest to see how those values are configured.

3. Configure container admission policies.

   The ContainerAdmissionPolicies resources are used to configure policies for Admission Controller.

   Deploy container policy.

   ```yaml
   kubectl create -f - <<-EOF
   apiVersion: containersecurity.tigera.io/v1beta1
   kind: ContainerAdmissionPolicy
   metadata:
     name: reject-failed-and-non-dockerhub
   spec:
     selector: all()
     namespaceSelector: "apply-container-policies == 'true'"
     order: 10
     rules:
     - action: Allow
       imagePath:
         operator: IsOneOf
         values:
         - "^registry.hub.docker.com/.*"
       imageScanStatus:
         operator: IsOneOf
         values:
         - Pass
         - Warn
       imageLastScan:
         operator: "gt"
         duration:
           days: 7
     - action: Reject
   EOF
   ```
3. Set up alerts on vulnerabilities.

   Deploy an alert to be triggered whenever there is at least one event for an image from the specified registry/ repo that has a max CVSS score greater than 7.9 within the past 30 minutes. Providing control over the exact max CVSS score threshold lets you define a trigger that is different from what the CVSS score threshold is configured for Fail scan result on the Scan Results page in Manager UI.

   ```yaml
   kubectl create -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: GlobalAlert
   metadata:
     name: alarm-website-fail
   spec:
     summary: "Vulnerabilities for a specific repo based on results"
     description: "Vulnerabilities for a specific repo based on results"
     severity: 100
     period: 1m
     lookback: 1m
     dataSet: vulnerability
     query: registry="registry.hub.docker.com/regisftm" AND repository="node" AND result="Fail"
     metric: count
     condition: gt
     threshold: 1
   EOF
   ```

4. Create the namespace `website` adding the label to allow the Admission Controller to watch it.

   ```bash
   kubectl create namespace website
   kubectl label namespace website apply-container-policies=true
   ```

5. Deploy the application to test the enviroment.

   ```bash
   kubectl create -f ./manifests/website.yaml
   ```

6. Look the alert generated for this attempt to deploy a compromised image.

IMAGE HERE !

7. Create the exceptions in the Calico Cloud UI.

   The application will not be allowed to be deployed because the image failed to the scanning process.
   When this happen ideally you should fix the vulneabilities in the image before trying to deploy it again. However we know that this can be a slow and cumbersome process. As a workaround after evaluation the impact of the detected vulnerabilities, you may decide to create **exceptions** for the CVE's in the image, changing its status from `Fail` to `Warn`.

   ![exception](https://user-images.githubusercontent.com/104035488/207643561-ed2eec90-03a8-4fc7-a085-c845121fd21a.gif)

8. Delete the pods and let the replicaset to create a new one.

   ```bash
   kubectl delete pods --all
   ```

9. The image is accepted.



   
   --- 

[:arrow_right: Module 5 - Zero-trust access control using identity-aware microsegmentation](/modules/module-5-zero-trust.md ) <br>

[:arrow_left: Module 3 - Scan Container Images](/modules/module-3-scan-images.md)   
[:leftwards_arrow_with_hook: Back to Main](/README.md)  
