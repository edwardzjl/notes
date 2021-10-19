# Kubernetes Notes

## Pre requirements

### Prepare user

- create user

```bash
adduser aix
```

- add user to sudo group

```bash
adduser aix sudo
```

- (optional) config default editor

```bash
update-alternatives --config editor
```

- (optional) allow user to execute sudo without password

```bash
visudo
```

add

```bash
$USER ALL=(ALL) NOPASSWD: ALL
```

### Prepare container runtime (containerd in my case)

- install dependencies

```bash
sudo apt update
sudo apt install libseccomp2
```

- dowload tarball (to fix version accross nodes, it was predownloaded)
- install containerd

```bash
sudo tar --no-overwrite-dir -C / -xzf cri-containerd-cni-${VERSION}-linux-amd64.tar.gz
sudo systemctl daemon-reload
sudo systemctl start containerd
```

- config containerd
  - generate default config

    ```bash
    mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml
    ```

  - change cgroup

    ```toml
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    ...
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
      SystemdCgroup = true
    ```

  - change pause image registry

    ```toml
    [plugins."io.containerd.grpc.v1.cri"]
    ...
    # sandbox_image = "k8s.gcr.io/pause:3.5"
    sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.5"
    ```

  - enable and restart containerd

    ```bash
    sudo systemctl enable containerd
    sudo systemctl restart containerd
    ```

## Install

- enable ipv4 ip forward

  ```bash
  echo 1 | tee /proc/sys/net/ipv4/ip_forward
  ```

- aliyun mirror for kubeadm, kubelet and kubectl

  ```bash
  sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg
  echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
  ```

- install specific version

  ```bash
  sudo apt install -qy kubeadm=1.21.2-00 kubelet=1.21.2-00 kubectl=1.21.2-00
  ```

## Reset

- remove all other nodes

  ```bash
  # must add --ignore-daemonsets to delete kubernetes cluster nodes
  kubectl drain $node_name --ignore-daemonsets --delete-local-data
  kubectl delete node $node_name
  ```

- reset by kubeadm

  ```bash
  kubeadm reset
  ```

- remove kubernetes remainings

  ```bash
  rm -rf /etc/kubernetes
  ```

- remove cni remainings

  ```bash
  rm -rf /etc/cni
  rm -rf /var/lib/cni
  ```

- reset iptables

  ```bash
  iptables -F
  iptables -X
  ```

- clear ip config

  ```bash
  ip netns list #check whether there's cni remaining
  ip netns delete $remaining_ip
  ```

- restart container runtime

  ```bash
  ctr c ls # check whether there's remaining containers, if yes, clear them
  systemctl restart containerd # depends on your container runtime
  ```

- if using external `etcd`, might need to vacuum it
  - I just reset my `etcd` cluster by `etcdadm`

## Join nodes

- join worker
  - print join command on control-plane

  ```bash
  sudo kubeadm join k8s.zjuici.com:6443 --token r2f998.7oyadiiynlqb9tif --discovery-token-ca-cert-hash sha256:184d341c7a928698e6e3b651408319c5562de7b2a33c9f37341edba0abd89053
  ```

- join master
  - re upload certs in the already working master node

  ```bash
  sudo kubeadm init phase upload-certs --upload-certs
  ```

  - print join command in the already working master node

  ```bash
  kubeadm token create --print-join-command
  ```

  - Add the --control-plane --certificate-key and execute

  ```bash
   ${join command from step 02} --control-plane --certificate-key ${key from step 01}
  ```

## Management

- rolling restart deploy

```bash
kubectl rollout restart deployment/demo
```

### Network

- get services ip range

```bash
kubectl cluster-info dump | grep -m 1 service-cluster-ip-range
```

- get pods ip range

```bash
kubectl cluster-info dump | grep -m 1 cluster-cidr
```

### Nodes

- get node labels

```bash
kubectl get nodes --show-labels
```

- add label to node

```bash
kubectl label nodes <node-name> <label-key>=<label-value>
```

- remove label from node

```bash
kubectl label nodes <node-name> <label-key>-
```

### Manage storage

- run rook-ceph-tool

```bash
kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') -- bash
```

- check ceph status

```bash
ceph status
ceph osd status
ceph df
rados df
```

- execute this command after node restart

> see <https://github.com/rook/rook/issues/3640#issuecomment-522262868> for more details

```bash
sudo vgchange -ay
```

## Configure cloud provider

> Our cluster is hosted by aliyun ecs, but not created by ACK Kubernetes(Aliyun Container Service Kubernetes).
> Thus we need to configure cloud provider, to enable LoadBalancer service to have an external-ip

- execute following commands to get node's region-id and instance-id

  ```bash
  META_EP=http://100.100.100.200/latest/meta-data
  echo `curl -s $META_EP/region-id`.`curl -s $META_EP/instance-id`
  ```

- patch each node with following command:

  ```bash
  kubectl patch node ${NODE_NAME} -p "{\"spec\":{\"providerID\": \"${region-id}.${instance-id}\" }}"
  ```

- follow [Install Alibaba CloudProvider support](https://github.com/kubernetes/cloud-provider-alibaba-cloud/blob/master/docs/getting-started.md#install-alibaba-cloudprovider-support)
