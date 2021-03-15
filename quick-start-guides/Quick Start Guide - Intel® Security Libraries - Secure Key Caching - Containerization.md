# Quick start Guide - SGX Containerization 

## Hardware & OS Requirements

1. **Machines**

   a. RHEL 8.2 Build Machine

   b. K8S Master Node Setup on CSP (VMs/Physical Nodes + SGX enabled Physical Nodes)

   c. K8S Master Node Setup on Enterprise (VMs/Physical Nodes)

   > **Note:** The supported K8s distributions are `microk8s(single-node)` & `kubeadm(multi-node)` K8s distributions

2. **OS Requirements**

   * RHEL 8.2 or Ubuntu 18.04 for deployments

     >  **Note:** SKC Solution is built, installed and tested with root privileges. Please ensure that all the following instructions are executed with root privileges



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



## **RHEL Package Requirements**

Access required for the following packages in all systems

1. **BaseOS**

2. **Appstream**

3. **CodeReady**

   

## Deployment Model

### Single Node

The single Node uses `**microk8s**` as a supported K8s distribution



### Multi Node

The multi node supports `**kubeadm`** as a supported K8s distribution

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

The deployment can be done on either RHEL 8.2 OS K8s cluster or Ubuntu 18.04 OS K8s cluster. Follow the below instructions for detailed steps

### Pre-requisites

* Install `openssl`

* Ensure a docker registry is running locally or remotely. 

  > **Note:** For single node microk8s deployment, a registry can be brought up by using microk8s add-ons. More details present in Microk8s documentation
  >
  > **Note:** For multi-node kubeadm deployment, a docker registry needs to be setup by the user

* Push all container images to docker registry. Example below

  ```shell
  # With TLS enabled
  skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/<image-name>:<image-tag> --dest-tls-verify=false
  
  # Without TLS enabled
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

> TODO: add deploy steps for single node and multi-node

#### Multi-Node

  > TODO: add deploy steps for single node and multi-node



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






