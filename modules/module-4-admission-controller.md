# Calico Cloud Admission Controller

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
   kubectl apply -f ./tigera-image-assurance-admission-controller-deploy.yaml
   ```

   > The Admission Controller only watches the namespaces it is configured to track. You can configure namespace label via    `IN_NAMESPACE_SELECTOR_KEY` and `IN_NAMESPACE_SELECTOR_VALUES` variables used in the commands above. Explore    `tigera-image-assurance-admission-controller-deploy.yaml` manifest to see how those values are configured.

3. Configure container admission policies.

   The ContainerAdmissionPolicies resources are used to configure policies for Admission Controller.

   Deploy container policies.



   --- 

[:arrow_right: Module 4 - Prerequisites](./modules/module-1-prereq.md) <br>

[:arrow_left: Module 2 - Create an EKS cluster and Connect it to Calico Cloud](./modules/module-2-create-eks.md)  
[:leftwards_arrow_with_hook: Back to Main](/README.md)  
