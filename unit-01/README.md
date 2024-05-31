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
  Attempting to download meshctl version v2.5.6
  Downloading meshctl-darwin-arm64...
  Download complete!, validating checksum...
  Checksum valid.
  meshctl was successfully installed 🎉

  Add the Gloo Mesh CLI to your path with:
    export PATH=$HOME/.gloo-mesh/bin:$PATH

  Now run:
    meshctl install     # install Gloo Mesh management plane
  Please see visit the Gloo Mesh website for more info:  https://www.solo.io/products/gloo-mesh/```

</details>


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
In most cases `glooctl` check simplifies this process significantly by aggregating these checks into a single command.

## Use `kubectl`

There is no single `kubectl` command that performs all the checks glooctl check does. You can use a combination of `kubectl` commands to achieve similar checks.

Check Deployments and Pods:

```bash
kubectl get deployments -n gloo-system
kubectl get pods -n gloo-system
```

Check Deployments and Pods:

```bash
kubectl get crds | grep gloo
```

Check the Status of Custom Resources:

```bash
kubectl get virtualservices -n gloo-system
kubectl get gateways -n gloo-system
kubectl get upstreams -n gloo-system
```

Inspect Logs for Proxies:

```bash
kubectl logs -l app=gloo -n gloo-system
kubectl logs -l app=discovery -n gloo-system
```

You can approximate the comprehensive checks performed by `glooctl` check. 

# Summary

* **Ease of Use**: `glooctl` is simpler to use for installing Gloo Edge because it is designed specifically for this purpose and handles many default configurations automatically. `Helm` offers more flexibility and customization but requires more initial setup and understanding of Helm charts.

* **Configuration**: `glooctl` uses predefined configurations that are suitable for most standard installations. `Helm` provides a values.yaml file that allows extensive customization, making it suitable for complex and large-scale deployments.

* **Upgrade and Rollback**: `Helm` provides robust mechanisms for upgrading and rolling back installations, making it ideal for managing versions in production environments. `glooctl` also supports upgrades but lacks the fine-grained control provided by Helm.
