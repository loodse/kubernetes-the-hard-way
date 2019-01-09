# Configuring kubectl for Remote Access

In this lab you will generate a kubeconfig file for the `kubectl` command line utility based on the `admin` user credentials.

> Run the commands in this lab from the same directory used to generate the admin client certificates.

## The Admin Kubernetes Configuration File

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used.

Generate a kubeconfig file suitable for authenticating as the `admin` user:

```
{
  LOADBALANCER_IP=$(openstack floating ip list --port $(neutron lbaas-loadbalancer-show kubernetes-the-hard-way -f=value -c vip_port_id) -f value -c "Floating IP Address")

  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${LOADBALANCER_IP}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```

## Verification

Check the health of the remote Kubernetes cluster:

```
kubectl get componentstatuses
```

> output

```
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health":"true"}
etcd-2               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

> output

```
NAME                STATUS   ROLES    AGE    VERSION
kube-controller-1   Ready    <none>   35s   v1.12.0
kube-controller-2   Ready    <none>   36s   v1.12.0
kube-controller-3   Ready    <none>   36s   v1.12.0
kube-worker-1       Ready    <none>   36s   v1.12.0
kube-worker-2       Ready    <none>   36s   v1.12.0
kube-worker-3       Ready    <none>   36s   v1.12.0
```

Next: [Provisioning CNI](11-pod-network-routes.md)
