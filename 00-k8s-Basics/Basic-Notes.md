## Namespace

- Create a new namespace

    ```command
    kubectl apply -f 01-Namespaces/test-namespace.yaml
    ````

- List all namespaces:

    ```console
    kubectl get namespaces
    ```

- Delete a Namespace:
    ```console
   kubectl delete -f 01-Namespaces/test-namespace.yaml
    ```
    OR 

    ```
    kubectl delete namespace test
    ```

----------------------------------------------------------------
## Pod

- pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: happypanda
spec:
  containers:
  - name: nginx
    image: nginx
```
- Create a pod

```console
kubectl apply -f 02-Pod/pod.yaml
```

- Validation

```console
kubectl get pods
```

- Delete a Pod

```console
kubectl delete -f 02-Pod/pod.yaml
```

OR 

```console
kubectl delete pod happypanda
```

- Check if the happypanda pod has been deleted:

```console
kubectl get pods
```

## Schedule a pod in a namespace

- Notice that happypanda pod has been configured to be scheduled in the dev-service1 namespace.
  
```console
kubectl apply -f 02-Pod/pod-namespace.yaml
```

- Validation
  
```console
kubectl get pods -n dev-service1
```

---------------

## Updating pod

- We are going to update our happypanda pod running in dev-service1 namespace and to do that you need to apply the pod-update.yaml configuration.

- Look at the file `02-Pod/pod-update.yaml`

  1. Pods labels has been added in the metadata section
  2. Container image has been updated in the containers section. Notice that you can specify image tags if not, latest is used
  3. Pod ports has been added in the containers section

```console
kubectl apply -f 02-Pod/pod-update.yaml
```
- The error we are getting is the following:

> The Pod "happypanda" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)

- Fixing the highlighted problem
  
  1. Delete the pod:
   ```console
   kubectl delete pod happypanda -n dev-service1
   ```
  2. Apply the yaml file:
   ```console
    kubectl apply -f 02-Pod/pod-update.yaml
   ```
   3. Check it out:
   
     ```console
        kubectl describe pod happypanda --namespace dev-service1
     ```
    OR

     ```console
    kubectl get pod -n dev-service1
     ```
-----------

## Exercise

- Let's practice!

>Create namespace: lab1

>Create a pod with the following constraints:
Pod name: happyelephant
Deploy pod in a lab1 namespace
Add 3 labels
Use this container image: jenkins:latest
Set the container with 8080 port
Check happyelephant is running successfully
Delete happyelephant pod (remember set up path: /pods-manifests/)


- Please refer to file `02-Pod/myPod.yaml`
  
```console
kubectl apply -f 02-Pod/myPod.yaml
```

