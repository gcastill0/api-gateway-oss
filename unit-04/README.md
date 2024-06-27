# Default Logs and Metrics

### Inspect logs

```bash
kubectl logs -l gloo=gloo -n gloo-system
```

<details>
<summary>See expected results</summary>

```bash
{"level":"info","ts":"2024-05-31T20:06:31.126Z","logger":"gloo.v1.event_loop.setup.gateway-validation-webhook","caller":"k8sadmission/validating_admission_webhook.go:198","msg":"received validation request on webhook","version":"1.16.14"}
{"level":"info","ts":"2024-05-31T20:06:31.130Z","logger":"gloo.v1.event_loop.setup.gateway-validation-webhook","caller":"k8sadmission/validating_admission_webhook.go:266","msg":"ready to write response ...","version":"1.16.14"}
{"level":"info","ts":"2024-05-31T20:06:31.135Z","logger":"gloo.v1.event_loop.setup.gateway-validation-webhook","caller":"k8sadmission/validating_admission_webhook.go:198","msg":"received validation request on webhook","version":"1.16.14"}
{"level":"info","ts":"2024-05-31T20:06:31.139Z","logger":"gloo.v1.event_loop.setup.gateway-validation-webhook","caller":"k8sadmission/validating_admission_webhook.go:266","msg":"ready to write response ...","version":"1.16.14"}
{"level":"info","ts":"2024-05-31T20:06:31.144Z","logger":"gloo.v1.event_loop.setup.gateway-validation-webhook","caller":"k8sadmission/validating_admission_webhook.go:198","msg":"received validation request on webhook","version":"1.16.14"}
{"level":"info","ts":"2024-05-31T20:06:31.147Z","logger":"gloo.v1.event_loop.setup.gateway-validation-webhook","caller":"k8sadmission/validating_admission_webhook.go:266","msg":"ready to write response ...","version":"1.16.14"}
{"level":"info","ts":"2024-05-31T20:06:31.152Z","logger":"gloo.v1.event_loop.setup.gateway-validation-webhook","caller":"k8sadmission/validating_admission_webhook.go:198","msg":"received validation request on webhook","version":"1.16.14"}
{"level":"info","ts":"2024-05-31T20:06:31.155Z","logger":"gloo.v1.event_loop.setup.gateway-validation-webhook","caller":"k8sadmission/validating_admission_webhook.go:266","msg":"ready to write response ...","version":"1.16.14"}
{"level":"info","ts":"2024-05-31T20:06:31.160Z","logger":"gloo.v1.event_loop.setup.gateway-validation-webhook","caller":"k8sadmission/validating_admission_webhook.go:198","msg":"received validation request on webhook","version":"1.16.14"}
{"level":"info","ts":"2024-05-31T20:06:31.164Z","logger":"gloo.v1.event_loop.setup.gateway-validation-webhook","caller":"k8sadmission/validating_admission_webhook.go:266","msg":"ready to write response ...","version":"1.16.14"}
```

</details>
<br>

```bash
kubectl logs -l gloo=discovery -n gloo-system
```
<details>
<summary>See expected results</summary>

```bash
{"level":"info","ts":"2024-05-31T20:10:27.979Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":12}
{"level":"info","ts":"2024-05-31T20:10:27.987Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":12}
{"level":"info","ts":"2024-05-31T20:10:27.995Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":12}
{"level":"info","ts":"2024-05-31T20:10:28.004Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":12}
{"level":"info","ts":"2024-05-31T20:10:28.012Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":12}
{"level":"info","ts":"2024-05-31T20:10:28.020Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":12}
{"level":"info","ts":"2024-05-31T20:10:28.029Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":12}
{"level":"info","ts":"2024-05-31T20:10:28.038Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":12}
{"level":"info","ts":"2024-05-31T20:10:28.046Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":12}
{"level":"info","ts":"2024-05-31T20:10:28.054Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":12}
```

</details>
<br>

```bash
kubectl logs -l gloo=gateway-proxy -n gloo-system
```

<details>
<summary>See expected results</summary>

```bash
[2024-05-31 20:06:24.524][1][warning][misc] [external/envoy/source/common/protobuf/message_validator_impl.cc:21] Deprecated field: type envoy.config.route.v3.HeaderMatcher Using deprecated option 'envoy.config.route.v3.HeaderMatcher.exact_match' from file route_components.proto. This configuration will be removed from Envoy soon. Please see https://www.envoyproxy.io/docs/envoy/latest/version_history/version_history for details. If continued use of this field is absolutely necessary, see https://www.envoyproxy.io/docs/envoy/latest/configuration/operations/runtime#using-runtime-overrides-for-deprecated-features for how to apply a temporary and highly discouraged override.
[2024-05-31 20:06:24.544][1][info][config] [external/envoy/source/server/configuration_impl.cc:130] loading stats configuration
[2024-05-31 20:06:24.548][1][info][main] [external/envoy/source/server/server.cc:937] starting main dispatch loop
[2024-05-31 20:06:24.552][1][info][runtime] [external/envoy/source/common/runtime/runtime_impl.cc:577] RTDS has finished initialization
[2024-05-31 20:06:24.552][1][info][upstream] [external/envoy/source/common/upstream/cluster_manager_impl.cc:222] cm init: initializing cds
[2024-05-31 20:06:30.069][1][info][upstream] [external/envoy/source/common/upstream/cds_api_helper.cc:32] cds: add 0 cluster(s), remove 4 cluster(s)
[2024-05-31 20:06:30.069][1][info][upstream] [external/envoy/source/common/upstream/cds_api_helper.cc:69] cds: added/updated 0 cluster(s), skipped 0 unmodified cluster(s)
[2024-05-31 20:06:30.069][1][info][upstream] [external/envoy/source/common/upstream/cluster_manager_impl.cc:226] cm init: all clusters initialized
[2024-05-31 20:06:30.070][1][info][main] [external/envoy/source/server/server.cc:918] all clusters initialized. initializing init manager
[2024-05-31 20:06:30.076][1][info][config] [external/envoy/source/extensions/listener_managers/listener_manager/listener_manager_impl.cc:858] all dependencies initialized. starting workers
```

</details>
<br>

### Inspect Metrics

```bash
kubectl -n gloo-system port-forward deployment/gateway-proxy 19000
```

```bash
curl http://localhost:19000/stats/prometheus
```

<details>
<summary>See expected results</summary>

```bash
...
# TYPE envoy_server_initialization_time_ms histogram
envoy_server_initialization_time_ms_bucket{le="0.5"} 0
envoy_server_initialization_time_ms_bucket{le="1"} 0
envoy_server_initialization_time_ms_bucket{le="5"} 0
envoy_server_initialization_time_ms_bucket{le="10"} 0
envoy_server_initialization_time_ms_bucket{le="25"} 0
envoy_server_initialization_time_ms_bucket{le="50"} 0
envoy_server_initialization_time_ms_bucket{le="100"} 0
envoy_server_initialization_time_ms_bucket{le="250"} 0
envoy_server_initialization_time_ms_bucket{le="500"} 0
envoy_server_initialization_time_ms_bucket{le="1000"} 0
envoy_server_initialization_time_ms_bucket{le="2500"} 0
envoy_server_initialization_time_ms_bucket{le="5000"} 0
envoy_server_initialization_time_ms_bucket{le="10000"} 1
envoy_server_initialization_time_ms_bucket{le="30000"} 1
envoy_server_initialization_time_ms_bucket{le="60000"} 1
envoy_server_initialization_time_ms_bucket{le="300000"} 1
envoy_server_initialization_time_ms_bucket{le="600000"} 1
envoy_server_initialization_time_ms_bucket{le="1800000"} 1
envoy_server_initialization_time_ms_bucket{le="3600000"} 1
envoy_server_initialization_time_ms_bucket{le="+Inf"} 1
envoy_server_initialization_time_ms_sum{} 5550
envoy_server_initialization_time_ms_count{} 1
```

</details>
<br>

---

# Install and configure Loki and Grafana

Install Grafana and Loki on Kubernetes using Helm. Here are the steps and commands to do this:

### 1. Install Helm

If Helm is not already installed, you can install it by following the instructions on the [Helm installation guide](https://helm.sh/docs/intro/install/).

### 2. Add Helm Repositories

First, add the Helm repositories for Grafana and Loki.

```sh
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### 3. Install Loki

Loki is a log aggregation system optimized for use with Prometheus and Grafana.

```sh
helm upgrade --install loki grafana/loki-stack --namespace monitoring --create-namespace
```

This command installs Loki along with Promtail (a log shipping agent) and Grafana in a namespace called `monitoring`. If the namespace does not exist, it will be created.

### 4. Install Grafana

If you need a standalone Grafana instance, you can install it with:

```sh
helm upgrade --install grafana grafana/grafana --namespace monitoring
```

This command installs Grafana in the `monitoring` namespace.

### 5. Accessing Grafana

After installing Grafana, you can access it by port-forwarding to the Grafana service. Run the following command:

```sh
kubectl port-forward --namespace monitoring svc/grafana 3000:80
```

Then, open your browser and navigate to `http://localhost:3000`.

### 6. Default Credentials

The default username and password for Grafana are typically:

- **Username**: `admin`
- **Password**: `prom-operator` (or `admin` for standalone Grafana)

You can retrieve the password with:

```sh
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### 7. Configuring Loki as a Data Source in Grafana

To configure Loki as a data source in Grafana:

1. Open Grafana in your browser.
2. Go to **Configuration > Data Sources**.
3. Click **Add data source**.
4. Select **Loki**.
5. Set the URL to `http://loki:3100`.
6. Click **Save & Test**.

---

# Install and configure Prometheus

Install Prometheus on Kubernetes using Helm. Below are the steps to install Prometheus using the Prometheus Community Helm chart.

### 1. Add the Prometheus Community Helm Repository

Add the Prometheus Community Helm repository to your Helm configuration:

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 2. Install Prometheus

Use Helm to install Prometheus in a namespace called `monitoring`. If the namespace doesn't exist, it will be created:

```sh
helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
```

### 3. Verify the Installation

Check that Prometheus pods are running:

```sh
kubectl get pods --namespace monitoring
```

You should see pods with names starting with `prometheus-server`, `prometheus-alertmanager`, etc.

### 4. Accessing Prometheus

To access the Prometheus server, you can port-forward the Prometheus service to your local machine:

```sh
kubectl port-forward --namespace monitoring svc/prometheus-server 9090:80
```

Then, open your browser and navigate to `http://localhost:9090` to access the Prometheus UI.

### 5. Configure Grafana to Use Prometheus 

After Grafana is installed, configure it to use Prometheus and Loki as data sources.

#### Port-Forward to Access Grafana

```sh
kubectl port-forward --namespace monitoring svc/grafana 3000:80
```

Open your browser and navigate to `http://localhost:3000`.

#### Add Prometheus as a Data Source

1. Open Grafana in your browser.
2. Go to **Configuration > Data Sources**.
3. Click **Add data source**.
4. Select **Prometheus**.
5. Set the URL to `http://prometheus-server.monitoring.svc.cluster.local`.
6. Click **Save & Test**.