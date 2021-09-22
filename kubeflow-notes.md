# Kubeflow Notes

## install

### install by juju

> Follow <https://www.kubeflow.org/docs/distributions/charmed/install-kubeflow> for latest instruction

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
  - `common/oidc-authservice/base/envoy-filter.yaml` will secure all requests into the k8s cluster, need to exclude some for our own services.
    - use `ExtAuthzPerRoute` to disable ext_authz:

    ```yaml
      apiVersion: networking.istio.io/v1alpha3
      kind: EnvoyFilter
      metadata:
        name: bypass-authserver-filter # a distinct name for each bypass filter
        namespace: istio-system
      spec:
        workloadSelector:
          labels:
            istio: ingressgateway
        configPatches:
          - applyTo: HTTP_ROUTE
            match:
              context: GATEWAY
              routeConfiguration:
                vhost:
                  route:
                    name: authserver-route # matches VirtualService.spec.http.name which we need to exclude for auth
            patch:
              operation: MERGE
              value:
                name: envoy.ext_authz_disabled
                typed_per_filter_config:
                  envoy.ext_authz:
                    "@type": type.googleapis.com/envoy.extensions.filters.http.ext_authz.v3.ExtAuthzPerRoute
                    disabled: true
    ```

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
- [kfserving](https://github.com/kubeflow/manifests/blob/master/README.md#kfserving)
- [katib](https://github.com/kubeflow/manifests/blob/master/README.md#kfserving)
- [central dashboard](https://github.com/kubeflow/manifests/blob/master/README.md#central-dashboard)
- [admission webhook](https://github.com/kubeflow/manifests/blob/master/README.md#admission-webhook)
- [notebooks](https://github.com/kubeflow/manifests/blob/master/README.md#notebooks)
  - notebook controller
  - jupyter web app
- [profiles + KFAM](https://github.com/kubeflow/manifests/blob/master/README.md#profiles--kfam)
- [volumes web app](https://github.com/kubeflow/manifests/blob/master/README.md#volumes-web-app)
- [tensorboard](https://github.com/kubeflow/manifests/blob/master/README.md#tensorboard)
  - tensorboard web app
  - tensorboard controller
- [tfjob operator](https://github.com/kubeflow/manifests/blob/v1.3-branch/README.md#tfjob-operator)
- [pytorch operator](https://github.com/kubeflow/manifests/blob/v1.3-branch/README.md#pytorch-operator)
- [mpi operator](https://github.com/kubeflow/manifests/blob/v1.3-branch/README.md#mpi-operator)
  - stuck here: `flag provided but not defined: -kubectl-delivery-image`
- [mxnet operator](https://github.com/kubeflow/manifests/blob/v1.3-branch/README.md#mxnet-operator)
- [xgboost operator](https://github.com/kubeflow/manifests/blob/v1.3-branch/README.md#xgboost-operator)
- [user namespace](https://github.com/kubeflow/manifests/blob/v1.3-branch/README.md#user-namespace)
