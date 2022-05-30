# MongApp

## MongoDB

> Should read every time the `DockerHub` documentation before creating a new Deployment.

At first I've created the secret that contains the `root-username` and the `root-password`. The **Type** I used is *Opaque*. The values of the data must be encoded in base64

> echo -n "whatyouwant" | base64

And create the secrets before your deployment.

This is how my secret configuration file looks like:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  root-username: cm9vdA==
  root-password: dG9vcg==
```

Then apply.

```bash
kubectl apply -f mongodb-secret.yaml
```

Now you can reference your *secret* in your deployment file. It looks something like this:

```yaml
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: root-username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: root-username
```

Now I'll create the Deployment.

```bash
kubectl apply -f mongodb-deployment.yaml

❯ kubectl get pod
NAME                                  READY   STATUS    RESTARTS   AGE
mongodb-deployment-684488f8f8-78h6r   1/1     Running   0          43s
```

Now I had to rename my `mongodb-deployment.yaml` filename to `mongodb.yaml` 'cause I found that the *Service* should be stay together with the *Deployment*.

```bash
❯ k apply -f mongodb.yaml
deployment.apps/mongodb-deployment unchanged
service/mongodb-service created

❯ k get service
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP     3d
mongodb-service   ClusterIP   10.109.111.173   <none>        27017/TCP   58s
```

> I created an alias for kubectl: k. 

Now I'm just gonna create my deployment for the mongo-express. I'll reuse my secrets: `mongodb-secret` and configure a new **ConfigMap**.

For that I created a new file (*mongodb-configmap.yaml*) with the following configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service
```

As you can see the `database_url` is the name of the services. So each time a Node is been recreated and the IP addresses changes: the config-map will recompute all mappers and will return the correctly IP address from the Pod.

Now I'm gonna apply:

```bash
❯ k apply -f mongodb-configmap.yaml
configmap/mongodb-configmap created
```

And this is my final `mongo-exprees` spec indicator:

```yaml
    spec:
      containers:
        - name: mongo-express
          image: mongo-express
          ports:
            - containerPort: 8081
          env:
            - name: ME_CONFIG_MONGODB_ADMINUSERNAME
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: root-username
            - name: ME_CONFIG_MONGODB_ADMINPASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: root-username
            - name: ME_CONFIG_MONGODB_SERVER
              valueFrom:
                configMapKeyRef:
                  name: mongodb-configmap
                  key: database_url
```

Create a new External Service (type: LoadBalancer ~ really??? ~)

> Assigns service an External IP address and so accpets external requests. You must specify `nodePort` (the port exposed) and must be between 30000-32767

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30000
```

```bash
❯ k get service
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes              ClusterIP      10.96.0.1        <none>        443/TCP          3d
mongo-express-service   LoadBalancer   10.103.199.29    <pending>     8081:30000/TCP   18s
mongodb-service         ClusterIP      10.109.111.173   <none>        27017/TCP        24m
```

> Minikube TIP: minikube service mongo-express-service
> This command will assign and external IP Address to that service

#### Didn't you see your mongapp?

Keep calm. Run the following commands:

```bash
> minikube tunnel

❯ k get services
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes              ClusterIP      10.96.0.1        <none>        443/TCP          3d2h
mongo-express-service   LoadBalancer   10.103.199.29    127.0.0.1     8081:30030/TCP   122m
mongodb-service         ClusterIP      10.109.111.173   <none>        27017/TCP        146m
```

Now take the external ip of your service and run it on the first port

> 127.0.0.1:8081 [More information here](https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-service-with-tunnel)