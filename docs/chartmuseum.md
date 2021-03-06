# ChartMuseum

https://github.com/chartmuseum/chartmuseum

### Requirements

1. You have a working Kubernetes cluster.
2. Your `kubectl` is properly configured and authorized to administer your k8s cluster.
3. You have Tiller deployed in your k8s cluster with proper RBAC configuration (optional in CaaSP).

### Installing ChartMuseum

This will install a single instance of ChartMuseum, chart storage will not be persistent.

```bash
$ kubectl run chartmuseum --image=chartmuseum/chartmuseum:latest --port=8080 -- \
--debug --port=8080 --storage=local --storage-local-rootdir=./chartstorage\

$ kubectl expose deployment chartmuseum --type=NodePort

# Find the IP of the node ChartMuseum was deployed to and the port mapped to 8080, 
# export them as $CM_IP and $CM_PORT for future commands.
```

### Publishing a chart

```bash
$ curl --data-binary "@mychart-0.1.0.tgz" http://${CM_IP}:${CM_PORT}/api/charts
```

### Add ChartMuseum as a Helm Repository

```bash
$ helm repo add chartmuseum http://${CM_IP}:${CM_PORT}
```

### Update Helm's index of ChartMuseum's charts

```bash
$ helm repo update chartmuseum
```

### Search ChartMuseums's charts

```bash
$ helm search chartmuseum/
```

### Install release from a ChartMuseum chart

```bash
$ helm install chartmuseum/mychart
```
## FAQ

### Does the API support basic chart management functionality AKA CRUD?

ChartMuseum supports create, delete, and read operations but not update. Once a chart is created it cannot be updated, only different versions of the chart can be created.

### Does ChartMuseum have any way of verifying that charts created/uploaded via the API are secure?

Only that the charts are secure in the sense that they are verified as signed by an author - trusting the chart's author is required. But this feature could be rendered useless via a MITM attack (see below).

### Is it possible to do auth to only allow some user to upload charts?

No, the API does not support any form of authentication or authorization.

### Is there some way of ensuring that there is not MITM and downloaded charts are secure? (eg. SSL / signatures, etc.)

Not really. ChartMuseum supports chart [provenance files](https://github.com/kubernetes/helm/blob/master/docs/provenance.md) which provide the ability to authenicate a chart assuming that the end user trusts the author but without a secure connection (SSL is not supported) a MITM attack could theoretically manipulate the provenance file itself, rendering it insecure.
