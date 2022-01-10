# AIX Kubernetes components

## nodes

- currently 2 master nodes (control plane nodes), 5 worker nodes
- aliyun load balancer before 2 master nodes

## etcd

- we are running a hybrid etcd topology for several reasons.
  - we use [external etcd](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/#external-etcd-topology) described here, as it's more scalable.
  - due to lack of resources, we deployed etcd cluster (2 instances) on the same machines which control plane lays on.
- deployed using [etcdadm](https://github.com/kubernetes-sigs/etcdadm)

## container runtime

- containerd as it's recommended

## kubernetes deployment

- follow [this guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/) to deploy a HA cluster.

## storage

- we use [rook](https://github.com/rook/rook) to deploy ceph on kubernetes, to provide rbd `storage class`

## RDBMS

- deployed a HA postgres cluster by [kubegres](https://github.com/reactive-tech/kubegres)
