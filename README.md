# Ansible role - `k8s_istio`

Deploy and manage Istio on Kubernetes clusters.

This role aims to manage the whole lifecycle of an Istio deployment:

* Installation and deployment
* Resources customization (e.g. `IngressGateway` ports)
* Custom resources deployment (e.g. generic `Gateway`)
* Upgrade (downgrade not yet fully supported)

## Tags

| Tag      | Description                                      |
|----------|--------------------------------------------------|
| `apply`  | Install, deploy, upgrade and (re)configure Istio |
| `delete` | Uninstall Istio and remove its resources         |

## Variables

| Variable                          | Type     | Required | Defaut              | Description                                                  |
|-----------------------------------|----------|----------|---------------------|--------------------------------------------------------------|
| `k8s_istio_version`               | `string` | No       | `1.4.3`             | Istio version (`major.minor.patch`)                          |
| `k8s_istio_profile`               | `string` | No       | `default`           | Istio profile name                                           |
| `k8s_istio_parameters_extra`      | `list`   | No       | `[]` (empty list)   | Istio parameters (passed to `istioctl` with `--set`)         |
| `k8s_istio_sidecar_namespaces`    | `string` | No       | `[default]`         | Namespaces watched by Istio to inject its sidecar            |
| `k8s_istio_tls_crt`               | `string` | No       | `""` (empty string) | Istio `IngressGateway` TLS certificate                       |
| `k8s_istio_tls_key`               | `string` | No       | `""` (empty string) | Istio `IngressGateway` TLS provate key                       |
| `k8s_istio_ingressgw_annotations` | `dict`   | No       | `{}` (empty dict)   | Istio `IngressGateway` annotations (e.g. for `MetalLB` pool) |
| `k8s_istio_ingressgw_custom`      | `bool`   | No       | `yes`               | Uses a custom `IngressGateway` manifest                      |
| `k8s_istio_ingressgw_ports`       | `string` | No       | `no`                | Custom `IngressGateway` ports                                |
| `k8s_istio_custom_manifests`      | `list`   | No       | `[]` (empty list)   | Custom manifests to deploy (located in role's `templates/`)  |

## Ingress gateway configuration

Istio's ingress gateway inbound ports are configured by Ansible. Two set of ports are injected in the configuration:

* `k8s_istio_ingressgw_default_ports`: Defaults inbound port (DNS, status, Prometheus, etc.)
* `k8s_istio_ingressgw_ports`: Custom ports

It is strongly advised to provide only the `port` and `name` fields, which are generic and compatible with all deployment type:

```yaml
ports:
- port: <int>           # Required; Port number
  name: <str>           # Optional; Port protocol and name; Syntax is <protocol>-<suffix>; Defaults to 'tcp-<port>'
```

When deploying default Istio (i.e. without custom ingress gateway template), each `port` item should provide the following values:

```yaml
ports:
- port: <int>           # Required; Port number
  targetPort: <int>     # Optional; Target port number; Defaults to <port>.
  name: <str>           # Optional; Port protocol and name; Syntax is <protocol>-<suffix>; Defaults to 'tcp-<port>'
                        # More info: https://istio.io/latest/docs/ops/configuration/traffic-management/protocol-selection/
```

When deploying Istio with a custom ingress gateway template, each `port` item should provide the following values:

```yaml
ports:
- port: <int>           # Optional; Port number
  containerPort: <int>  # Optional; Container port number; Defaults to <port>
  hostPort: <int>       # Optional; Host port number; Defaults to <port>
  protocol: <str>       # Optional; Protocol name; Defaults to 'tcp'
  name: <str>           # Optional; Port name; No defaults
```

## Generic gateway

You can deploy a generic istio `Gateway` using this role.

* This `Gateway` is SSL-aware
* This `Gateway` is deployed in `istio-system` namespace; You can refeer to it using `istio-system/istio-gateway` (see example bellow)

Example usage in an Istio's `VirtualService`:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: test-vs
  namespace: default
spec:
  hosts:
  - "*"
  gateways:
  - "istio-system/istio-gateway"
  http:
  - route:
    - destination:
        host: test-svc
        port:
          number: 80
```

## Templates and custom manifests

| Template                | Description                                                                          |
|-------------------------|--------------------------------------------------------------------------------------|
| `values.yml.j2`         | Default Istio's `IngressGateway` customization                                       |
| `ingressgw-*.yml.j2`    | Custom Istio's `IngressGateway` (`kind` changed to `DaemonSet` with templated ports) |
| `custom/gateway.yml.j2` | Generic and SSL-aware `Gateway`                                                      |

## Tasks sequence

```text
main.yml
|
| -------------------------------- tags: apply --
\__ delete.yml
|
\__ apply.yml
|
\__ ingressgw.yml
|
\__ test.yml
|
|
| ------------------------------- tags: delete --
\__ delete.yml
```
