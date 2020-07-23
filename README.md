# quarkus-configmap project

This project demonstrates how to read properties from a Kubernetes ConfigMap.

## Prerequisites

* Quarkus 1.6.1.Final or later (because that's the version I've tested)
* minikube 1.11.0 or later (because that's the version I've tested)
* JDK 11 (because that's the version I've tested)
* Maven 3.6.2+

## Instructions

In this example, application.properties configures configmap access.
application.yaml, located in the top-level directory,
is used as the configmap for application values.
This example also assumes you are already logged into the desired
namespace.

Create the configmap:
```
kubectl create configmap greeting --from-file=application.yaml
```
Use minikube container registry:

```
eval $(minikube -p minikube docker-env)
```

Deploy the application:

```
mvn \
   clean \
   install \
   -DskipTests \
   -Dquarkus.kubernetes.deploy=true
```

Access the endpoint:

```
NODEPORT=`kubectl get svc quarkus-configmap -o jsonpath='{.spec.ports[0].nodePort}'`

curl http://`minikube ip`:$NODEPORT/hello
```

##  Update the property and 'restart' the pod

Since the application is running in  Kubernetes, this example embraces immutable infrastructure.
First update the configmap and then force a pod 'restart'.
Technically, the old pod is destroyed and a new oneis created :-)

Update the Configmap, and change `greeting` value from "(configmap) Hello" to "Hello from Kubernetes"

```
kubectl edit cm/greeting
```

Update the value of _hello_:

```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#

apiVersion: v1
data:
  application.yaml: 'greeting: Hello'  # <---  Change 'Hello' to 'Hello from Kubernetes'
kind: ConfigMap
metadata:
  creationTimestamp: "2020-07-23T21:04:39Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:application.yaml: {}
    manager: kubectl
    operation: Update
    time: "2020-07-23T21:04:39Z"
  name: greeting
  namespace: default
  resourceVersion: "403548"
  selfLink: /api/v1/namespaces/default/configmaps/greeting
  uid: b0f323c0-135c-4f31-a660-d8465f8b62a1
  ```

  Force a pod restart.
  This example does this by updating the deployment with an unused environmental variable.

  ```
  kubectl set env deployment quarkus-configmap --env="LAST_RESTART=$(date)"
  ```

  Wait for the pod to restart, then get updated greeting:

```
  NODEPORT=`kubectl get svc quarkus-configmap -o jsonpath='{.spec.ports[0].nodePort}'`

  curl http://`minikube ip`:$NODEPORT/hello
```