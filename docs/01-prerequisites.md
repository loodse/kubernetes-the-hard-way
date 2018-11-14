# Prerequisites

## Google Cloud Platform

This tutorial leverages the [Google Cloud Platform](https://cloud.google.com/) to streamline provisioning of the compute infrastructure required to bootstrap a Kubernetes cluster from the ground up. You get the GCP Account for those course from your Loodse instructor.

## Google Cloud Platform SDK

### optional: Install the Google Cloud SDK (if you're running it locally)

Follow the Google Cloud SDK [documentation](https://cloud.google.com/sdk/) to install and configure the `gcloud` command line utility.

Verify the Google Cloud SDK version is 218.0.0 or higher:

```
gcloud version
```

### Set a Default Compute Region and Zone

This tutorial assumes a default compute region and zone have been configured.

If you are using the `gcloud` command-line tool for the first time `init` is the easiest way to do this:

```
gcloud init
```

Otherwise set a default compute region:

```
gcloud config set compute/region europe-west3
```

Set a default compute zone:

```
gcloud config set compute/zone europe-west3-c
```

> Use the `gcloud compute zones list` command to view additional regions and zones.

## Next

Next: [Installing the Client Tools](02-client-tools.md)
