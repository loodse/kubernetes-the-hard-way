# Provisioning CNI

In order for Pods to be able to communicate with each other, a CNI solution must be deployed. For this we will be using [Flannel](https://github.com/coreos/flannel)

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```

Next: [Deploying the DNS Cluster Add-on](12-dns-addon.md)
