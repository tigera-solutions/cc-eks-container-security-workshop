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

## Pull the images to be scanned

Lets pull two images:

1. Pull the website images website:v1.0.0 and website:v1.1.0

   ```bash
   docker pull registry.hub.docker.com/regisftm/website:v1.0.0
   docker pull registry.hub.docker.com/regisftm/website:v1.1.0
   ```

2. Verify the downloaded image.

   ```bash
   docker image
   ```

## Scan the images

1. First, let's scan the images locally, without exporting the results to Calico Cloud

   ```bash
   tigera-scanner scan registry.hub.docker.com/regisftm/website:v1.0.0
   ```
   
   This command will scan the image and present all the vulnerabilities found on it. However, as we didn't define the threshold for `PASS`, `WARN` or `FAIL` results, the reported result will be `UNKNOWN`

   <pre>
   $ tigera-scanner scan registry.hub.docker.com/regisftm/website:v1.0.0
   INFO[0000] Vulnerability database director not set, setting it to the cache default direct /home/ec2-user/.cache. 
   
    scanning registry.hub.docker.com/regisftm/website:v1.0.0... 
   
    Summary: 
   
    Name: registry.hub.docker.com/regisftm/website:v1.0.0
    Digest: 
    Number of dependencies: 42.
    Total vulnerabilities: 10, critical: 4, high: 6, medium: 0, low: 0, N/A: 0 
   
    Scan result:   UNKNOWN Please set fail_threshold(-f), warn_threshold(-w) for a scan result. 
   +------------+----------+----------------+------+--------------------------------+----------------------+------------------------------------------------------------------------------------------+
   | DEPENDENCY | SEVERITY |     CVE-ID     | CVSS |          DESCRIPTION           |      FIX RESULT      |                                        REFERENCES                                        |
   +------------+----------+----------------+------+--------------------------------+----------------------+------------------------------------------------------------------------------------------+
   | curl       | Critical | CVE-2022-32221 |  9.8 | When doing HTTP(S) transfers,  | fixed in [7.83.1-r4] | https://hackerone.com/reports/   1704017                                                 |
   |            |          |                |      | libcurl might erroneously      |                      |       
   </pre>



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