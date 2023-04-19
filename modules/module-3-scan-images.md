# Module 3 - Scan Container Images

## Install the CLI `tigera-scanner` 

1. Download the latest version of the Tigera CLI scanner.

   [Installations Instructions](https://docs.calicocloud.io/image-assurance/scan-image-registries#start-the-cli-scanner)

   Linux
   
   ```bash
   curl -Lo tigera-scanner https://installer.calicocloud.io/tigera-scanner/v3.16.1-0/image-assurance-scanner-cli-linux-amd64
   sudo chmod +x ./tigera-scanner
   sudo mv ./tigera-scanner /usr/local/bin
   tigera-scanner version
   ```

## Pull the images to be scanned

Lets pull two images:

1. Pull the website image website:v1.0.0

   ```bash
   docker pull registry.hub.docker.com/regisftm/website:v1.0.0
   ```

2. Verify the downloaded image.

   ```bash
   docker images
   ```

## Scan the images

1. First, let's scan the images locally, without exporting the results to Calico Cloud.

   ```bash
   tigera-scanner scan registry.hub.docker.com/regisftm/website:v1.0.0
   ```
   
   This command will scan the image and present all the vulnerabilities found on it. However, as we didn't define the threshold for `PASS`, `WARN` or `FAIL` results, the reported `Scan result:` will be `UNKNOWN`.

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
   ...
   [omitted output]
   </pre>

   Scan the image again, but now define the thresholds using --fail_threshold (or -f) and --warn_threshold (or -w)

   ```bash
   tigera-scanner scan registry.hub.docker.com/regisftm/website:v1.0.0 -f 7.9 -w 3.9
   ```
   
   This time you will the the `Scan Result: FAIL`

2. Run the scan again, now exporting the result to the Calico Cloud.

   To export it to the Calico Cloud you will need to get the `apiurl` and `token` information from the Calico Cloud UI. Also check the `Enable Runtime View`.

   Go to Image Assurance > Scan Results > Settings  and copy the API URL and the API TOKEN
   
   ![apiurl](https://user-images.githubusercontent.com/104035488/207679431-02b5a56c-ca10-4fb6-b147-e881bf631cb7.gif)

   Export the values to enviroment variables:

   ```bash
   export APIURL=< paste the api url here! >
   ```

   ```bash
   export APITOKEN=< paste the api token here! >
   ```

   Run the `tigera-scanner` passing the `apiurl` and `token` parameters, so the result will be exported to Calico Cloud.

   ```bash
   tigera-scanner scan registry.hub.docker.com/regisftm/website:v1.0.0 --apiurl $APIURL --token $APITOKEN
   ```

   This is the expected output. 

   <pre>
   $ tigera-scanner scan registry.hub.docker.com/regisftm/website:v1.0.0 --apiurl $APIURL --token $APITOKEN
   INFO[0001] Vulnerability database director not set, setting it to the cache default direct /home/ec2-user/.cache. 
   
    scanning registry.hub.docker.com/regisftm/website:v1.0.0... 
   INFO[0001] Rebuilding dependencies with results from a previous scan of the image. 
   NOTE: Uploading results, this might take a while...
   NOTE: Uploaded vulnerability results for repository path / digest registry.hub.docker.com/regisftm/website:v1.0.   0@sha256:79a9e8505d68fb535fb0d3cfe33425b1876c2a52fb7d180d5f5de86ec2cdd557
   
    Summary: 
   
    Name: registry.hub.docker.com/regisftm/website:v1.0.0
    Digest: sha256:79a9e8505d68fb535fb0d3cfe33425b1876c2a52fb7d180d5f5de86ec2cdd557
    Number of dependencies: 42.
    Total vulnerabilities: 10, critical: 4, high: 6, medium: 0, low: 0, N/A: 0 
   
    Scan result:   ⚠ WARN (warn_threshold - 3.9, fail_threshold - 7.9, Using thresholds from Calico Cloud)  
    </pre>

     Now you can visualize the scan results in the Calico Cloud UI.

--- 

[:arrow_right: Module 4 - Calico Cloud Admission Controller](/modules/module-4-admission-controller.md) <br>

[:arrow_left: Module 2 - Create an EKS cluster and Connect it to Calico Cloud](./modules/module-2-create-eks.md)  
[:leftwards_arrow_with_hook: Back to Main](/README.md)  
