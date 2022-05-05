# matrixdb-operator

Kubernetes operator for MatrixDB cluster deployment.

For document in Chinese, see: [Document on ymatrix.cn](https://ymatrix.cn/doc/latest/install/kubernetes)

## Prerequisites

- Kubernetes 1.20+
  - RBAC enabled
  - DNS enabled
  - StorageClass that is suitable for running database
- cert-manager 1.6+

## Installation

### Install `cert-manager`

See: https://artifacthub.io/packages/helm/cert-manager/cert-manager

Make sure the parameter `installCRDs` is `true`.

### Install from `ymatrix` helm repository

```
helm repo add ymatrix https://ymatrix-data.github.io/charts
helm repo update

helm upgrade --install \
  matrixdb-operator ymatrix/matrixdb-operator \
  --namespace matrixdb-system \
  --create-namespace
```

## Usage

### Prepare a namespace for MatrixDB cluster to be deployed

It is recommended to deploy the cluster in a standalone namespace.

### Prepare yaml file of CRD

Check the CRD examples from [artifacthub.io](https://artifacthub.io/packages/helm/ymatrix/matrixdb-operator),
or check the [full documentation](https://ymatrix.cn/doc/latest/install/kubernetes_crd) of CRD yaml file supported.

Prepare a yaml file `db0.yaml` and change the parameters based on your requirement.

### Deploy a MatrixDB cluster

Then apply it to Kubernetes cluster

```
kubectl apply -f db0.yaml --namespace your-db-namespace
```

### Check the status of cluster

Check the status of DB cluster

```
kubectl get mxdb --namespace your-db-namespace
```

When the state is `Running`, the DB is fully deployed.

### Use the deployed database

Run the following command to get the services of the cluster

```
kubectl get svc --namespace your-db-namespace
```

The output should be something like this:

```
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
db0            ClusterIP   None             <none>        22/TCP     7d22h
db0-cylinder   ClusterIP   172.20.148.232   <none>        4637/TCP   7d22h
db0-gate       ClusterIP   None             <none>        4617/TCP   7d22h
db0-pg         ClusterIP   172.20.216.189   <none>        5432/TCP   7d22h
db0-ui         ClusterIP   172.20.157.163   <none>        8240/TCP   7d22h
```

`*-ui` is the service of UI which can be accessed from browser.
`*-pg` is the service of MatrixDB which can be accessed with a DB client.

The default user is `mxadmin` and default password is `changeme`.
Make sure to change this password before actually using the DB cluster.
