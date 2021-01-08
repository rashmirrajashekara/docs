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

[![isecl-arch](https://github.com/intel-secl/intel-secl/raw/v3.3.1/develop/docs/diagrams/isecl-arch.png)](https://github.com/intel-secl/intel-secl/blob/v3.3.1/develop/docs/diagrams/isecl-arch.png)



## Use Cases

### Foundational Security & Launch Time Protection

Foundational and Workload Security refers to a collection of software security solutions provided by Intel SecL-DC that leverage Intel silicon to provide boot-time integrity attestation of platform components.  Starting with a Hardware Root of Trust, a chain of measurements of system components that includes the system BIOS/UEFI and OS kernel is extended to a Trusted Platform Module for remote attestation against expected measurements.  Use cases include auditing the integrity of Cloud platforms, Asset or Geolocation Tagging, Platform Integrity-aware Cloud orchestration, and VM and container encryption.  This acts as a firm, hardware-rooted foundation upon which to build a Cloud platform with auditable integrity verification.  

[Foundational and Workload Security Product Guide](https://github.com/intel-secl/docs/blob/v3.3.1/develop/product-guides/Product%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Datacenter%20Foundational%20Security.md)

[Foundational & Workload Security Quick Start Guide](https://github.com/intel-secl/docs/blob/v3.3.1/develop/quick-start-guides/Quick%20Start%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Foundational%20%26%20Workload%20Security.md)

[Foundational & Workload Security Swagger Documents](https://github.com/intel-secl/docs/tree/v3.3.1/develop/swagger-docs/foundational-and-workload-security)

### SGX Attestation Infrastructure

The SGX Attestation infrastructure provides an end to end support for registering SGX hosts and provisioning them with SGX material (PCK certificates) and SGX collateral (security patches information - TCB Information - and Certificate Revocation Lists - CRLs). 

The SGX Attestation infrastructure also provides support for generating SGX quotes for SGX enclaves hosted by workloads and verifying them by a remote attesting application. The remote attesting application can also use the SGX Attestation infrastructure to enforce enclave policies (like requiring a specific enclave signer).

Optionally, the SGX Attestation Infrastructure allows to integrate with Cloud Orchestrators like Openstack and Kubernetes. 

The SGX Attestation infrastructure does not make any assumption on the user SGX workload and enclave policy. 

[SGX Attestation Infrastructure and Secure Key Caching Product Guide](https://github.com/intel-secl/docs/blob/v3.3.1/develop/product-guides/Product%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Secure%20Key%20Caching.md)

[SGX Attestation Infrastructure and Secure Key Caching Quick Start Guide](https://github.com/intel-sec/docs/-/blob/v3.3.1/develop/quick-start-guides/Quick%20Start%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Secure%20Key%20Caching.md)

[SGX Attestation Infrastructure and Secure Key Caching Swagger Documents](https://github.com/intel-secl/docs/tree/v3.3.1/develop/swagger-docs/secure-key-caching)

### Secure Key Caching

Secure Key Caching (SKC) leverages the SGX Attestation Infrastructure to support the Secure Key Caching (SKC) use case. 

SKC provides key protection at rest and in-use using Intel Software Guard Extensions (SGX). SGX implements the Trusted Execution Environment (TEE) paradigm.

Using the SKC Client -- a set of libraries -- applications can retrieve keys from the Intel SecL-DC Key Broker Service (KBS) and load them to an SGX-protected memory (called SGX enclave) in the application memory space. KBS performs the SGX enclave attestation to ensure that the application will store the keys in a genuine SGX enclave. The attestation involves the KBS verification of a signed SGX quote generated by the SKC Client. The SGX quote contains the hash of the public key of an enclave generated RSA key pair. 

Application keys are wrapped with a Symmetric Wrapping Key (SWK) by KBS prior to transferring to the SGX enclave. The SWK is generated by KBS and wrapped with the enclave RSA public key, which ensures that the SWK is only known to KBS and the enclave . Consequently, application keys are protected from infrastructure admins, malicious applications and compromised HW/BIOS/OS/VMM. SKC does not require the refactoring of the application because it supports a standard PKCS#11 interface.

[SGX Attestation Infrastructure and Secure Key Caching Product Guide](https://github.com/intel-secl/docs/blob/v3.3.1/develop/product-guides/Product%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Secure%20Key%20Caching.md)

[SGX Attestation Infrastructure and Secure Key Caching Quick Start Guide](https://github.com/intel-secl/docs/blob/v3.3.1/develop/quick-start-guides/Quick%20Start%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Secure%20Key%20Caching.md)

[SGX Attestation Infrastructure and Secure Key Caching Swagger Documents](https://github.com/intel-secl/docs/tree/v3.3.1/develop/swagger-docs/secure-key-caching)

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
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.3.1 -m manifest/all-components.xml
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

## License 

## Support 

## Known Issues
 1. SKC Library adding incorrect entry in /etc/hosts file for kbshostname
 
    Details: This issue is not reproducible with fresh VM. It comes only when we already have some other entry in hosts file with same hostname. It will be fixed in upcoming release.
 2. AAS_URL not updating in authservice.env by skc automation scripts
    
    Details: This issue is not impacting AAS deployment as AAS by default consider it’s own IP if it didn’t find in env file. It will be fixed in upcoming release.
 3. HvsTrustExpiry label should not display in SGX node label
    
    Details: While validating common cluster use case. HvsTrustExpiry label (which is part of TA agent), is displaying under SGX node labels. Although there is no impact of this. It will be fixed in upcoming release.
 4. Single and multiple instances launch from encrypted image failed
    
    Details: As part of Trust Agent, Single and multiple instances launch from encrypted image failing.  It will be fixed in upcoming release.

 5. On openstack compute node and build VM, same OS package repositories should be used to build the components. If there is a mismatch in repos, key transfer flow would fail due to the package mismatch. It will be fixed in the upcoming release.

## Contributing 

## Legalities
