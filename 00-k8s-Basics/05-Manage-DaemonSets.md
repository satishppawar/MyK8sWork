# Manage DaemonSets

- A `DaemonSet` ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are evicted. Deleting a DaemonSet will clean up the Pods it created.

- Some typical uses of a DaemonSet are:

  * running a cluster storage daemon, such as `glusterd` and `ceph`, on each node.
  * running a logs collection daemon on every node, such as `fluentd` or `logstash`.
  * running a node monitoring daemon on every node, such as `Prometheus Node` `Exporter (node_exporter)`, `collectd`, `Datadog agent`, `New Relic agent`, or `Ganglia gmond`.

- In a simple case, one DaemonSet, covering all nodes, would be used for each type of daemon. 
- A more complex setup might use multiple DaemonSets for a single type of daemon, but with different flags and/or different memory and cpu constraints for different hardware types.

----------------------------------------------------------------

## Create a DaemonSet

- In this scenario, we're going to create an `nginx` DaemonSet. Initially, we'll run this on our worker nodes (myk8spractise), but then we will manipulate the DaemonSet to get it to run on the master node too.


### nginx DaemonSet

- In your terminal, you'll see a file named `05-Manage-DaemonSets/nginx-daemonset.yaml`. This is the DaemonSet which we will be using to run nginx across both of our nodes.

- First, let's create all the prerequisites needed for this DaemonSet to run: `05-Manage-DaemonSets/nginx-ds-prereqs.yaml`

```yml
---
kind: Namespace
apiVersion: v1
metadata:
  name: contino
  labels:
    name: contino

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-svc-acct
  namespace: contino
  labels:
    name: nginx-svc-acct
```

```bash
kubectl create -f 05-Manage-DaemonSets/nginx-ds-prereqs.yaml
```

- Now we've created the namespace (and other prerequisites), let's inspect the manifest for the nginx DaemonSet: `cat 05-Manage-DaemonSets/nginx-daemonset.yaml; echo`

```yml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx
  namespace: contino
  labels:
    app: nginx
    name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
      name: nginx
  template:
    metadata:
      labels:
        app: nginx
        name: nginx
    spec:
      serviceAccountName: nginx-svc-acct
      containers:
      - image: katacoda/docker-http-server:latest
        name: nginx
        ports:
        - name: http
          containerPort: 80
```

- As you can see, we're running a basic DaemonSet - in the `contino` namespace - which exposes port 80 inside the container.

- Create it:
  
```bash
kubectl create -f 05-Manage-DaemonSets/nginx-daemonset.yaml
```

- Now check the status of the DaemonSet:

```bash
kubectl get daemonsets -n contino
```
---------- 

## Accessing nginx

-  Now that we've created our nginx DaemonSet, let's see what host it's running on:

```conosole
kubectl get po -n contino -l app=nginx -o 'jsonpath={.items[0].spec.nodeName}'; echo
```

- Testing the Webserver

```bash
kubectl get po -n contino -l app=nginx -o 'jsonpath={.items[0].status.podIP}'; echo
```

Curl it:

```bash
curl `kubectl get po -n contino -l app=nginx -o 'jsonpath={.items[0].status.podIP}'`
```
----------
### Updating a DaemonSet (Rolling Update)

- As mentioned in the previous chapter, workload isn't scheduled to master nodes unless specifically tainted. In this scenario, we want to run the nginx DaemonSet across both master and node01.

- We need to update the DaemonSet, so we're going to use the ` 05-Manage-DaemonSets/nginx-daemonset-tolerations.yaml` file to replace the manifest.

- First, let's see what we added to the -tolerations.yaml file:

```yml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx
  namespace: contino
  labels:
    app: nginx
    name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        name: nginx
    spec:
      serviceAccountName: nginx-svc-acct
      containers:
      - image: katacoda/docker-http-server:latest
        name: nginx
        ports:
        - name: http
          containerPort: 80
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
```
- As you can see, we've added the following to the spec section:

```yml
tolerations:
- key: node-role.kubernetes.io/master
  effect: NoSchedule
```

- This is what manifests need to be tainted with in order to be ran on master nodes. Proceed to update the DaemonSet:

```console
kubectl replace -f 05-Manage-DaemonSets/nginx-daemonset-tolerations.yaml
```

### Deleting a DaemonSet

```console
kubectl delete daemonset nginx -n contino
```
OR

```console
kubectl delete ds nginx -n contino
```

- Success - you've deleted the DaemonSet. Check for pods:
  
```console
kubectl get pods -n contino
```