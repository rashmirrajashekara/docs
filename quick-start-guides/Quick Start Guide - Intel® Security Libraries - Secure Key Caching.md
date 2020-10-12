# **Secure Key Caching (SKC) Quick Start Guide**

[[_TOC_]]

## **1. Hardware & OS Requirements**

1. **Four Hosts or VMs**

   a.    Build System

   b.    CSP managed Services 

   c.    Enterprise Managed Services
   
   d.    K8S Master Node Setup

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

***Sample***: Ensure that all the SKC service ports are opened up with firewall


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
dnf install https://dl.fedoraproject.org/pub/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/makeself-2.4.0-5.fc32.noarch.rpm
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
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.1.0 -m manifest/skc.xml
repo sync
```

**Install, Enable and start the Docker daemon**

  ```shell
  dnf install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.10-3.2.el7.x86_64.rpm
  dnf install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-cli-19.03.5-3.el7.x86_64.rpm
  dnf install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-19.03.5-3.el7.x86_64.rpm
  systemctl enable docker
  systemctl start docker
  ```

**Ignore the below steps if not running behind a proxy**

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


### Usecases Workflow Tools Installation

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  > **Note:** The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/master/tools/api-collections)


## **8. Deployment**

The below details would enable the deployment through Ansible Role for Intel® SecL-DC Foundational & Workload Security Usecases. However the services can still be installed manually using the Product Guide. More details on Ansible Role for Intel® SecL-DC in [Ansible-Role](https://github.com/intel-secl/utils/tree/master/tools/ansible-role) repository.

### Download the Ansible Role 

The role can be cloned locally from git and the contents can be copied to the roles folder used by your ansible server 

```shell
#Create directory for using ansible deployment
mkdir -p /root/intel-secl/deploy/

#Clone the repository
cd /root/intel-secl/deploy/ && git clone https://github.com/intel-secl/utils.git

#Checkout to specific release version
cd utils/
git checkout <release-version of choice>
cd tools/ansible-role

#Update ansible.cfg roles_path to point to path(/root/intel-secl/deploy/utils/tools/)
```


### Update Ansible Inventory

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


### Create and Run Playbook

The following are playbook and CLI for deploying Intel® SecL-DC binaries for Foundational and Workload Security

> **Note:** If running behind a proxy, update the proxy variables under `vars/main.yml` and run as below

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
    no_proxy: "{{no_proxy}}"```
```

and

```shell
ansible-playbook <playbook-name>
```

> **Note:** Update the `roles_path` under `ansible.cfg` to point to the cloned repository so that the role can be read by Ansible

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

> **Note:** Update the `roles_path` under `ansible.cfg` to point to the cloned repository so that the role can be read by Ansible


### Additional Examples & Tips

* For `secure-key-caching` & `security-aware-orchestration` usecase following options can be provided during runtime in the playbook for providing the PCS server key

  ```shell
   ansible-playbook <playbook-name> --extra-vars setup=<setup var from supported usecases> --extra-vars binaries_path=<path where built binaries are copied to> --extra-vars intel_provisioning_server_api_key=<pcs server key>
  ```

  or 

  Update the following vars in `defaults/main.yml`

  ```yaml
  intel_provisioning_server_api_key_sandbox: <pcs server key>
  ```

* If any service installation fails due to any misconfiguration, just uninstall the specific service manually , fix the misconfiguration in ansible and rerun the playbook. The successfully installed services wont be reinstalled.


### Usecase Setup Options

| Usecase            | Variable                                                     |
| ------------------ | ------------------------------------------------------------ |
| Secure Key Caching | `setup: secure-key-caching` in playbook or via `--extra-vars` as `setup=secure-key-caching`in CLI |
| Security Aware Orchestration | `setup: security-aware-orchestration` in playbook or via `--extra-vars` as `setup=security-aware-orchestration`in CLI |


> **Note:**  Orchestrator installation is not bundled with the role and need to be done independently. Also, components dependent on the orchestrator like `isecl-k8s-extensions` and `integration-hub` are installed either partially or not installed


## **9. Usecase Workflows with Postman API Collections**

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/utils/tree/master/tools/api-collections) repository

### Use Case Collections

| Use case                     | Sub-Usecase | API Collection     |
| ---------------------------- | ----------- | ------------------ |
| Secure Key Caching           | -           | ✔️                  |
| Security Aware Orchestration | -           | ✔️(Kubernetes Only) |


### Download Postman API Collections

* Postman API Network for latest released collections: https://explore.postman.com/intelsecldc

  or 

* Github repo for allreleases

  ```shell
  #Clone the github repo for api-collections
  git clone https://github.com/intel-secl/utils.git
  
  #Switch to specific release-version of choice
  cd utils/
  git checkout <release-version of choice>
  
  #Import Collections from
  cd tools/api-collections
  ```

>  **Note:**  The postman-collections are also available when cloning the repos via build manifest under `utils/tools/api-collections`



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



## **10. Deployment Using Binaries**

#### Setup K8S Cluster & Deploy Isecl-k8s-extensions

Setup master and worker node for k8s. Worker node should be setup on SGX host machine. Master node can be any VM machine.

Please note whatever hostname has been used on worker node while registering SGX_Agent with SHVS, use same node-name in join command.

Once the master/worker setup is done, copy the binaries directory generated in the build system VM to the /root/ directory on the Master Node.

Go to /root/binaries directory and run ./isecl-k8s-extensions-* .

Edit /etc/kubernetes/manifests/kube-scheduler.yaml and remove/comment the following content and restart kubelet.

```
    --policy-config-file=/opt/isecl-k8s-extensions/iseclk8sscheduler/config/scheduler-policy.json
    systemctl restart kubelet
```

Wait for the isecl-controller and isecl-scheduler pods to be into running state.

```
    kubectl get pods -n isecl
```

Create role bindings on the Kubernetes Master.

```
    kubectl create clusterrolebinding isecl-clusterrole --clusterrole=system:node --user=system:serviceaccount:isecl:default
    kubectl create clusterrolebinding isecl-crd-clusterrole --clusterrole=iseclcontroller --user=system:serviceaccount:isecl:default
```
	
Copy /etc/kubernetes/pki/apiserver.crt from master node to CSP VM. Update KUBERNETES_CERT_FILE in /root/binaries/env/ihub.env on CSP VM with kubernetes certificate path.

Get k8s token in master, using below commands.

```
    kubectl get secrets -n isecl
    kubectl describe secret <ouput-secret-name> -n isecl.
```

Update KUBERNETES_TOKEN in /root/binaries/env/ihub.env on CSP VM with above kubernetes token.


#### Deploy CSP SKC Services

Copy the binaries directory generated in the build system VM to the /root/ directory on the CSP VM

Update the IP addresses for CMS/AAS/SCS/SHVS/IHUB/K8S services in csp_skc.conf

Also update the Intel PCS Server API URL and API Keys in csp_skc.conf

./install_csp_skc.sh

Copy IHUB public key to the master node and restart kubelet.

```
    scp -r /etc/ihub/ihub_public_key.pem <master-node IP>:/opt/isecl-k8s-extensions/isecl-k8s-scheduler/config/
    systemctl restart kubelet
```
	
Run this command to validate if the data has been pushed to CRD: 

```
    kubectl get -o json hostattributes.crd.isecl.intel.com
```
	
Run this command to validate that the labels have been populated:

```
    kubectl get nodes --show-labels.
```
	
Sample labels:

```
    EPC-Memory=2.0GB,FLC-Enabled=true,SGX-Enabled=true,SGX-Supported=true,TCBUpToDate=true,TrustTagExpiry=2020-08-28T15.28.41Z
```

Create sample yml file for nginx workload and add SGX labels to it such as:

```
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
           
Validate if pod can be launched on the node. Run following commands:

```
    kubectl apply -f pod.yml
    kubectl get pods
    kubectl describe pods nginx 
```

Pod should be in running state and launched on the host as per values in pod.yml.
	
    
#### Deploy Enterprise SKC Services

Copy the binaries directory generated in the build system VM to the /root/ directory on Enterprise VM

Update the IP addresses for CMS/AAS/SCS/SQVS/KBS services in enterprise_skc.conf

Also update the Intel PCS Server API URL and API Keys in enterprise_skc.conf

./install_enterprise_skc.sh


#### Deploy SGX Agent

Copy sgx_agent.tar, sgx_agent.sha2 and agent_untar.sh from binaries directoy to a directory in SGX compute node

./agent_untar.sh

Edit agent.conf Update the IP address for CMS/AAS/SHVS services deployed on CSP VM

Update CMS TLS SHA Value (using cms tlscertsha384 on CSP VM where CMS is deployed)

./deploy_sgx_agent.sh



#### Deploy SKC Library

Copy skc_library.tar, skc_library.sha2 and skclib_untar.sh from binaries directoy to a directory in SGX compute node

./skclib_untar.sh

Edit skc_library.conf and Update the IP address for CMS/AAS/KBS services deployed on Enterprise VM

Also update the IP Address for SGX Caching Service deployed in CSP VM

Update the Hostname of the Enterprise VM where KBS is deployed

./deploy_skc_library.sh


## **11. System User Configuration**

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

## Creating AES and RSA Keys in Key Broker Service

**Configuration Update to create Keys in KBS**

​	cd into /root/workspace/utils/build/skc-tools/kbs_script folder

​	Update KBS and AAS IP addresses in run.sh

**Create RSA Key**

​	Execute the command

​	./run.sh reg

- copy the generated cert file to SGX Compute node where skc_library is deployed. Also make a note of the key id generated

## Configuration for NGINX testing

**Note:** Below mentioned OpenSSL and NGINX configuration updates are provided as patches (nginx.patch and openssl.patch) as part of skc_library deployment script.

**OpenSSL**

Update openssl configuration file /etc/pki/tls/openssl.cnf with below changes:

[openssl_def]
engines = engine_section

[engine_section]
pkcs11 = pkcs11_section

[pkcs11_section]
engine_id = pkcs11

dynamic_path =/usr/lib64/engines-1.1/pkcs11.so

MODULE_PATH =/opt/skc/lib/libpkcs11-api.so

init = 0

**Nginx**

Update nginx configuration file /etc/nginx/nginx.conf with below changes:

user root;

ssl_engine pkcs11;

Update the location of certificate with the loaction where it was copied into the skc_library machine. 

ssl_certificate "add absolute path of crt file";

Update the KeyID with the KeyID received when RSA key was generated in KBS

ssl_certificate_key "engine:pkcs11:pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234";

**SKC Configuration**

​ Create keys.txt in /tmp folder. The keyID should match the keyID of RSA key created in KBS. Other contents should match with nginx.conf. File location should match on pkcs11-apimodule.ini; 

​	pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234";

​	**Note:** Content of this file should match with the nginx conf file

​	**/opt/skc/etc/pkcs11-apimodule.ini**

​	**[core]**

​	preload_keys=/tmp/keys.txt

​	keyagent_conf=/opt/skc/etc/key-agent.ini

​	mode=SGX

​	debug=true

​	**[SW]**

​	module=/usr/lib64/pkcs11/libsofthsm2.so

​	**[SGX]**

​	module=/opt/intel/cryptoapitoolkit/lib/libp11sgx.so

# KBS key-transfer flow validation

Execute below commands for KBS key-transfer:

```
    pkill nginx
```

Remove any existing pkcs11 token

```
    rm -rf /opt/intel/cryptoapitoolkit/tokens/*
```

Initiate Key transfer from KBS

```
    systemctl restart nginx
```

Establish ssh session with the nginx using the key transferred inside the enclave

```
    wget https://localhost:2443 --no-check-certificate
```
