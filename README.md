# sample-docker-swarm-practice

## Procedure

### environment

```
docker-compose up -d
```

```
docker exec manager docker swarm init
```

```
docker exec worker01 docker swarm join --token ${JOIN Token} ${Node}
docker exec worker02 docker swarm join --token ${JOIN Token} ${Node}
docker exec worker03 docker swarm join --token ${JOIN Token} ${Node}
```

```
docker exec manager docker network create --driver=overlay --attachable todoapp
```

### datastore

```
ghq get git@github.com:gihyodocker/tododb.git
```

gihyodocker/tododbに移動して

```
docker image build -t ch04/tododb:latest .
```

```
docker image tag ch04/tododb:latest localhost:5000/ch04/tododb:latest
```

```
docker image push localhost:5000/ch04/tododb:latest
```

戻って

```
docker exec manager docker stack deploy -c /stack/todo-mysql.yml todo_mysql
```

```
docker exec manager docker service ps todo_mysql_master --no-trunc --filter "desired-state=running"
```

```
docker exec manager docker service ps todo_mysql_master --no-trunc --filter "desired-state=running" --format "docker exec -it {{.Node}} docker exec -it {{.Name}}.{{.ID}} bash"
```

```
docker exec e77ebe193e09 docker exec -it todo_mysql_master.1.4jlf7aksx0zwi4obi7y8oqce4 init-data.sh
```

```
docker exec -it e77ebe193e09 docker exec -it todo_mysql_master.1.4jlf7aksx0zwi4obi7y8oqce4 mysql -u gihyo -pgihyo tododb
> SELECT * FROM todo LIMIT 1
```

```
docker exec manager docker service ps todo_mysql_slave --no-trunc --filter "desired-state=running" --format "docker exec -it {{.Node}} docker exec -it {{.Name}}.{{.ID}} bash"

docker exec -it 95097cb2f466 docker exec -it todo_mysql_slave.1.8ruq5h1w7qhbgyn83lm4wttlk mysql -u gihyo -pgihyo tododb
> SELECT * FROM todo LIMIT 1
```

### api server

```
ghq get git@github.com:gihyodocker/todoapi.git
```

gihyodocker/todoapiに移動して

```
docker image build -t ch04/todoapi:latest .
```

```
docker mage tag ch04/todoapi:latest localhost:5000/ch04/todoapi:latesti
```

```
docker image push localhost:5000/ch04/todoapi:latest
```

戻って

```
docker exec manager docker stack deploy -c /stack/todo-app.yml todo_app
```

```
docker exec manager docker service logs -f todo_app_api
```

### nginx

```
ghq get git@github.com:gihyodocker/todonginx.git
```

gihyodocker/todonginxに移動して

```
docker image build -t ch04/nginx:latest .
```

```
docker image tag ch04/nginx:latest localhost:5000/ch04/nginx:latest
```

```
docker image push localhost:5000/ch04/nginx:latest
```

戻って

```
docker exec manager docker stack deploy -c /stack/todo-app.yml todo_app
```

### web server

```
ghq get git@github.com:gihyodocker/todoweb.git
```

gihyodocker/todowebに移動して

```
docker image build -t ch04/todoweb:latest .
```

```
docker image tag ch04/todoweb:latest localhost:5000/ch04/todoweb:latest
```

```
docker image push localhost:5000/ch04/todoweb:latest
```

gihyodocker/todonginxに移動して

```
docker image build -f Dockerfile-nuxt -t ch04/nginx-nuxt:latest .
```

```
docker image tag ch04/nginx-nuxt:latest localhost:5000/ch04/nginx-nuxt:latest
```

```
docker image push localhost:5000/ch04/nginx-nuxt:latest
```

戻って

```
docker exec manager docker stack deploy -c /stack/todo-frontend.yml todo_frontend
```

### ingress

```
docker exec manager docker stack deploy -c /stack/todo-ingress.yml todo_ingress
```

### visualizer

```
docker exec manager docker stack deploy -c /stack/visualizer.yml visualizer
```
