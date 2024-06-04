# Deploy a Sample Application
Deploy a sample application to test the Gloo Edge setup. You can deploy it using kubectl with Gloo Edge configurations:

## Petstore Configuration Breakdown
The Petstore configuration is defined in a single YAML file. This file specifies routing rules, upstream services, and security settings in a human-readable format. The configuration can be version-controlled, replicated, and applied consistently across environments.

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

```bash
wget -nv https://raw.githubusercontent.com/solo-io/gloo/main/example/petstore/petstore.yaml
```

```bash
kubectl apply -f petstore.yaml
```

## Confirm deployment
```bash
kubectl -n default get pods
```

```bash
kubectl -n default get svc petstore
```

---
## Test the application

```bash
kubectl -n default port-forward svc/petstore 18080:8080
```

```bash
Forwarding from 127.0.0.1:18080 -> 8080
Forwarding from [::1]:18080 -> 8080
```

```bash
curl http://127.0.0.1:18080/api/pets
```

```bash
[{"id":1,"name":"Dog","status":"available"},{"id":2,"name":"Cat","status":"pending"}]
```

```json
[
  {
    "id": 1,
    "name": "Dog",
    "status": "available"
  },
  {
    "id": 2,
    "name": "Cat",
    "status": "pending"
  }
]
```
# Configure API Gateway Upstream
The Gloo Edge API Server provides a single point of interaction for managing all configurations. The YAML file from the example can be submitted directly to the API server. Operators can manage routing, load balancing, and security policies from a single, unified API.

## Examine the application upstream
The Discovery Service automatically detects new services and updates the configuration accordingly. In the Petstore example, upstream services are discovered and managed dynamically.
```bash
glooctl get upstream default-petstore-8080
```

```bash
+-----------------------+------------+----------+-------------------------+
|       UPSTREAM        |    TYPE    |  STATUS  |         DETAILS         |
+-----------------------+------------+----------+-------------------------+
| default-petstore-8080 | Kubernetes | Accepted | svc name:      petstore |
|                       |            |          | svc namespace: default  |
|                       |            |          | port:          8080     |
|                       |            |          |                         |
+-----------------------+------------+----------+-------------------------+
```
The service discovery mechanism ensures that routing and load balancing configurations are always up-to-date without manual intervention. This approach reduces the manual effort needed to register and update services.

```bash
kubectl label namespace default discovery.solo.io/function_discovery=enabled
```

```bash
glooctl get upstream default-petstore-8080
```

```bash
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

## Create a route 
The API Server provides a single interface for managing traffic policies. Changes to the Petstore YAML configuration are processed centrally. Operators can define and manage advanced traffic management policies from a single point.

```bash
glooctl add route \
  --path-exact /all-pets \
  --dest-name default-petstore-8080 \
  --prefix-rewrite /api/pets
```

```bash
{"level":"info","ts":"2024-06-04T14:00:14.870Z","caller":"add/route.go:156","msg":"Created new default virtual service","virtualService":"virtual_host:{domains:\"*\" routes:{matchers:{exact:\"/all-pets\"} route_action:{single:{upstream:{name:\"default-petstore-8080\" namespace:\"gloo-system\"}}} options:{prefix_rewrite:{value:\"/api/pets\"}}}} namespaced_statuses:{} metadata:{name:\"default\" namespace:\"gloo-system\" resource_version:\"2339\" generation:1}"}
+-----------------+--------------+---------+------+--------+-----------------+-----------------------------------+
| VIRTUAL SERVICE | DISPLAY NAME | DOMAINS | SSL  | STATUS | LISTENERPLUGINS |              ROUTES               |
+-----------------+--------------+---------+------+--------+-----------------+-----------------------------------+
| default         |              | *       | none |        |                 | /all-pets ->                      |
|                 |              |         |      |        |                 | gloo-system.default-petstore-8080 |
|                 |              |         |      |        |                 | (upstream)                        |
+-----------------+--------------+---------+------+--------+-----------------+-----------------------------------+
```

```bash
glooctl get virtualservices
```

```bash
+-----------------+--------------+---------+------+----------+-----------------+-----------------------------------+
| VIRTUAL SERVICE | DISPLAY NAME | DOMAINS | SSL  |  STATUS  | LISTENERPLUGINS |              ROUTES               |
+-----------------+--------------+---------+------+----------+-----------------+-----------------------------------+
| default         |              | *       | none | Accepted |                 | /all-pets ->                      |
|                 |              |         |      |          |                 | gloo-system.default-petstore-8080 |
|                 |              |         |      |          |                 | (upstream)                        |
+-----------------+--------------+---------+------+----------+-----------------+-----------------------------------+
```

## Use the API Gateway Proxy
 The Gateway Proxy acts as a centralized entry point, enforcing traffic management policies defined in the YAML configuration.
```bash
glooctl proxy url
```

```bash
http://10.5.1.149:80
```

```bash
curl -vv http://10.5.1.149:80/all-pets
```

```bash
*   Trying 10.5.1.149:80...
* Connected to 10.5.1.149 (10.5.1.149) port 80 (#0)
> GET /all-pets HTTP/1.1
> Host: 10.5.1.149
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< content-type: application/xml
< date: Tue, 04 Jun 2024 14:06:05 GMT
< content-length: 86
< x-envoy-upstream-service-time: 0
< server: envoy
< 
[{"id":1,"name":"Dog","status":"available"},{"id":2,"name":"Cat","status":"pending"}]
* Connection #0 to host 10.5.1.149 left intact
```

# Summary

* Gloo Edge simplifies API management through its use of declarative configuration, allowing administrators to define the desired state of the API gateway using YAML or JSON files. This approach is similar to how Kubernetes manages infrastructure, making it familiar and accessible to teams already using Kubernetes. Administrators can use these configuration files as templates, which can be stored and managed in version control systems like Git.

* Using templates stored in version control, administrators can manage routing rules, security policies, and traffic management settings without manual configuration. This reduces the risk of errors and inconsistencies. They can easily replicate and apply configurations across multiple instances, streamlining deployment and management tasks.

* This automation improves the reliability and availability of services by ensuring that the API gateway can quickly adapt to changes in the service environment. It minimizes downtime and enhances overall system resilience by promptly reflecting the latest service topology. Additionally, automated service discovery in Gloo Edge reduces the operational burden on administrators, freeing them from the need to manually manage service configurations. This leads to increased operational efficiency and allows the team to focus on higher-level tasks, further improving the performance and stability of the service infrastructure.

