# Basic Deployments

- The Kubernetes documentation states that a Deployment allows you to:

> ... describe a desired state in a Deployment object, and the Deployment controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

----    ------     -

## Creating a deployment through kubectl

- `Kubernetes deployments` manage stateless services running in your cluster (as opposed to - for example - StatefulSets, which manage stateful services). Their purpose is to keep a set of identical pods running and upgrade them in a controlled way – performing a rolling update by default. There are different deployment strategies that work with Deployments. They are, however, out of scope of this scenario. For more information on deployment strategies, [read the Kubernetes Documentation here.](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#writing-a-deployment-spec)

----------

![K8s Deployment](deployment-high-level.png)

----------

### Creating a Deployment in kubectl

- Before we start, you should already have a namespace created called `contino`. If you do not have this namespace yet, or you have deleted it, then please re-create it:

```sh
kubectl create namespace contino
```
- Create a basic deployment called nginx-deployment using the nginx image, and expose port 80 in the container:

```sh
kubectl run nginx-deployment -n contino --image=nginx --port 80
```

- Now let's inspect the deployment that we've just created:

```sh
kubectl get deployment nginx-deployment -n contino -o yaml
```

### Deployment Object

<details >
<summary>Deployment Object</summary>

```yml
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      run: nginx-deployment
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: nginx-deployment
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx-deployment
        ports:
        - containerPort: 80
          protocol: TCP
```
</details>

- The spec (specification) of the deployment has two keys you must set:

    **replicas:** describes how many pods this deployment should have. In our case, there will be one only one pod created.

    **template:** describes how each pod should look like. It describes a list of containers that should be in the Pod.

- The two other keys can be set to customize the behavior of the deployment.

**selector:** determines which pods are considered to be part of this deployment. This uses labels to 'select' pods.

**strategy:** states how an update to a deployment should be rolled out. See earlier at the start of this chapter for the Kubernetes API link on more information on deployment strategies.

----------

## Creating a deployment through manifests


- Because you have already created a deployment earlier called `nginx-deployment`, we need to delete it before we can create a new deployment with the YAML manifest:

```console
kubectl delete deployment nginx-deployment -n contino
```
`cat 06-Basic-Deployments/dep-manifest.yaml; echo`

```yml
apiVersion: v1
kind: Deployment
metadata:
  labels:
    run: nginx-deployment
  name: nginx-deployment
  namespace: contino
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      run: nginx-deployment
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: nginx-deployment
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx-deployment
        ports:
        - containerPort: 80
          protocol: TCP
```

- Now create the deployment:

```console
kubectl create -f  06-Basic-Deployments/dep-manifest.yaml
```

- Get the deployment:

```console
kubectl get deployments -n contino -o yaml
```

--------------------------------

## Scaling replicas in a deployment

- Now that you've created your deployment, let's look at scaling. Similar to how we created a deployment, you can update the replicas through the deployment manifest or by the kubectl command. For example:

```console
kubectl scale deployment nginx-deployment --replicas=10 -n contino
```
> This will scale the nginx-deployment Deployment to 10 replicas.

You can also edit this in the manifest. For example:

```sh
vi dep-manifest.yaml
```

Find the line 9 in the manifest, and update it to 10 replicas. :wq in vim to save and exit. Then replace the deployment:

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx-deployment
  name: nginx-deployment
  namespace: contino
spec:
  replicas: 9
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      run: nginx-deployment
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: nginx-deployment
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx-deployment
        ports:
        - containerPort: 80
          protocol: TCP
```

```console
kubectl replace -f 06-Basic-Deployments/dep-manifest.yaml
```

- Once the deployment has been updated, check the pods:
```console
kubectl get po -n contino
```

- You should see 10 nginx pods now. You've successfully just scaled your deployment. Equally, to scale down your deployment, simply change the number of replicas and update the deployment again.

----------

### Proportional scaling

- RollingUpdate Deployments support running multiple versions of an application at the same time. When you scale a RollingUpdate Deployment that is in the middle of a rollout (either in progress or paused), then the Deployment controller will balance the additional replicas in the existing active ReplicaSets (ReplicaSets with Pods) in order to mitigate risk. This is called proportional scaling.

- For example, you are running a Deployment with 10 replicas, maxSurge=3, and maxUnavailable=2.

```console
kubectl get deployments -n contino
```

- You update to a new image which happens to be unresolvable from inside the cluster.

```
kubectl replace -f 06-Basic-Deployments/dep-manifest-update.yaml
kubectl get deployments -n contino
```
```console
kubectl set image deploy/nginx-deployment nginx-deployment=nginx:unresolvabletag -n contino
```

----------

## Create and scaling a replication controller

- The `Replication Controller` is the original form of replication in Kubernetes. It has been replaced by ReplicaSets, so we won't cover it in too much detail, but it's worth understanding.

- Similar to Deployments, it allows you to tell the Kubernetes scheduler how many pods to ensure stay running. If a pod does die, then the replication controller will create another pod to ensure the replica count is matched.

### Creating a Replication Controller

- Look at the replication controller manifest:

```sh
cat rc-nginx.yaml; echo
```
<details>

```yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
  namespace: contino
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
</details>

- Now create the replication controller:

```console
kubectl create -f 06-Basic-Deployments/rc-nginx.yaml
```

- Check for RCs:

```
kubectl get rc -n contino
```

- Now check for pods:

```
kubectl get po -n contino
```
> As you can see, there are now 3 pods, as per the replication controller manifest that we just created.

- To delete a replication controller:

```
kubectl delete rc nginx -n contino
```

## Scaling ReplicaSets

- Scaling RCs is very similar to deployments. Edit the replicas in the manifest and use the kubectl replace -f command:

```
kubectl replace -f 06-Basic-Deployments/rc-nginx.yaml
```