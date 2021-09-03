# Kubeflow Notes

## install

### install by juju

> Follow https://www.kubeflow.org/docs/distributions/charmed/install-kubeflow/ for latest instruction

```bash
sed -i 's/archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list
sed -i 's/security.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list
export DEBIAN_FRONTEND=noninteractive
```

- bootstrap cluster with external access

```bash
juju bootstrap myk8s --debug --config controller-service-type=loadbalancer
```

- patch `ingressgateway-operator` to have permission to access resources

```bash
kubectl patch role -n kubeflow istio-ingressgateway-operator -p '{"apiVersion":"rbac.authorization.k8s.io/v1","kind":"Role","metadata":{"name":"istio-ingressgateway-operator"},"rules":[{"apiGroups":["*"],"resources":["*"],"verbs":["*"]}]}'
```

### uninstall

```bash
juju destroy-model kubeflow --destroy-storage
```

or

```bash
juju destroy-model kubeflow --yes --destroy-storage --force
```

### install by kubeflow manifest

> Follow <https://github.com/kubeflow/manifests/blob/master/README.md> for latest instruction

- [install cert manager](https://github.com/kubeflow/manifests/blob/master/README.md#cert-manager)
  - need to config quay image registry
- [install istio](https://github.com/kubeflow/manifests/blob/master/README.md#istio)
  - I only applied `kustomize build common/istio-1-9/istio-crds/base | kubectl apply -f -`, as I have my own istio installed
- [install dex](https://github.com/kubeflow/manifests/blob/master/README.md#dex)
  - need to config quay image registry
  - customization must be made to work with k8s v1.21 (kubeflow version v1.4)
    - see [this issue](https://github.com/dexidp/dex/issues/2082) for solution
- [install oidc-authserver](https://github.com/kubeflow/manifests/blob/master/README.md#oidc-authservice)
  > this component cause all services on k8s to require authentication from dex, need to figure out how to config this
  - need to config gcr image registry (did it in kustomization.yaml)
  - istio-system.authservice still 0/1, seems dex service is not up yet
- [install knative](https://github.com/kubeflow/manifests/blob/master/README.md#knative)
  - need to config gcr image registry
- [kubeflow namespace](https://github.com/kubeflow/manifests/blob/master/README.md#kubeflow-namespace)
- [kubeflow roles](https://github.com/kubeflow/manifests/blob/master/README.md#kubeflow-roles)
- [kubeflow istio resources](https://github.com/kubeflow/manifests/blob/master/README.md#kubeflow-istio-resources)
- [kubeflow pipelines](https://github.com/kubeflow/manifests/blob/master/README.md#kubeflow-istio-resources)
  - we are running under containrd instead of docker, thus run

  ```bash
  kustomize build apps/pipeline/upstream/env/platform-agnostic-multi-user-pns | kubectl apply -f -
  ```

  - need to config gcr image registry
  - currently deleted due to limited resource
- [kfserving](https://github.com/kubeflow/manifests/blob/master/README.md#kfserving)
  - currently deleted due to limited resource
- [katib](https://github.com/kubeflow/manifests/blob/master/README.md#kfserving)
  - currently deleted due to limited resource
- [central dashboard](https://github.com/kubeflow/manifests/blob/master/README.md#central-dashboard)
  - currently deleted due to limited resource
- [admission webhook](https://github.com/kubeflow/manifests/blob/master/README.md#admission-webhook)
  
- currently deleted due to limited resource
