# Build

## Foundational Security

* Sync the repo

```{.bash linenums="1"}
mkdir -p /root/intel-secl/build/fs && cd /root/intel-secl/build/fs
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.3.1 -m manifest/fs.xml
repo sync
```

* Run the `pre-requisites` setup script

```{.bash linenums="1"}
cd utils/build/foundational-security/
chmod +x fs-prereq.sh
./fs-prereq.sh -s
```

* Build all repos

```{.bash linenums="1"}
cd /root/intel-secl/build/fs/
make all
```

* Built Binaries

```{.bash linenums="1"}
/root/intel-secl/build/fs/binaries
```

## Workload Security

### VM Confidentiality

* Sync the repo

```{.bash linenums="1"}
mkdir -p /root/intel-secl/build/vmc && cd /root/intel-secl/build/vmc
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.3.1 -m manifest/vmc.xml
repo sync
```

* Run the pre-req script

```{.bash linenums="1"}
cd utils/build/workload-security
chmod +x ws-prereq.sh
./ws-prereq.sh -v
```
  
* Build repo

```{.bash linenums="1"}
cd /root/intel-secl/build/vmc/
make all
```

* Built Binaries

```{.bash linenums="1"}
/root/intel-secl/build/vmc/binaries/
```

### Container Confidentiality with Docker Runtime

* Sync the repo

```{.bash linenums="1"}
mkdir -p /root/intel-secl/build/cc-docker && cd /root/intel-secl/build/cc-docker
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.3.1 -m manifest/cc-docker.xml
repo sync
```
  
* Run the `pre-requisites` script

```{.bash linenums="1"}
cd utils/build/workload-security
chmod +x ws-prereq.sh
./ws-prereq.sh -d
```

* Enable and start the Docker daemon

```{.bash linenums="1"}
systemctl enable docker
systemctl start docker
```

* Ignore the below steps if not running behind a proxy

```{.bash linenums="1"}
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/proxy.conf
  
# Add the below lines in proxy.conf
[Service]
Environment="HTTP_PROXY=<http_proxy>"
Environment="HTTPS_PROXY=<https_proxy>"
Environment="NO_PROXY=<no_proxy>"
```

```{.bash linenums="1"}
systemctl daemon-reload
systemctl restart docker
```
  
* Build repos

```{.bash linenums="1"}
cd /root/intel-secl/build/cc-docker/
make all 
```
  
* Built binaries

```{.bash linenums="1"}
/root/intel-secl/build/cc-docker/binaries/
```

### Container Confidentiality with CRIO Runtime

* Sync the repo

```{.bash linenums="1"}
mkdir -p /root/intel-secl/build/cc-crio && cd /root/intel-secl/build/cc-crio
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v3.3.1 -m manifest/cc-crio.xml
repo sync
```

* Run the `pre-requisites` script

```{.bash linenums="1"}
cd utils/build/workload-security
chmod +x ws-prereq.sh
./ws-prereq.sh -c
```
  
* Enable and start the Docker daemon

```{.bash linenums="1"}
systemctl enable docker
systemctl start docker
```

* Ignore the below steps if not running behind a proxy

```{.bash linenums="1"}
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/proxy.conf
  
#Add the below lines in proxy.conf
[Service]
Environment="HTTP_PROXY=<http_proxy>"
Environment="HTTPS_PROXY=<https_proxy>"
Environment="NO_PROXY=<no_proxy>"
```

```{.bash linenums="1"}
#Reload docker
systemctl daemon-reload
systemctl restart docker
```

* Download go dependencies

```{.bash linenums="1"}
cd /root/
go get github.com/cpuguy83/go-md2man
mv /root/go/bin/go-md2man /usr/bin/
```

* Build the repos

```{.bash linenums="1"}
cd /root/intel-secl/build/cc-crio
make all
```
  
???+ note
    The crio use case uses containerd that is bundled with `docker-ce-19.03.13` during build time. As of this release , the version being used is `containerd-1.3.7`. If the remote docker-ce repo gets updated for newer containerd version, then the version of containerd might be incompatible for building crio use case. It is recommended to use the version 1.3.7 in that case.
  
* Built binaries
  
```{.bash linenums="1"}
/root/intel-secl/build/cc-crio/binaries/
```