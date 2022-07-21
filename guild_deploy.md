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

In order to get access to pull docker images in new kubernetes namespace, create new secret for new namespace.
At first check difference between "default" and "dev"
```
vscode@nyu:/app/deploy$ kc get secret -n default
NAME                  TYPE                                  DATA   AGE
all-icr-io            kubernetes.io/dockerconfigjson        1      2d4h
default-token-psxkw   kubernetes.io/service-account-token   3      2d4h
postgres-creds        Opaque
```

```
vscode@nyu:/app/deploy$ kc get secret -n dev
NAME                  TYPE                                  DATA   AGE
default-token-9ljvx   kubernetes.io/service-account-token   3      2m28s
```

Create secret for "dev":
```
vscode@nyu:/app/deploy$ kc get secret all-icr-io -o yaml > icr.yaml
```

Change namespace to "dev":
```
apiVersion: v1
data:
  .dockerconfigjson: eyAiYXV0aHMiOiB7ICJpY3IuaW8iOiB7ICJhdXRoIjogImFXRnRZWEJwYTJWNU9taDBjQzAzWlcwd2NtRldWamh5Y2toYWVGWTNVVzFLUWtrNVYyaHNYMk5PU0dRd2IyVjVWbk5zV0RGMSIsICJ1c2VybmFtZSI6ICJpYW1hcGlrZXkiLCAiZW1haWwiOiAiaWFtYXBpa2V5IiwgInBhc3N3b3JkIjogImh0cC03ZW0wcmFWVjhyckhaeFY3UW1KQkk5V2hsX2NOSGQwb2V5VnNsWDF1IiB9LCJ1cy5pY3IuaW8iOiB7ICJhdXRoIjogImFXRnRZWEJwYTJWNU9taDBjQzAzWlcwd2NtRldWamh5Y2toYWVGWTNVVzFLUWtrNVYyaHNYMk5PU0dRd2IyVjVWbk5zV0RGMSIsICJ1c2VybmFtZSI6ICJpYW1hcGlrZXkiLCAiZW1haWwiOiAiaWFtYXBpa2V5IiwgInBhc3N3b3JkIjogImh0cC03ZW0wcmFWVjhyckhaeFY3UW1KQkk5V2hsX2NOSGQwb2V5VnNsWDF1IiB9LCJ1ay5pY3IuaW8iOiB7ICJhdXRoIjogImFXRnRZWEJwYTJWNU9taDBjQzAzWlcwd2NtRldWamh5Y2toYWVGWTNVVzFLUWtrNVYyaHNYMk5PU0dRd2IyVjVWbk5zV0RGMSIsICJ1c2VybmFtZSI6ICJpYW1hcGlrZXkiLCAiZW1haWwiOiAiaWFtYXBpa2V5IiwgInBhc3N3b3JkIjogImh0cC03ZW0wcmFWVjhyckhaeFY3UW1KQkk5V2hsX2NOSGQwb2V5VnNsWDF1IiB9LCJkZS5pY3IuaW8iOiB7ICJhdXRoIjogImFXRnRZWEJwYTJWNU9taDBjQzAzWlcwd2NtRldWamh5Y2toYWVGWTNVVzFLUWtrNVYyaHNYMk5PU0dRd2IyVjVWbk5zV0RGMSIsICJ1c2VybmFtZSI6ICJpYW1hcGlrZXkiLCAiZW1haWwiOiAiaWFtYXBpa2V5IiwgInBhc3N3b3JkIjogImh0cC03ZW0wcmFWVjhyckhaeFY3UW1KQkk5V2hsX2NOSGQwb2V5VnNsWDF1IiB9LCJhdS5pY3IuaW8iOiB7ICJhdXRoIjogImFXRnRZWEJwYTJWNU9taDBjQzAzWlcwd2NtRldWamh5Y2toYWVGWTNVVzFLUWtrNVYyaHNYMk5PU0dRd2IyVjVWbk5zV0RGMSIsICJ1c2VybmFtZSI6ICJpYW1hcGlrZXkiLCAiZW1haWwiOiAiaWFtYXBpa2V5IiwgInBhc3N3b3JkIjogImh0cC03ZW0wcmFWVjhyckhaeFY3UW1KQkk5V2hsX2NOSGQwb2V5VnNsWDF1IiB9LCJqcC5pY3IuaW8iOiB7ICJhdXRoIjogImFXRnRZWEJwYTJWNU9taDBjQzAzWlcwd2NtRldWamh5Y2toYWVGWTNVVzFLUWtrNVYyaHNYMk5PU0dRd2IyVjVWbk5zV0RGMSIsICJ1c2VybmFtZSI6ICJpYW1hcGlrZXkiLCAiZW1haWwiOiAiaWFtYXBpa2V5IiwgInBhc3N3b3JkIjogImh0cC03ZW0wcmFWVjhyckhaeFY3UW1KQkk5V2hsX2NOSGQwb2V5VnNsWDF1IiB9LCJwcml2YXRlLmljci5pbyI6IHsgImF1dGgiOiAiYVdGdFlYQnBhMlY1T21oMGNDMDNaVzB3Y21GV1ZqaHlja2hhZUZZM1VXMUtRa2s1VjJoc1gyTk9TR1F3YjJWNVZuTnNXREYxIiwgInVzZXJuYW1lIjogImlhbWFwaWtleSIsICJlbWFpbCI6ICJpYW1hcGlrZXkiLCAicGFzc3dvcmQiOiAiaHRwLTdlbTByYVZWOHJySFp4VjdRbUpCSTlXaGxfY05IZDBvZXlWc2xYMXUiIH0sInByaXZhdGUudXMuaWNyLmlvIjogeyAiYXV0aCI6ICJhV0Z0WVhCcGEyVjVPbWgwY0MwM1pXMHdjbUZXVmpoeWNraGFlRlkzVVcxS1FrazVWMmhzWDJOT1NHUXdiMlY1Vm5Oc1dERjEiLCAidXNlcm5hbWUiOiAiaWFtYXBpa2V5IiwgImVtYWlsIjogImlhbWFwaWtleSIsICJwYXNzd29yZCI6ICJodHAtN2VtMHJhVlY4cnJIWnhWN1FtSkJJOVdobF9jTkhkMG9leVZzbFgxdSIgfSwicHJpdmF0ZS51ay5pY3IuaW8iOiB7ICJhdXRoIjogImFXRnRZWEJwYTJWNU9taDBjQzAzWlcwd2NtRldWamh5Y2toYWVGWTNVVzFLUWtrNVYyaHNYMk5PU0dRd2IyVjVWbk5zV0RGMSIsICJ1c2VybmFtZSI6ICJpYW1hcGlrZXkiLCAiZW1haWwiOiAiaWFtYXBpa2V5IiwgInBhc3N3b3JkIjogImh0cC03ZW0wcmFWVjhyckhaeFY3UW1KQkk5V2hsX2NOSGQwb2V5VnNsWDF1IiB9LCJwcml2YXRlLmRlLmljci5pbyI6IHsgImF1dGgiOiAiYVdGdFlYQnBhMlY1T21oMGNDMDNaVzB3Y21GV1ZqaHlja2hhZUZZM1VXMUtRa2s1VjJoc1gyTk9TR1F3YjJWNVZuTnNXREYxIiwgInVzZXJuYW1lIjogImlhbWFwaWtleSIsICJlbWFpbCI6ICJpYW1hcGlrZXkiLCAicGFzc3dvcmQiOiAiaHRwLTdlbTByYVZWOHJySFp4VjdRbUpCSTlXaGxfY05IZDBvZXlWc2xYMXUiIH0sInByaXZhdGUuYXUuaWNyLmlvIjogeyAiYXV0aCI6ICJhV0Z0WVhCcGEyVjVPbWgwY0MwM1pXMHdjbUZXVmpoeWNraGFlRlkzVVcxS1FrazVWMmhzWDJOT1NHUXdiMlY1Vm5Oc1dERjEiLCAidXNlcm5hbWUiOiAiaWFtYXBpa2V5IiwgImVtYWlsIjogImlhbWFwaWtleSIsICJwYXNzd29yZCI6ICJodHAtN2VtMHJhVlY4cnJIWnhWN1FtSkJJOVdobF9jTkhkMG9leVZzbFgxdSIgfSwicHJpdmF0ZS5qcC5pY3IuaW8iOiB7ICJhdXRoIjogImFXRnRZWEJwYTJWNU9taDBjQzAzWlcwd2NtRldWamh5Y2toYWVGWTNVVzFLUWtrNVYyaHNYMk5PU0dRd2IyVjVWbk5zV0RGMSIsICJ1c2VybmFtZSI6ICJpYW1hcGlrZXkiLCAiZW1haWwiOiAiaWFtYXBpa2V5IiwgInBhc3N3b3JkIjogImh0cC03ZW0wcmFWVjhyckhaeFY3UW1KQkk5V2hsX2NOSGQwb2V5VnNsWDF1IiB9LCJici5pY3IuaW8iOiB7ICJhdXRoIjogImFXRnRZWEJwYTJWNU9taDBjQzAzWlcwd2NtRldWamh5Y2toYWVGWTNVVzFLUWtrNVYyaHNYMk5PU0dRd2IyVjVWbk5zV0RGMSIsICJ1c2VybmFtZSI6ICJpYW1hcGlrZXkiLCAiZW1haWwiOiAiaWFtYXBpa2V5IiwgInBhc3N3b3JkIjogImh0cC03ZW0wcmFWVjhyckhaeFY3UW1KQkk5V2hsX2NOSGQwb2V5VnNsWDF1IiB9LCJwcml2YXRlLmJyLmljci5pbyI6IHsgImF1dGgiOiAiYVdGdFlYQnBhMlY1T21oMGNDMDNaVzB3Y21GV1ZqaHlja2hhZUZZM1VXMUtRa2s1VjJoc1gyTk9TR1F3YjJWNVZuTnNXREYxIiwgInVzZXJuYW1lIjogImlhbWFwaWtleSIsICJlbWFpbCI6ICJpYW1hcGlrZXkiLCAicGFzc3dvcmQiOiAiaHRwLTdlbTByYVZWOHJySFp4VjdRbUpCSTlXaGxfY05IZDBvZXlWc2xYMXUiIH0sImNhLmljci5pbyI6IHsgImF1dGgiOiAiYVdGdFlYQnBhMlY1T21oMGNDMDNaVzB3Y21GV1ZqaHlja2hhZUZZM1VXMUtRa2s1VjJoc1gyTk9TR1F3YjJWNVZuTnNXREYxIiwgInVzZXJuYW1lIjogImlhbWFwaWtleSIsICJlbWFpbCI6ICJpYW1hcGlrZXkiLCAicGFzc3dvcmQiOiAiaHRwLTdlbTByYVZWOHJySFp4VjdRbUpCSTlXaGxfY05IZDBvZXlWc2xYMXUiIH0sInByaXZhdGUuY2EuaWNyLmlvIjogeyAiYXV0aCI6ICJhV0Z0WVhCcGEyVjVPbWgwY0MwM1pXMHdjbUZXVmpoeWNraGFlRlkzVVcxS1FrazVWMmhzWDJOT1NHUXdiMlY1Vm5Oc1dERjEiLCAidXNlcm5hbWUiOiAiaWFtYXBpa2V5IiwgImVtYWlsIjogImlhbWFwaWtleSIsICJwYXNzd29yZCI6ICJodHAtN2VtMHJhVlY4cnJIWnhWN1FtSkJJOVdobF9jTkhkMG9leVZzbFgxdSIgfSwiZnIyLmljci5pbyI6IHsgImF1dGgiOiAiYVdGdFlYQnBhMlY1T21oMGNDMDNaVzB3Y21GV1ZqaHlja2hhZUZZM1VXMUtRa2s1VjJoc1gyTk9TR1F3YjJWNVZuTnNXREYxIiwgInVzZXJuYW1lIjogImlhbWFwaWtleSIsICJlbWFpbCI6ICJpYW1hcGlrZXkiLCAicGFzc3dvcmQiOiAiaHRwLTdlbTByYVZWOHJySFp4VjdRbUpCSTlXaGxfY05IZDBvZXlWc2xYMXUiIH0sInByaXZhdGUuZnIyLmljci5pbyI6IHsgImF1dGgiOiAiYVdGdFlYQnBhMlY1T21oMGNDMDNaVzB3Y21GV1ZqaHlja2hhZUZZM1VXMUtRa2s1VjJoc1gyTk9TR1F3YjJWNVZuTnNXREYxIiwgInVzZXJuYW1lIjogImlhbWFwaWtleSIsICJlbWFpbCI6ICJpYW1hcGlrZXkiLCAicGFzc3dvcmQiOiAiaHRwLTdlbTByYVZWOHJySFp4VjdRbUpCSTlXaGxfY05IZDBvZXlWc2xYMXUiIH0sImpwMi5pY3IuaW8iOiB7ICJhdXRoIjogImFXRnRZWEJwYTJWNU9taDBjQzAzWlcwd2NtRldWamh5Y2toYWVGWTNVVzFLUWtrNVYyaHNYMk5PU0dRd2IyVjVWbk5zV0RGMSIsICJ1c2VybmFtZSI6ICJpYW1hcGlrZXkiLCAiZW1haWwiOiAiaWFtYXBpa2V5IiwgInBhc3N3b3JkIjogImh0cC03ZW0wcmFWVjhyckhaeFY3UW1KQkk5V2hsX2NOSGQwb2V5VnNsWDF1IiB9LCJwcml2YXRlLmpwMi5pY3IuaW8iOiB7ICJhdXRoIjogImFXRnRZWEJwYTJWNU9taDBjQzAzWlcwd2NtRldWamh5Y2toYWVGWTNVVzFLUWtrNVYyaHNYMk5PU0dRd2IyVjVWbk5zV0RGMSIsICJ1c2VybmFtZSI6ICJpYW1hcGlrZXkiLCAiZW1haWwiOiAiaWFtYXBpa2V5IiwgInBhc3N3b3JkIjogImh0cC03ZW0wcmFWVjhyckhaeFY3UW1KQkk5V2hsX2NOSGQwb2V5VnNsWDF1IiB9fX0K
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{".dockerconfigjson":"eyAiYXV0aHMiOiB7..."},"kind":"Secret","metadata":{"annotations":{},"creationTimestamp":null,"name":"all-icr-io","namespace":"default"},"type":"kubernetes.io/dockerconfigjson"}
  creationTimestamp: "2022-07-18T21:22:26Z"
  name: all-icr-io
  namespace: dev
  resourceVersion: "336"
  uid: bd0a1ee7-32b7-463b-addb-30d7624fe2c9
type: kubernetes.io/dockerconfigjson

```

Update new configure:
```
vscode@nyu:/app/deploy$ kc create -f icr.yaml
secret/all-icr-io created
```

Check authority of "dev":
```
vscode@nyu:/app/deploy$ kc get secret -n dev
NAME                  TYPE                                  DATA   AGE
all-icr-io            kubernetes.io/dockerconfigjson        1      43s
default-token-9ljvx   kubernetes.io/service-account-token   3      7m39s
```







