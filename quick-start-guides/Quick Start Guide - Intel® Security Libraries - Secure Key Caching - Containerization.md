# Quick start Guide - SGX Containerization 

Table of Contents
=================

   * [Quick start Guide - SGX Containerization](#quick-start-guide---sgx-containerization)
      * [Hardware &amp; OS Requirements](#hardware--os-requirements)
      * [Network Requirements](#network-requirements)
      * [RHEL Package Requirements](#rhel-package-requirements)
      * [Deployment Model](#deployment-model)
         * [Single Node](#single-node)
         * [Multi Node](#multi-node)
      * [Build](#build)
         * [Pre-requisites](#pre-requisites)
            * [System Tools and Utilities](#system-tools-and-utilities)
            * [Repo tool](#repo-tool)
            * [Golang](#golang)
            * [Docker](#docker)
            * [Skopeo](#skopeo)
            * [Libkmip for KBS](#libkmip-for-kbs)
         * [Build OCI Container images and K8s Manifests](#build-oci-container-images-and-k8s-manifests)
            * [Single Node](#single-node-1)
            * [Multi Node](#multi-node-1)
      * [Deployment](#deployment)
         * [Pre-requisites](#pre-requisites-1)
         * [Deploy](#deploy)
            * [Single-Node](#single-node-2)
               * [Pre-requisites](#pre-requisites-2)
               * [Deploy steps](#deploy-steps)
                  * [Update .env file](#update-env-file)
                  * [Run scripts on K8s master](#run-scripts-on-k8s-master)
            * [Multi-Node](#multi-node-2)
               * [Pre-requisites](#pre-requisites-3)
               * [Deploy steps](#deploy-steps-1)
                  * [Update .env file](#update-env-file-1)
                  * [Run scripts on K8s master](#run-scripts-on-k8s-master-1)
      * [Usecase Workflows API Collections](#usecase-workflows-api-collections)
         * [Pre-requisites](#pre-requisites-4)
         * [Use Case Collections](#use-case-collections)
         * [Downloading API Collections](#downloading-api-collections)
         * [Running API Collections](#running-api-collections)

## Hardware & OS Requirements

1. **Machines**

   * RHEL 8.2 Build Machine

   * K8S Master Node Setup on CSP (VMs/Physical Nodes + SGX enabled Physical Nodes)

   * K8S Master Node Setup on Enterprise (VMs/Physical Nodes)

   > **Note:** The supported K8s distributions are `microk8s(single-node)` & `kubeadm(multi-node)` K8s distributions

2. **OS Requirements**

   * RHEL 8.2 for build

   * RHEL 8.2 or Ubuntu 18.04 for deployments
   
     >  **Note:** SKC Solution is built, installed and tested with root privileges. Please ensure that all the following instructions are executed with root privileges
   
3. **Container Runtime**

   * Docker
   
4. **Storage**

   * `hostPath` in case of single-node `microk8s`
   * `NFS` in case of multi-node `kubeadm`



## Network Requirements

1. **Build System**

   Internet access required

2. **CSP Managed Services**

   Internet access required for SGX Caching Service deployed on CSP system/SGX Compute Node;

3. **Enterprise Managed Services**

   Internet access required for SGX Caching Service deployed on Enterprise system;

4. **SGX Enabled Host**

   Internet access required to access KBS running on Enterprise environment

**Firewall Settings**

Ensure that all the SKC service ports are accessible with firewall



## RHEL Package Requirements

Access required for the following packages in all systems

1. **BaseOS**

2. **Appstream**

3. **CodeReady**

   

## Deployment Model

### Single Node

The single Node uses `**microk8s**` as a supported K8s distribution

> TODO: Praveen to provide diagram

### Multi Node

The multi node supports `**kubeadm**` as a supported K8s distribution

![K8s Deployment-sqx](./images/k8s-deployment-sgx.jpg)

## Build

### Pre-requisites

The below steps need to be done on RHEL 8.2 Build machine (VM/Physical Node)

#### System Tools and Utilities

```shell
dnf install git wget tar python3 gcc gcc-c++ zip tar make yum-utils openssl-devel
dnf install https://dl.fedoraproject.org/pub/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/makeself-2.4.0-5.fc32.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip
```

#### Repo tool

```shell
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir
```

#### Golang

```shell
wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
tar -xzf go1.14.4.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.14.1.linux-amd64.tar.gz
```

#### Docker

```shell
dnf module enable container-tools
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce-19.03.13 docker-ce-cli-19.03.13

systemctl enable docker
systemctl start docker

#Ignore the below steps if not running behind a proxy
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/proxy.conf

#Add the below lines in proxy.conf
[Service]
Environment="HTTP_PROXY=<http_proxy>"
Environment="HTTPS_PROXY=<https_proxy>"
Environment="NO_PROXY=<no_proxy>"

systemctl daemon-reload
systemctl restart docker
```

#### Skopeo

```shell
dnf install skopeo
```

#### Libkmip for KBS

```shell
git clone https://github.com/openkmip/libkmip.git
cd libkmip
make install
```

### Build OCI Container images and K8s Manifests

The build process for OCI containers images and K8s manifests for RHEL 8.2 & Ubuntu 18.04 deployments must be done on RHEL 8.2 machine only

#### Single Node

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/skc-k8s-single-node && cd /root/intel-secl/build/skc-k8s-single-node
  repo init -u https://github.com/intel-secl/build-manifest.git -m manifest/skc-k8s-single-node.xml -b refs/tags/v3.5.0
  repo sync
  ```

* Build

  ```shell
  make all
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/skc-k8s-single-node/k8s/
  ```

#### Multi Node

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/skc-k8s-multi-node && cd /root/intel-secl/build/skc-k8s-multi-node
  repo init -u https://github.com/intel-secl/build-manifest.git -m manifest/skc-k8s-multi-node.xml -b refs/tags/v3.5.0
  repo sync
  ```

* Build

  ```shell
  make all
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/skc-k8s-multi-node/k8s/
  ```



## Deployment

### Pre-requisites

* Install `openssl` on K8s master

* Ensure a docker registry is running locally or remotely. 

  > **Note:** For single node microk8s deployment, a registry can be brought up by using microk8s add-ons. More details present in Microk8s documentation. This is not mandatory, if a remote registry already exists, the same can be  used as well for single-node
  >
  > **Note:** For multi-node kubeadm deployment, a docker registry needs to be setup by the user

* Push all container images to docker registry. Example below

  ```shell
  # Without TLS enabled
  skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/<image-name>:<image-tag> --dest-tls-verify=false
  
  # With TLS enabled
  skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/image-name>:<image-tag>
  ```

  > **Note:**  In case of microk8s deployment, when docker registry is enabled locally, the OCI container images need to be copied to the node where registry is enabled and then the above example command can be run. The same would not be required when registry is remotely installed

* On each worker node with SGX enabled and registered to K8s-master, the following pre-req needs to be done

  * RHEL 8.2 enabled K8s worker node with SGX:

    > TODO: add the pre-req for sgxagent here

  * Ubuntu 18.04 enabled K8s worker node with SGX:

    > TODO: add the pre-req for sgxagent here

### Deploy

#### Single-Node

##### Pre-requisites 

###### Setup

* `Microk8s` being the default supported single node K8s distribution, users would need to install microk8s on a Physical server
* Copy all manifests and OCI container images as required to K8s master
* Ensure docker registry is running locally or remotely

###### Manifests

* Update all the K8s manifests with the image names to be pulled from the registry

* The `tolerations` and `node-affinity` in case of isecl-scheduler and isecl-controller needs to be updated in the respective manifests under the `manifests/k8s-extensions-controller`  and `manifests/k8s-extensions-scheduler` directories to `microk8s.io/cluster`
##### Deploy steps

The bootstrap script would facilitate the deployment of all SGX components at a usecase level. Sample one given below. All the env files will be updated dynamically as per the system on which its deployed

###### Update .env file

```shell
#Update isecl-k8s-skc.env for the following fields
K8S_DISTRIBUTION=microk8s
K8S_MASTER_IP=<ip-address of K8s master>
K8S_MASTER_HOSTNAME=<hostname of K8s master>
AAS_API_CLUSTER_ENDPOINT_URL=https://<Master IP>:30444/aas/v1

#For microk8s
#K8S_API_SERVER_CERT=/var/snap/microk8s/current/certs/server.crt
K8S_API_SERVER_CERT=

#For microk8s
#IHUB_PUB_KEY_PATH=/etc/ihub/ihub_public_key.pem
IHUB_PUB_KEY_PATH=

# ISECL-Scheduler
# For microk8s
# K8S_CA_KEY=/var/snap/microk8s/current/certs/ca.key
# K8S_CA_CERT=/var/snap/microk8s/current/certs/ca.crt
K8S_CA_KEY=
K8S_CA_CERT=
```

###### Run scripts on K8s master

The bootstrap scripts are sample scripts to allow for a quick start of SKC services and agents. Users are free to modify the script or directly use the K8s manifests as per their deployment model requirements

```shell
#Pre-reqs.sh
chmod +x pre-requisites.sh
./pre-requisites.sh

#skc-bootstrap-db-services.sh
#Reference
#Usage: ./skc-bootstrap-db-services.sh [-help/up/purge]
#    -help      print help and exit
#     up        Bootstrap Database Services for Authservice, SGX Caching Service and SGX Host verification Service
#     purge     Delete Database Services for Authservice, SGX Caching Service and SGX Host verification Service

chmod +x skc-bootstrap-db-services.sh
./skc-bootstrap-db-services.sh

#skc-bootstrap.sh
#Reference
#Usage: ./skc-bootstrap.sh [-help/up/down/purge]
#    -help                                     Print help and exit
#    up   [all/<agent>/<service>/<usecase>]    Bootstrap SKC K8s environment for specified agent/service/usecase
#    down [all/<agent>/<service>/<usecase>]    Delete SKC K8s environment for specified agent/service/usecase [will not delete data,config and logs]
#    purge                                     Delete SKC K8s environment with data,config,logs

#    Available Options for up/down command:
#        agent      Can be one of sagent,skclib
#        service    Can be one of cms,authservice,scs,shvs,ihub,sqvs,kbs,isecl-controller,isecl-scheduler
#        usecase    Can be one of secure-key-caching,sgx-attestation,sgx-orchestration-k8s

chmod +x skc-bootstrap.sh
./skc-bootstrap.sh up <all/usecase of choice>
```

> TODO: Add steps for configuring K8s scheduler in case of microk8s

#### Multi-Node

##### Pre-requisites

###### Setup

* `kubeadm` being the default supported multi-node K8s distribution, users would need to install a kubeadm master node setup

* Copy all manifests and OCI container images as required to K8s master

* Ensure images are pushed to registry

* `NFS` storage class is used in kubernetes environment for data persistence and supported in SKC. User needs to setup NFS server and create directory structure along with granting permission for a given user id. From security point of view, its been recommended to create a separate user id and grant the permission for all isecl directories for this user id. Below are some samples for reference

  * Snapshot showing directory structure for which user needs to create on NFS volumes manually or using custom scripts.

  ![NFS Directory Structure SKC Usecase](./images/nfs_skc_directory_structure.JPG)

  * Snapshot showing ownership and permissions for directories for which user needs to manually grant the ownership.

  ![NFS Directory Permissions SKC Usecase](./images/nfs_skc_permissions_dir.JPG)

  * Snapshot for configuring PV and PVC , user need to provide the NFS server IP or hostname and paths for each of the service directories. Sample manifest for creating `config-pv` for cms service

  ```
  ---
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: cms-config-pv
  spec:
    capacity:
      storage: 128Mi
    volumeMode: Filesystem
    accessModes:
      - ReadWriteMany
    persistentVolumeReclaimPolicy: Retain
    storageClassName: nfs
    nfs:
      path: /<NFS-vol-base-path>/isecl/cms/config
      server: <NFS Server IP/Hostname>
  ```

  Sample manifest for creating config-pvc for cms service

  ```
  ---
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: cms-config-pvc
    namespace: isecl
  spec:
    storageClassName: nfs
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 128Mi
  ```

  > **Note:** The user id specified in security context in `deployment.yml` for a given service and owner of the service related directories in NFS must be same

###### Manifests

* Update all the K8s manifests with the image names to be pulled from the registry

* The `tolerations` and `node-affinity` in case of isecl-scheduler and isecl-controller needs to be updated in the respective manifests under the `manifests/k8s-extensions-controller`  and `manifests/k8s-extensions-scheduler` directories to `node-role.kubernetes.io/master`
* All NFS PV yaml files needs to be updated with the  `path: /<NFS-vol-path>`  and `server: <NFS Server IP/Hostname>` under each service manifest file for `config`, `logs` , `db-data`

##### Deploy steps

###### Update .env file

```shell
#Update isecl-k8s-skc.env for the following fields
K8S_DISTRIBUTION=kubeadm
K8S_MASTER_IP=<ip-address of K8s master>
K8S_MASTER_HOSTNAME=<hostname of K8s master>
AAS_API_CLUSTER_ENDPOINT_URL=https://<Master IP>:30444/aas/v1

# For Kubeadm
# K8S_API_SERVER_CERT=/etc/kubernetes/pki/apiserver.crt
K8S_API_SERVER_CERT=

# For Kubeadm
# In case of kubeadm, user needs to manually copy the pub key from NFS server location under <NFS-path-for-isecl>/ihub/config/ihub_public_key.pem to K8s master and update the path variable below
IHUB_PUB_KEY_PATH=

# ISECL-Scheduler
# For Kubeadm
# K8S_CA_KEY=/etc/kubernetes/pki/ca.key
# K8S_CA_CERT=/etc/kubernetes/pki/ca.crt
K8S_CA_KEY=
K8S_CA_CERT=
```

###### Run scripts on K8s master

* The bootstrap scripts are sample scripts to allow for a quick start of SKC services and agents. Users are free to modify the script or directly use the K8s manifests as per their deployment model requirements

```shell
#If using sample script provided for creating nfs directories
#Copy the script to the base path of the NFS location configured
chmod +x create-skc-dirs-nfs.sh
./create-skc-dirs.nfs.sh

#Pre-reqs.sh
chmod +x pre-requisites.sh
./pre-requisites.sh

#skc-bootstrap-db-services.sh
#Reference
#Usage: ./skc-bootstrap-db-services.sh [-help/up/purge]
#    -help      print help and exit
#     up        Bootstrap Database Services for Authservice, SGX Caching Service and SGX Host verification Service
#     purge     Delete Database Services for Authservice, SGX Caching Service and SGX Host verification Service

chmod +x skc-bootstrap-db-services.sh
./skc-bootstrap-db-services.sh

#skc-bootstrap.sh
#Reference
#Usage: ./skc-bootstrap.sh [-help/up/down/purge]
#    -help                                     Print help and exit
#    up   [all/<agent>/<service>/<usecase>]    Bootstrap SKC K8s environment for specified agent/service/usecase
#    down [all/<agent>/<service>/<usecase>]    Delete SKC K8s environment for specified agent/service/usecase [will not delete data,config and logs]
#    purge                                     Delete SKC K8s environment with data,config,logs

#    Available Options for up/down command:
#        agent      Can be one of sagent,skclib
#        service    Can be one of cms,authservice,scs,shvs,ihub,sqvs,kbs,isecl-controller,isecl-scheduler
#        usecase    Can be one of secure-key-caching,sgx-attestation,sgx-orchestration-k8s

chmod +x skc-bootstrap.sh
./skc-bootstrap.sh up <all/usecase of choice>

#NOTE: The isecl-scheduler will not be deployed in case of multi-node K8s as there is dependency on the NFS server IHUB public key to be copied to allow the successful installation of isecl-scheduler. Post update of the isecl-k8s-skc.env for IHUB_PUB_KEY_PATH on K8s master, user needs to run the following

./skc-bootstrap.sh up isecl-scheduler
```



* Configure kube-scheduler to establish communication with isecl-scheduler. Add `scheduler-policy.json` under kube-scheduler section, `mountPath` under container section and `hostPath` under volumes section in` /etc/kubernetes/manifests/kube-scheduler.yaml` as mentioned below

```yaml
spec:
  containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json
```

```yaml
containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
```

```yaml
volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched
```

> **Note:** Make sure to use proper indentation and don't delete existing mountPath and hostPath sections in kube-scheduler.yaml.



* Restart Kubelet which restart all the k8s services including kube base scheduler

```shell
systemctl restart kubelet
```



* Check if CRD data is populated

```shell
kubectl get -o json hostattributes.crd.isecl.intel.com
```



## Default Service and Agent Mount Paths

#### Single Node Deployments

Single node Deployments use `hostPath` mounting pod(container) files directly on host. Following is the complete list of the files being mounted on host

```yaml
#Certificate-Management-Service
Config: /etc/cms
Logs: /var/log/cms

#Authentication Authorization Service
Config: /etc/authservice
Logs: /var/log/authservice
Pg-data: /usr/local/kube/data/authservice/pgdata

#SGX Host Attestation Service
Config: /etc/shvs
Logs: /var/log/shvs
Pg-data: /usr/local/kube/data/shvs

#Integration-Hub
Config: /etc/ihub
Log: /var/log/ihub

#SGX Quote Verificaton Service
Config: /etc/sqvs
Logs: /var/log/sqvs

#Key-Broker-Service
Config: /etc/kbs
Log: /var/log/kbs
Opt: /opt/kbs

#SGX Library:
Config: /root/lib/resources/sgx_default_qcnl.conf
openssl-config: /etc/pki/tls/openssl.cnf
pkcs11-config: /opt/skc/etc/pkcs11-apimodule.ini
kms-cert: /root/9e9db4b5-5893-40fe-b6c4-d54ec6609c55.crt
haproxy-hosts: /etc/hosts
nginx-config: /etc/nginx/nginx.conf
skc-lib-config: /root/lib/skc_library.conf
kms-key: /tmp/keys.txt
nginx-logs: /var/log/sqvs/

#SGX Agent:
Config: /etc/sgx-agent
Logs: /var/log/sgx-agent
EFI: /sys/firmware/efi/efivars
```

#### Multi Node Deployments

Multi node Deployments use k8s persistent volume and persistent volume claims for mounting pod(container) files on NFS volumes for all services, agents will continue to use `hostPath`. Following is a sample list of the files being mounted on NFS base volumes

```yaml
#Certificate-Management-Service
Config: <NFS-vol-base-path>/isecl/cms/config
Logs: <NFS-vol-base-path>/isecl/cms/logs

#Authentication Authorization Service
Config: <NFS-vol-base-path>/isecl/aas/config
Logs: <NFS-vol-base-path>/isecl/aas/logs
Pg-data: <NFS-vol-base-path>/isecl/aas/db

#SGX Host Attestation Service
Config: <NFS-vol-base-path>/isecl/shvs/config
Logs: <NFS-vol-base-path>/isecl/shvs/logs
Pg-data: <NFS-vol-base-path>/usr/local/kube/data/shvs

#Integration-Hub
Config: <NFS-vol-base-path>/isecl/ihub/config
Log: <NFS-vol-base-path>/isecl/ihub/logs

#SGX Quote Verificaton Service
Config: <NFS-vol-base-path>/isecl/sqvs/config
Logs: <NFS-vol-base-path>/isecl/sqvs/log

#Key-Broker-Service
Config: <NFS-vol-base-path>/isecl/kbs/config
Log: <NFS-vol-base-path>/isecl/kbs/logs
Opt: <NFS-vol-base-path>/isecl/kbs/kbs/opt

#SGX Agent:
Config: /etc/sgx-agent
Logs: /var/log/sgx-agent
EFI: /sys/firmware/efi/efivars
```



## Default Service Ports

For both Single node and multi-node deployments following ports are used. All services exposing APIs will use the below ports

```yaml
CMS: 30445
AAS: 30444
SCS: 30501
SHVS: 30500
SQVS: 30502
IHUB: None
KBS: 30448
K8s-scheduler: 30888
K8s-controller: None
SKC Library: None
SGXAgent: 31001
```



## Usecase Workflows API Collections

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/utils/tree/v3.5/develop/tools/api-collections) repository

### Pre-requisites

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  >  **Note:** The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/v3.5/develop/tools/api-collections)

### Use Case Collections


| Use case                     | Sub-Usecase | API Collection     |
| ---------------------------- | ----------- | ------------------ |
| Secure Key Caching           | -           | ✔️                  |
| SGX Discovery, Provisioning and Orchestration | -           | ✔️                  |

### Downloading API Collections

* Postman API Network for latest released: https://explore.postman.com/intelsecldc

  or 

* Github repo for all releases

  ```shell
  #Clone the github repo for api-collections
  git clone https://github.com/intel-secl/utils.git
  
  #Switch to specific release-version of choice
  cd utils/
  git checkout <release-version of choice>
  
  #Import Collections from
  cd tools/api-collections
  ```

> **Note:**  The postman-collections are also available when cloning the repos via build manifest under `utils/tools/api-collections`

### Running API Collections

* Import the collection into Postman API Client

  > **Note:** This step is required only when not using Postman API Network and downloading from Github

  ![importing-collection](./images/importing_collection.gif)

* Update env as per the deployment details for specific usecase

  ![updating-env](./images/updating_env.gif)

* View Documentation

  ![view-docs](./images/view_documentation.gif)

* Run the workflow

  ![running-collection](./images/running_collection.gif)



## Appendix

### Running behind Proxy

```shell
#Set proxy in ~/.bash_profile
export http_proxy=<proxy-url>
export https_proxy=<proxy-url>
export no_proxy=<ip_address/hostname>
```

### Git Config Sample (~/.gitconfig)

```
[user]
        name = <username>
        email = <email-id>
[color]
        ui = auto
 [push]
        default = matching 
```

### Rebuilding Repos

In order to rebuild repos, ensure the following steps are followed as a pre-requisite

```shell
# Clean all go-mod packages
rm -rf ~/go/pkg/mod/*

#Navigate to specific folder where repos are built, example below
cd /root/isecl/build/skc-k8s-single-node
rm -rf .repo *
#Rebuild as before
repo init ...
repo sync
make all
```

### SKC Key Transfer Flow

> TODO: TBA by Vinit

### Setup Task Flow

Setup tasks flows have been updated to have K8s native flow to be more agnostic to K8s workflows. Following would be the flow for setup task

- User would create a new `configMap`  object with the environment variables specific to the setup task. The Setup task variables would be documented in the Product Guide
- Users can provide variables for more than one setup task
- Users need to add `SETUP_TASK: "<setup task name>/<comma separated setup task name>"` in the same `configMap`
- Provide a unique name to the new `configMap`
- Provide the same name in the `deployment.yaml` under `configMapRef` section
- Deploy the specific service again with  ` kubectl kustomize . | kubectl  apply -f -`

### Configuration Update Flow

Configuration Update flows have been updated to have K8s native flow to be more agnostic to K8s workflows using `configMap` only. Following would be the flow for configuration update

- User would create a new `configMap`  object using existing one and update the new values. The list of config variables would be documented in the Product Guide
- Users can update variables for more than one
- Users need to add `SETUP_TASK: "update-service-config"` in the same `configMap`
- Provide a unique name to the new `configMap`
- Provide the same name in the `deployment.yaml` under `configMapRef` section
- Deploy the specific service again with  ` kubectl  kustomize . | kubectl  apply -f -`

> **Note:** Incase of agents, setup tasks or configuration updates done through above flows will be applied for all the agents running on different BMs. In order to run setup task or update configuration for individual agents, then user need to perform `kubectl exec -it <pod_name> /bin/bash` into a particular agent pod and run the specific setup task.

### Cleanup workflows

#### Single-node

In order to cleanup and setup fresh again on single node without data,config from previous deployment

```shell
#Purge all data and pods,deploy,cm,secrets
./skc-bootstrap.sh purge <all/usecase of choice>

#Purge all db data,pods,deploy,cm,secrets
./skc-bootstrap-db-services.sh purge

#Setup fresh
./skc-bootstrap-db-services.sh up
./skc-bootstrap.sh up <all/usecase of choice>
```



In order to cleanup and setup fresh again on single node with data,config from previous deployment

```shell
#Down all data and pods,deploy,cm,secrets with deleting config,data,logs
./skc-bootstrap.sh down <all/usecase of choice>

#Setup fresh
./skc-bootstrap-db-services.sh up
./skc-bootstrap.sh up <all/usecase of choice>
```



#### Multi-node

In order to cleanup and setup fresh again on multi-node without data,config from previous deployment

```shell
#Purge all data and pods,deploy,cm,secrets
./skc-bootstrap.sh down <all/usecase of choice>

#Purge all db data,pods,deploy,cm,secrets
./skc-bootstrap-db-services.sh purge

#Cleanup all data from NFS share --> User controlled

#Cleanup data from each worker node
rm -rf /etc/sgx_agent
rm -rf /var/log/sgx_agent

#Setup fresh
./skc-bootstrap-db-services.sh up
./skc-bootstrap.sh up <all/usecase of choice>
```



In order to cleanup and setup fresh again on multi-node with data,config from previous deployment

```shell
#Down all pods,deploy,cm,secrets with removing persistent data
./skc-bootstrap.sh down <all/usecase of choice>

#Setup fresh
./skc-bootstrap-db-services.sh up
./skc-bootstrap.sh up <all/usecase of choice>
```

