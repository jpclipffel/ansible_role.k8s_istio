# IaC / Ansible / Roles / K8S Istio

Install Istio on Kubernetes clusters.

## Usage

Include the role in your playbook using the `include_role` module (do not specify the `file` attribute):

```yaml
include_role:
  name: k8s_istio
```

Do not forget to set the common `k8s_` variables:

* `k8s_primary_master_name`
* `k8s_node_type`

## Tags

|Tag      |Description|
|---------|-----------|
|`apply`  |Install or upgrade Istio|
|`delete` |Uninstall Istio|

## Variables

Role variables are prefixed with `k8s_istio_`.

|Variable|Type|Required|Defaut|Description|
|--------|----|--------|------|-----------|
|`k8s_primary_master_name`|`string`|Yes|-|K8S cluster primary master name (as set in Ansible inventory)|
|`k8s_node_type`|`string`|Yes|-|Host role in K8S cluster (`master` or `worker`)|
||||||
|`k8s_istio_version`|`string`|No|`1.4.3`|Istio version (`major.minor.patch`)|
|`k8s_istio_profile`|`string`|No|`default`|Istio profile name|
|`k8s_istio_parameters_extra`|`list`|No|`[]` (empty list)|Istio parameters (passed to `istioctl` with `--set`)|
|`k8s_istio_sidecar_namespaces`|`string`|No|`[default]`|Namespaces watched by Istio to inject its sidecar|
|`k8s_istio_tls_crt`|`string`|No|`""` (empty string)|Istio `IngressGateway` TLS certificate|
|`k8s_istio_tls_key`|`string`|No|`""` (empty string)|Istio `IngressGateway` TLS provate key|
|`k8s_istio_ingressgw_custom`|`bool`|No|`yes`|Uses a custom `IngressGateway` manifest|
|`k8s_istio_ingressgw_ports`|`string`|No|`no`|Custom `IngressGateway` ports|

## Templates

|Template|Description|
|--------|-----------|
|`ingressgw-*.yml.j2`|Custom Istio's `IngressGateway` (king changed to `DaemonSet` with templated ports)|

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
