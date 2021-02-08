
# Appendix

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

### Installing the Intel® SecL Kubernetes Extensions and Integration Hub

Intel® SecL uses Custom Resource Definitions to add the ability to base
orchestration decisions on Intel® SecL security attributes to
Kubernetes. These CRDs allow Kubernetes administrators to configure pods
to require specific security attributes so that the Kubernetes Control Plane
Node will schedule those pods only on Worker Nodes that match the
specified attributes.

Two CRDs are required for integration with Intel® SecL – an extension
for the Control Plane nodes, and a scheduler extension. The extensions are deployed as a Kubernetes
deployment in the `isecl` namespace.


#### Deploy Intel® SecL Custom Controller

1.  Copy `isecl-k8s-extensions-*.tar.gz` to Kubernetes Control plane machine and extract the contents
    
    ```shell
    #Copy
    scp /<build_path>/binaries/isecl-k8s-extensions-*.tar.gz <user>@<k8s_master_machine>:/<path>/
    
    #Extract
    tar -xvzf /<path>/isecl-k8s-extensions-*.tar.gz
    cd /<path>/isecl-k8s-extensions/
    ```
    
2.  Create `hostattributes.crd.isecl.intel.com` CRD
    
    ```shell
    #1.14<=k8s_version<=1.16
    kubectl apply -f yamls/crd-1.14.yaml

    #1.16<=k8s_version<=1.18
    kubectl apply -f yamls/crd-1.17.yaml
    ```
    
3. Check whether the CRD is created

   ```shell
   kubectl get crds
   ```

4. Load the `isecl-controller` docker image

   ```shell
   docker load -i docker-isecl-controller-*.tar
   ```

5. Deploy `isecl-controller`

   ```shell
   kubectl apply -f yamls/isecl-controller.yaml
   ```

6. Check whether the isecl-controller is up and running

   ```shell
   kubectl get deploy -n isecl
   ```

7. Create clusterRoleBinding for ihub to get access to cluster nodes

   ```shell
   kubectl create clusterrolebinding isecl-clusterrole --clusterrole=system:node --user=system:serviceaccount:isecl:isecl
   ```

8. Fetch token required for ihub installation

   ```shell
   kubectl get secrets -n isecl
   
   #The below token will be used for ihub installation when configured with Kubernetes Tenant
   kubectl describe secret default-token-<name> -n isecl
   ```

9. Additional Optional Configurable fields for isecl-controller configuration in `isecl-controller.yaml`

   | Field                 | Required   | Type     | Default | Description                                                  |
   | --------------------- | ---------- | -------- | ------- | ------------------------------------------------------------ |
   | LOG_LEVEL             | `Optional` | `string` | INFO    | Determines the log level                                     |
   | LOG_MAX_LENGTH        | `Optional` | `int`    | 1500    | Determines the maximum length of characters in a line in log file |
   | TAG_PREFIX            | `Optional` | `string` | isecl   | A custom prefix which can be applied to isecl attributes that are pushed from IH. For example, if the tag-prefix is **isecl.** and **trusted** attribute in CRD becomes **isecl.trusted**. |
   | TAINT_UNTRUSTED_NODES | `Optional` | `string` | false   | If set to true. NoExec taint applied to the nodes for which trust status is set to false, Applicable only for HVS based attestation |



#### Installing the Intel® SecL Integration Hub

1. Copy the API Server certificate of K8s Master to machine where Integration Hub will be installed to `/root/` directory

   >  **Note:**  In most  Kubernetes distributions the Kubernetes certificate and key is normally present under `/etc/kubernetes/pki`. However this might differ in case of some specific Kubernetes distributions.

2. Update the token obtained in  Step 8 of `Deploy Intel® SecL Custom Controller` along with other relevant tenant configuration options in `ihub.env`

3. Install Integration Hub

4. Copy the `/etc/ihub/ihub_public_key.pem` to Kubernetes Master machine to `/<path>/secrets/` directory

   ```shell
   #On K8s-Master machine
   mkdir -p /<path>/secrets
   
   #On IHUB machine, copy
   scp /etc/ihub/ihub_public_key.pem <user>@<k8s_master_machine>:/<path>/secrets/hvs_ihub_public_key.pem
   ```



#### Deploy Intel® SecL Extended Scheduler

1. Install `cfssl` and `cfssljson` on Kubernetes Control Plane

   ```shell
   #Install wget
   dnf install wget -y
   
   #Download cfssl to /usr/local/bin/
   wget -O /usr/local/bin/cfssl http://pkg.cfssl.org/R1.2/cfssl_linux-amd64
   chmod +x /usr/local/bin/cfssl
   
   #Download cfssljson to /usr/local/bin
   wget -O /usr/local/bin/cfssljson http://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
   chmod +x /usr/local/bin/cfssljson
   ```

2. Create TLS key-pair for `isecl-scheduler` service which is signed by Kubernetes `apiserver.crt`

   ```shell
   cd /<path>/isecl-k8s-extensions/
   chmod +x create_k8s_extsched_cert.sh
   
   #Set K8s_MASTER_IP,HOSTNAME
   export MASTER_IP=<k8s_machine_ip>
   export HOSTNAME=<k8s_machine_hostname>
   
   #Create TLS key-pair
   ./create_k8s_extsched_cert.sh -n "K8S Extended Scheduler" -s "$MASTER_IP","$HOSTNAME" -c <k8s_ca_authority_cert> -k <k8s_ca_authority_key>
   ```

   > **Note:**  In most  Kubernetes distributions the Kubernetes certificate and key is normally present under `/etc/kubernetes/pki`. However this might differ in case of some specific Kubernetes distributions.

3. Copy the TLS key-pair generated to `/<path>/secrets/` directory

   ```shell
   cp /<path>/isecl-k8s-extensions/server.key /<path>/secrets/
   cp /<path>/isecl-k8s-extensions/server.crt /<path>/secrets/
   ```

4. Load the `isecl-scheduler` docker image

   ```shell
   cd /<path>/isecl-k8s-extensions/
   docker load -i docker-isecl-scheduler-*.tar
   ```
   
5. Create scheduler-secret for isecl-scheduler

   ```shell
   cd /<path>/
   kubectl create secret generic scheduler-certs --namespace isecl --from-file=secrets
   ```

6. The `isecl-scheduler.yaml` file includes support for both SGX and Workload Security put together. For only working with Workload Security scenarios , the following line needs to be made empty in the yaml file. The scheduler and controller yaml files are located under `/<path>/isecl-k8s-extensions/yamls`

   ```yaml
   - name: SGX_IHUB_PUBLIC_KEY_PATH
     value: ""
   ```

7. Deploy `isecl-scheduler`

   ```shell
   cd /<path>/isecl-k8s-extensions/
   kubectl apply -f yamls/isecl-scheduler.yaml      
   ```

8. Check whether the `isecl-scheduler` is up and running

   ```shell
   kubectl get deploy -n isecl
   ```

9. Additional optional fields for isecl-scheduler configuration in `isecl-scheduler.yaml`

   | Field                    | Required   | Type     | Default | Description                                                  |
   | ------------------------ | ---------- | -------- | ------- | ------------------------------------------------------------ |
   | LOG_LEVEL                | `Optional` | `string` | INFO    | Determines the log level                                     |
   | LOG_MAX_LENGTH           | `Optional` | `int`    | 1500    | Determines the maximum length of characters in a line in log file |
   | TAG_PREFIX               | `Optional` | `string` | isecl.  | A custom prefix which can be applied to isecl attributes that are pushed from IH. For example, if the tag-prefix is ***\*isecl.\**** and ***\*trusted\**** attribute in CRD becomes ***\*isecl.trusted\****. |
   | PORT                     | `Optional` | `int`    | 8888    | ISecl scheduler service port                                 |
   | HVS_IHUB_PUBLIC_KEY_PATH | `Required` | `string` |         | Required for IHub with HVS Attestation                       |
   | SGX_IHUB_PUBLIC_KEY_PATH | `Required` | `string` |         | Required for IHub with SGX Attestation                       |
   | TLS_CERT_PATH            | `Required` | `string` |         | Path of tls certificate signed by kubernetes CA              |
   | TLS_KEY_PATH             | `Required` | `string` |         | Path of tls key                                              |



#### Configuring kube-scheduler to establish communication with isecl-scheduler

> **Note:** The below is a sample when using `kubeadm` as the Kubernetes distribution, the scheduler configuration files would be different for any other Kubernetes distributions being used.

1.  Add a mount path to the
    `/etc/kubernetes/manifests/kube-scheduler.yaml` file for the Intel
    SecL scheduler extension:

    ```yaml
    - mountPath: /<path>/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
    ```

2. Add a volume path to the
   `/etc/kubernetes/manifests/kube-scheduler.yaml` file for the Intel
   SecL scheduler extension:

   ```yaml
   - hostPath:
       path: /<path>/isecl-k8s-extensions/
       type: ""
     name: extendedsched
   ```

3. Add `policy-config-file` path in the
   `/etc/kubernetes/manifests/kube-scheduler.yaml` file under `command` section:

   ```yaml
   - command:
     - kube-scheduler
     - --policy-config-file=/<path>/isecl-k8s-extensions/scheduler-policy.json
     - --bind-address=127.0.0.1
     - --kubeconfig=/etc/kubernetes/scheduler.conf
     - --leader-elect=true
   ```

4. Restart kubelet 

   ```shell
   systemctl restart kubelet
   ```
