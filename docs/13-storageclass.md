# Adding support for storage

In order to create volumes on OpenStack a [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/#openstack-cinder) must be created.
A StorageClass enables the creation of PersistentVolumes for PersistentVolumeClaims.

```
kubectl apply -f https://raw.githubusercontent.com/loodse/kubernetes-the-hard-way/openstack/deployments/storageclass.yaml
```

Next: [Smoke Test](14-smoke-test.md)
