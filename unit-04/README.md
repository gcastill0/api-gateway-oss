Inspect Logs:

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