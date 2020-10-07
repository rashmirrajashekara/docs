# **Foundational & Workload Security Quick Start Guide**

[[_TOC_]]

## **1. Hardware & OS Requirements**

#### Physical Server requirements

* Intel® SecL-DC  supports and uses a variety of Intel security features, but there are some key requirements to consider before beginning an installation. Most important among these is the Root of Trust configuration. This involves deciding what combination of TXT, Boot Guard, tboot, and UEFI Secure Boot to enable on platforms that will be attested using Intel® SecL.

  > **Note:** At least one "Static Root of Trust" mechanism must be used (TXT and/or BtG). For Legacy BIOS systems, tboot must be used. For UEFI mode systems, UEFI SecureBoot must be used* Use the chart below for a guide to acceptable configuration options. 

  ![hardware-options](./images/trusted-boot-options.PNG)

> **Note:** A security bug related to UEFI Secure Boot and Grub2 modules has resulted in some modules required by tboot to not be available on RedHat 8 UEFI systems. Tboot therefore cannot be used currently on RedHat 8. A future tboot release is expected to resolve this dependency issue and restore support for UEFI mode.

#### OS Requirements

* RHEL 8.2 or later
* `rhel-8-for-x86_64-baseos-rpms` and `rhel-8-for-x86_64-appstream-rpms` repositories need to be enabled on the OS

#### User Access

The services need to be built and installed as `root` user. Ensure root privileges are present for the user to work with Intel® SecL-DC.
All Intel® SecL-DC service & agent ports should be allowed in firewall rules. 



## **2. Deployment Model**

![deploy_model](./images/isecl_deploy_model.PNG)

* Build + Deployment Machine

* CSP - ISecL Services Machine

* CSP - Physical Server as per supported configurations

* Enterprise - ISecL Services Machine

  

## **3. System Tools and Utilities Installation**

The below installation is required on the Build and Deployment VM only

#### Install basic utilities for getting started

```shell
dnf install git wget tar python3 yum-utils
```

#### Create symlink for python3

```shell
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip
```

#### Install repo tool

```shell
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir
```



## **4. Build Services and packages**

The below steps needs to be carried out on the Build and Deployment VM

#### Pre-requisites

* The repos can be built only as `root` user

* RHEL 8.2 VM for building repos

* Enable the following RHEL repos:

  * `rhel-8-for-x86_64-appstream-rpms`
  * `rhel-8-for-x86_64-baseos-rpms`

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
  
* Additional packages for **Foundational Security** & **Workload Security** usecases

  ```shell
  dnf install java-1.8.0-openjdk.x86_64 wget gcc gcc-c++ ant git patch zip unzip make tpm2-tss-2.0.0-4.el8.x86_64 tpm2-abrmd-2.1.1-3.el8.x86_64 openssl-devel
  ```

  ```shell
  dnf install https://download-ib01.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/m/makeself-2.4.2-1.el8.noarch.rpm \
  	          http://mirror.centos.org/centos/8/PowerTools/x86_64/os/Packages/tpm2-abrmd-devel-2.1.1-3.el8.x86_64.rpm \
              http://mirror.centos.org/centos/8/PowerTools/x86_64/os/Packages/trousers-devel-0.3.14-4.el8.x86_64.rpm \
              https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.10-3.2.el7.x86_64.rpm \
              https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-cli-19.03.5-3.el7.x86_64.rpm \
              https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-19.03.5-3.el7.x86_64.rpm 
  ```

* Enable and start the Docker daemon

  ```shell
  systemctl enable docker
  systemctl start docker
  ```

* If Running behind a proxy, configure docker to run behind proxy

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

#### Building

##### Foundational Security Usecase

```shell
mkdir -p /root/isecl/fs && cd /root/isecl/fs
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.1.0 -m manifest/fs.xml
repo sync
make all
```

##### Workload Security Usecase

```shell
#Container Confidentiality with Docker Runtime
mkdir -p /root/isecl/cc-docker && cd /root/isecl/cc-docker
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.1.0 -m manifest/cc-docker.xml
repo sync
make all

#Container Confidentiality with CRIO Runtime
mkdir -p /root/isecl/cc-crio && cd /root/isecl/cc-crio
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.1.0 -m manifest/cc-crio.xml
repo sync
make all

#VM Confidentiality
mkdir -p /root/isecl/vmc && cd /root/isecl/vmc
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.1.0 -m manifest/vmc.xml
repo sync
make all
```



## **5. Deployment & Usecase Workflow Tools Installation**

The below installation is required on the Build & Deployment VM only and the Platform(Windows,Linux or MacOS) for Usecase Workflow Tool Installation

#### Deployment Tools Installation

* Install Ansible on Build VM

  ```shell
  pip3 install ansible==2.9.10
  ```

* Download role from `ansible-galaxy`

  ```shell
  ansible-galaxy install intel-secl.isecl
  ```

  > **Note: ** The ansible galaxy role will have always the latest version of the role. For older releases , refer the github repository of [Ansible-Role](https://github.com/intel-secl/ansible-role) for specific release tags

#### Usecases Workflow Tools Installation

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  >  **Note:** The Postman API Network will always have the latest released version of the API Collections. For older releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tools/api-collections)



## **6. Deployment**

The below details would enable the deployment through Ansible Role for Intel® SecL-DC Foundational & Workload Security Usecases. However the services can still be installed manually using the Product Guide. More details on Ansible Role for Intel® SecL-DC in [Ansible-Role](https://github.com/intel-secl/ansible-role) repository.

#### Download the Ansible Role

The role can be downloaded from ansible galaxy as follows:

```shell
#search for isecl role
ansible-galaxy search isecl 

#describe isecl role details
ansible-galaxy info isecl

 #install isecl role
ansible-galaxy install intel-secl.isecl  
```

or 

The role can be cloned locally from git and the contents can be copied to the roles folder used by your ansible server 

```shell
#Create directory for using ansible deployment
mkdir -p /root/intel-secl/deploy/

#Clone the repository
cd /root/intel-secl/deploy/ && git clone https://github.com/intel-secl/ansible-role

#Checkout to specific release tag
git checkout <release-tag of choice>

#Update ansible.cfg roles_path to point to /root/intel-secl/deploy/ansible-role
```



#### Update Ansible Inventory

The following is the inventory to be used and updated. Ansible requires `ssh` and `root` user access to remote machines.

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

[Enterpise:vars]
isecl_role=enterprise
ansible_user=root
ansible_password=<password>

[Node:vars]
isecl_role=node
ansible_user=root
ansible_password=<password>
```



#### Create and Run Playbook

The following are playbook and CLI for deploying Intel® SecL-DC binaries for Foundational and Workload Security

> **Note :** If running behind a proxy, update the proxy variables under `vars/main.yml` and run as below

**Option 1**

```yaml
- hosts: all
  gather_facts: yes
  any_errors_fatal: true
  vars:
    setup: <setup var from supported usecases>
    binaries_path: <path where built binaries are copied to>
  roles:   
  - intel-secl.isecl
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
  - intel-secl.isecl
  environment:
    http_proxy: "{{http_proxy}}"
    https_proxy: "{{https_proxy}}"
    no_proxy: "{{no_proxy}}"
```

and

```shell
ansible-playbook <playbook-name> --extra-vars setup=<setup var from supported usecases> --extra-vars binaries_path=<path where built binaries are copied to>
```



Additional Examples and Tips
----------------------------

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
  skip_secure_docker_daemon: 'yes'
  ```

* If using Docker notary when working with `Launch Time Protection - Workload Confidentiality with Docker Runtime`, following options can be provided during runtime in the playbook

  ```shell
  ansible-playbook <playbook-name> --extra-vars setup=<setup var from supported usecases> --extra-vars binaries_path=<path where built binaries are copied to> --extra-vars insecure_verify=<insecure_verify[TRUE/FALSE]> --extra-vars registry_ipaddr=<registry ipaddr> --extra-vars registry_scheme=<registry schedme[http/https]>
  ```
  or
 
  Update the following vars in `defaults/main.yml`

  ```yaml
  # [TRUE/FALSE based on registry configured with http/https respectively]
  # Required for Workload Integrity with containers
  insecure_skip_verify: <insecur_skip_verify>

  # The registry IP for the Docker registry from where container images are pulled
  # Required for Workload Integrity with containers
  registry_ip: <registry_ipaddr>

  # The registry protocol for talking to the remote registry [http/https]
  # Required for Workload Integrity with containers
  registry_scheme_type: <registry_scheme>
  ```

* If any service installation fails due to any misconfiguration, just uninstall the specific service manually , fix the misconfiguration in ansible 
  and rerun the playbook. The successfully installed services wont be reinstalled.



#### Usecase Setup Options

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



## **7. Usecase Workflows with Postman API Collections**

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/utils/tools/api-collections) repository

#### Use Case Collections

| Use case               | Sub-Usecase                                   | API Collection     |
| ---------------------- | --------------------------------------------- | ------------------ |
| Foundational Security  | Host Attestation                              | ✔️                  |
|                        | Data Fencing  with Asset Tags                 | ✔️                  |
|                        | Trusted Workload Placement                    | ✔️(Kubernetes Only) |
|                        | Application Integrity                         | ✔️                  |
| Launch Time Protection | VM Confidentiality                            | ❌                  |
|                        | Container Confidentiality with Docker Runtime | ✔️                  |
|                        | Container Confidentiality with CRIO Runtime   | ✔️                  |



#### Downloading API Collections

* Postman API Network for latest release

  <TODO: add image>

  or 

* Github repo for older releases

  ```shell
  #Clone the github repo for api-collections
  git clone https://github.com/intel-secl/api-collections
  
  #Switch to specific release tag of choice
  git checkout <release-tag of choice>
  ```



#### Running API Collections

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

##### Running behind Proxy

```shell
#Set proxy in ~/.bash_profile
export http_proxy=<proxy-url>
export https_proxy=<proxy-url>
export no_proxy=<ip_address/hostname>
```

##### Git Config Sample (~/.gitconfig)

```
[user]
        name = <username>
        email = <email-id>
[color]
        ui = auto
 [push]
        default = matching 
```

