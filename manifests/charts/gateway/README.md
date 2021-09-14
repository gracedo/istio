# Istio Gateway Helm Chart

This chart installs an Istio gateway deployment.

## Setup Repo Info

```console
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```

_See [helm repo](https://helm.sh/docs/helm/helm_repo/) for command documentation._

## Installing the Chart

To install the chart with the release name `istio-ingressgateway`:

```console
helm install istio-ingressgateway istio/gateway
```

## Uninstalling the Chart

To uninstall/delete the `istio-ingressgateway` deployment:

```console
helm delete istio-ingressgateway
```

## Configuration

To view support configuration options and documentation, run:

```console
helm show values istio/gateway
```

### Examples

#### Egress Gateway

Deploying a Gateway to be used as an [Egress Gateway](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway/):

```yaml
service:
  # Egress gateways do not need an external LoadBalancer IP
  type: ClusterIP
```

#### Multi-network/VM Gateway

Deploying a Gateway to be used as a [Multi-network Gateway](https://istio.io/latest/docs/setup/install/multicluster/) for network `network-1`:

```yaml
networkGateway: network-1
```

### Migrating from other installation methods

Installations from other installation methods (such as istioctl, Istio Operator, other helm charts, etc) can be migrated to use the new Helm charts
following the guidance below.
If you are able to, a clean installation is simpler. However, this often requires an external IP migration which can be challenging.

WARNING: when installing over an existing deployment, the two deployments will be merged together by Helm, which may lead to unexpected results.

#### General concerns

For a smooth migration, the resource names and `Deployment.spec.selector` labels must match.

If you install with `helm install istio-gateway istio/gateway`, resources will be named `istio-gateway` and the `selector` labels set to:

```yaml
app: istio-gateway
istio: gateway # the release name with leading istio- prefix stripped
```

If your existing installation doesn't follow these names, you can override them. For example, if you have resources named `my-custom-gateway` with `selector` labels
`foo=bar,istio=ingressgateway`:

```yaml
name: my-custom-gateway # Override the name to match existing resources
labels:
  app: "" # Unset default app selector label
  istio: ingressgateway # override default istio selector label
  foo: bar # Add the existing custom selector label
```

#### Migrating an existing Helm release

An existing helm release can be `helm upgrade`d to this chart by using the same release name. For example, if a previous
installation was done like:

```console
helm install istio-ingress manifests/charts/gateways/istio-ingress -n istio-system
```

It could be upgraded with

```console
helm upgrade istio-ingress manifests/charts/gateway -n istio-system --set name=istio-ingressgateway --set labels.app=istio-ingressgateway --set labels.istio=ingressgateway
```

Note the name and labels are overridden to match the names of the existing installation

#### Other migrations

If you see errors like `rendered manifests contain a resource that already exists` during installation, you may need to forcibly take ownership.

The script below can handle this for you. Replace `RELEASE` and `NAMESPACE` with the name and namespace of the release:

```console
KINDS=(service deployment)
RELEASE=istio-ingressgateway
NAMESPACE=istio-system
for KIND in "${KINDS[@]}"; do
    kubectl --namespace $NAMESPACE --overwrite=true annotate $KIND $RELEASE meta.helm.sh/release-name=$RELEASE
    kubectl --namespace $NAMESPACE --overwrite=true annotate $KIND $RELEASE meta.helm.sh/release-namespace=$NAMESPACE
    kubectl --namespace $NAMESPACE --overwrite=true label $KIND $RELEASE app.kubernetes.io/managed-by=Helm
done
```

You may ignore errors about resources not being found.