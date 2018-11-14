# Prerequisites

## Google Cloud Platform

This tutorial leverages the [Google Cloud Platform](https://cloud.google.com/) to streamline provisioning of the compute infrastructure required to bootstrap a Kubernetes cluster from the ground up. You get the GCP Account for those course from your Loodse instructor.


### Set a Default Compute Region and Zone

This tutorial assumes a default compute region and zone have been configured.

```
$ gcloud config set compute/region europe-west3
```

Set a default compute zone:

```
$ gcloud config set compute/zone europe-west3-c
```

> Use the `gcloud compute zones list` command to view additional regions and zones.

## Next

Next: [Installing the Client Tools](02-client-tools.md)
