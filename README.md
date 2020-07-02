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

| Variable                       | Type     | Required | Defaut              | Description                                                   |
|--------------------------------|----------|----------|---------------------|---------------------------------------------------------------|
| `k8s_istio_version`            | `string` | No       | `1.4.3`             | Istio version (`major.minor.patch`)                           |
| `k8s_istio_profile`            | `string` | No       | `default`           | Istio profile name                                            |
| `k8s_istio_parameters_extra`   | `list`   | No       | `[]` (empty list)   | Istio parameters (passed to `istioctl` with `--set`)          |
| `k8s_istio_sidecar_namespaces` | `string` | No       | `[default]`         | Namespaces watched by Istio to inject its sidecar             |
| `k8s_istio_tls_crt`            | `string` | No       | `""` (empty string) | Istio `IngressGateway` TLS certificate                        |
| `k8s_istio_tls_key`            | `string` | No       | `""` (empty string) | Istio `IngressGateway` TLS provate key                        |
| `k8s_istio_ingressgw_custom`   | `bool`   | No       | `yes`               | Uses a custom `IngressGateway` manifest                       |
| `k8s_istio_ingressgw_ports`    | `string` | No       | `no`                | Custom `IngressGateway` ports                                 |
| `k8s_istio_custom_manifests`   | `list`   | No       | `[]` (empty list)   | Custom manifests to deploy (located in role's `templates/`)   |

## Ingress gateway configuration

Istio's ingress gateway inbound ports are configured by Ansible. Two set of ports are injected in the configuration:

* `k8s_istio_ingressgw_default_ports`: Defaults inbound port (DNS, status, Prometheus, etc.)
* `k8s_istio_ingressgw_ports`: Custom ports

Custom ports are defined with the same syntax as Istio manifest:

```yaml
k8s_istio_ingressgw_ports:
  # HTTP
  - containerPort: 80
    hostPort: 80
    protocol: TCP
    name: http2
  # HTTPS
  - containerPort: 443
    hostPort: 443
    protocol: TCP
    name: https
  # ...
```

| Variable        | Type               | Required | Default         | Description                                     |
|-----------------|--------------------|----------|-----------------|-------------------------------------------------|
| `containerPort` | `int`, port number | Yes      |                 | Target port in the mesh                         |
| `hostPort`      | `int`, port number | No       | `containerPort` | Port on the host OS                             |
| `protocol`      | `string`           | No       | `TCP`           | Port protocol (must be recognized by Istio)     |
| `name`          | `string`           | No       |                 | Port name (highly recomended by still optional) |

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
