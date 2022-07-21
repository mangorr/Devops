Before start docker container, reset environment:
```
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

```
docker system prune -f
docker volume prune -f
```

Start vscode and reopen in container

```
vscode@nyu:/app$ make login
```

Create new IBM Cloud namespace
```
vscode@nyu:/app$ ibmcloud cr namespace-add yachiru
```
Export to new namespace and build // takes some time
```
vscode@nyu:/app$ export NAMESPACE=yachiru
vscode@nyu:/app$ make build
```
Use docker images to list the local images
```
vscode@nyu:/app$ docker images
REPOSITORY                 TAG       IMAGE ID       CREATED          SIZE
us.icr.io/yachiru/orders   1.0       62e2b26c2586   30 seconds ago   390MB
```

Use docker push to send to IBM Cloud Container Registry
```
vscode@nyu:/app$ docker push us.icr.io/yachiru/orders:1.0
The push refers to repository [us.icr.io/yachiru/orders]
bc151fe41219: Pushed 
44925dcc104e: Pushed 
9f267d2dedd6: Pushed 
9bbcb61a1092: Pushed 
a594c80eb584: Pushed 
9de0365805f7: Pushed 
755710f34a21: Pushed 
83aed24cdd2b: Pushed 
cc3de6b21a41: Pushed 
0255573f4829: Pushed 
43b3c4e3001c: Pushed 
1.0: digest: sha256:9191488a9482b431f503923e877d6855f88813618b533d0ac950a7776cf53538 size: 2626
```

Use ibmcloud cr images to list the IBM Cloud images
```
vscode@nyu:/app$ ibmcloud cr images
Listing images...

Repository                 Tag   Digest         Namespace   Created         Size     Security status   
us.icr.io/yachiru/orders   1.0   9191488a9482   yachiru     2 minutes ago   151 MB   No Issues   

OK
```

Use the kubectl get all command to see what is running in the **default** namespace
```
vscode@nyu:/app$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   172.21.0.1   <none>        443/TCP   5h41m
```

Create new namespace "dev" amd "prod":
```
vscode@nyu:/app/deploy$ kubectl create namespace dev
namespace/dev created
vscode@nyu:/app/deploy$ kubectl create namespace prod
namespace/prod created
```

Check namespace in kubernetes:
```
vscode@nyu:/app/deploy$ kubectl get namespace
NAME              STATUS   AGE
default           Active   2d4h
dev               Active   63s
ibm-cert-store    Active   2d3h
ibm-operators     Active   2d4h
ibm-system        Active   2d4h
kube-node-lease   Active   2d4h
kube-public       Active   2d4h
kube-system       Active   2d4h
prod              Active   8h
```






