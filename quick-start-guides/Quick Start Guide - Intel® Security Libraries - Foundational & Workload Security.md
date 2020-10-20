# **Foundational & Workload Security Quick Start Guide**

Table of Contents
-----------------

 * [<strong>1. Hardware &amp; OS Requirements</strong>](#1-hardware--os-requirements)
         * [Physical Server requirements](#physical-server-requirements)
         * [OS Requirements](#os-requirements)
         * [User Access](#user-access)
      * [<strong>2. Deployment Model</strong>](#2-deployment-model)
      * [<strong>3. Build Services and packages</strong>](#3-build-services-and-packages)
         * [Pre-requisites](#pre-requisites)
         * [Building](#building)
            * [Foundational Security Usecase](#foundational-security-usecase)
            * [Workload Security Usecase](#workload-security-usecase)
               * [VM Confidentiality](#vm-confidentiality)
               * [Container Confidentiality with Docker Runtime](#container-confidentiality-with-docker-runtime)
               * [Container Confidentiality with CRIO Runtime](#container-confidentiality-with-crio-runtime)
      * [<strong>4. Deployment</strong>](#4-deployment)
         * [Pre-requisites](#pre-requisites-1)
         * [Download the Ansible Role](#download-the-ansible-role)
         * [Usecase Setup Options](#usecase-setup-options)
         * [Update Ansible Inventory](#update-ansible-inventory)
         * [Create and Run Playbook](#create-and-run-playbook)
         * [Additional Examples &amp; Tips](#additional-examples--tips)
      * [<strong>5. Usecase Workflows with Postman API Collections</strong>](#5-usecase-workflows-with-postman-api-collections)
         * [Pre-requisites](#pre-requisites-2)
         * [Use Case Collections](#use-case-collections)
         * [Downloading API Collections](#downloading-api-collections)
         * [Running API Collections](#running-api-collections)
      * [<strong>Appendix</strong>](#appendix)
         * [Running behind Proxy](#running-behind-proxy)
         * [Git Config Sample (~/.gitconfig)](#git-config-sample-gitconfig)
         * [Rebuilding Repos](#rebuilding-repos)

## **1. Hardware & OS Requirements**

### Physical Server requirements

* Intel® SecL-DC  supports and uses a variety of Intel security features, but there are some key requirements to consider before beginning an installation. Most important among these is the Root of Trust configuration. This involves deciding what combination of TXT, Boot Guard, tboot, and UEFI Secure Boot to enable on platforms that will be attested using Intel® SecL.

  > **Note:** At least one "Static Root of Trust" mechanism must be used (TXT and/or BtG). For Legacy BIOS systems, tboot must be used. For UEFI mode systems, UEFI SecureBoot must be used* Use the chart below for a guide to acceptable configuration options. 

  ![hardware-options](./images/trusted-boot-options.PNG)

> **Note:** A security bug related to UEFI Secure Boot and Grub2 modules has resulted in some modules required by tboot to not be available on RedHat 8 UEFI systems. Tboot therefore cannot be used currently on RedHat 8. A future tboot release is expected to resolve this dependency issue and restore support for UEFI mode.

### OS Requirements

* `RHEL 8.2` OS
* `rhel-8-for-x86_64-baseos-rpms` and `rhel-8-for-x86_64-appstream-rpms` repositories need to be enabled on the OS
* Date and time should be in sync across the machines

### User Access

The services need to be installed as `root` user. Ensure root privileges are present for the user to work with Intel® SecL-DC.
All Intel® SecL-DC service & agent ports should be allowed in firewall rules. 


## **2. Deployment Model**

![deploy_model](./images/isecl_deploy_model.PNG)

* Build + Deployment Machine

* CSP - ISecL Services Machine

* CSP - Physical Server as per supported configurations

* Enterprise - ISecL Services Machine
  

## **3. Build Services and packages**

The below steps needs to be carried out on the Build and Deployment VM

### Pre-requisites

* The repos can be built only as `root` user

* RHEL 8.2 Machine for building repos

* Enable the following RHEL repos:

  * `rhel-8-for-x86_64-appstream-rpms`
  * `rhel-8-for-x86_64-baseos-rpms`

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

* Extract Install `go` version > `go1.13` & <= `go1.14.4` from `https://golang.org/dl/` 
  and set `GOROOT` & `PATH`

  ```shell
  export GOROOT=<path_to_go>
  export PATH=$GOROOT/bin:$PATH
  ```
  
* Extract and Install `Maven` & set in `PATH`

  ```shell
  wget https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
  tar -xf apache-maven-3.6.3-bin.tar.gz
  export M2_HOME=<path_to_maven>
  export PATH=$M2_HOME/bin:$PATH
  ```

  - Add the below profile element under the `<profiles>` section of `settings.xml` located under `<path_to_maven>/conf/` folder

    ```xml
    <profile>
        <id>artifacts</id>
        <repositories>
        <repository>
            <id>mulesoft-releases</id>
            <name>MuleSoft Repository</name>
            <url>http://repository.mulesoft.org/releases/</url>
            <layout>default</layout>
        </repository>
        <repository>
            <id>maven-central</id>
            <snapshots><enabled>false</enabled></snapshots>
            <url>http://central.maven.org/maven2</url>
        </repository>
        </repositories>
    </profile>
    ```

  - Enable `<activeProfiles>` to include the above profile.

    ```xml
    <activeProfiles>
        <activeProfile>artifacts</activeProfile>
    </activeProfiles>
    ```

  - If you are behind a proxy, enable proxy setting under maven `settings.xml`

    ```xml
    <!-- proxies
    | This is a list of proxies which can be used on this machine to connect to the network.
    | Unless otherwise specified (by system property or command-line switch), the first proxy
    | specification in this list marked as active will be used.
    |-->
    <proxies>
        <!-- proxy
        | Specification for one proxy, to be used in connecting to the network.
        |
        <proxy>
        <id>optional</id>
        <active>true</active>
        <protocol>http</protocol>
        <username>proxyuser</username>
        <password>proxypass</password>
        <host>proxy.host.net</host>
        <port>80</port>
        <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
        </proxy>
        -->
    </proxies> 
    ```
  
### Building

#### Foundational Security Usecase

* Sync the repo

  ```shell
  mkdir -p /root/intel-secl/build/fs && cd /root/intel-secl/build/fs
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.1.0 -m manifest/fs.xml
  repo sync
  ```

* Run the `pre-requisites` setup script

  ```shell
  cd utils/build/foundational-security/
  chmod +x fs-prereq.sh
  ./fs-prereq.sh -s
  ```

* Build all repos

  ```shell
  cd /root/intel-secl/build/fs/
  make all
  ```

* Built Binaries

  ```shell
  /root/intel-secl/build/fs/binaries
  ```

#### Workload Security Usecase

##### VM Confidentiality

* Sync the repo

  ```shell
  mkdir -p /root/intel-secl/build/vmc && cd /root/intel-secl/build/vmc
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.2.0 -m manifest/vmc.xml
  repo sync
  ```

* Run the pre-req script

  ```shell
  cd utils/build/workload-security
  chmod +x ws-prereq.sh
  ./ws-prereq.sh -s
  ```
  
* Build repo

  ```shell
  cd /root/intel-secl/build/vmc/
  make all
  ```

* Built Binaries
  ```shell
  /root/intel-secl/build/vmc/binaries/
  ```

##### Container Confidentiality with Docker Runtime

* Sync the repo

  ```shell
  mkdir -p /root/intel-secl/build/cc-docker && cd /root/intel-secl/build/cc-docker
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.2.0 -m manifest/cc-docker.xml
  repo sync
  ```
  
* Run the `pre-requisites` script

  ```shell
  cd utils/build/workload-security
  chmod +x ws-prereq.sh
  ./ws-prereq.sh -s
  ```

* Enable and start the Docker daemon

  ```shell
  systemctl enable docker
  systemctl start docker
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
  ```

  ```shell
  #Reload docker
  systemctl daemon-reload
  systemctl restart docker
  ```
  
* Build repos

  ```shell
  cd /root/intel-secl/build/cc-docker/
  make all 
  ```
  
* Built binaries

  ```shell
  /root/intel-secl/build/cc-docker/binaries/
  ```

##### Container Confidentiality with CRIO Runtime

* Sync the repo

  ```shell
  mkdir -p /root/intel-secl/build/cc-crio && cd /root/intel-secl/build/cc-crio
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.2.0 -m manifest/cc-crio.xml
  repo sync
  ```

* Run the `pre-requisites` script

  ```shell
  cd utils/build/workload-security
  chmod +x ws-prereq.sh
  ./ws-prereq.sh -s
  ```

* Download go dependencies

  ```shell
  cd /root/
  go get github.com/cpuguy83/go-md2man
  mv /root/go/bin/go-md2man /usr/bin/
  ```

* Build the repos

  ```shell
  cd /root/isecl/cc-crio/
  make all
  ```
  
* Built binaries
  ```shell
  /root/intel-secl/build/cc-crio/binaries/
  ```

## **4. Deployment**

The below details would enable the deployment through Ansible Role for Intel® SecL-DC Foundational & Workload Security Usecases. However the services can still be installed manually using the Product Guide. More details on Ansible Role for Intel® SecL-DC in [Ansible-Role](https://github.com/intel-secl/utils/tree/master/tools/ansible-role) repository.


### Pre-requisites

* Install Ansible on Build Machine

  ```shell
  pip3 install ansible==2.9.10
  ```

* Install `epel-release` repository and install `sshpass` for ansible to connect to remote hosts using SSH

  ```shell
  dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
  dnf install sshpass
  ```

* Create directory for ansible default configuration and hosts file

  ```shell
  mkdir -p /etc/ansible/
  touch /etc/ansible/ansible.cfg
  ```

* Copy the default `ansible.cfg` contents from https://raw.githubusercontent.com/ansible/ansible/v2.9.10/examples/ansible.cfg and paste it under `/etc/ansible/ansible.cfg`


### Download the Ansible Role

The role can be cloned locally from git and the contents can be copied to the roles folder used by your ansible server 

```shell
#Create directory for using ansible deployment
mkdir -p /root/intel-secl/deploy/

#Clone the repository
cd /root/intel-secl/deploy/ && git clone https://github.com/intel-secl/utils.git

#Checkout to specific release-version
cd utils/
git checkout <release-version of choice>
cd tools/ansible-role

#Update ansible.cfg roles_path to point to path(/root/intel-secl/deploy/utils/tools/)
```

### Usecase Setup Options

| Usecase                                                      | Variable                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Host Attestation                                             | `setup: host-attestation` in playbook or via `--extra-vars` as `setup=host-attestation` in CLI |
| Application Integrity                                        | `setup: application-integrity` in playbook or via `--extra-vars` as `setup=application-integrity` in CLI |
| Data Fencing & Asset Tags                                    | `setup: data-fencing` in playbook or via `--extra-vars` as `setup=data-fencing` in CLI |
| Trusted Workload Placement - Containers                      | `setup: trusted-workload-placement-containers` in playbook or via `--extra-vars` as `setup=trusted-workload-placement-containers` in CLI |
| Launch Time Protection - VM Confidentiality                  | `setup: workload-conf-vm` in playbook or via `--extra-vars` as `setup=workload-conf-vm` in CLI |
| Launch Time Protection - Container Confidentiality with Docker Runtime | `setup: workload-conf-containers-docker` in playbook or via `--extra-vars` as `setup=workload-conf-containers-docker`in CLI |
| Launch Time Protection - Container Confidentiality with CRIO Runtime | `setup: workload-conf-containers-crio` in playbook or via `--extra-vars` as `setup=workload-conf-crio`in CLI |

> **Note:**  Orchestrator installation is not bundled with the role and need to be done independently. Also, components dependent on the orchestrator like `isecl-k8s-extensions` and `integration-hub` are installed either partially or not installed
>
> **Note:** `Key Broker Service` is not configured with KMIP compliant KMS when installing through ansible role  

### Update Ansible Inventory

The following is the inventory to be used and updated. 

> **Note:** Ansible requires `ssh` and `root` user access to remote machines.

```
[CSP]
<machine1_ip/hostname>

[Enterprise]
<machine2_ip/hostname>

[Node]
<machine3_ip/hostname>

[CSP:vars]
isecl_role=csp
ansible_user=root
ansible_password=<password>

[Enterprise:vars]
isecl_role=enterprise
ansible_user=root
ansible_password=<password>

[Node:vars]
isecl_role=node
ansible_user=root
ansible_password=<password>
```

### Create and Run Playbook

The following are playbook and CLI for deploying Intel® SecL-DC binaries for Foundational and Workload Security

> **Note :** If running behind a proxy, update the proxy variables under `vars/main.yml` and run as below

> **Note:** Go through the `Additional Examples and Tips` section for specific workflow samples

**Option 1**

```yaml
- hosts: all
  gather_facts: yes
  any_errors_fatal: true
  vars:
    setup: <setup var from supported usecases>
    binaries_path: <path where built binaries are copied to>
  roles:   
  - ansible-role
  environment:
    http_proxy: "{{http_proxy}}"
    https_proxy: "{{https_proxy}}"
    no_proxy: "{{no_proxy}}"
```

and

```shell
ansible-playbook <playbook-name>
```

OR

**Option 2:**

```yaml
- hosts: all
  gather_facts: yes
  any_errors_fatal: true
  roles:   
  - ansible-role
  environment:
    http_proxy: "{{http_proxy}}"
    https_proxy: "{{https_proxy}}"
    no_proxy: "{{no_proxy}}"
```

and

```shell
ansible-playbook <playbook-name> --extra-vars setup=<setup var from supported usecases> --extra-vars binaries_path=<path where built binaries are copied to>
```

### Additional Examples & Tips

* If the Trusted Platform Module(TPM) is already owned, the owner secret(SRK) can be provided directly during runtime in the playbook:
  
  ```shell
  ansible-playbook <playbook-name> --extra-vars setup=<setup var from supported usecases> --extra-vars binaries_path=<path where built binaries are copied to> --extra-vars tpm_secret=<tpm owner secret>
  ```
  or

  Update the following vars in `defaults/main.yml`

  ```yaml
  # The TPM Storage Root Key(SRK) Password to be used if TPM is already owned
  tpm_owner_secret: <tpm_secret>
  ```

* If using for `Launch Time Protection - Workload Confidentiality with CRIO Runtime` , following option can be provided during runtime in playbook

  ```shell
  ansible-playbook <playbook-name> --extra-vars setup=<setup var from supported usecases> --extra-vars binaries_path=<path where built binaries are copied to> --extra-vars skip_sdd=yes
  ```
  or

  Update the following vars in `defaults/main.yml`

  ```yaml
  #Enable/disable container security for CRIO runtime
  # [yes - Launch Time Protection with CRIO Containers, NA - others]
  skip_secure_docker_daemon: <skip_sdd>
  ```

* If using Docker notary when working with `Launch Time Protection - Workload Confidentiality with Docker Runtime`, following options can be provided during runtime in the playbook

  ```shell
  ansible-playbook <playbook-name> --extra-vars setup=<setup var from supported usecases> --extra-vars binaries_path=<path where built binaries are copied to> --extra-vars insecure_verify=<insecure_verify[TRUE/FALSE]> --extra-vars registry_ipaddr=<registry ipaddr> --extra-vars registry_scheme=<registry scheme[http/https]>
  ```
  or

  Update the following vars in `defaults/main.yml`

  ```yaml
  # [TRUE/FALSE based on registry configured with http/https respectively]
  # Required for Workload Integrity with containers
  insecure_skip_verify: <insecure_skip_verify>

  # The registry IP for the Docker registry from where container images are pulled
  # Required for Workload Integrity with containers
  registry_ip: <registry_ipaddr>

  # The registry protocol for talking to the remote registry [http/https]
  # Required for Workload Integrity with containers
  registry_scheme_type: <registry_scheme>
  ```

* If any service installation fails due to any misconfiguration, just uninstall the specific service manually , fix the misconfiguration in ansible and rerun the playbook. The successfully installed services wont be reinstalled.


## **5. Usecase Workflows with Postman API Collections**

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/utils/tree/master/tools/api-collections) repository

### Pre-requisites

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  >  **Note:** The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/master/tools/api-collections)

### Use Case Collections

| Use case               | Sub-Usecase                                   | API Collection     |
| ---------------------- | --------------------------------------------- | ------------------ |
| Foundational Security  | Host Attestation(RHEL & VMWARE)                              | ✔️                  |
|                        | Data Fencing  with Asset Tags(RHEL & VMWARE)                 | ✔️                  |
|                        | Trusted Workload Placement                    | ✔️(Kubernetes Only) |
|                        | Application Integrity                         | ✔️                  |
| Launch Time Protection | VM Confidentiality                            | ❌                  |
|                        | Container Confidentiality with Docker Runtime | ✔️                  |
|                        | Container Confidentiality with CRIO Runtime   | ✔️                  |

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


## **Appendix**

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
cd /root/isec/fs
rm -rf * .repo
```

### Installing the Intel® SecL Custom Resource Definitions(isecl-k8s-extensions)

Intel® SecL uses Custom Resource Definitions to add the ability to base
orchestration decisions on Intel® SecL security attributes to
Kubernetes. These CRDs allow Kubernetes administrators to configure pods
to require specific security attributes so that the Kubernetes Control Plane
Node will schedule those pods only on Worker Nodes that match the
specified attributes.

Two CRDs are required for integration with Intel® SecL – an extension
for the Control Plane nodes, and a scheduler extension. A single installer will
deploy both of these CRDs. The extensions are deployed as a Kubernetes
deployment in the `isecl` namespace.

To deploy the Kubernetes integration CRDs for Intel® SecL:

1.  Copy the `isecl-k8s-extensions` installer to the Kubernetes Control Plane Node
    and execute the installer
    
    ```shell
    ./isecl-k8s-extensions-v3.0.0.bin
    ```
    
2.  Add a mount path to the
    `/etc/kubernetes/manifests/kube-scheduler.yaml` file for the Intel
    SecL scheduler extension:

    ```yaml
    - mountPath: /opt/isecl-k8s-extensions/isecl-k8s-scheduler/config/
      name: extendedsched
      readOnly: true
    ```

3.  Add a volume path to the
    `/etc/kubernetes/manifests/kube-scheduler.yaml` file for the Intel
    SecL scheduler extension:

    ```yaml
    - hostPath:
        path: /opt/isecl-k8s-extensions/isecl-k8s-scheduler/config/
        type: ""
      name: extendedsched
    ```

4.  Add `policy-config-file` path in the
    `/etc/kubernetes/manifests/kube-scheduler.yaml` file under `command` section:

    ```yaml
    - command:
      - kube-scheduler
      - --policy-config-file=/opt/isecl-k8s-extensions/isecl-k8s-scheduler/config/scheduler-policy.json
      - --bind-address=127.0.0.1
      - --kubeconfig=/etc/kubernetes/scheduler.conf
      - --leader-elect=true
    ```

5.  Wait for the isecl-controller and isecl-scheduler pods to be into
    running state

    ```shell
kubectl get pods -n isecl
    ```
    
6. Create role bindings on the Kubernetes Control Plane Node:

```
kubectl create clusterrolebinding isecl-clusterrole --clusterrole=system:node --user=system:serviceaccount:isecl:isecl

kubectl create clusterrolebinding isecl-crd-clusterrole --clusterrole=isecl-controller --user=system:serviceaccount:isecl:isecl
```



7. Copy the Integration Hub public key to the Kubernetes Control Plane Node:

```shell
scp -r /etc/ihub/ihub_public_key.pem k8s.maseter.server:/opt/isecl-k8s-extensions/isecl-k8s-scheduler/config/
```

8. Run the command `systemctl restart kubelet` to restart all the control
   plane container services, including the base scheduler.

The scheduler yaml is present under
`/opt/isecl-k8s-extensions/yamls/isecl-scheduler.yaml`

9. If the Controller and/or Scheduler deployments are deleted, the
   following steps need to be performed:

a. Edit `/etc/kubernetes/manifests/kube-scheduler.yaml` and
remove/comment the following content and restart kubelet

​	`--policy-config-file=/opt/isecl-k8s-extensions/isecl-k8sscheduler/config/scheduler-policy.json`

​	`systemctl restart kubelet`

b. Redeploy scheduler and controller

```
kubectl apply -f /opt/isecl-k8s-extensions/yamls/isecl-controller.yaml
kubectl apply -f /opt/isecl-k8s-extensions/yamls/isecl-scheduler.yaml
```

c. Edit `/etc/kubernetes/manifests/kube-scheduler.yaml` and
add/uncomment the following content and restart kubelet

​	`--policy-config-file=/opt/isecl-k8s-extensions/isecl-k8sscheduler/config/scheduler-policy.json`

​	`systemctl restart kubelet`

d. Logs will be appended to older logs in
/var/log/isecl-k8s-extensions

10. Whenever the CRD's are deleted and restarted for updates, the CRD's
    using the yaml files present under `/opt/isecl-k8s-extensions/yamls/`.
    Kubernetes Version 1.14-1.15 uses crd-1.14.yaml and 1.16-1.17 uses
    crd-1.17.yaml

```shell
kubectl delete hostattributes.crd.isecl.intel.com
kubectl apply -f /opt/isecl-k8s-extensions/yamls/crd-<version>.yaml
```

11. (Optional) Verify that the Intel ® SecL Custom Resource Definitions
    have been started:

To verify the Intel SecL CRDs have been deployed:

```shell
kubectl get crds
```



































