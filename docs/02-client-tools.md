# Installing the Client Tools

In this lab you will install the command line utilities required to complete this tutorial: [cfssl](https://github.com/cloudflare/cfssl), [cfssljson](https://github.com/cloudflare/cfssl), and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl) and also create a working/provisioner machine.


## Provision the provisoner 

Run this command in the Cloud Shell.

```
$ gcloud beta compute instances create provisioner --zone=europe-west3-c --machine-type=n1-standard-1 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --scopes=https://www.googleapis.com/auth/cloud-platform --image=ubuntu-1804-bionic-v20181029 --image-project=ubuntu-os-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=provisioner
```

## Install CFSSL

The `cfssl` and `cfssljson` command line utilities will be used to provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) and generate TLS certificates.

Download and install `cfssl` and `cfssljson` from the [cfssl repository](https://pkg.cfssl.org):

### Linux or Cloud Shell

```
$ wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
```

```
$ chmod +x cfssl_linux-amd64 cfssljson_linux-amd64
```

```
$ sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
```

```
$ sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
```

### OS X (only if you're running it from you local OS X machine)

```
$ curl -o cfssl https://pkg.cfssl.org/R1.2/cfssl_darwin-amd64
$ curl -o cfssljson https://pkg.cfssl.org/R1.2/cfssljson_darwin-amd64
```

```
$ chmod +x cfssl cfssljson
```

```
$ sudo mv cfssl cfssljson /usr/local/bin/
```

### Verification

Verify `cfssl` version 1.2.0 or higher is installed:

```
$ cfssl version
```

> output

```
Version: 1.2.0
Revision: dev
Runtime: go1.6
```

> The cfssljson command line utility does not provide a way to print its version.

## Install kubectl (already pre-installed on the Cloud Shell)

The `kubectl` command line utility is used to interact with the Kubernetes API Server. Download and install `kubectl` from the official release binaries:

### Linux

```
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kubectl
```

```
$ chmod +x kubectl
```

```
$ sudo mv kubectl /usr/local/bin/
```

### Verification

Verify `kubectl` version 1.12.0 or higher is installed:

```
$ kubectl version --client
```

> output

```
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.0", GitCommit:"0ed33881dc4355495f623c6f22e7dd0b7632b7c0", GitTreeState:"clean", BuildDate:"2018-09-27T17:05:32Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
```

Next: [Provisioning Compute Resources](03-compute-resources.md)
