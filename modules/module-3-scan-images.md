# Module 3: Scan Container Images

## Install the CLI `tigera-scanner` 

1. Download the latest version of the CLI scanner.

   [Installations Instructions](https://docs.calicocloud.io/image-assurance/scan-image-registries#start-the-cli-scanner)

   Linux
   
   ```bash
   curl -Lo -o /tmp/tigera-scanner https://installer.calicocloud.io/tigera-scanner/v3.14.1-11/image-assurance-scanner-cli-linux-amd64
   sudo chmod +x /tmp/tigera-scanner
   sudo mv /tmp/tigera-scanner /usr/local/bin
   tigera-scanner version
   ```

**Pull Images**


1. pull adservice image v0.3.2

```bash
docker image pull tigeralabs.azurecr.io/boutiqueshop-demo/adservice:v0.3.2
```

2. verify the downloaded image.

```bash
docker image list
```

**Scan images**
========================

**Online scan**
> For online scanning you need to provide calico cloud apiurl and token

```bash
./tigera-scanner scan ubuntu:latest --apiurl https://<my-org>.calicocloud.io --token ezBhbGcetc...
```


**Offline scan**

Scan image locally, do not report results

```bash
./tigera-scanner scan tigeralabs.azurecr.io/boutiqueshop-demo/adservice:v0.3.2
```

Scan images with fail and warn thresholds

```bash
./tigera-scanner scan tigeralabs.azurecr.io/boutiqueshop-demo/adservice:v0.3.2 --fail_threshold 7.0 --warn_threshold 3.9 
```

**Enable Runtime scanning**
========================

 ![enable-runtime](../img/enable-runtime.png)

---
[Next -> Module 5](../modules/configure-demo-resources.md)