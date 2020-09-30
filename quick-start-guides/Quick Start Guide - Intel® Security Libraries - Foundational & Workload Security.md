# **Foundational & Workload Security Quick Start Guide**

[[_TOC_]]

## **1. Hardware & OS Requirements**

#### Hardware

* Build and Deploy VM
* CSP Managed Services (VM and 1 Physical Server)
* Enterprise Managed Services VM

#### OS Requirements

* RHEL 8.2 or later
* `rhel-8-for-x86_64-baseos-rpms` and `rhel-8-for-x86_64-appstream-rpms` repositories need to be enabled on the OS

#### User Access

The services need to be built and installed as `root` user. Ensure root privileges are present for the user to work with Intel® SecL-DC



## **2. Network Requirements**

1. **Build and Deployment VM**

   a.    Internet access required

2. **CSP Managed Services** 

   a.    Internet access required

3. **Enterprise Managed Services**

   a.    Internet access required




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



## **4. Deployment & Usecase Workflow Tools Installation**

The below installation is required on the Build & Deployment VM only and the Platform(Windows,Linux or MacOS) for Usecase Workflow Tool Installation

#### Deployment Tools Installation

* Install Ansible on Build VM

  ```shell
  pip3 install ansible==2.9.10
  ```

* Download role from `ansible-galaxy`

  ```shell
  ansible-galaxy install intel-secl.ansible
  ```

  > **Note: ** The ansible galaxy role will have always the latest version of the role. For older releases , refer the github repository `intel-secl/ansible-role` for specific release tags

#### Usecases Workflow Tools Installation

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  >  **Note:** The Postman API Network will always have the latest released version of the API Collections. For older releases, refer the github repository `intel-secl/api-collections`



## **5. Build Services and packages**

The below steps needs to be carried out on the Build and Deployment VM

#### Building from Source

```shell
mkdir -p /root/isecl && cd /root/isecl
```

##### Foundational Security Usecase

```shell
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.1.0 -m manifest/fs.xml
repo sync
make all
```

##### Workload Security Usecase

```shell
#Container Confidentiality with Docker Runtime
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.1.0 -m manifest/cc.xml
repo sync
make all

#or

#Container Confidentiality with CRIO Runtime
??
repo sync
make all

#or 

#VM Confidentiality
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.1.0 -m manifest/vmc.xml
repo sync
make all
```



## **6. Deployment**

The below details would enable the deployment through Ansible Role for Intel® SecL-DC Foundational & Workload Security Usecases. However the services can still be installed manually using the Product Guide. More details on Ansible Role for Intel® SecL-DC in [Ansible-Role](https://github.com/intel-secl/ansible-role) repository.

#### Download the Ansible Role

The role can be downloaded from ansible galaxy as follows:

```shell
#search for isecl role
ansible-galaxy search intel-secl 

#describe isecl role details
ansible-galaxy info intel-secl 

 #install isecl role
ansible-galaxy install intel-isecl  
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

The following is the inventory to be used and updated. Ansible requires `ssh` and `root` user access to remote machines.

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

[Node]
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
  vars:
    setup: <setup var from supported usecases>
    binaries_path: <path where built binaries are copied to>
  gather_facts: yes
  any_errors_fatal: true
  roles:   
  - intel-secl.ansible-role
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
  any_errors_fatal:true
  roles:   
  - intel-secl.ansible-role
  environment:
    http_proxy: "{{http_proxy}}"
    https_proxy: "{{https_proxy}}"
    no_proxy: "{{no_proxy}}"
```

and

```shell
ansible-playbook <playbook-name> --extra-vars setup=<setup var from supported usecases> --extra-vars binaries_path=<path where built binaries are copied to>
```



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



## **7. Usecase Workflows with Postman API Collections**

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/api-collections) repository

#### Download Postman API Collections

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



#### Running the Collections

* Import the collection into Postman API Client

  <TODO: add image/gif>

* Update env as per the deployment details for specific usecase

  <TODO: add image/gif>

* View Documentation

  <TODO: add image/image>

* Run the workflow

  <TODO: add image/gif>



## Appendix

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

