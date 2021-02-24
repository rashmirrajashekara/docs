​	

# **SGX Attestation Infrastructure and Secure Key Caching (SKC) Quick Start Guide**

[[_TOC_]]

## **1. Hardware & OS Requirements**

1. **Two Hosts or VMs**

   a.    Build System

   b.    Deployment System

2. **SGX Enabled Host**

3. **OS Requirements**

   RHEL 8.2. SKC Solution is built, installed and tested with root privileges. Please ensure that all the following instructions are executed with root privileges

   **Assumption:**

   Deployment will be done through Ansible-Galaxy role;

## **2. Network Requirements**

1. **Build System**

   Internet access required

2. **Deployment System**

   Internet access required for SGX Caching Service on deployment system;

4. **SGX Enabled Host**

   Internet access required to access KBS

**Setting Proxy and No Proxy**

```
export http_proxy=http://<proxy-url>:<proxy-port>
export https_proxy=http://<proxy-url>:<proxy-port>
export no_proxy=0.0.0.0,127.0.0.1,localhost, <Deployment System IP>, <SGX Compute Node IP>, <KBS system Hostname>
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
dnf install https://dl.fedoraproject.org/pub/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/makeself-2.4.0-5.fc32.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip

Install latest libkmip for KBS
git clone https://github.com/openkmip/libkmip.git
cd libkmip
make install

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


## **6. Build Services, Libraries and Install packages**

Note: currently, the repos contain the source code of both the SGX Attestation Infrastructure and SKC. Make will build and package all the binaries and installation scripts  but the SGX Attestation Infrastructure can be installed and deployed separately. SKC cannot be installed without the SGX Attestation Infrastructure. 

The rest of this document will indicate steps that are only needed for SKC. 

**Pulling Source Code**

```
mkdir -p /root/workspace && cd /root/workspace
repo init -u ssh://git@gitlab.devtools.intel.com:29418/sst/isecl/build-manifest.git -b v3.4/develop -m manifest/skc.xml
repo sync
```

**Building All SKC Components**

```
make
```

**Copy Binaries to a clean folder**

```
Copy the generated binaries directory to the /root directory on the deployment system
```

### Usecases Workflow Tools Installation

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  > **Note:** The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/v3.4.0/develop/tools/api-collections)


## **8. Deployment**

## **9. Usecase Workflows with Postman API Collections**

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/utils/tree/v3.4.0/develop/tools/api-collections) repository

### Use Case Collections

| Use case                     | Sub-Usecase | API Collection     |
| ---------------------------- | ----------- | ------------------ |
| Secure Key Caching           | -           | ✔️                  |
| SGX Discovery, Provisioning and Orchestration | -           | ✔️(Kubernetes Only) |
| SGX Discovery and Provisioning           | -           | ✔️                  |


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

#### Deploying SKC Services
```
Copy the binaries directory generated in the build system to the /root/ directory on the deployment system
Update skc.conf with the following
  - Deployment system IP address
  - Database name, Database username and passwords for AAS, SCS and SHVS services
  - Intel PCS Server API URL and API Keys
Save and Close
./install_basic.sh
```

#### Deploy SGX Agent
```
Copy sgx_agent.tar, sgx_agent.sha2 and agent_untar.sh from binaries directoy to a directory in SGX compute node
./agent_untar.sh
Edit agent.conf with the following
  - System IP address where CMS/AAS/SHVS services deployed
  - CMS TLS SHA Value (Run "cms tlscertsha384" on deployment system)
  - For Each Agent installation on a SGX compute node, please change AGENT_USER (Changing AGENT_PASSWORD is optional)
Save and Close
./deploy_sgx_agent.sh
```

#### Deploy SKC Library
```
Copy skc_library.tar, skc_library.sha2 and skclib_untar.sh from binaries directoy to a directory in SGX compute node
./skclib_untar.sh
Update skc_library.conf with the following
  - IP address of the system where CMS/AAS/KBS services are deployed
  - Hostname of the system where KBS is deployed
Save and Close
./deploy_skc_library.sh
```

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

* Make sure system date and time of SGX machine and deployment sysetm both are in sync. Also, if the system is configured to read the RTC time in the local time zone, then use RTC in UTC by running `timedatectl set-local-rtc 0` command on both the machine. Otherwise SGX Agent deployment will fail with certificate expiry error. 

## Appendix

## Creating RSA Keys in Key Broker Service

**Configuration Update to create Keys in KBS**

On Deployment System

	cd into /root/binaries/kbs_script folder
	
	Update Deployment system IP addresses in run.sh
	
	Update CACERT_PATH variable with trustedca certificate inside directory /etc/kbs/certs/trustedca/<id.pem>. 

**Create RSA Key**

	Execute the command
	
	./run.sh reg

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

ssl_engine pkcs11;

Update the location of certificate with the loaction where it was copied into the skc_library machine. 

ssl_certificate "add absolute path of crt file";

Update the KeyID with the KeyID received when RSA key was generated in KBS

ssl_certificate_key "engine:pkcs11:pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234";

**SKC Configuration**

 Create keys.txt in /tmp folder. This provides key preloading functionality in skc_library.

  Any number of keys can be added in keys.txt. Each PKCS11 URL should contain different Key IDs which need to be transferred from KBS along with respective object tag for each key id specified

  Sample PKCS11 url is as below
  
  pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234;
  
  Last PKCS11 url entry in keys.txt should match with the one in nginx.conf

  The keyID should match the keyID of RSA key created in KBS. Other contents should match with nginx.conf. File location should match with preload_keys directive in pkcs11-apimodule.ini; 

  Sample /opt/skc/etc/pkcs11-apimodule.ini file
	
	[core]
	preload_keys=/tmp/keys.txt
	keyagent_conf=/opt/skc/etc/key-agent.ini
	mode=SGX
	debug=true
	
	[SW]
	module=/usr/lib64/pkcs11/libsofthsm2.so
	
	[SGX]
	module=/opt/intel/cryptoapitoolkit/lib/libp11sgx.so

# KBS key-transfer flow validation

On SGX Compute node, Execute below commands for KBS key-transfer:

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
