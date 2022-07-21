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
