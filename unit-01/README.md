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
discovery       1/1     1            1           2m32s
gateway-proxy   1/1     1            1           2m32s
gloo            1/1     1            1           2m32s
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
NAME                                READY   STATUS      RESTARTS   AGE
discovery-5d58dcbf7f-jb9lv          1/1     Running     0          3m5s
gateway-proxy-6b648fc7b9-c22z2      1/1     Running     0          3m5s
gloo-77cb789d9f-mm78p               1/1     Running     0          3m5s
gloo-resource-rollout-check-jjcdm   0/1     Completed   0          3m5s
gloo-resource-rollout-rxrzs         0/1     Completed   0          3m5s```

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

Check the Gloo Gateway Virtual services:

```bash
kubectl get virtualservices -n gloo-system
```

<details>
<summary>See expected results</summary>

```bash
No resources found in gloo-system namespace.
```

</details>
<br>

Check Gloo Gateway access points:

```bash
kubectl get gateways -n gloo-system
```

<details>
<summary>See expected results</summary>

```bash
NAME                AGE
gateway-proxy       3m49s
gateway-proxy-ssl   3m49s
```

</details>
<br>


Check the Gloo Gateway Upstreams:

```bash
kubectl get upstreams -n gloo-system
```

<details>
<summary>See expected results</summary>

```bash
NAME                              AGE
default-kubernetes-443            3m51s
gloo-system-gateway-proxy-31500   3m51s
gloo-system-gateway-proxy-32500   3m51s
gloo-system-gloo-443              3m51s
gloo-system-gloo-9966             3m51s
gloo-system-gloo-9976             3m51s
gloo-system-gloo-9977             3m51s
gloo-system-gloo-9979             3m51s
gloo-system-gloo-9988             3m51s
kube-system-kube-dns-53           3m51s
kube-system-kube-dns-9153         3m51s
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

You can approximate the comprehensive checks performed by `glooctl` check. 

# Summary

* **Ease of Use**: `glooctl` is simpler to use for installing Gloo Edge because it is designed specifically for this purpose and handles many default configurations automatically. `Helm` offers more flexibility and customization but requires more initial setup and understanding of Helm charts.

* **Configuration**: `glooctl` uses predefined configurations that are suitable for most standard installations. `Helm` provides a values.yaml file that allows extensive customization, making it suitable for complex and large-scale deployments.

* **Upgrade and Rollback**: `Helm` provides robust mechanisms for upgrading and rolling back installations, making it ideal for managing versions in production environments. `glooctl` also supports upgrades but lacks the fine-grained control provided by Helm.
