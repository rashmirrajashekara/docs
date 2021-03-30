# Intel® Security Libraries for Data Center (Intel® SecL-DC)

Table of Contents
=================

   * [Intel® Security Libraries for Data Center (Intel® SecL-DC)](#intel-security-libraries-for-data-center-intel-secl-dc)
     * [Overview](#overview)
     * [Architecture](#architecture)
     * [Use Cases](#use-cases)
       * [Foundational Security &amp; Launch Time Protection](#foundational-security--launch-time-protection)
       * [SGX Attestation Infrastructure](#sgx-attestation-infrastructure)
       * [Secure Key Caching](#secure-key-caching)
     * [Building all components](#building-all-components)
       * [Pre-requisites](#pre-requisites)
       * [Building](#building)
     * [Common Cluster Setup and Validation](#common-cluster-setup-and-validation)
       * [Setup K8S Cluster and Deploy Isecl-k8s-extensions](#setup-k8s-cluster-and-deploy-isecl-k8s-extensions)
       * [Untar packages and push OCI images to registry](#untar-packages-and-push-oci-images-to-registry)
       * [Deploy isecl-controller](#deploy-isecl-controller)
       * [Deploy ihub](#deploy-ihub)
       * [Deploy isecl-scheduler](#deploy-isecl-scheduler)
       * [Configure kube-scheduler to establish communication with isecl-scheduler](#configure-kube-scheduler-to-establish-communication-with-isecl-scheduler)
       * [Validate POD Launch](#validate-pod-launch)
     * [License](#license)
     * [Support](#support)
     * [Known Issues](#known-issues)
     * [Contributing](#contributing)
     * [Legalities](#legalities)


## Overview

Intel® Security Libraries for Data Center (Intel® SecL-DC) enables security use cases for data center using Intel® hardware security technologies.

Hardware-based cloud security solutions provide a higher level of protection as compared to software-only security measures. There are many Intel platform security technologies, which can be used to secure customers' data. Customers have found adopting and deploying these technologies at a broad scale challenging, due to the lack of solution integration and deployment tools. Intel® Security Libraries for Data Centers (Intel® SecL - DC) was built to aid our customers in adopting and deploying Intel Security features, rooted in silicon, at scale.

Intel® SecL-DC is an open-source remote attestation implementation comprising a set of building blocks that utilize Intel Security features to discover, attest, and enable critical foundation security and confidential computing use-cases. It applies the remote attestation fundamentals and standard specifications to maintain a platform data collection service, and an efficient verification engine to perform comprehensive trust evaluations. These trust evaluations can be used to govern different trust and security policies applied to any given workload.

For more details please visit : https://01.org/intel-secl

## Architecture

The below diagram depicts the high level architecture of the Intel®SecL-DC,

[![isecl-arch](https://github.com/intel-secl/intel-secl/raw/v3.5.0/docs/diagrams/isecl-arch.png)](https://github.com/intel-secl/intel-secl/raw/v3.5.0/docs/diagrams/isecl-arch.png)



## Use Cases

### Foundational Security & Launch Time Protection

Foundational and Workload Security refers to a collection of software security solutions provided by Intel SecL-DC that leverage Intel silicon to provide boot-time integrity attestation of platform components.  Starting with a Hardware Root of Trust, a chain of measurements of system components that includes the system BIOS/UEFI and OS kernel is extended to a Trusted Platform Module for remote attestation against expected measurements.  Use cases include auditing the integrity of Cloud platforms, Asset or Geolocation Tagging, Platform Integrity-aware Cloud orchestration, and VM and container encryption.  This acts as a firm, hardware-rooted foundation upon which to build a Cloud platform with auditable integrity verification.  

[Foundational and Workload Security Product Guide](https://github.com/intel-secl/docs/blob/v3.5/develop/product-guides/Product%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Datacenter%20Foundational%20Security.md)

[Foundational & Workload Security Quick Start Guide](https://github.com/intel-secl/docs/blob/v3.5/develop/quick-start-guides/Quick%20Start%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Foundational%20%26%20Workload%20Security.md)

[Foundational & Workload Security Swagger Documents](https://github.com/intel-secl/docs/tree/v3.5/develop/swagger-docs/foundational-and-workload-security)

### SGX Attestation Infrastructure

The SGX Attestation infrastructure provides an end to end support for registering SGX hosts and provisioning them with SGX material (PCK certificates) and SGX collateral (security patches information - TCB Information - and Certificate Revocation Lists - CRLs). 

The SGX Attestation infrastructure also provides support for generating SGX quotes for SGX enclaves hosted by workloads and verifying them by a remote attesting application. The remote attesting application can also use the SGX Attestation infrastructure to enforce enclave policies (like requiring a specific enclave signer).

Optionally, the SGX Attestation Infrastructure allows to integrate with Cloud Orchestrators like Openstack and Kubernetes. 

The SGX Attestation infrastructure does not make any assumption on the user SGX workload and enclave policy. 

[SGX Attestation Infrastructure and Secure Key Caching Product Guide](https://github.com/intel-secl/docs/blob/v3.5/develop/product-guides/Product%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Secure%20Key%20Caching.md)

[SGX Attestation Infrastructure and Secure Key Caching Quick Start Guide](https://github.com/intel-secl/docs/blob/v3.5/develop/quick-start-guides)

[SGX Attestation Infrastructure and Secure Key Caching Swagger Documents](https://github.com/intel-secl/docs/tree/v3.5/develop/swagger-docs/secure-key-caching)

### Secure Key Caching

Secure Key Caching (SKC) leverages the SGX Attestation Infrastructure to support the Secure Key Caching (SKC) use case. 

SKC provides key protection at rest and in-use using Intel Software Guard Extensions (SGX). SGX implements the Trusted Execution Environment (TEE) paradigm.

Using the SKC Client -- a set of libraries -- applications can retrieve keys from the Intel SecL-DC Key Broker Service (KBS) and load them to an SGX-protected memory (called SGX enclave) in the application memory space. KBS performs the SGX enclave attestation to ensure that the application will store the keys in a genuine SGX enclave. The attestation involves the KBS verification of a signed SGX quote generated by the SKC Client. The SGX quote contains the hash of the public key of an enclave generated RSA key pair. 

Application keys are wrapped with a Symmetric Wrapping Key (SWK) by KBS prior to transferring to the SGX enclave. The SWK is generated by KBS and wrapped with the enclave RSA public key, which ensures that the SWK is only known to KBS and the enclave . Consequently, application keys are protected from infrastructure admins, malicious applications and compromised HW/BIOS/OS/VMM. SKC does not require the refactoring of the application because it supports a standard PKCS#11 interface.

[SGX Attestation Infrastructure and Secure Key Caching Product Guide](https://github.com/intel-secl/docs/blob/v3.5/develop/product-guides/Product%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Secure%20Key%20Caching.md)

[SGX Attestation Infrastructure and Secure Key Caching Quick Start Guide](https://github.com/intel-secl/docs/blob/v3.5/develop/quick-start-guides/)

[SGX Attestation Infrastructure and Secure Key Caching Swagger Documents](https://github.com/intel-secl/docs/tree/v3.5/develop/swagger-docs/secure-key-caching)

## Building all components

The Quick start guides facilitate building components for Foundational Security, Workload Security and Secure Key Caching individually. However, if need be, all components can be built together on a single Build Machine.

Following steps facilitate the building of all components:

### Pre-requisites

* RHEL 8.2 machine

* `rhel-8-for-x86_64-baseos-rpms` and `rhel-8-for-x86_64-appstream-rpms` and `codeready-builder-for-rhel-8-x86_64-rpms` repositories need to be enabled on the OS

* The services need to be built and installed as `root` user. Ensure root privileges are present for the user to work with Intel® SecL-DC. All Intel® SecL-DC service & agent ports should be allowed in firewall rules.

* Install basic utilities for getting started

  ```shell
  dnf install git wget tar python3 yum-utils
  ```

* Create symlink for python3

  ```shell
  ln -s /usr/bin/python3 /usr/bin/python
  ln -s /usr/bin/pip3 /usr/bin/pip
  ```

* Install repo tool

  ```shell
  tmpdir=$(mktemp -d)
  git clone https://gerrit.googlesource.com/git-repo $tmpdir
  install -m 755 $tmpdir/repo /usr/local/bin
  rm -rf $tmpdir
  ```

* Extract Install `go` version > `go1.13` & <= `go1.14.4` from `https://golang.org/dl/` and set `GOROOT` & `PATH`

  ```
  export GOROOT=<path_to_go>
  export PATH=$GOROOT/bin:$PATH
  ```

### Building

* Sync the repo

  ```shell
  mkdir -p /root/isecl/all && cd /root/isecl/all
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.5.0 -m manifest/all-components.xml
  repo sync
  ```

* Run the `pre-requisites` setup script

  ```shell
  cd utils/build/
  chmod +x all-components.sh
  ./all-components.sh
  ```

* Download go dependencies

  ```shell
  cd /root/
  go get github.com/cpuguy83/go-md2man
  mv /root/go/bin/go-md2man /usr/bin/
  ```

* Ignore the below steps if not running behind a proxy

  ```shell
  mkdir -p /etc/systemd/system/docker.service.d
  touch /etc/systemd/system/docker.service.d/proxy.conf
  
  #Add the below lines in proxy.conf
  [Service]
  Environment="HTTP_PROXY=<http_proxy>"
  Environment="HTTPS_PROXY=<https_proxy>"
  Environment="NO_PROXY=<no_proxy>"
  #Reload docker
  systemctl daemon-reload
  systemctl restart docker
  ```

* Build all repos

  ```shell
  cd /root/isecl/all
  make all
  ```
  
* Built Binaries

  ```shell
  /root/isecl/all/binaries/
  ```

  

> **Note:** Post successful build, the  deployment and usecase collections can be followed from Quick start guides posted above


## Common Cluster Setup and Validation

### Setup K8S Cluster and Deploy Isecl-k8s-extensions

* Setup master and worker node for k8s. Worker node should be setup on SGX enabled host machine. Master node can be any system.

* To setup k8 cluster follow https://phoenixnap.com/kb/how-to-install-kubernetes-on-centos Once the master/worker setup is done, follow below steps on Master Node:

### Untar packages and push OCI images to registry

* Copy tar output isecl-k8s-extensions-*.tar.gz from build system's binaries folder to /opt/ directory on the Master Node and extract the contents.
  
  ```shell
    cd /opt/
    tar -xvzf isecl-k8s-extensions-*.tar.gz
    cd isecl-k8s-extensions/
  ```
  
* Configure private registry

* Push images to private registry using skopeo command, (this can be done from build vm also)
  
  ```shell
     skopeo copy oci-archive:isecl-k8s-controller-v3.5.0-<commitid>.tar docker://<registryIP>:<registryPort>/isecl-k8s-controller:v3.5.0
     skopeo copy oci-archive:isecl-k8s-scheduler-v3.5.0-<commitid>.tar docker://<registryIP>:<registryPort>/isecl-k8s-scheduler:v3.5.0
  ```
  
* Add the image names in isecl-controller.yml and isecl-scheduler.yml in /opt/isecl-k8s-extensions/yamls with full image name including registry IP/hostname (e.g <registryIP>:<registryPort>/isecl-k8s-scheduler:v3.5.0). It will automatically pull the images from registry.
  
### Deploy isecl-controller

* Create hostattributes.crd.isecl.intel.com crd
  
  ```shell
    kubectl apply -f yamls/crd-1.17.yaml
  ```
  
* Check whether the crd is created

  ```shell  
    kubectl get crds
  ```
  
* Deploy isecl-controller
  
  ```shell
    kubectl apply -f yamls/isecl-controller.yaml
  ```
  
* Check whether the isecl-controller is up and running
  
  ```shell
    kubectl get deploy -n isecl
  ```
  
* Create clusterrolebinding for ihub to get access to cluster nodes
  
  ```shell
    kubectl create clusterrolebinding isecl-clusterrole --clusterrole=system:node --user=system:serviceaccount:isecl:isecl
  ```
  
* Fetch token required for ihub installation and follow below IHUB installation steps,
  
  ```shell
    kubectl get secrets -n isecl
    kubectl describe secret default-token-<name> -n isecl
  ```
  
### Deploy ihub

For IHUB installation, make sure to update below configuration in /root/binaries/env/ihub.env before installing ihub on both CSP systems (SGX and FS):

* Copy /etc/kubernetes/pki/apiserver.crt from master node to /root on CSP systems. Update KUBERNETES_CERT_FILE.

* Get k8s token in master, using above commands and update KUBERNETES_TOKEN

* Update the value of CRD name

  ```shell
	KUBERNETES_CRD=custom-isecl-sgx (On SGX CSP)
	KUBERNETES_CRD=custom-isecl-hvs (On FS CSP)
  ```
  
* Deploy ihub for both SGX and FS

* After iHub deployment, copy /etc/ihub/ihub_public_key.pem from both ihub systems to /opt/isecl-k8s-extensions/ directory on k8 master system and prefix the attestation type for ihub_public_key.pem

* Follow below steps on k8 master node

  ```shell
    scp <user>@<SGX iHub host>:/etc/ihub/ihub_public_key.pem /opt/isecl-k8s-extensions/
    mv /opt/isecl-k8s-extensions/ihub_public_key.pem /opt/isecl-k8s-extensions/sgx_ihub_public_key.pem
    scp <user>@<FS iHub host>:/etc/ihub/ihub_public_key.pem /opt/isecl-k8s-extensions/
    mv /opt/isecl-k8s-extensions/ihub_public_key.pem /opt/isecl-k8s-extensions/hvs_ihub_public_key.pem
  ```
  
### Deploy isecl-scheduler

* The isecl-scheduler default configuration is provided for common cluster support in /opt/isecl-k8s-extensions/yamls/isecl-scheduler.yaml. Variables HVS_IHUB_PUBLIC_KEY_PATH and SGX_IHUB_PUBLIC_KEY_PATH are by default set to default paths. 
 
* Install cfssl and cfssljson on Kubernetes Control Plane

  ```shell
    wget -O /usr/local/bin/cfssl http://pkg.cfssl.org/R1.2/cfssl_linux-amd64
    chmod +x /usr/local/bin/cfssl
    wget -O /usr/local/bin/cfssljson http://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
    chmod +x /usr/local/bin/cfssljson
  ```
  
* Create tls key pair for isecl-scheduler service, which is signed by k8s apiserver.crt
  
  ```shell
    cd /opt/isecl-k8s-extensions/
    chmod +x create_k8s_extsched_cert.sh
    ./create_k8s_extsched_cert.sh -n "K8S Extended Scheduler" -s "<K8_MASTER_IP>","<K8_MASTER_HOST>" -c /etc/kubernetes/pki/ca.crt -k /etc/kubernetes/pki/ca.key
  ```
  
* Copy tls key pair generated in previous step to secrets directory.

  ```shell
    mkdir secrets
    cp /opt/isecl-k8s-extensions/server.key secrets/
    cp /opt/isecl-k8s-extensions/server.crt secrets/
    mv /opt/isecl-k8s-extensions/sgx_ihub_public_key.pem secrets/
    mv /opt/isecl-k8s-extensions/hvs_ihub_public_key.pem secrets/
  ```
  
* Create kubernetes secrets scheduler-secret for isecl-scheduler
  
  ```shell
    kubectl create secret generic scheduler-certs --namespace isecl --from-file=secrets
  ```
  
* Deploy isecl-scheduler

  ```shell
    kubectl apply -f yamls/isecl-scheduler.yaml
  ```
  
* Check whether the isecl-scheduler is up and running

  ```shell
    kubectl get deploy -n isecl
  ```
  
### Configure kube-scheduler to establish communication with isecl-scheduler

* Add scheduler-policy.json under kube-scheduler section, mountPath under container section and hostPath under volumes section in /etc/kubernetes/manifests/kube-scheduler.yaml as mentioned below

```shell
spec:
  containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json
  
  containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
  
  volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched
```

Note: Make sure to use proper indentation and don't delete existing mountPath and hostPath sections in kube-scheduler.yaml.

* Restart Kubelet which restart all the k8s services including kube base schedular
  
  ```shell
	systemctl restart kubelet
  ```
  
* Check if CRD data is populated
  
  ```shell
	kubectl get -o json hostattributes.crd.isecl.intel.com
	kubectl get nodes --show-labels
  ```
  
### Validate POD Launch

* Create sample nginx.yml file for SGX workload and add SGX labels to it such as:

```shell
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  affinity:
    nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: SGX-Enabled
           operator: In
           values:
           - "true"
         - key: EPC-Memory
           operator: In
           values:
           - "2.0GB"
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

* Create sample trusted-enc-asset-tag-1.yml file for FS workload

* Launch both the PODs from k8 master node

  ```shell
    kubectl apply -f nginx.yml (For SGX)
    kubectl apply -f deployments/trusted-enc-asset-tag-1.yml (For FS)
  ```
  
* Validate if pod can be launched on the node. Run following commands:

  ```shell
    kubectl get pods
    kubectl describe pods nginx (For SGX)
    kubectl describe pods trusted-enc-assettag1-deployment-xx (For FS)
  ```
  
Pod should be in running state and launched on the worker host as per values in yml. 

* Validate running below commands on both worker nodes:
  
  ```shell
	docker ps
  ```


## License 

## Support 

 1. To get SGX Attributes from a SGX ECDSA Quote, use the following command, once SKC library component is built on the build system
   
   ```shell
   /opt/intel/sgxsdk/bin/x64/sgx_sign dump -enclave /opt/intel/cryptoapitoolkit/lib/libp11SgxEnclave.signed.so -dumpfile info.txt
   ```
   
   ```shell
   From info.txt
   metadata->enclave_css.body.enclave_hash.m value represents the MRENCLAVE value of CTK enclave
   mrsigner->value represents MRSIGNER
   metadata->enclave_css.body.isv_prod_id shows the PROD_ID
   metadata->enclave_css.body.isv_svn show ISVSVN and so on
   ```

   These values can be used to cross verify against the ones in Key transfer policy.


## Known Issues

 1. SKC workflow is not supported on openstack setup. So end to end SKC key transfer flow will not work with openstack.

## Contributing 

## Legalities
