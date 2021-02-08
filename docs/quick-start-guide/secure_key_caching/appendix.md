# Appendix

## Deployment Using Binaries

#### Setup K8S Cluster & Deploy Isecl-k8s-extensions

* Setup master and worker node for k8s. Worker node should be setup on SGX host machine. Master node can be any system.
* Please note whatever hostname has been used on worker node while registering SGX_Agent with SHVS, use same node-name in join command.
* Once the master/worker setup is done, follow below steps:

##### Untar packages and load docker images
* Copy tar output isecl-k8s-extensions-*.tar.gz from build system's binaries folder to /opt/ directory on the Master Node and extract the contents.
```
    cd /opt/
    tar -xvzf isecl-k8s-extensions-*.tar.gz
```
* Load the docker images
```
    cd isecl-k8s-extensions
    docker load -i docker-isecl-controller-v*.tar
    docker load -i docker-isecl-scheduler-v*.tar
```

##### Deploy isecl-controller
* Create hostattributes.crd.isecl.intel.com crd
```
    kubectl apply -f yamls/crd-1.17.yaml
```
* Check whether the crd is created
```
    kubectl get crds
```
* Deploy isecl-controller
```
    kubectl apply -f yamls/isecl-controller.yaml
```
* Check whether the isecl-controller is up and running
```
    kubectl get deploy -n isecl
```
* Create clusterrolebinding for ihub to get access to cluster nodes
```
    kubectl create clusterrolebinding isecl-clusterrole --clusterrole=system:node --user=system:serviceaccount:isecl:isecl
```
* Fetch token required for ihub installation and follow below IHUB installation steps,
```
    kubectl get secrets -n isecl
    kubectl describe secret default-token-<name> -n isecl
```

For IHUB installation, make sure to update below configuration in /root/binaries/env/ihub.env before installing ihub on CSP system:
* Copy /etc/kubernetes/pki/apiserver.crt from master node to /root on CSP system. Update KUBERNETES_CERT_FILE.
* Get k8s token in master, using above commands and update KUBERNETES_TOKEN
* Update the value of CRD name
```
	KUBERNETES_CRD=custom-isecl-sgx
```

##### Deploy isecl-scheduler
* The isecl-scheduler default configuration is provided for common cluster support in isecl-scheduler.yaml. Variables HVS_IHUB_PUBLIC_KEY_PATH and SGX_IHUB_PUBLIC_KEY_PATH are by default set to default paths. Please use and set only required variables based on the use case. 
For example, if only sgx based attestation is required then remove/comment HVS_IHUB_PUBLIC_KEY_PATH variables.

* Install cfssl and cfssljson on Kubernetes Control Plane
```
    #Download cfssl to /usr/local/bin/
    wget -O /usr/local/bin/cfssl http://pkg.cfssl.org/R1.2/cfssl_linux-amd64
    chmod +x /usr/local/bin/cfssl

    #Download cfssljson to /usr/local/bin
    wget -O /usr/local/bin/cfssljson http://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
    chmod +x /usr/local/bin/cfssljson
```

* Create tls key pair for isecl-scheduler service, which is signed by k8s apiserver.crt
```
    cd /opt/isecl-k8s-extensions/
    chmod +x create_k8s_extsched_cert.sh
    ./create_k8s_extsched_cert.sh -n "K8S Extended Scheduler" -s "<K8_MASTER_IP>","<K8_MASTER_HOST>" -c /etc/kubernetes/pki/ca.crt -k /etc/kubernetes/pki/ca.key
```
* After iHub deployment, copy /etc/ihub/ihub_public_key.pem from ihub to /opt/isecl-k8s-extensions/ directory on k8 master system. Also, copy tls key pair generated in previous step to secrets directory.
```
    mkdir secrets
    cp /opt/isecl-k8s-extensions/server.key secrets/
    cp /opt/isecl-k8s-extensions/server.crt secrets/
    mv /opt/isecl-k8s-extensions/ihub_public_key.pem /opt/isecl-k8s-extensions/sgx_ihub_public_key.pem
    cp /opt/isecl-k8s-extensions/sgx_ihub_public_key.pem secrets/
```
Note: Prefix the attestation type for ihub_public_key.pem before copying to secrets folder.
* Create kubernetes secrets scheduler-secret for isecl-scheduler
```
    kubectl create secret generic scheduler-certs --namespace isecl --from-file=secrets
```
* Deploy isecl-scheduler
```
    kubectl apply -f yamls/isecl-scheduler.yaml
```
* Check whether the isecl-scheduler is up and running
```
    kubectl get deploy -n isecl
```

##### Configure kube-scheduler to establish communication with isecl-scheduler
* Add scheduler-policy.json under kube-scheduler section, mountPath under container section and hostPath under volumes section in /etc/kubernetes/manifests/kube-scheduler.yaml as mentioned below
```
spec:
  containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json
```

```
  containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
```

```
  volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched
```

Note: Make sure to use proper indentation and don't delete existing mountPath and hostPath sections in kube-scheduler.yaml.
* Restart Kubelet which restart all the k8s services including kube base schedular
```
	systemctl restart kubelet
```

* Check if CRD data is populated
```
	kubectl get -o json hostattributes.crd.isecl.intel.com
```

#### Deploying SKC Services on Single System
```
Copy the binaries directory generated in the build system to the /root/ directory on the deployment system
Update skc.conf with the following
  - Deployment system IP address
  - TENANT as KUBERNETES or OPENSTACK (based on the orchestrator chosen)
  - System IP address where Kubernetes or Openstack is deployed
  - Database name, Database username and passwords for AAS, SCS and SHVS services
  - Intel PCS Server API URL and API Keys
Save and Close
./install_skc.sh
```

#### Deploy CSP SKC Services
```
Copy the binaries directory generated in the build system system to the /root/ directory on the CSP system
Update csp_skc.conf with the following
  - CSP system IP Address
  - TENANT as KUBERNETES or OPENSTACK (based on the orchestrator chosen)
  - System IP address where Kubernetes or Openstack is deployed
  - Database name, Database username and passwords for AAS, SCS and SHVS services
  - Intel PCS Server API URL and API Keys
Save and Close
./install_csp_skc.sh
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

Pod should be in running state and launched on the host as per values in pod.yml. Validate running below commands on sgx host:
```
	docker ps
```
#### Openstack Setup and Associate Traits

* Setup Compute and Controller node for Openstack. Compute node should be setup on SGX host machine, Controller node can be any system. After the compute/controller setup is done, follow the below steps:

* IHUB should be installed and configured with Openstack

  Note:
  * While using deployment scripts to install the components, in the env directory of the binaries folder comment "KUBERNETES_TOKEN" in the ihub.env before installation.
  * Openstack compute node and build VM should have the same OS package repositories, else there will be package mismatch for SKC library.
  
* On the openstack controller, if resource provider is not listing the resources then install the "osc-placement"
```
  pip3 install osc-placement
```
* source the admin-openrc credentials to gain access to user-only CLI commands and export the os_placement_API_version
```
   source admin-openrc
```
* List the set of resources mapped to the Openstack
```
  openstack resource provider list
```
* Set the required traits for SGX Hosts
```
  #For example 'cirros' image can be used for the instances
  openstack image set --property trait:CUSTOM_ISECL_SGX_ENABLED_TRUE=required <image name>
  
```
* Veiw the Traits that has been set:
```
  #The trait should be set and assinged to the respective image successfully. For example 'cirros' image can be used for the instances 
   openstack image show <image name>
```
* Verify the trait is enabled for the SGX Host:
```
  openstack resource provider trait list <uuid of the host which the openstack resoruce provider lists>

  #SGX Supported, SGX TCB upto Date, SGX FLC enabled, SGX EPC size attritubes of the SGX host for which the 'required' trait set to TRUE or FALSE is displayed. For example,if required trait is set as TRUE:
  
  CUSTOM_ISECL_SGX_ENABLED_TRUE
  CUSTOM_ISECL_SGX_SUPPORTED_TRUE
  CUSTOM_ISECL_SGX_TCBUPTODATE_FALSE
  CUSTOM_ISECL_SGX_FLC_ENABLED_TRUE
  CUSTOM_ISECL_SGX_EPC_SIZE_2_0_GB

  For example, if the required trait is set as FALSE
  CUSTOM_ISECL_SGX_ENABLED_FALSE
  CUSTOM_ISECL_SGX_SUPPORTED_TRUE
  CUSTOM_ISECL_SGX_TCBUPTODATE_FALSE
  CUSTOM_ISECL_SGX_FLC_ENABLED_FALSE
  CUSTOM_ISECL_SGX_EPC_SIZE_0_B
```
* Create the instances
```
  openstack server create --flavor tiny --image <image name> --net vmnet <vm instance name>

  Instances should be created and the status should be "Active". Instance should be launched successfully.
  openstack server list
```
 Note : To unset the trait, use the following CLI commands:
```
 openstack image unset --property trait:CUSTOM_ISECL_SGX_ENABLED_TRUE <image name>

 openstack image unset --property trait:CUSTOM_ISECL_SGX_ENABLED_FALSE <image name>
```
#### Deploy Enterprise SKC Services
```
Copy the binaries directory generated in the build system to the /root/ directory on Enterprise system
Update enterprise_skc.conf with the following
  - Enterprise system IP address
  - Database name, Database username and passwords for AAS and SCS services
  - Intel PCS Server API URL and API Keys
Save and Close
./install_enterprise_skc.sh
```

#### Deploy SGX Agent
```
Copy sgx_agent.tar, sgx_agent.sha2 and agent_untar.sh from binaries directoy to a directory in SGX compute node
./agent_untar.sh
Edit agent.conf with the following
  - CSP system IP address where CMS/AAS/SHVS services deployed
  - CMS TLS SHA Value (Run "cms tlscertsha384" on CSP system)
  - For Each Agent installation on a SGX compute node, please change AGENT_USER (Changing AGENT_PASSWORD is optional)
Save and Close
./deploy_sgx_agent.sh
```

#### Deploy SKC Library
```
Copy skc_library.tar, skc_library.sha2 and skclib_untar.sh from binaries directoy to a directory in SGX compute node
./skclib_untar.sh
Update skc_library.conf with the following
  - IP address for CMS/AAS/KBS services deployed on Enterprise system
  - CSP_CMS_IP should point to the IP of CMS service deployed on CSP system
  - CSP system IP address where SGX Caching Service deployed
  - Hostname of the Enterprise system where KBS is deployed
Save and Close
./deploy_skc_library.sh
```

## System User Configuration

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


## Creating RSA Keys in Key Broker Service

**Configuration Update to create Keys in KBS**

	cd into /root/binaries/kbs_script folder
	
	Update KBS and AAS IP addresses in run.sh
	
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

## KBS key-transfer flow validation

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
