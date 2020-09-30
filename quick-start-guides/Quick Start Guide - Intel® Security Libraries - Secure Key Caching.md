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
systemctl firewalld stop
```


## **3. RHEL Package Requirements**

Access required for the following packages in all systems

1. **BaseOS**

2. **Appstream**

3. **CodeReady**

**Sample RHEL Repo Configuration**

```
[codeready-builder-for-rhel-8-x86_64-rpms]
baseurl = <URL>/codeready-builder-for-rhel-8-x86_64-rpms
enabled = 1
gpgcheck = 0
name =  CodeReady Builder Local Repo

[rhel-8-for-x86_64-baseos-rpms]
baseurl = <URL>/rhel-8-for-x86_64-baseos-rpms
enabled = 1
gpgcheck = 0
name =  RHEL8 BaseOS Local Repo

[rhel-8-for-x86_64-appstream-rpms]
baseurl = <URL>/rhel-8-for-x86_64-appstream-rpms
enabled = 1
gpgcheck = 0
name =  RHEL8 appstreams Local Repo
```


## **4. System Tools and Utilities**

**System Tools and utils**

```
yum install git wget tar python3 yum-utils
```

***Softlink for Python3***

```
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


## **5. Deployment & Testing Tools**

**Build System**

**Deployment Tools**

**Install Ansible and Pull ISECL role from Ansible Galaxy**

```
```

**Testing Tools**

**Install Postman & pull collections**

```
```

## **6. System User Configuration**

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

## **7. Build Services, Libraries and Install packages**

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


**Build SGX Agent Package**

**Build Pre-requisites**

```
cd utils/build/skc-tools/sgx_agent/build_scripts
./sgxagent_build.sh

This script installs the following packages
    wget tar git gcc-c++ make curl-devel makeself
```
    
**Building SKC Library Package**

**Build Pre-requisites**

```
cd utils/build/skc-tools/skc_library/build_scripts
./skc_library_build.sh

This script installs the following packages
    bc wget tar git gcc-c++ make automake autoconf libtool yum-utils p11-kit-devel cppunit-devel openssl-devel
```


**Copy Binaries to a clean folder**

```
copy the generated binaries directory to the home directory on the CSP/Enterprise VM
```

## 8. Deployment

**Assumption:** Ansible-Galaxy role deploys both CSP managed (Services and Agent) and Enterprise Managed (services and workload)

**Deployment Using Ansible:**

**Update Ansible Galaxy Configuration in Build System:**

1. Update inventories (MUST)

2. Update variables (MUST)

3. Deploy Playbook for SKC

**Deployment**

```
cd <Ansible Folder>
Update Inventories <Sample>
Update Variables <Sample>
<<   ansible-playbook -i <inventories> <skc-playbook>  >>
```

**Deployment Using Binaries**

**Deploy CSP SKC Services**

Copy the binaries directory generated in the build system VM to the home directory on the CSP VM

Update the IP addresses for CMS/AAS/SCS/SHVS/IHUB/K8S services in csp_skc.conf

Also update the Intel PCS Server API URL and API Keys in csp_skc.conf

./install_csp_skc.sh



**Deploy Enterprise SKC Services**

Copy the binaries directory generated in the build system VM to the home directory on Enterprise VM

Update the IP addresses for CMS/AAS/SCS/SQVS/KBS services in csp_skc.conf

Also update the Intel PCS Server API URL and API Keys in csp_skc.conf

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

Edit skc_library.conf and Update the IP address for CMS/AAS/SCS/KBS services deployed on CSP VM

Update the Hostname of the Enterprise VM where KBS is deployed

./deploy_skc_library.sh


## **9. Testing Using Postman Collections**

1. Update inventories in postman collections matching deployment

2. Update config files matching deployment

3. Execute Use cases 

**Testing**

```
Update Postman Inventories <Sample>
Update Postman Variables <Sample>
Execute Postman Collections for the use cases
```


## Appendix

**User Specific Environment**

**SSH Key Generation**

```
ssh-keygen -t rsa
```

**OpenSSL Config**

[engine_section]
pkcs11 = pkcs11_section

[pkcs11_section]
engine_id = pkcs11
dynamic_path =/usr/lib64/engines-1.1/pkcs11.so
MODULE_PATH =/opt/skc/lib/libpkcs11-api.so
init = 0

**SGX Measurement example**

Retrieving Quote+Public key - ECDSA
SGX_ISSUER[size:32]:cd171c56941c6ce49690b455f691d9c8a04c2e43e0a4d30f752fa5285c7ee57f
**SGX_MEASUREMENT[size:32]:31085b62833ed5490f71f1d051b1dc5a0c078af8671419752a903692d0cfb865**
SGX_CONFIG_ID[size:64]:00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
SGX_PRODUCT_ID[size:2]:00
SGX_EXT_PRODUCT_ID[size:16]:00000000000000000000000000000000
SGX_CONFIG_ID_SVN[size:2]:00
SGX_ENCLAVE_SVN[size:2]:1
Retrieving Quote+Public key - EPID

