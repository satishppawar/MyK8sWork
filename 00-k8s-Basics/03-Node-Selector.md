# Node Selector

- nodeSelector is a selector which allows you to assign a pod to a specific node. It matches a node key/value pair, also known as a Label, which tells the Kubernetes scheduler which node to schedule the pod to.


## Discover node labels

- First of all, let's look at the current node labels:

```console
kubectl get nodes --show-labels
```

## Add a new node label

- First list all the available nodes

```console
kubectl get nodes 
```

- Now, add the label disk=ssd to the myk8spractise node:

```console
kubectl label nodes myk8spractise disk=ssd
```


- Ensure the label we've just created has been added to myk8spractise:
  
```console
kubectl get nodes --show-labels
```

## Assign the happypanda pod to node01, matching disk:ssd label

- cat 03-Node-Selector/pod-nodeselector.yaml
  
  ```yml
  apiVersion: v1
    kind: Pod
    metadata:
    name: happypanda
    labels: 
        app: redis
        segment: backend
        company: mycompany 
        disk: ssd  
    spec:
    containers:
    - name: redis
        image: redis
        ports:
        - name: redisport
        containerPort: 6379
        protocol: TCP
    nodeSelector:
        disk: ssd
  ```

- Notice that `nodeSelector` is matching the label we just added to the node - `disk:ssd`. The Kubernetes scheduler will use this label to try and schedule pods onto a node with the label.

- Next, we will create the happypanda pod by running the following command:

```console
kubectl apply -f 03-Node-Selector/pod-nodeselector.yaml
```

- Let's verify
  
Check that pod `happypanda` has been successfully scheduled on the `myk8spractise` node:

## Deleting happypanda pod and label

- Delete happypanda pod:

```console
kubectl delete -f 03-Node-Selector/pod-nodeselector.yaml
```

OR 

```console
kubectl delete pod happypanda
```

- Remove label from `myk8spractise`

```console
kubectl label node myk8spractise disk-
```

---------------

## Node Affinity and Anti-Affinity


- In the previous chapter, we introduced `nodeSelector`. This works well if your nodes have the required node labels, but if the nodeSelector doesn't match a label on a node, then the pod will not be scheduled. 
  
- `Node/Pod Affinity and Anti-Affinity` resolves this issue by introducing `soft and hard conditions`.

### Node Affinity

When you look at the [Kubernetes API Reference](https://v1-18.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#nodeaffinity-v1-core), you'll notice that the two specs for Node Affinity are:

- `spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution` 
  -  *Soft NodeAffinity and Anti-Affinity:* If the node label exists, the Pod will be ran there. If not, then the Pod will be rescheduled elsewhere within the cluster.

- `spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution` 
  - *Hard NodeAffinity and Anti-Affinity:* If the node label doesn't exist, then the pod won't be scheduled at all.


### Soft Node Affinity

Let's inspect the node-soft-affinity.yaml manifest:

```
cat 03-Node-Selector/node-soft-affinity.yaml
```

The manfifest reads as: "If there are no nodes labelled as apple, then still schedule the pod to a node".

### Hard Node Affinity

- Now inspect the node-hard-affinity.yaml manifest:

```
cat 03-Node-Selector/node-hard-affinity.yaml
```

The manifest reads as: "If there are no nodes labelled as apple, then this pod won't be assigned a node by the scheduler".


### Node Anti-Affinity

- Node anti-affinity can be achieved by using the `NotIn` operator. This will help us to ignore nodes while scheduling.

- For a node anti-affinity check the `node-hard-anti-affinity.yaml` and `node-soft-anti-affinity.yaml` manifest files:
  
-----------

## Pod Affinity and Anti-Affinity

- NodeAffinity allows you to schedule pods on specific nodes. But what if you want to run multiple pods on specific nodes? Pod affinity helps with that.


### Pod Affinity

When you look at the [Kubernetes API Reference](https://v1-18.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#podaffinity-v1-core), you'll notice that the two specs for Pod Affinity are:

* `spec.affinity.podAffinity.preferredDuringSchedulingIgnoredDuringExecutionis` for `Soft Pod Affinity`. 
  
  - If the preferred option is available, the Pod will run there. If not, the Pod can still be scheduled elsewhere.

* `spec.affinity.podAffinity.requiredDuringSchedulingIgnoredDuringExecution` is for Hard `Pod Affinity`. 

  * If the required option is not available, the Pod cannot run.


### Hard Pod Affinity

- Let's inspect the pod-hard-affinity.yaml file:

```console
cat 03-Node-Selector/pod-hard-affinity.yaml
```

``` YAML
apiVersion: v1
kind: Pod
metadata:
  name: happypanda
  labels: 
    app: redis
    segment: backend
    company: mycompany 
    disk: ssd  
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
            - key: fruit
              operator: In
              values:
              - apple
          topologyKey: kubernetes.io/hostname
  containers:
  - name: redis
    image: redis
    ports:
    - name: redisport
      containerPort: 6379
      protocol: TCP
```

- This is a hard pod affinity. If none of the nodes are labelled with fruit=apple, then the pod won't be scheduled.


- The topologyKey is a label of a node, such as kubernetes.io/hostname

### Soft Pod Affinity

- Soft Pod Affinity will schedule the Pod even though is not finding a pod running with label fruit=apple.


## Pod Anti-Affinity

When you look at the [Kubernetes API Reference](https://v1-18.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#podantiaffinity-v1-core), you'll notice that the `two specs for Pod Anti-Affinity are`:

* `spec.affinity.podAntiAffinity.preferredDuringSchedulingIgnoredDuringExecution` is for `Soft Pod Anti-Affinity`.
  *  If the preferred option is available, the Pod will run there. If not, the Pod can still be scheduled elsewhere.

* `spec.affinity.podAntiAffinity.requiredDuringSchedulingIgnoredDuringExecution` is for `Hard Pod Anti-Affinity.`
  
  *  If the required option is not available, the Pod cannot run.

* Pod anti-affinity works the opposite way of pod affinity. If one of the nodes has a pod running with label fruit=apple, the pod will be scheduled on different node.


