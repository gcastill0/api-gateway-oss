# Introduction
Gloo Gateway is a feature-rich, Kubernetes-native ingress controller and next-generation API gateway. 

Gloo Gateway OSS, rebranded from Gloo Edge in 2023, is a next-generation API gateway built on the Envoy Proxy. It was developed by Solo.io and designed to handle modern, Cloud-native routing and security needs.

Since its inception, Gloo Gateway has focused on providing a feature-rich, Kubernetes-native ingress controller and API gateway to support complex routing, transformation, and direct integration with legacy applications, microservices, and serverless functions.

## Requirements

| | |
| ------------- | ------------- |
| macOS or Linux | Kubernetes <br>(1.11.x or newer is required) |
| glooctl | helm |
| Python | jq |
| jq | openssl |

## Project 
Set up a demo using Gloo Edge OSS with a deployment, API entry point, and integration into a GitHub Actions workflow.

# Prepare working environment

## Install the Gloo Edge command line tool

Install the Gloo Edge command line, glooctl, to help install, configure, and debug Gloo Edge. Depending on your operating system, you have several installation options.

```bash
brew install glooctl
```

<details>
<summary>See expected results</summary>

```bash
==> Downloading https://formulae.brew.sh/api/formula.jws.json
################################################################################################################################################################################################################## 100.0%
==> Downloading https://formulae.brew.sh/api/cask.jws.json
################################################################################################################################################################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/glooctl/manifests/1.16.10
Already downloaded: /Users/gcastillo/Library/Caches/Homebrew/downloads/77db8c47d2e26f7c025baff51d762185ffab960620126b8dad1ada71f5cbfb7b--glooctl-1.16.10.bottle_manifest.json
==> Fetching glooctl
==> Downloading https://ghcr.io/v2/homebrew/core/glooctl/blobs/sha256:e6ef8e5a97b474fe4d45aaf7ac00a5993ce5303b168c2e88cdd2220dfb0dc602
Already downloaded: /Users/gcastillo/Library/Caches/Homebrew/downloads/ff78e8ffe8d6d26c09154b972cf486a05b71e77d231946cc95a396dd44b449b2--glooctl--1.16.10.arm64_sonoma.bottle.tar.gz
==> Pouring glooctl--1.16.10.arm64_sonoma.bottle.tar.gz
==> Caveats
zsh completions have been installed to:
  /opt/homebrew/share/zsh/site-functions
==> Summary
🍺  /opt/homebrew/Cellar/glooctl/1.16.10: 7 files, 146.8MB
==> Running `brew cleanup glooctl`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
```

</details>
<br>

As an alternative, you can reference the following installation script. The procedure requires Python to execute properly.

```bash
curl -sL https://run.solo.io/gloo/install | sh
export PATH=$HOME/.gloo/bin:$PATH
```
</details>
<br>
Verify the glooctl CLI is installed and running the appropriate version. In the output, the Client is your local version.

```bash
glooctl version
```

<details>
<summary>See expected results</summary>

```bash
Client: {"version":"1.16.14"}
Server: version undefined, could not find any version of gloo running
```

</details>
<br>

# Deploy Gloo Gateway

Install the Gloo Edge Gateway on your Kubernetes cluster. Make sure your Kubernetes version is compatible with Gloo Edge requirements​.

When installing Gloo Edge, two common methods are using the `glooctl` command-line tool and using Helm. Both approaches achieve the same end result—installing Gloo Edge—but they differ in terms of process, configuration, and steps involved.

## Use `glooctl`

Start by installing Gloo Edge using either glooctl or Helm. The glooctl command line tool simplifies the installation process.

```bash
cat <<EOF | glooctl install gateway --values -
gatewayProxies:
  gatewayProxy:
    service:
      type: NodePort
      httpPort: 31500
      httpsPort: 32500
      httpNodePort: 31500
      httpsNodePort: 32500
EOF
```

<details>
<summary>See expected results</summary>

```bash
Creating namespace gloo-system... Done.
Starting Gloo Edge installation...

Gloo Edge was successfully installed!
```

</details>
<br>

During the installation process `glooctl` automatically detects the Kubernetes cluster configuration from the local kubeconfig file. The tool creates a namespace, typically `gloo-system`, for Gloo Edge components. 

Also, `glooctl` applies a set of Kubernetes resource definitions (YAML manifests) to the cluster. These include Custom Resource Definitions (CRDs), Deployments, Services, ConfigMaps, and more.

## Use `helm`

Helm is a package manager for Kubernetes that uses "charts" to define, install, and upgrade complex Kubernetes applications. 

Add the official Gloo Edge Helm repository:

```bash
helm repo add gloo https://storage.googleapis.com/solo-public-helm
helm repo update
```

Run the Helm install command to deploy Gloo Edge:

```bash
helm repo add gloo https://storage.googleapis.com/solo-public-helm
helm repo update
helm install gloo gloo/gloo --namespace gloo-system --create-namespace
```

By default, Helm installs the latest version of the Gloo Edge chart. You can specify a version using the `--version` flag if needed.

The `--create-namespace` flag ensures that the `gloo-system` namespace is created if it doesn't already exist. Using `gloo-system` is our standard convention, but you can name the namespace anything you want.

Helm applies the chart, which is a collection of YAML templates that Kubernetes will use to create the necessary resources.

# Validate Deployment 

Your deployment should include all the necessary resources to deploy Gloo Edge components:

* Gloo Edge API Server
* Discovery Service
* Gateway Proxy (Envoy Proxy)
* Translator Service
* External Auth Service (if applicable)
* Rate Limiting Service (if applicable)

## Use `glooctl`

The `glooctl` check command is used to verify the health and status of Gloo Edge resources within a Kubernetes cluster. It performs a series of checks to ensure that all components are running correctly and that there are no configuration errors.

The command validates various Gloo Edge resources such as deployments, pods, upstreams, virtual services, gateways, proxies, and more. It checks if these resources are correctly defined and operating as expected.

```bash
glooctl check
```

<details>
<summary>See expected results</summary>

```bash
Checking deployments... OK
Checking pods... Note: Unhandled pod condition PodReadyToStartContainersNote: Unhandled pod condition PodReadyToStartContainersNote: Unhandled pod condition PodReadyToStartContainersOK
Checking upstreams... OK
Checking upstream groups... OK
Checking auth configs... OK
Checking rate limit configs... OK
Checking VirtualHostOptions... OK
Checking RouteOptions... OK
Checking secrets... OK
Checking virtual services... OK
Checking gateways... OK
Checking proxies... OK
No problems detected.
Skipping Gloo Instance check -- Gloo Federation not detected
```

</details>
<br>

In most cases `glooctl` check simplifies this process significantly by aggregating these checks into a single command.

## Use `kubectl`

There is no single `kubectl` command that performs all the checks glooctl check does. You can use a combination of `kubectl` commands to achieve similar checks.

Check Deployments:

```bash
kubectl get deployments -n gloo-system
```

<details>
<summary>See expected results</summary>

```bash
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
discovery       1/1     1            1           3h27m
gateway-proxy   1/1     1            1           3h27m
gloo            1/1     1            1           3h27m
```

</details>
<br>

Check Pods:

```bash
kubectl get pods -n gloo-system
```

<details>
<summary>See expected results</summary>

```bash
NAME                             READY   STATUS    RESTARTS   AGE
discovery-77855f5888-qb4nn       1/1     Running   0          3h27m
gateway-proxy-74fb449d9d-rlk8s   2/2     Running   0          3h27m
gloo-645d6b8c4d-6qw24            3/3     Running   0          3h27m
```

</details>
<br>

Check CRDs:

```bash
kubectl get crds | grep gloo
```

<details>
<summary>See expected results</summary>

```bash
authconfigs.enterprise.gloo.solo.io                 2024-05-31T15:49:33Z
graphqlapis.graphql.gloo.solo.io                    2024-05-31T15:49:34Z
proxies.gloo.solo.io                                2024-05-31T15:49:34Z
settings.gloo.solo.io                               2024-05-31T15:49:34Z
upstreamgroups.gloo.solo.io                         2024-05-31T15:49:34Z
upstreams.gloo.solo.io                              2024-05-31T15:49:34Z
```

</details>
<br>

Check the Status of Custom Resources:

```bash
kubectl get virtualservices -n gloo-system
kubectl get gateways -n gloo-system
kubectl get upstreams -n gloo-system
```

<details>
<summary>See expected results</summary>

```bash
NAME      AGE
default   172m
NAME                AGE
gateway-proxy       3h28m
gateway-proxy-ssl   3h28m
NAME                                                              AGE
default-kubernetes-443                                            3h28m
gloo-system-discovery-9091                                        3h28m
gloo-system-gateway-proxy-31500                                   3h28m
gloo-system-gateway-proxy-32500                                   3h28m
gloo-system-gateway-proxy-monitoring-service-8081                 3h28m
gloo-system-gloo-443                                              3h28m
gloo-system-gloo-9091                                             3h28m
gloo-system-gloo-9966                                             3h28m
gloo-system-gloo-9976                                             3h28m
gloo-system-gloo-9977                                             3h28m
gloo-system-gloo-9979                                             3h28m
gloo-system-gloo-9988                                             3h28m
kube-system-kube-dns-53                                           3h28m
kube-system-kube-dns-9153                                         3h28m
```

</details>
<br>

Inspect Logs:

```bash
kubectl logs -l gloo=gloo -n gloo-system
```

<details>
<summary>See expected results</summary>

```bash
Defaulted container "envoy-sidecar" out of: envoy-sidecar, sds, gloo
[2024-05-31 15:49:51.868][1][info][config] [external/envoy/source/server/configuration_impl.cc:103] loading 0 static secret(s)
[2024-05-31 15:49:51.868][1][info][config] [external/envoy/source/server/configuration_impl.cc:109] loading 3 cluster(s)
[2024-05-31 15:49:51.907][1][info][config] [external/envoy/source/server/configuration_impl.cc:113] loading 2 listener(s)
[2024-05-31 15:49:51.939][1][info][config] [external/envoy/source/server/configuration_impl.cc:130] loading stats configuration
[2024-05-31 15:49:51.940][1][info][runtime] [external/envoy/source/common/runtime/runtime_impl.cc:577] RTDS has finished initialization
[2024-05-31 15:49:51.940][1][info][upstream] [external/envoy/source/common/upstream/cluster_manager_impl.cc:226] cm init: all clusters initialized
[2024-05-31 15:49:51.943][1][info][main] [external/envoy/source/server/server.cc:918] all clusters initialized. initializing init manager
[2024-05-31 15:49:51.961][1][info][main] [external/envoy/source/server/server.cc:937] starting main dispatch loop
[2024-05-31 15:49:55.248][1][info][config] [external/envoy/source/extensions/listener_managers/listener_manager/listener_manager_impl.cc:858] all dependencies initialized. starting workers
[2024-05-31 16:04:55.253][1][info][main] [external/envoy/source/server/drain_manager_impl.cc:175] shutting down parent after drain
```

</details>
<br>

```bash
kubectl logs -l gloo=discovery -n gloo-system
```
<details>
<summary>See expected results</summary>

```bash
{"level":"info","ts":"2024-05-31T19:21:53.521Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":50}
{"level":"info","ts":"2024-05-31T19:21:53.571Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":50}
{"level":"info","ts":"2024-05-31T19:21:53.621Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":50}
{"level":"info","ts":"2024-05-31T19:21:53.670Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":50}
{"level":"info","ts":"2024-05-31T19:21:53.719Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":50}
{"level":"info","ts":"2024-05-31T19:21:53.768Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":50}
{"level":"info","ts":"2024-05-31T19:21:53.818Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":50}
{"level":"info","ts":"2024-05-31T19:21:53.867Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":50}
{"level":"info","ts":"2024-05-31T19:21:53.916Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":50}
{"level":"info","ts":"2024-05-31T19:21:53.966Z","logger":"uds.v1.event_loop.uds","caller":"discovery/discovery.go:154","msg":"reconciled upstreams","version":"1.16.14","discovered_by":"kubernetesplugin","upstreams":50}
```

</details>
<br>

```bash
kubectl logs -l gloo=gateway-proxy -n gloo-system
```

<details>
<summary>See expected results</summary>

```bash
Defaulted container "gateway-proxy" out of: gateway-proxy, sds
[2024-05-31 16:26:04.157][1][warning][misc] [external/envoy/source/common/protobuf/message_validator_impl.cc:21] Deprecated field: type envoy.config.cluster.v3.Cluster Using deprecated option 'envoy.config.cluster.v3.Cluster.http2_protocol_options' from file cluster.proto. This configuration will be removed from Envoy soon. Please see https://www.envoyproxy.io/docs/envoy/latest/version_history/version_history for details. If continued use of this field is absolutely necessary, see https://www.envoyproxy.io/docs/envoy/latest/configuration/operations/runtime#using-runtime-overrides-for-deprecated-features for how to apply a temporary and highly discouraged override.
[2024-05-31 16:26:04.157][1][warning][misc] [external/envoy/source/common/protobuf/message_validator_impl.cc:21] Deprecated field: type envoy.config.cluster.v3.Cluster Using deprecated option 'envoy.config.cluster.v3.Cluster.http2_protocol_options' from file cluster.proto. This configuration will be removed from Envoy soon. Please see https://www.envoyproxy.io/docs/envoy/latest/version_history/version_history for details. If continued use of this field is absolutely necessary, see https://www.envoyproxy.io/docs/envoy/latest/configuration/operations/runtime#using-runtime-overrides-for-deprecated-features for how to apply a temporary and highly discouraged override.
[2024-05-31 16:26:04.158][1][warning][misc] [external/envoy/source/common/protobuf/message_validator_impl.cc:21] Deprecated field: type envoy.config.cluster.v3.Cluster Using deprecated option 'envoy.config.cluster.v3.Cluster.http2_protocol_options' from file cluster.proto. This configuration will be removed from Envoy soon. Please see https://www.envoyproxy.io/docs/envoy/latest/version_history/version_history for details. If continued use of this field is absolutely necessary, see https://www.envoyproxy.io/docs/envoy/latest/configuration/operations/runtime#using-runtime-overrides-for-deprecated-features for how to apply a temporary and highly discouraged override.
[2024-05-31 16:26:04.159][1][info][upstream] [external/envoy/source/common/upstream/cds_api_helper.cc:32] cds: add 96 cluster(s), remove 5 cluster(s)
[2024-05-31 16:26:04.458][1][info][upstream] [external/envoy/source/common/upstream/cds_api_helper.cc:69] cds: added/updated 96 cluster(s), skipped 0 unmodified cluster(s)
[2024-05-31 16:26:04.476][1][info][upstream] [external/envoy/source/extensions/listener_managers/listener_manager/lds_api.cc:86] lds: add/update listener 'listener-::-8080'
[2024-05-31 16:27:26.146][1][info][upstream] [external/envoy/source/extensions/listener_managers/listener_manager/lds_api.cc:62] lds: remove listener 'listener-::-8080'
[2024-05-31 16:33:03.156][1][info][upstream] [external/envoy/source/extensions/listener_managers/listener_manager/lds_api.cc:86] lds: add/update listener 'listener-::-8443'
[2024-05-31 16:45:30.157][1][info][upstream] [external/envoy/source/extensions/listener_managers/listener_manager/lds_api.cc:86] lds: add/update listener 'listener-::-8443'
[2024-05-31 17:45:30.164][1][warning][config] [external/envoy/source/extensions/config_subscription/grpc/grpc_stream.h:152] StreamAggregatedResources gRPC config stream to gloo.gloo-system.svc.cluster.local:9977 closed: 13, 
```

</details>
<br>

You can approximate the comprehensive checks performed by `glooctl` check. 

# Summary

* **Ease of Use**: `glooctl` is simpler to use for installing Gloo Edge because it is designed specifically for this purpose and handles many default configurations automatically. `Helm` offers more flexibility and customization but requires more initial setup and understanding of Helm charts.

* **Configuration**: `glooctl` uses predefined configurations that are suitable for most standard installations. `Helm` provides a values.yaml file that allows extensive customization, making it suitable for complex and large-scale deployments.

* **Upgrade and Rollback**: `Helm` provides robust mechanisms for upgrading and rolling back installations, making it ideal for managing versions in production environments. `glooctl` also supports upgrades but lacks the fine-grained control provided by Helm.
