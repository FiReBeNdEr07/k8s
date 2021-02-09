## Rook Ceph setup

#### Prerequisites
Rook can be installed on any existing Kubernetes cluster as long as it meets the minimum version and Rook is granted the required privileges.

##### Minimum Version
Kubernetes v1.11 or higher is supported by Rook.

##### Ceph Prerequisites
In order to configure the Ceph storage cluster, at least one of these local storage options are required:

* Raw devices (no partitions or formatted filesystems)
* Raw partitions (no formatted filesystem)
* PVs available from a storage class in block mode

### TL;DR
If you’re feeling lucky, a simple Rook cluster can be created with the following kubectl commands and example yaml files.
```
git clone --single-branch --branch v1.4.9 https://github.com/rook/rook.git
cd rook/cluster/examples/kubernetes/ceph
kubectl create -f common.yaml
kubectl create -f operator.yaml
kubectl create -f cluster.yaml
```
After the cluster is running, you can create block, object, or file storage to be consumed by other applications in your cluster.