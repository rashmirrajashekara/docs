# **Secure Key Caching (SKC) Quick Start Guide**

[[_TOC_]]

## **1. Hardware & OS Requirements**

1. **Three Hosts or VMs**

   a.    Build System

   b.    CSP managed Services 

   c.    Enterprise Managed Services

2. **SGX Enabled Host**

3. **OS Requirements**

   RHEL 8.2. SKC Solution is built, installed and tested with root privileges. Please ensure that all the following instructions are executed with root privileges

   **Assumption:**

   CSP and Enterprise side deployment will be done through Ansible-Galaxy role;

## **2. Network Requirements**

1. **Build System**

   Internet access required

2. **CSP Managed Services** 

   Internet access required for SGX Caching Service deployed on CSP VM/SGX Compute Node;

3. **Enterprise Managed Services**

   Internet access required for SGX Caching Service deployed on Enterprise VM;

4. **SGX Enabled Host**

   Internet access required to access KBS running on Enterprise environment

**Setting Proxy and No Proxy**

```
export http_proxy=http://<proxy-url>:<proxy-port>
export https_proxy=http://<proxy-url>:<proxy-port>
export no_proxy=0.0.0.0,127.0.0.1,localhost,<CSP_VM IP>,<Enterprise VM IP>, <SGX Compute Node IP>, <KBS VM Hostname>
```

**Firewall Settings**

***Sample***: Disable firewall

```
systemctl stop firewalld
```


## **3. RHEL Package Requirements**

Access required for the following packages in all systems

1. **BaseOS**

2. **Appstream**

3. **CodeReady**

## **4. Deployment Model**

![deploy_model](./images/isecl_deploy_model.PNG)

* Build + Deployment Machine

* CSP - ISecL Services Machine

* CSP - Physical Server as per supported configurations

* Enterprise - ISecL Services Machine


## **5. System Tools and Utilities**

**System Tools and utils**

```
dnf install git wget tar python3 make yum-utils
dnf install https://dl.fedoraproject.org/pub/fedora/linux/releases/30/Everything/x86_64/os/Packages/m/makeself-2.4.0-3.fc30.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip

```

***Repo Tool***

```
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir
```

***Golang Installation***

```
wget https://dl.google.com/go/go1.14.1.linux-amd64.tar.gz
tar -xzf go1.14.1.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.14.1.linux-amd64.tar.gz
```

***Maven Installation***

```
wget https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
tar -xvf apache-maven-3.6.3-bin.tar.gz
mv apache-maven-3.6.3/ /usr/local/
rm -rf apache-maven-3.6.3-bin.tar.gz
export M2_HOME=/usr/local/apache-maven-3.6.3/
export PATH=$M2_HOME/bin:$PATH
```

  Add the below profile element under the `<profiles>` section of `settings.xml` located under `<path_to_maven>/conf/` folder

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

  Enable `<activeProfiles>` to include the above profile.

    <activeProfiles>
        <activeProfile>artifacts</activeProfile>
    </activeProfiles>

  If you are behind a proxy, enable proxy setting under maven `settings.xml`

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


## **6. Build Services, Libraries and Install packages**

**Pulling Source Code**

```
mkdir -p /root/workspace && cd /root/workspace
repo init -u ssh://git@gitlab.devtools.intel.com:29418/sst/isecl/build-manifest.git -b v3.1/develop -m manifest/skc.xml
repo sync
```

**Building All SKC Components**
```
make

This script installs the following packages
    wget gcc gcc-c++ ant git zip java-1.8.0 make makeself

```


**Copy Binaries to a clean folder**

```
copy the generated binaries directory to the /root directory on the CSP/Enterprise VM
```


## **7. Deployment & Usecase Workflow Tools Installation**

The below installation is required on the Build & Deployment VM only and the Platform(Windows,Linux or MacOS) for Usecase Workflow Tool Installation

**Deployment Tools Installation**

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


## **8. Deployment**

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
  gather_facts: yes
  any_errors_fatal: true
  vars:
    setup: <setup var from supported usecases>
    binaries_path: <path where built binaries are copied to>
  roles:   
  - intel-secl.ansible-role
  environment:
    http_proxy: "{{http_proxy}}"
    https_proxy: "{{https_proxy}}"
    no_proxy: "{{no_proxy}}"```
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
| Secure Key Caching                                           |                                                              |

> **Note:**  Orchestrator installation is not bundled with the role and need to be done independently. Also, components dependent on the orchestrator like `isecl-k8s-extensions` and `integration-hub` are installed either partially or not installed


## **9. Usecase Workflows with Postman API Collections**

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/api-collections) repository

#### Use Case Collections

| Use case                     | Sub-Usecase                                   | API Collection     |
| ---------------------------- | --------------------------------------------- | ------------------ |
| Secure Key Caching           |                                               | ✔️                  |
| Security Aware Orchestration |                                               | ✔️(Kubernetes Only) |


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



**Deployment Using Binaries**

**Deploy CSP SKC Services**

Copy the binaries directory generated in the build system VM to the home directory on the CSP VM

Update the IP addresses for CMS/AAS/SCS/SHVS/IHUB/K8S services in csp_skc.conf

Also update the Intel PCS Server API URL and API Keys in csp_skc.conf

./install_csp_skc.sh


**Deploy Enterprise SKC Services**

Copy the binaries directory generated in the build system VM to the home directory on Enterprise VM

Update the IP addresses for CMS/AAS/SCS/SQVS/KBS services in enterprise_skc.conf

Also update the Intel PCS Server API URL and API Keys in enterprise_skc.conf

./install_enterprise_skc.sh



**Deploy SGX Agent**

Copy sgx_agent.tar, sgx_agent.sh2 and agent_untar.sh from binaries directoy to SGX compute node

./agent_untar.sh

Edit agent.conf Update the IP address for CMS/AAS/SHVS services deployed on CSP VM

Update CMS TLS SHA Value (using cms tlscertsha384 on CSP VM where CMS is deployed)

./deploy_sgx_agent.sh



**Deploy SKC Library**

Copy skc_library.tar, skc_library.sh2 and skclib_untar.sh from binaries directoy to SGX compute node

./skclib_untar.sh

Edit skc_library.conf and Update the IP address for CMS/AAS/KBS services deployed on Enterprise VM

Also update the IP Address for CS Service deployed in CSP VM

Update the Hostname of the Enterprise VM where KBS is deployed

./deploy_skc_library.sh


## **10. System User Configuration**

**Build System**

**Setup ~/.gitconfig to update the git user details. A sample config is provided below**

GIT Configuration**

```
[user]
        name = John Doe
        email = john.doe@abc.com
[color]
        ui = auto
 [push]
        default = matching 
```


## Appendix

**OpenSSL Config**
****

Add the folowing block in openssl.cnf in order to enable pkcs11 engine support in openssl.

[engine_section]

pkcs11 = pkcs11_section

[pkcs11_section]

engine_id = pkcs11

dynamic_path =/usr/lib64/engines-1.1/pkcs11.so

MODULE_PATH =/opt/skc/lib/libpkcs11-api.so

init = 0



