# What the F**K is K8S?

> I don't know.

## Minikube

*Minikube is usually used just for local setup and internal testing.*

But the thing I know is that you should to start these commands:

Install `minikube`.

[Of course I put the official documentation](https://minikube.sigs.k8s.io/docs/start/)

So `kubectl` is a dependency of minikube then you shouldn't install it.

```bash
> minikube
> kubectl
```

Ok now start your local minikluster.

```bash
> minikube start
```

Now just wait to see the magic people done for you.

After the preparation, check your k8s nodes:

```bash
> kubectl get nodes
```

The output should be something like this:

```bash
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   4m18s   v1.23.3
```

What about the **minikube** status?

```bash
> minikube status
```

```bash
minikube
type: Control Plane
host: Running
**kubelet: Running**
apiserver: Running
kubeconfig: Configured
```

## Kubectl

Now gonna create my first **Pod** (in your local environment you don't create a Pod, but you create a deployment, that it's a Pod abstraction), following the output of `kubectl create` documentation.


```bash
kubectl create deployment nginx-deployment --image=nginx
```

Verify your deployment

```bash
$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/1     1            0           19s
```

Now check your **pods**

```bash
$ kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-794f656f8b-ff2bk   0/1     ContainerCreating   0          52s
````

If your pods is still no running, just wait for a second and re-run the `get pods` command.

```bash
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-794f656f8b-ff2bk   1/1     Running   0          99s
```

What about my replicas pod? (never work directly on replicas, neither update, delete or create. just work on deployments)

```bash
$ kubectl get replicaset
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-794f656f8b   1         1         1       3m1s
```

- Deploys Manages a ReplicaSet
- A ReplicaSet mannages a Pod
- A pod is an abstraction of your a container

### Editing a deployment

```bash
kubectl edit deployment nginx-deployment
```

This command will return you the `yaml` deployment configuration file so you can edit whetever you want and save the file.

After editing check your pods

```bash
$ kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-6bbc464978-4xd9l   0/1     Running                 0      11s
nginx-deployment-794f656f8b-ff2bk   1/1     Terminating             0      10m
```

As you can see the **Status** columns indicates that the old Pod is being terminating and the other one with the changes made is running.

```bash
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6bbc464978-4xd9l   1/1     Running   0          75s
```

> Now the old **Pod** is gone.

What about my Replicas?

```bash
$ kubectl get replicaset
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6bbc464978   1         1         1       3m6s
nginx-deployment-794f656f8b   0         0         0       13m
```

The old **ReplicaSet** has no more **Pods** and the new one has been automatically created.

### Debugging Pods

> Definitely one of the commands I will use the most....

```bash 
$ kubectl logs nginx-deployment-6bbc464978-4xd9l
```

The output is empty because `nginx` not logged data yet. So I gonna create another deployment, just for test.

```bash
kubectl create deployment mongo-deployment --image=mongo
```

Check your **Pods**:

```bash
$ kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
mongo-deployment-7994f64674-llvx5   0/1     ContainerCreating   0          9s
nginx-deployment-6bbc464978-4xd9l   1/1     Running             0          7m35s
```

And know check your logs.

```bash
$ kubectl logs mongo-deployment-7994f64674-llvx5
Error from server (BadRequest): container "mongo" in pod "mongo-deployment-7994f64674-llvx5" is waiting to start: ContainerCreating
```

Probably the most useless log I ever seen. So I wait for a minute that k8s finish for create the container.

Meanwhile I'll describe the **pod**

```bash
$ kubectl describe pod mongo-deployment-7994f64674-llvx5
Name:         mongo-deployment-7994f64674-llvx5
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Sun, 29 May 2022 14:03:36 +0200
Labels:       app=mongo-deployment
              pod-template-hash=7994f64674
Annotations:  <none>
Status:       Running
IP:           172.17.0.3
IPs:
  IP:           172.17.0.3
Controlled By:  ReplicaSet/mongo-deployment-7994f64674
Containers:
  mongo:
    Container ID:   docker://2b6b6ea54592fe2271ba135c5d7815a0e8bf1291ea091ea413c3cd1397a9468b
    Image:          mongo
    Image ID:       docker-pullable://mongo@sha256:50d8918de7b076feceb9ba1ee264afd5f67fb4baaff07949f3b9de92cdca79c2
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 29 May 2022 14:07:18 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5k6v4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-5k6v4:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  10m   default-scheduler  Successfully assigned default/mongo-deployment-7994f64674-llvx5 to minikube
  Normal  Pulling    10m   kubelet            Pulling image "mongo"
  Normal  Pulled     7m8s  kubelet            Successfully pulled image "mongo" in 3m38.3143875s
  Normal  Created    7m6s  kubelet            Created container mongo
  Normal  Started    7m5s  kubelet            Started container mong
  ```


> After 4m13s minutes

```bash
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
mongo-deployment-7994f64674-llvx5   1/1     Running   0          4m13s
nginx-deployment-6bbc464978-4xd9l   1/1     Running   0          11m

$ kubectl logs mongo-deployment-7994f64674-llvx5
{"t":{"$date":"2022-05-29T12:07:18.054+00:00"},"s":"I",  "c":"CONTROL",  "id":23285,   "ctx":"-","msg":"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"}

{"t":{"$date":"2022-05-29T12:07:18.067+00:00"},"s":"I",  "c":"NETWORK",  "id":4915701, "ctx":"main","msg":"Initialized wire specification","attr":{"spec":{"incomingExternalClient":{"minWireVersion":0,"maxWireVersion":13},"incomingInternalClient":{"minWireVersion":0,"maxWireVersion":13},"outgoing":{"minWireVersion":0,"maxWireVersion":13},"isInternalClient":true}}}

... bla bla bla
```

Another way to debug a Pod is running the `exec` command. By this way you could interact with the pod.

```bash
$ kubectl exec -it mongo-deployment-7994f64674-llvx5 -- bin/bash
root@mongo-deployment-7994f64674-llvx5:/#
```

> Now you can erase all data 😈

### Deleting a Deployment

```bash
$ kubectl delete deployment mongo-deployment
deployment.apps "mongo-deployment" deleted

$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6bbc464978-4xd9l   1/1     Running   0          24m

$ kubectl get replicaset
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6bbc464978   1         1         1       24m
nginx-deployment-794f656f8b   0         0         0       34m
```

### Apply from a file

> Commands are beautiful and useful, but bro. I'm lazy. Now I gonna create all mi configuration files.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.16
          ports:
            - containerPort: 80
```

This is how `nginx-deployment.yaml` file looks like.

```bash
$ kubectl apply -f nginx-deployment.yaml 
deployment.apps/nginx-deployment created
```

And after that the magic is done.

```bash
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7956bd8bb9-dsvqp   1/1     Running   0          12s
```

> I'm so excited right now.

K8s automatic detect when to create or update a deployment. Now I'm gonna set 2 replicas for my Deployment.


```bash
$ vim nginx-deployment.yaml
```

> Change the replicas from 1 to 2 and re-apply your changes

```bash
$ kubectl apply -f nginx-deployment.yaml 
deployment.apps/nginx-deployment configured

$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7956bd8bb9-dsvqp   1/1     Running   0          4m36s
nginx-deployment-7956bd8bb9-tx2jr   1/1     Running   0          78s

$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           4m51s

$ kubectl get replicaset
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-7956bd8bb9   2         2         2       3m39s
```

K8s updated my deployment and set the replicas from 1 to 2.

