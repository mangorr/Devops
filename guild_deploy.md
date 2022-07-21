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
make login
```

Create new IBM Cloud namespace
```
vscode@nyu:/app$ ibmcloud cr namespace-add yachiru
```
Export to new namespace and build
```
vscode@nyu:/app$ export NAMESPACE=yachiru
vscode@nyu:/app$ make build
```
