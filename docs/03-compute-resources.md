# Provisioning Compute Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. In this lab you will provision the compute resources required for running a secure and highly available Kubernetes cluster.

## Networking

The Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model) assumes a flat network in which containers and nodes can communicate with each other. In cases where this is not desired [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) can limit how groups of containers are allowed to communicate with each other and external network endpoints.

> Setting up network policies is out of scope for this tutorial.

### Virtual Private Cloud Network

In this section a dedicated  network will be setup to host the Kubernetes cluster.

Create the `kubernetes-the-hard-way` network, a subnet and a router:

```
openstack network create kubernetes-the-hard-way
```

A subnet must be provisioned with an IP address range large enough to assign a private IP address to each node in the Kubernetes cluster.


Create the `kubernetes-the-hard-way` subnet in the `kubernetes-the-hard-way` VPC network:

```
openstack subnet create --dns-nameserver=8.8.8.8 --subnet-range 10.11.9.0/24 --network kubernetes-the-hard-way kubernetes-the-hard-way
```

> The `10.11.9.0/24` IP address range can host up to 254 compute instances.

In order to reach external hosts, a router must be configured:

```
openstack router create kubernetes-the-hard-way
# Replace ext-net with your external network if it has a different name
openstack router set --external-gateway=ext-net kubernetes-the-hard-way
openstack router add subnet kubernetes-the-hard-way kubernetes-the-hard-way
```

### Firewall Rules


Create a firewall rule that allows external SSH and HTTPS for the Kubernetes API server:

```
openstack security group create kubernetes
openstack security group rule create --dst-port=22 kubernetes
openstack security group rule create --dst-port=6443 kubernetes
```

> An external load balancer will be used to expose the Kubernetes API Servers to remote clients.


```
neutron lbaas-loadbalancer-create --name kubernetes-the-hard-way kubernetes-the-hard-way
```

### Kubernetes Public IP Address

Allocate a static IP address that will be attached to the external load balancer fronting the Kubernetes API Servers:

```
neutron floatingip-associate  \
  $(openstack floating ip create -f value -c id ext-net) \
  $(neutron lbaas-loadbalancer-show kubernetes-the-hard-way -f=value -c vip_port_id)
neutron port-update --security-group kubernetes $( neutron lbaas-loadbalancer-show kubernetes-the-hard-way -f=value -c vip_port_id)
neutron lbaas-listener-create --name=kubernetes-the-hard-way-api --loadbalancer kubernetes-the-hard-way --protocol TCP --protocol-port 6443
```

## Compute Instances

The compute instances in this lab will be provisioned using [Ubuntu Server](https://www.ubuntu.com/server) 18.04, which has good support for the [containerd container runtime](https://github.com/containerd/containerd). Each compute instance will be provisioned with a fixed private IP address to simplify the Kubernetes bootstrapping process.

### Kubernetes Controllers

Create three compute instances which will host the Kubernetes control plane:

```
openstack server create \
  --image='Ubuntu 18.04 LTS - 2018-08-10' \
  --security-group=kubernetes \
  --key-name=my-key \
  --network=kubernetes-the-hard-way \
  --flavor=l1.small \
  --min=3 --max=3 \
  kube-controller
```

Put the instances behind the LoadBalancer:

```
neutron lbaas-pool-create  --name kubernetes-the-hard-way-api-pool --listener kubernetes-the-hard-way-api --lb-algorithm ROUND_ROBIN --protocol=tcp
for server in kube-controller-{1..3}; do
  neutron lbaas-member-create \
    --subnet=kubernetes-the-hard-way \
    --address $(openstack server show $server -c addresses -f value|cut -d'=' -f2)
    --protocol-port=6443
    kubernetes-the-hard-way-api-pool
done
```

### Kubernetes Workers

Create three compute instances which will host the Kubernetes worker nodes:

```
openstack server create \
  --image='Ubuntu 18.04 LTS - 2018-08-10' \
  --security-group=kubernetes \
  --key-name=my-key \
  --network=kubernetes-the-hard-way \
  --flavor=l1.small \
  --min=3 --max=3 \
  kube-worker
```

### Verification

List the compute instances:

```
openstack server list
```

> output

```
$ openstack server list
+--------------------------------------+-------------------+--------+------------------------------------+-------------------------------+----------+
| ID                                   | Name              | Status | Networks                           | Image                         | Flavor   |
+--------------------------------------+-------------------+--------+------------------------------------+-------------------------------+----------+
| bc5d77d8-1e63-4ac4-80e7-0b636df687ca | kube-worker-3     | ACTIVE | kubernetes-the-hard-way=10.11.9.15 | Ubuntu 18.04 LTS - 2018-08-10 | l1.small |
| ed95edc9-9b7b-44f7-91de-d826f921976c | kube-worker-2     | ACTIVE | kubernetes-the-hard-way=10.11.9.2  | Ubuntu 18.04 LTS - 2018-08-10 | l1.small |
| 09e681a0-d7ea-43ca-9b61-efc14faa5f7d | kube-worker-1     | ACTIVE | kubernetes-the-hard-way=10.11.9.6  | Ubuntu 18.04 LTS - 2018-08-10 | l1.small |
| 8d3561bc-e00a-4b8b-82f9-61d15af066fb | kube-controller-3 | ACTIVE | kubernetes-the-hard-way=10.11.9.5  | Ubuntu 18.04 LTS - 2018-08-10 | l1.small |
| c0528716-87bd-45b5-8567-26f0608ebd59 | kube-controller-2 | ACTIVE | kubernetes-the-hard-way=10.11.9.13 | Ubuntu 18.04 LTS - 2018-08-10 | l1.small |
| 2d864102-373a-457a-bbd5-6e0af332bb71 | kube-controller-1 | ACTIVE | kubernetes-the-hard-way=10.11.9.9  | Ubuntu 18.04 LTS - 2018-08-10 | l1.small |
+--------------------------------------+-------------------+--------+------------------------------------+-------------------------------+----------+
```

## Configuring SSH Access

SSH will be used to configure the controller and worker instances.

Add floating IPs:

```
for server in kube-controller-{1..3} kube-worker-{1..3}; do
  openstack server add floating ip $server $(openstack floating ip create -f value -c id ext-net);
done
```

Test conecting to one of the nodes:

```
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no ubuntu@$(openstack server show kube-controller-1 -f value -c addresses|cut -d',' -f2|tr -d ' ')
```

You should be able to connect:

```
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-30-generic x86_64)

...

Last login: Sun May 13 14:34:27 2018 from XX.XXX.XXX.XX
```

Type `exit` at the prompt to exit the `kube-controller-1` compute instance:

```
$USER@kube-controller-1:~$ exit
```
> output

```
logout
Connection to XX.XXX.XXX.XXX closed
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
