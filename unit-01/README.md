# 1 - Introduction
Gloo Gateway is a feature-rich, Kubernetes-native ingress controller and next-generation API gateway. 

Gloo Gateway OSS, rebranded from Gloo Edge in 2023, is a next-generation API gateway built on the Envoy Proxy. It was developed by Solo.io and designed to handle modern, Cloud-native routing and security needs.

Since its inception, Gloo Gateway has focused on providing a feature-rich, Kubernetes-native ingress controller and API gateway to support complex routing, transformation, and direct integration with legacy applications, microservices, and serverless functions.

## 1.1 - Requirements

| | |
| ------------- | ------------- |
| macOS or Linux |  This exercise is tested and validated on <br> macOS 14.4.1 (Sonoma) and Ubuntu 22.04.2 |
| Kubernetes | Gloo Edge Enterprise is supported and tested for the <br> latest Kubernetes version and all Kubernetes versions <br>  released up to 1 year before the latest version. <br> <br> Gloo Edge supports a large collection of Kubernetes variants <br> including MiniKube, Minishift, Kind, OpenShift, GKE, and AKS <br><br> This exercise is tested and validated on version 1.27.1 and 1.29.2 | 
| glooctl | Command line utility for Gloo Edge |
| Python | Used by the Gloo Edge install script |
| helm | Utility for managing charts and deploying applications on Kubernetes |
| jq | Utility for manipulating JSON |
| curl | Utility for transferring data to/from a server, especially with HTTP/S |
| openssl | Cryptography toolkit for working with SSL and TLS |

## 1.2 - Objectives 

Set up a demo using Gloo Edge OSS with a deployment, API entry point, and integration into a GitHub Actions workflow.


# 2 - Prepare working environment

## 2.1 - Install the Gloo Edge command line tool

Install the Gloo Edge command line, glooctl, to help install, configure, and debug Gloo Edge. Depending on your operating system, you have several installation options.

```bash
# On macOS
brew install glooctl
```

<details>
<summary>See sample results</summary>

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

You can reference the following installation script on most Linux variants. The procedure requires Python to execute properly.

```bash
# On most Linux variants
curl -sL https://run.solo.io/gloo/install | sh
export PATH=$HOME/.gloo/bin:$PATH
```

<details>
<summary>See sample results</summary>

```
Using /usr/bin/python3
Python 3.12. Use setuptools or check PEP 632 for potential alternatives
Attempting to download glooctl version v1.16.14
Downloading glooctl-linux-amd64...
Download complete!, validating checksum...
Checksum valid.
Gloo Edge was successfully installed 🎉

Add the gloo CLI to your path with:
  export PATH=$HOME/.gloo/bin:$PATH

Now run:
  glooctl install gateway     # install gloo's function gateway functionality into the 'gloo-system' namespace
  glooctl install ingress     # install very basic Kubernetes Ingress support with Gloo into namespace gloo-system
  glooctl install knative     # install Knative serving with Gloo configured as the default cluster ingress
Please see visit the Gloo Installation guides for more:  https://docs.solo.io/gloo-edge/latest/installation/
```
</details>
<br>
Verify the glooctl CLI is installed and running the appropriate version. In the output, the Client is your local version.

```bash
glooctl version
```

<details>
<summary>See sample results</summary>

```bash
Client: {"version":"1.16.14"}
Server: version undefined, could not find any version of gloo running
```

</details>
<br>

# 3 - Deploy Gloo Gateway

Install the Gloo Edge Gateway on your Kubernetes cluster. Make sure your Kubernetes version is compatible with Gloo Edge requirements​.

When installing Gloo Edge, two common methods are using the `glooctl` command-line tool and using Helm. Both approaches achieve the same end result—installing Gloo Edge—but they differ in terms of process, configuration, and steps involved.

## 3.1 - Use `glooctl`

Start by installing Gloo Edge using either `glooctl` or `Helm`. The `glooctl` command line tool simplifies the installation process. 

The [Chart Values](https://docs.solo.io/gloo-edge/latest/reference/helm_chart_values/open_source_helm_chart_values) page describes all the values that you can override in your custom values file when working with the Helm chart for Open Source Gloo Edge. 

The following dynamic use of these values helps us establish mTLS as a standard, and enables the publication of a metrics service through Envoy.

```bash
cat <<EOF | glooctl install gateway --values -
global:
  glooMtls:
    enabled: true
  glooStats:
    enabled: true
    serviceMonitorEnabled: true
    podMonitorEnabled: true
    enableStatsRoute: true
EOF
```

<details>
<summary>See sample results</summary>

```bash
Creating namespace gloo-system... Done.
Starting Gloo Edge installation...

Gloo Edge was successfully installed!
```

</details>
<br>

During the installation process `glooctl` automatically detects the Kubernetes cluster configuration from the local kubeconfig file. The tool creates a namespace, typically `gloo-system`, for Gloo Edge components. 

Also, `glooctl` applies a set of Kubernetes resource definitions (YAML manifests) to the cluster. These include Custom Resource Definitions (CRDs), Deployments, Services, ConfigMaps, and more.

## 3.2 - Use `helm`

Helm is a package manager for Kubernetes that uses "charts" to define, install, and upgrade complex Kubernetes applications. 

Add the official Gloo Edge Helm repository:

```bash
helm repo add gloo https://storage.googleapis.com/solo-public-helm
helm repo update
```

<details>
<summary>See sample results</summary>

```bash
"gloo" has been added to your repositories
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "gloo" chart repository
Update Complete. ⎈Happy Helming!⎈
```

</details>
<br>

Run the Helm install command to deploy Gloo Edge:

```bash
helm install gloo gloo/gloo --namespace gloo-system --create-namespace
```

<details>
<summary>See sample results</summary>

```bash
NAME: gloo
LAST DEPLOYED: Mon Jun  3 18:50:42 2024
NAMESPACE: gloo-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

</details>
<br>

By default, Helm installs the latest version of the Gloo Edge chart. You can specify a version using the `--version` flag if needed.

The `--create-namespace` flag ensures that the `gloo-system` namespace is created if it doesn't already exist. Using `gloo-system` is our standard convention, but you can name the namespace anything you want.

Helm applies the chart, which is a collection of YAML templates that Kubernetes will use to create the necessary resources.

# 4 - Validate Deployment 

Your deployment should include all the necessary resources to deploy Gloo Edge components:

* Gloo Edge API Server
* Discovery Service
* Gateway Proxy (Envoy Proxy)
* Translator Service
* External Auth Service (if applicable)
* Rate Limiting Service (if applicable)

## 4.1 - Use `glooctl`

The `glooctl` check command is used to verify the health and status of Gloo Edge resources within a Kubernetes cluster. It performs a series of checks to ensure that all components are running correctly and that there are no configuration errors.

The command validates various Gloo Edge resources such as deployments, pods, upstreams, virtual services, gateways, proxies, and more. It checks if these resources are correctly defined and operating as expected.

```bash
glooctl check
```

<details>
<summary>See sample results</summary>

```bash
Checking deployments... OK
Checking pods... OK
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

## 4.2 - Use `kubectl`

There is no single `kubectl` command that performs all the checks glooctl check does. You can use a combination of `kubectl` commands to achieve similar checks.

Check Pods:

```bash
kubectl get pods -n gloo-system
```

<details>
<summary>See sample results</summary>

```bash
NAME                                READY   STATUS      RESTARTS   AGE
discovery-5d58dcbf7f-jb9lv          1/1     Running     0          3m5s
gateway-proxy-6b648fc7b9-c22z2      1/1     Running     0          3m5s
gloo-77cb789d9f-mm78p               1/1     Running     0          3m5s
gloo-resource-rollout-check-jjcdm   0/1     Completed   0          3m5s
gloo-resource-rollout-rxrzs         0/1     Completed   0          3m5s
```

</details>
<br>

Check Services:

```bash
kubectl get services -n gloo-system
```

<details>
<summary>See sample results</summary>

```bash
NAME                               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                                                         AGE
discovery                          ClusterIP      10.43.82.79    <none>        9091/TCP                                                        3m7s
gateway-proxy-monitoring-service   ClusterIP      10.43.95.159   <none>        8081/TCP                                                        3m7s
gloo                               ClusterIP      10.43.198.68   <none>        9977/TCP,9976/TCP,9988/TCP,9966/TCP,9979/TCP,443/TCP,9091/TCP   3m7s
gateway-proxy                      LoadBalancer   10.43.111.63   10.5.1.32     80:32120/TCP,443:31587/TCP                                      3m7s
```

</details>
<br>

Check Deployments:

```bash
kubectl get deployments -n gloo-system
```

<details>
<summary>See sample results</summary>

```bash
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
discovery       1/1     1            1           3m38s
gateway-proxy   1/1     1            1           3m38s
gloo            1/1     1            1           3m38s
```

</details>
<br>

Check CRDs:

```bash
kubectl get crds | grep gloo
```

<details>
<summary>See sample results</summary>

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
<summary>See sample results</summary>

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
<summary>See sample results</summary>

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
<summary>See sample results</summary>

```bash
NAME                                                  AGE
default-kubernetes-443                                4m5s
gloo-system-gloo-443                                  4m5s
gloo-system-discovery-9091                            4m5s
gloo-system-gloo-9966                                 4m5s
gloo-system-gloo-9976                                 4m5s
gloo-system-gloo-9977                                 4m5s
gloo-system-gloo-9979                                 4m5s
gloo-system-gloo-9988                                 4m5s
gloo-system-gloo-9091                                 4m5s
gloo-system-gateway-proxy-443                         4m5s
gloo-system-gateway-proxy-80                          4m5s
gloo-system-gateway-proxy-monitoring-service-8081     4m5s
kube-system-kube-dns-53                               4m5s
kube-system-kube-dns-9153                             4m5s
```

</details>
<br>

You can approximate the comprehensive checks performed by `glooctl` check. 

# 5 - Summary

## 5.1 - Ease of Use

`glooctl` is simpler to use for installing Gloo Edge because it is designed specifically for this purpose and handles many default configurations automatically. `Helm` offers more flexibility and customization but requires more initial setup and understanding of Helm charts.

## 5.2 - Configuration

`glooctl` uses predefined configurations that are suitable for most standard installations. `Helm` can use a customized `values.yaml` file that allows extensive customization, making it suitable for complex and large-scale deployments.

## 5.3 - Upgrade and Rollback

`Helm` provides robust mechanisms for upgrading and rolling back installations, making it ideal for managing versions in production environments. `glooctl` also supports upgrades but lacks the fine-grained control provided by Helm.
