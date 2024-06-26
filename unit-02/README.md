# 1 - Deploy a Sample Application
Deploy a sample application to test the Gloo Edge setup. You can deploy it using kubectl with Gloo Edge configurations:

## 1.1 - Deploy the Petstore sample application
The Petstore configuration is defined in a single YAML file. This file specifies the deployment and service specifications in a human-readable format. The configuration can be version-controlled, replicated, and applied consistently across environments.

<details>

```yaml
##########################
#                        #
#        Example         #
#        Service         #
#                        #
#                        #
##########################
# petstore service
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: petstore
  name: petstore
  namespace: default
spec:
  selector:
    matchLabels:
      app: petstore
  replicas: 1
  template:
    metadata:
      labels:
        app: petstore
    spec:
      containers:
      - image: soloio/petstore-example:latest
        name: petstore
        ports:
        - containerPort: 8080
          name: http
---
apiVersion: v1
kind: Service
metadata:
  name: petstore
  namespace: default
  labels:
    service: petstore
spec:
  ports:
  - name: http
    port: 8080
    protocol: TCP
  selector:
    app: petstore
```
</details>

### 1.1.1 - Get the latest Petstore
This is step is optional. A copy of the `petstore.yaml` is included in this folder. 
```bash
wget -nv https://raw.githubusercontent.com/solo-io/gloo/main/example/petstore/petstore.yaml -O petstore.yaml
```

<details>
<summary>See sample results</summary>

```
2024-06-04 15:22:37 URL:https://raw.githubusercontent.com/solo-io/gloo/main/example/petstore/petstore.yaml [822/822] -> "petstore.yaml" [1]
```
</details>

### 1.1.2 - Deploy the latest Petstore

```bash
kubectl apply -f petstore.yaml
```

<details>
<summary>See sample results</summary>

```
deployment.apps/petstore created
service/petstore created
```
</details>

### 1.1.3 - Confirm deployment
```bash
kubectl -n default get pods
```

<details>
<summary>See sample results</summary>

```
NAME                        READY   STATUS    RESTARTS   AGE
petstore-66cddd5bb4-4tdjt   1/1     Running   0          72s
```
</details>

### 1.1.4 - Confirm deployment
```bash
kubectl -n default get svc petstore
```

<details>
<summary>See sample results</summary>

```
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
petstore   ClusterIP   10.43.126.102   <none>        8080/TCP   96s
```
</details>

---
## 1.2 - Test the application

### 1.2.1 - Expose the application URL

```bash
kubectl -n default port-forward svc/petstore 18080:8080 &
```

<details>
<summary>See sample results</summary>

```
[1] 11371
Forwarding from 127.0.0.1:18080 -> 8080
Forwarding from [::1]:18080 -> 8080
```
</details>

### 1.2.1 - Test application URL at `/api/pets`

```bash
curl -vv http://127.0.0.1:18080/api/pets
```

<details>
<summary>See sample results</summary>

```
*   Trying 127.0.0.1:18080...
* Connected to 127.0.0.1 (127.0.0.1) port 18080 (#0)
> GET /api/pets HTTP/1.1
> Host: 127.0.0.1:18080
> User-Agent: curl/7.81.0
> Accept: */*
> 
Handling connection for 18080
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: application/xml
< Date: Tue, 04 Jun 2024 16:08:06 GMT
< Content-Length: 86
< 
[{"id":1,"name":"Dog","status":"available"},{"id":2,"name":"Cat","status":"pending"}]
* Connection #0 to host 127.0.0.1 left intact
```

</details>
<br>

# 2 - Configure API Gateway
The Gloo Edge API Server provides a single point of interaction for managing all configurations. The YAML file from the example can be submitted directly to the API server. Operators can manage routing, load balancing, and security policies from a single, unified API.

## 2.1 - Examine the application upstream
The Discovery Service automatically detects new services and updates the configuration accordingly. In the Petstore example, upstream services are discovered and managed dynamically.

### 2.1.1 - Gloo Edge interprets the Petstore service upstream
```bash
glooctl get upstream default-petstore-8080
```

<details>
<summary>See sample results</summary>

```
+-----------------------+------------+----------+-------------------------+
|       UPSTREAM        |    TYPE    |  STATUS  |         DETAILS         |
+-----------------------+------------+----------+-------------------------+
| default-petstore-8080 | Kubernetes | Accepted | svc name:      petstore |
|                       |            |          | svc namespace: default  |
|                       |            |          | port:          8080     |
|                       |            |          |                         |
+-----------------------+------------+----------+-------------------------+
```
</details>
<br>
The service discovery mechanism ensures that routing and load balancing configurations are always up-to-date without manual intervention. This approach reduces the manual effort needed to register and update services.

### 2.1.2 - Enable auto-discovery on the Petstore namespace
```bash
kubectl label namespace default discovery.solo.io/function_discovery=enabled
```

<details>
<summary>See sample results</summary>

```
namespace/default labeled
```
</details>

### 2.1.3 - Compare discovered API functions from Petstore

```bash
glooctl get upstream default-petstore-8080
```

<details>
<summary>See sample results</summary>

```
+-----------------------+------------+----------+-------------------------+
|       UPSTREAM        |    TYPE    |  STATUS  |         DETAILS         |
+-----------------------+------------+----------+-------------------------+
| default-petstore-8080 | Kubernetes | Accepted | svc name:      petstore |
|                       |            |          | svc namespace: default  |
|                       |            |          | port:          8080     |
|                       |            |          | REST service:           |
|                       |            |          | functions:              |
|                       |            |          | - addPet                |
|                       |            |          | - deletePet             |
|                       |            |          | - findPetById           |
|                       |            |          | - findPets              |
|                       |            |          |                         |
+-----------------------+------------+----------+-------------------------+
```
</details>

## 2.2 - Create a route 
The API Server provides a single interface for managing traffic policies. Changes to the Petstore YAML configuration are processed centrally. Operators can define and manage advanced traffic management policies from a single point.

### 2.2.1 - Use `glooctl` to add an API route

```bash
glooctl add route \
  --path-exact /all-pets \
  --dest-name default-petstore-8080 \
  --prefix-rewrite /api/pets
```

<details>
<summary>See sample results</summary>

```
{"level":"info","ts":"2024-06-04T14:00:14.870Z","caller":"add/route.go:156","msg":"Created new default virtual service","virtualService":"virtual_host:{domains:\"*\" routes:{matchers:{exact:\"/all-pets\"} route_action:{single:{upstream:{name:\"default-petstore-8080\" namespace:\"gloo-system\"}}} options:{prefix_rewrite:{value:\"/api/pets\"}}}} namespaced_statuses:{} metadata:{name:\"default\" namespace:\"gloo-system\" resource_version:\"2339\" generation:1}"}
+-----------------+--------------+---------+------+--------+-----------------+-----------------------------------+
| VIRTUAL SERVICE | DISPLAY NAME | DOMAINS | SSL  | STATUS | LISTENERPLUGINS |              ROUTES               |
+-----------------+--------------+---------+------+--------+-----------------+-----------------------------------+
| default         |              | *       | none |        |                 | /all-pets ->                      |
|                 |              |         |      |        |                 | gloo-system.default-petstore-8080 |
|                 |              |         |      |        |                 | (upstream)                        |
+-----------------+--------------+---------+------+--------+-----------------+-----------------------------------+
```
</details>

### 2.2.1 - Examine derivate virtual service
```bash
glooctl get virtualservices
```
<details>
<summary>See sample results</summary>

```bash
+-----------------+--------------+---------+------+----------+-----------------+-----------------------------------+
| VIRTUAL SERVICE | DISPLAY NAME | DOMAINS | SSL  |  STATUS  | LISTENERPLUGINS |              ROUTES               |
+-----------------+--------------+---------+------+----------+-----------------+-----------------------------------+
| default         |              | *       | none | Accepted |                 | /all-pets ->                      |
|                 |              |         |      |          |                 | gloo-system.default-petstore-8080 |
|                 |              |         |      |          |                 | (upstream)                        |
+-----------------+--------------+---------+------+----------+-----------------+-----------------------------------+
```
</details>

## 2.3 - Use the API Gateway Proxy
 The Gateway Proxy acts as a centralized entry point, enforcing traffic management policies defined in the YAML configuration.

### 2.3.1 - Obtain default `http` URL
```bash
# Show me the Gloo Gateway default URL for http
glooctl proxy url 

# Save the Gloo Gateway default URL as a variable
export GLOO_PROXY_URL=$(glooctl proxy url)
```

<details>
<summary>See sample results</summary>

```bash
http://10.5.0.68:80
```
</details>

### 2.3.2 - Test the Proxy route
```bash
curl -vv $GLOO_PROXY_URL/all-pets
```

<details>
<summary>See sample results</summary>

```
*   Trying 10.5.0.68:80...
* Connected to 10.5.0.68 (10.5.0.68) port 80 (#0)
> GET /all-pets HTTP/1.1
> Host: 10.5.0.68
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< content-type: application/xml
< date: Tue, 04 Jun 2024 15:34:22 GMT
< content-length: 86
< x-envoy-upstream-service-time: 1
< server: envoy
< 
[{"id":1,"name":"Dog","status":"available"},{"id":2,"name":"Cat","status":"pending"}]
* Connection #0 to host 10.5.0.68 left intact
```
</details>
<br>

# 3 - Summary

## 3.1 - Ease of Use

Gloo Edge simplifies API management through its use of declarative configuration, allowing administrators to define the desired state of the API gateway using YAML or JSON files. This approach is similar to how Kubernetes manages infrastructure, making it familiar and accessible to teams already using Kubernetes. Administrators can use these configuration files as templates, which can be stored and managed in version control systems like Git.

## 3.2 - Configuration

Using templates stored in version control, administrators can manage routing rules, security policies, and traffic management settings without manual configuration. This reduces the risk of errors and inconsistencies. They can easily replicate and apply configurations across multiple instances, streamlining deployment and management tasks.

## 3.3 - Upgrade and Rollback

This automation improves the reliability and availability of services by ensuring that the API gateway can quickly adapt to changes in the service environment. It minimizes downtime and enhances overall system resilience by promptly reflecting the latest service topology. Additionally, automated service discovery in Gloo Edge reduces the operational burden on administrators, freeing them from the need to manually manage service configurations. This leads to increased operational efficiency and allows the team to focus on higher-level tasks, further improving the performance and stability of the service infrastructure.

