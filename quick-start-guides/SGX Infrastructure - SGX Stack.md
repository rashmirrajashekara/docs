# **SGX Attestation Infrastructure and Secure Key Caching (SKC) using SGX Stack**

Table of Contents
-----------------

   * [<strong>SGX Attestation &amp; Secure Key Caching using SGX Stack</strong>](#sgx-attestation-secure-key-caching-using-sgx-stack)
      * [Table of Contents](#table-of-contents)
      * [<strong>1. Hardware &amp; OS Requirements</strong>](#1-hardware-os-requirements)
      * [<strong>2. Network Requirements</strong>](#2-network-requirements)
      * [<strong>3. RHEL Package Requirements</strong>](#3-rhel-package-requirements)
      * [<strong>4. System Tools and Utilities</strong>](#4-system-tools-and-utilities)
      * [<strong>5. Build Services, Libraries and Install packages</strong>](#5-build-services-libraries-and-install-packages)
      * [<strong>6. Deployment & Usecase Workflow Tools Installation</strong>](#6-deployment-usecase-workflow-tools-installation)
      * [<strong>7. Deployment</strong>](#7-deployment)
         * [<strong>7.1. Deploy Linux Stacks for Intel SGX</strong>](#deploy-linux-stacks-for-intel-sgx)
         * [<strong>7.2. Deployment Using Ansible</strong>](#deployment-using-ansible)
            * [Download the Ansible Role](#download-the-ansible-role)
            * [Update the Ansible Role](#update-ansible-inventory)
            * [Create and Run Playbook](#create-and-run-playbook)
            * [Additional Examples & Tips](#additional-examples-tips)
            * [Usecase Setup Options](#usecase-setup-options)
      * [<strong>8. System User Configuration</strong>](#8-system-user-configuration)
      * [<strong>Appendix</strong>](#appendix)
         * [SGX Attestation flow](#sgx-attestation-flow)
         * [Creating RSA Keys in Key Broker Service](#creating-rsa-keys-in-key-broker-service)
         * [Configuration for NGINX testing](#configuration-for-nginx-testing)
         * [KBS key-transfer flow validation](#kbs-key-transfer-flow-validation)
         * [Note on Key Transfer Policy](#note-on-key-transfer-policy)
         * [Note on SKC Library Deployment](#note-on-skc-library-deployment)
         * [Extracting SGX Enclave values for Key Transfer Policy](#extracting-sgx-enclave-values-for-key-transfer-policy)


## **1. Hardware & OS Requirements**

#### Four VMs and a Host

    Build System or Ansible Controller Node

    CSP managed Services 

    Enterprise Managed Services

    Orchestrator Node
     
    SGX Enabled Host

### OS Requirements

    RHEL 8.4 

NOTE: SKC Solution is built, installed and tested with root privileges. Please ensure that all the following instructions are executed with root privileges.

## **2. Network Requirements**

Internet access is required for the following

##### Build System or Ansible Controller Node

##### CSP Managed Services

##### Enterprise Managed Services

##### Orchestrator Node

##### SGX Enabled Host

**Setting Proxy and No Proxy**

```
export http_proxy=http://<proxy-url>:<proxy-port>
export https_proxy=http://<proxy-url>:<proxy-port>
export no_proxy=0.0.0.0,127.0.0.1,localhost,<CSP IP>,<Enterprise IP>, <SGX Compute Node IP>, <KBS system Hostname>
```

**Firewall Settings**

Ensure that all the SKC service ports are accessible with firewall


## **3. RHEL Package Requirements**

Access required for the following packages in all systems

1. **BaseOS**

2. **Appstream**

3. **CodeReady**

## **4. System Tools and Utilities**

**System Tools and utils**

```
dnf install git wget tar python3 gcc gcc-c++ zip tar make yum-utils openssl-devel
dnf install -y skopeo --nobest
dnf install https://dl.fedoraproject.org/pub/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/makeself-2.4.0-5.fc32.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip
export PATH=/usr/local/bin:$PATH
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


## **5. Build Services, Libraries and Install packages**

Note: Currently, the repos contain the source code of both the SGX Attestation Infrastructure and SKC.
Make will build and package all the binaries and installation scripts but the SGX Attestation Infrastructure can be installed and deployed separately. SKC cannot be installed without the SGX Attestation Infrastructure. 

The rest of this document will indicate steps that are only needed for SKC. 

**Pulling Source Code**

```
mkdir -p /root/workspace && cd /root/workspace
repo init -u https://github.com/intel-secl/build-manifest -b refs/tags/v4.1.0-Beta -m manifest/skc.xml
repo sync
```

**Install, Enable and start the Docker daemon**

  ```shell
  dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
  dnf install -y docker-ce-20.10.8 docker-ce-cli-20.10.8
  systemctl enable docker
  systemctl start docker
  ```

**Ignore the below steps if not running behind a proxy**

  ```shell
  mkdir -p /etc/systemd/system/docker.service.d
  touch /etc/systemd/system/docker.service.d/proxy.conf
  
  #Add the below lines in proxy.conf and set proxy server details if proxy is used
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
make sgx-stack
```

**Copy Binaries to a clean folder**

```
For CSP/Enterprise Deployment Model, copy the generated binaries directory to the /root directory on the CSP/Enterprise system
For Single system model, copy the generated binaries directory to the /root directory on the deployment system
```

## **6. Deployment & Usecase Workflow Tools Installation**

The below installation is required on the Build & Deployment system only and the Platform(Windows,Linux or MacOS) for Usecase Workflow Tool Installation

**Deployment Tools Installation**

* Install Ansible on Build Machine or Ansible Controller Node

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

* Copy the `ansible.cfg` contents from https://raw.githubusercontent.com/ansible/ansible/v2.9.10/examples/ansible.cfg and paste it under `/etc/ansible/ansible.cfg`

## **7. Deployment**

### Deploy Linux Stacks for Intel SGX

* To setup and deploy Linux* Stacks for Intel® SGX, follow https://download.01.org/intelsgxstack/2021-07-28/Getting_Started.pdf
* Install Kubernetes Stack on control-plane node and SGX compute node using Linux SGX Stacks Ansible. Control-plane node can be any system, Compute node should be setup on SGX enabled host machine. 
* Once the Kubernetes stack deployed in control-plane and SGX compute node, follow the below steps for deploying Intel® SecL-DC Secure Key Caching Usecase through Ansible Role.	

### Deployment Using Ansible

The below details would enable the deployment through Ansible Role for Intel® SecL-DC Secure Key Caching Usecase. However the services can still be installed manually using the Product Guide. More details on Ansible Role for Intel® SecL-DC in [Ansible-Role](https://github.com/intel-secl/utils/tree/v3.6/develop/tools/ansible-role) repository.

### Download the Ansible Role 

The role can be cloned locally from git and the contents can be copied to the roles folder used by your ansible controller node

```shell
#Create directory for using ansible deployment
mkdir -p /root/intel-secl/deploy/

#Clone the repository
cd /root/intel-secl/deploy/ && git clone https://github.com/intel-secl/utils.git

#Checkout to specific release version
cd utils/
git checkout <release-version of choice>
cd tools/ansible-role

#Update ansible.cfg roles_path to point to path(/root/intel-secl/deploy/utils/tools/ansible-role/roles/)
```

### Update Ansible Inventory

The following inventory can be used and created under `/etc/ansible/hosts`

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

> **Note:** Ansible requires `ssh` and `root` user access to remote machines. The following command can be used to ensure ansible can connect to remote machines with host key check `
  ```shell
  ssh-keyscan -H <ip_address> >> /root/.ssh/known_hosts
  ```

### Create and Run Playbook

The following are playbook and CLI example for deploying Intel® SecL-DC binaries based on the supported deployment models and usecases. The below example playbooks can be created as `site-bin-isecl.yml`

> **Note:** If running behind a proxy, update the proxy variables under `<path to ansible role>/ansible-role/vars/main.yml` and run as below

> **Note:** Go through the `Additional Examples and Tips` section for specific workflow samples

**Option 1**

Update the PCS Server key with following vars in `<path to ansible role>/ansible-role/defaults/main.yml`

```yaml
  intel_provisioning_server_api_key_sandbox: <pcs server key>
```

Create playbook with following contents

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

Create playbook with following contents

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
ansible-playbook <playbook-name> --extra-vars setup=<setup var from supported usecases> --extra-vars binaries_path=<path where built binaries are copied to> --extra-vars intel_provisioning_server_api_key=<pcs server key>
```

> **Note:** If any service installation fails due to any misconfiguration, just uninstall the specific service manually , fix the misconfiguration in ansible and rerun the playbook. The successfully installed services won't be reinstalled.


### Usecase Setup Options

| Usecase                      | Variable                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| Secure Key Caching           | `setup: secure-key-caching` in playbook or via `--extra-vars` as `setup=secure-key-caching`in CLI |
| SKC No Orchestration         | `setup: skc-no-orchestration` in playbook or via `--extra-vars` as `setup=skc-no-orchestration`in CLI |
| SGX Attestation No Orchestration | `setup: sgx-attestation-no-orchestration` in playbook or via `--extra-vars` as `setup=sgx-attestation-no-orchestration`in CLI |

## **8. System User Configuration**

**Build System or Ansible Controller Node**

**Setup ~/.gitconfig to update the git user details. A sample config is provided below**

GIT Configuration**

```
[user]
        name = John Doe
        email = john.doe@abc.com
[color]
        ui = auto
 [push]
        default = matching +
```

* Make sure system date and time of SGX machine and CSP machine both are in sync. Also, if the system is configured to read the RTC time in the local time zone, then use RTC in UTC by running `timedatectl set-local-rtc 0` command on both the machine. Otherwise SGX Agent deployment will fail with certificate expiry error. 

## **Appendix**

### SGX Attestation flow
```
To Deploy SampleApp:
  Copy sample_apps.tar, sample_apps.sha2 and sampleapps_untar.sh from binaries directory to a directory in SGX compute node and untar it using './sample_apps_untar.sh'
  Install Intel® SGX SDK for Linux*OS into /opt/intel/sgxsdk using './install_sgxsdk.sh'
  Install SGX dependencies using './deploy_sgx_dependencies.sh'
Note: Make sure to deploy SQVS with includetoken configuration as false. 

To Verify the SampleApp flow:
  Update sample_apps.conf with the following
   - IP address for SQVS services deployed on Enterprise system
   - IP address for SCS services deployed on CSP system
   - ENTERPRISE_CMS_IP should point to the IP of CMS service deployed on Enterprise system
   - Network Port numbers for SCS services deployed on CSP system
   - Network Port numbers for SQVS and CMS services deployed on Enterprise system
   - Set RUN_ATTESTING_APP to yes if user wants to run both apps in same machine
  Run SampleApp using './run_sample_apps.sh'
  Check the output of attestedApp and attestingApp under out/attested_app_console_out.log and out/attesting_app_console_out.log files
 
```
### Creating RSA Keys in Key Broker Service

**Steps to run KMIP Server**

Note: Below mentioned steps are provided as script (install_pykmip.sh and pykmip.service) as part of kbs_script folder which will install KMIP Server as daemon. Refer to ‘Install KMIP Server as daemon’ section.

```
1. Install python3 and vim-common
   # dnf -y install python3-pip vim-common
   ln -s /usr/bin/python3 /usr/bin/python  > /dev/null 2>&1
   ln -s /usr/bin/pip3 /usr/bin/pip  > /dev/null 2>&1

2. Install pykmip
   # pip3 install pykmip==0.9.1

3. In the /etc/ directory create pykmip and policies folders
   mkdir -p /etc/pykmip/policies

4. Configure pykmip server using server.conf
   Update hostname in the server.conf

5. Copy the following to /etc/pykmip/ from kbs_script folder available under binaries directory
   create_certificates.py, run_server.py, server.conf

6. Create certificates
   > cd /etc/pykmip
   > python3 create_certificates.py <KMIP Host IP/KMIP Host FQDN>

7. Kill running KMIP Server processes and wait for 10 seconds until all the KMIP Server processes are killed. 
   > ps -ef | grep run_server.py | grep -v grep | awk '{print $2}' | xargs kill

8. Run pykmip server using run_server.py script
   > python3 run_server.py &

```

**Install KMIP Server as daemon**

```
1. cd into /root/binaries/kbs_script folder 

2. Configure pykmip server using server.conf
   Update hostname in the server.conf

3. Run the install_pykmip.sh script and KMIP server will be installed as daemon process
   ./install_pykmip.sh

```
**Create RSA key in PyKMIP and generate certificate**

NOTE: This step is required only when PyKMIP script is used as a backend KMIP server.

```
1. Update Host IP in /root/binaries/kbs_script rsa_create.py script
2. In the kbs_script folder, Run rsa_create.py script
    > cd /root/binaries/kbs_script
    > python3 rsa_create.py

This script will generate “Private Key ID” and “Server certificate”, which should be provided in the kbs.conf file for “KMIP_KEY_ID” and “SERVER_CERT”.

```
**Configuration Update to create Keys in KBS**
    
	cd into /root/binaries/kbs_script folder
	
    **To register keys with KBS KMIP**
    
    Update the following variables in kbs.conf:
    
        KMIP_KEY_ID (Private key ID registered in KMIP server)
        
        SERVER_CERT (Server certificate for created private key)
		
		Enterprise system IP address where CMS, AAS and KBS services are deployed
        
		Port of CMS, AAS and KBS services deployed on enterprise system
    
	    AAS admin and Enterprise admin credentials
        
NOTE: If KMIP_KEY_ID is not provided then RSA key register will be done with keystring.

Update sgx_enclave_measurement_anyof value in transfer_policy_request.json with enclave measurement value obtained using sgx_sign utility. Refer to "Extracting SGX Enclave values for Key Transfer Policy" section.

**Create RSA Key**

	Execute the command
	
	./run.sh reg

Copy the generated cert file to SGX Compute node where skc_library is deployed. Also make a note of the key id generated.

### Configuration for NGINX testing

**Note:** Below mentioned OpenSSL and NGINX configuration updates are provided as patches (nginx.patch and openssl.patch) as part of skc_library deployment script. Patch can be applied with default nginx and openssl file. In case nginx/openssl contains any external changes then refer manual step.

**Apply Patch**
        Execute the command with nginx version - nginx 1.14.1 (Rhel) and openssl version- Openssl 1.1.1g (Rhel)

        patch -b /etc/nginx/nginx.conf < nginx.patch
        patch -b /etc/pki/tls/openssl.cnf < openssl.patch

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

ssl_engine pkcs11;

Update the location of certificate with the loaction where it was copied into the skc_library machine. 

ssl_certificate "add absolute path of crt file";

Update the fields(token, object and pin-value) with the values given in keys.txt for the KeyID corresponding to the certificate.

ssl_certificate_key "engine:pkcs11:pkcs11:token=KMS;object=RSAKEY;pin-value=1234";

**SKC Configuration**

 Create keys.txt in /root folder. This provides key preloading functionality in skc_library.

  Any number of keys can be added in keys.txt. Each PKCS11 URL should contain different Key ID which need to be transferred from KBS along with respective object tag for each key id specified

  Sample PKCS11 url is as below
  
  pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234;
  
  Token, object and pin-value given in PKCS11 url entry in keys.txt should match with the one in nginx.conf.

  The keyID should match the keyID of RSA key created in KBS. File location should match with preload_keys directive in pkcs11-apimodule.ini; 

  Sample /opt/skc/etc/pkcs11-apimodule.ini file
	
	[core]
	preload_keys=/root/keys.txt
	keyagent_conf=/opt/skc/etc/key-agent.ini
	mode=SGX
	debug=true
	
	[SGX]
	module=/opt/intel/cryptoapitoolkit/lib/libp11sgx.so

### KBS key-transfer flow validation

On SGX Compute node, Execute below commands for KBS key-transfer:


Note: Before initiating key transfer make sure, PYKMIP server is running.

```
    pkill nginx
```

Remove any existing pkcs11 token

```
    rm -rf /opt/intel/cryptoapitoolkit/tokens/*
```

Install Nginx and add Group

```
    yum install -y nginx
    groupadd intel
    usermod -G intel

```

Initiate Key transfer from KBS

```
    systemctl restart nginx
```

Changing group ownership and permissions of pkcs11 token

```
    chown -R root:intel /opt/intel/cryptoapitoolkit/tokens/
```

```
    chmod -R 770 /opt/intel/cryptoapitoolkit/tokens/
```

Establish a tls session with the nginx using the key transferred inside the enclave

```
    wget https://localhost:2443 --no-check-certificate
```

### Note on Key Transfer Policy

Key transfer policy is used to enforce a set of policies which need to be compiled with before the secret can be securely provisioned onto a sgx enclave

A typical Key Transfer Policy would look as below
```
        "sgx_enclave_issuer_anyof":["83d719e77deaca1470f6baf62a4d774303c899db69020f9c70ee1dfc08c7ce9e"],
        "sgx_enclave_issuer_product_id_anyof":[0],
        "sgx_enclave_measurement_anyof":["ad46749ed41ebaa2327252041ee746d3791a9f2431830fee0883f7993caf316a"],
        "tls_client_certificate_issuer_cn_anyof":["CMSCA", "CMS TLS Client CA"],
        "client_permissions_allof":["nginx","USA"],
        "sgx_enforce_tcb_up_to_date":false
```
**sgx_enclave_issuer_anyof** establishes the signing identity provided by an authority who has signed the sgx enclave. in other words the owner of the enclave

**sgx_enclave_measurement_anyof** represents the cryptographic hash of the enclave log (enclave code, data)

**sgx_enforce_tcb_up_to_date** - If set to true, Key Broker service will provision the key only of the platform generating the quote conforms to the latest Trusted Computing Base

**client_permissions_allof** - Special permission embedded into the skc_library client TLS certificate which can enforce additional restrictons on who can get access to the key,
    In above example: the key is provisioned only to the nginx workload and platform which is tagged with value for ex: USA


### Note on SKC Library Deployment

SKC Library Deployment (Binary as well as container) needs to performed with root privilege

For binary deployment of SKC client Library, only one instance of Workload can use SKC Client Library. The config information for SKC client library is bound to the workload.
In future, Multiple workloads might be supported
For container deployment, since configmaps are used, each container instance of workload gets its own private SKC Client Library config information

The SKC Client Library TLS client certificate private key is stored in the configuration directories and can be read only with elevated root privileges
keys.txt (set of PKCS11 URIs for the keys to be securely provisioned into an SGX enclave) can only be modified with elevated privileges


### Extracting SGX Enclave values for Key Transfer Policy

Values that are specific to the enclave such as sgx_enclave_issuer_anyof, sgx_enclave_measurement_anyof and sgx_enclave_issuer_product_id_anyof can be retrived using `sgx_sign` utility that is available as part of Intel SGX SDK.

Run `sgx_sign` utility on the signed enclave (This command should be run on the build system).
```
    /opt/intel/sgxsdk/bin/x64/sgx_sign dump -enclave <path to the signed enclave> -dumpfile info.txt
```

- For `sgx_enclave_issuer_anyof`, in info.txt, search for "mrsigner->value" . E.g mrsigner->value :
  ```
  mrsigner->value: "0x83 0xd7 0x19 0xe7 0x7d 0xea 0xca 0x14 0x70 0xf6 0xba 0xf6 0x2a 0x4d 0x77 0x43 0x03 0xc8 0x99 0xdb 0x69 0x02 0x0f 0x9c 0x70 0xee 0x1d 0xfc 0x08 0xc7 0xce 0x9e"
  ```
  Remove the whitespace and 0x characters from the above string and add it to the policy file. E.g :
  ```
  "sgx_enclave_issuer_anyof":["83d719e77deaca1470f6baf62a4d774303c899db69020f9c70ee1dfc08c7ce9e"]
  ```
- For `sgx_enclave_measurement_anyof`, in info.txt, search for metadata->enclave_css.body.enclave_hash.m . E.g metadata->enclave_css.body.enclave_hash.m :
  ```
  metadata->enclave_css.body.enclave_hash.m:
  0xad 0x46 0x74 0x9e 0xd4 0x1e 0xba 0xa2 0x32 0x72 0x52 0x04 0x1e 0xe7 0x46 0xd3
  0x79 0x1a 0x9f 0x24 0x31 0x83 0x0f 0xee 0x08 0x83 0xf7 0x99 0x3c 0xaf 0x31 0x6a
  ```
  Remove the whitespace and 0x characters from the above string and add it to the policy file. E.g :
  ```
  "sgx_enclave_measurement_anyof":["ad46749ed41ebaa2327252041ee746d3791a9f2431830fee0883f7993caf316a"]
  ```
Please note that the SGX Enclave measurement value will depend on the toolchain used to build and link the SGX enclave. Hence the SGX Enclave measurement value would differ across OS flavours.
For more details please refer https://github.com/intel/linux-sgx/tree/master/linux/reproducibility
