# Cleaning Up

In this lab you will delete the compute resources created during this tutorial.

## Compute Instances

Delete the controller and worker compute instances:

```
for instance in kube-controller-{1..3} kube-worker-{1..3}; do openstack server delete $instance; done
```

## Networking

Delete the external load balancer network resources:

```
# Delete the lbaas pool
neutron lbaas-pool-delete kubernetes-the-hard-way-api-pool

# Delete the lbaas floating IP association
neutron floatingip-disassociate $(openstack floating ip list --port $(neutron lbaas-loadbalancer-show kubernetes-the-hard-way -f value -c vip_port_id) -f value -c "ID")

# Delete the lbaas listener
neutron lbaas-listener-delete  kubernetes-the-hard-way-api

# Delete the lbaas loadbalancer
neutron lbaas-loadbalancer-delete kubernetes-the-hard-way
```

Delete the `kubernetes-the-hard-way` security group:

```
openstack security group delete kubernete
```

Delete the `kubernetes-the-hard-way` router:

```
openstack router remove subnet kubernetes-the-hard-way kubernetes-the-hard-way
openstack router delete kubernetes-the-hard-way
```

Delete the `kubernetes-the-hard-way` network and subnet:

```
openstack subnet delete kubernetes-the-hard-way
openstack network delete kubernetes-the-hard-way
```

Delete the floating IPs. Be aware that this will delete _all_ floating IPs so do not execute it if you want to keep some:

```
for floating_ip in $(openstack floating ip list -f value -c ID); do openstack floating ip delete $floating_ip; done
```
