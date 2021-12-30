## Running MongoDB on Kubernetes 
>  Kubernetes is the industry-leading container orchestration platform. You can use any distribution of Kubernetes to manage the full lifecycle of your MongoDB clusters, wherever you choose to run them, from on-premises infrastructure to the public cloud.

<img src="https://www.cloudsavvyit.com/p/uploads/2021/07/f5932bc2.jpg?width=1198&trim=1,1&bg-color=000&pad=1,1" alt="mongodb" >


### Requirements

- kubectl 
- mongoshell


### Steps 

1) Clone this repository 
```bash

git clone https://github.com/harkiratsm/DigitalOcean-Kubernetes-Challenge.git

```

```bash
cd DigitalOcean-Kubernetes-Challenge
```

Take a Look At ```headless-service.yaml``` 

```vim
apiVersion: v1
kind: Service
metadata:
  name: mongo
  labels:
    app: mongo
spec:
  ports:
  - name: mongo
    port: 27017
    targetPort: 27017
  clusterIP: None
  selector:
    app: mongo
```

Take a look at ```mongodb-statefulset.yaml```

```vim
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  serviceName: "mongo"
  replicas: 3
  template:
    metadata:
      labels:
        app: mongo
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: mongo
        image: mongo
        command: 
        - mongod 
        - "--bind_ip_all"
        - "--replSet"
        - rs0
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongo-volume
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: mongo-volume
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
Now deploy those ```yaml``` file 

Use the Following command
```bash
kubectl create -f .
```
Now check it using the following command
```bash
kubectl get all
```
2) Initilize the replicaset i.e we are gonna connect mongo-0 mongo-1 mongo-2

```bash
kubectl exec -it mongo-0 -- mongosh
```
```bash
rs.initiate()
var cfg = rs.conf()
cfg.members[0].host="mongo-0.mongo:27017"
rs.reconfig()
rs.status()
rs.add("mongo-1.mongo:27017")
rs.add("mongo-2.mongo:27017")
```

Accessing the mongodb replica set 

They are two way with ehihc we can access **Within a Cluster** and **Outside the cluster**

Let discuss about **Outside the cluster**
We have to expose it as LoadBalancer for it we using **[MetalLB](https://metallb.universe.tf/installation/)**

1) Lets first install MetalLB  

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/metallb.yaml
```

2) Now apply ```config.yaml```
```bash
kubectl create -f config.yaml
```
3) Now lets expose the pods 
```bash
kubectl expose pod mongo-0 --port 27017 --target-port 27017 --type LoadBalancer
```
Now expose the other pods also .

4) ```bash
mongosh mongodb://192.168.59.51
```


How to add new Member to the replicaset for that We are going to scale the statefulset
 
```bash
kubectl scale sts mongo --replicas 4
```


Running inside Kubernetes

we are using stateful sets because deploying mongodb as statefulset because members of mongodb replicaset need to talk to each other so for that we need statefulsets .


Usage 
git clone
1) do ls ,  show the output [ss]
2) tell what is headless service
3) tell what is stateful set



