# Day 6 - Docker Compose

## Índice

- [Day 6 - Docker Compose](#day-6---docker-compose)
  - [Índice](#índice)
  - [Primeiro Docker Compose](#primeiro-docker-compose)
  - [Giropops Senhas no compose](#giropops-senhas-no-compose)
    - [Compose file](#compose-file)
    - [Apply compose file](#apply-compose-file)
  - [Docker Compose- Comandos Adicionais](#docker-compose--comandos-adicionais)
    - [Listar stacks docker compose](#listar-stacks-docker-compose)
    - [Obter logs de um serviço](#obter-logs-de-um-serviço)
    - [Obter estatisticas dos serviços](#obter-estatisticas-dos-serviços)
    - [Executar um comando dentro de um container](#executar-um-comando-dentro-de-um-container)
    - [Listar imagens dos serviços](#listar-imagens-dos-serviços)
    - [Stop, Start, Restart, Pause e Unpause Services](#stop-start-restart-pause-e-unpause-services)
  - [Volumes](#volumes)
    - [listar volumes](#listar-volumes)
  - [Build de imagem no Compose](#build-de-imagem-no-compose)
  - [Scale - Escalar services](#scale---escalar-services)
  - [Reservando e Definindo recursos como CPU e memória](#reservando-e-definindo-recursos-como-cpu-e-memória)
  - [Health check](#health-check)
  - [Docker Compose Avançado](#docker-compose-avançado)
    - [Contextos no build](#contextos-no-build)
    - [Environment](#environment)
    - [Volumes](#volumes-1)
    - [Réplicas](#réplicas)
    - [Labels](#labels)
    - [Update-config](#update-config)
    - [Restart Policy](#restart-policy)
    - [Attach USD Device](#attach-usd-device)
    - [DNS](#dns)
    - [Network Avançado (Subnet)](#network-avançado-subnet)
    - [Inspect](#inspect)

## Primeiro Docker Compose

**docker-compose.yaml**

```yaml
services:
  nginx:
    image: nginx:1.29-alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "8080:80"
```

**-d** detach

```bash
$ docker compose up -d
```

```bash
$ docker compose
Usage:  docker compose [OPTIONS] COMMAND

Define and run multi-container applications with Docker

Options:
      --all-resources              Include all resources, even those not used by services
      --ansi string                Control when to print ANSI control characters ("never"|"always"|"auto") (default "auto")
      --compatibility              Run compose in backward compatibility mode
      --dry-run                    Execute command in dry run mode
      --env-file stringArray       Specify an alternate environment file
  -f, --file stringArray           Compose configuration files
      --parallel int               Control max parallelism, -1 for unlimited (default -1)
      --profile stringArray        Specify a profile to enable
      --progress string            Set type of progress output (auto, tty, plain, json, quiet)
      --project-directory string   Specify an alternate working directory
                                   (default: the path of the, first specified, Compose file)
  -p, --project-name string        Project name

Management Commands:
  bridge      Convert compose files into another model

Commands:
  attach      Attach local standard input, output, and error streams to a service's running container
  build       Build or rebuild services
  commit      Create a new image from a service container's changes
  config      Parse, resolve and render compose file in canonical format
  cp          Copy files/folders between a service container and the local filesystem
  create      Creates containers for a service
  down        Stop and remove containers, networks
  events      Receive real time events from containers
  exec        Execute a command in a running container
  export      Export a service container's filesystem as a tar archive
  images      List images used by the created containers
  kill        Force stop service containers
  logs        View output from containers
  ls          List running compose projects
  pause       Pause services
  port        Print the public port for a port binding
  ps          List containers
  publish     Publish compose application
  pull        Pull service images
  push        Push service images
  restart     Restart service containers
  rm          Removes stopped service containers
  run         Run a one-off command on a service
  scale       Scale services
  start       Start services
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop services
  top         Display the running processes
  unpause     Unpause services
  up          Create and start containers
  version     Show the Docker Compose version information
  volumes     List volumes
  wait        Block until containers of all (or specified) services stop.
  watch       Watch build context for service and rebuild/refresh containers when files are updated

Run 'docker compose COMMAND --help' for more information on a command.
```

## Giropops Senhas no compose

### Compose file

**giropops-stack.yaml**

```yaml
services:
  giropops-senhas:
    image: pauloneves/giropops-senhas-assinada:1.0
    container_name: giropops-senhas-assinada
    restart: unless-stopped
    environment:
      - REDIS_HOST=redis
    ports:
      - "5000:5000"
    depends_on:
      - redis
    networks:
      - giropops-net
  redis:
    image: redis:8.4
    container_name: redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    networks:
      - giropops-net

networks:
  giropops-net:
    driver: bridge
```

### Apply compose file

```bash
$ docker compose -f giropops-stack.yaml up -d     
[+] up 3/3
 ✔ Network day-6_giropops-net        Created                                                                                                                           0.0s
 ✔ Container day-6-redis-1           Created                                                                                                                           0.0s
 ✔ Container day-6-giropops-senhas-1 Created
```

## Docker Compose- Comandos Adicionais

### Listar stacks docker compose

```bash
$ docker compose ls                          
NAME                      STATUS              CONFIG FILES
alt-esb-dev               running(1)          /data/compose/1/docker-compose.yml
day-6                     running(2)          /home/paulo/workspace/linuxtips/PICK-2026/day-6/giropops-stack.yaml
grafana_prometheus_demo   running(1)          /opt/workspace/Grafana_Prometheus_Demo/docker-compose.yml
```

### Obter logs de um serviço

```bash
$ docker compose -f giropops-stack.yaml logs -f giropops-senhas                                                        
giropops-senhas-1  |  * Serving Flask app 'app.py'
giropops-senhas-1  |  * Debug mode: off
giropops-senhas-1  | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
giropops-senhas-1  |  * Running on all addresses (0.0.0.0)
giropops-senhas-1  |  * Running on http://127.0.0.1:5000
giropops-senhas-1  |  * Running on http://192.168.80.3:5000
giropops-senhas-1  | Press CTRL+C to quit
```

### Obter estatisticas dos serviços

**stats**

```bash
$ docker compose -f giropops-stack.yaml stats --no-stream
CONTAINER ID   NAME                      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O     PIDS
c82f5610cd2a   day-6-giropops-senhas-1   0.01%     30.11MiB / 27.16GiB   0.11%     6.4kB / 126B    0B / 8.19kB   1
2157e113b553   day-6-redis-1             0.21%     9.734MiB / 27.16GiB   0.03%     6.57kB / 126B   0B / 0B       6
```

**top**

```bash
$ docker compose -f giropops-stack.yaml top                                                                                                                          1 ↵
SERVICE          #   UID      PID      PPID     C   STIME  TTY  TIME      CMD
giropops-senhas  1   65532    1302816  1302792  0   11:31  ?    00:00:00  python -m flask run
redis            1   dnsmasq  1302512  1302489  0   11:31  ?    00:00:00  redis-server *:6379
```

### Executar um comando dentro de um container

```bash
$ docker compose -f giropops-stack.yaml exec redis bash                                                                     
root@2157e113b553:/data# ls
root@2157e113b553:/data# ls /
bin  boot  data  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@2157e113b553:/data#
```

### Listar imagens dos serviços

```bash
$ docker compose -f giropops-stack.yaml images         
CONTAINER                 REPOSITORY                            TAG                 PLATFORM            IMAGE ID            SIZE                CREATED
day-6-giropops-senhas-1   pauloneves/giropops-senhas-assinada   1.0                 linux/amd64         fc7abfe606f1        83MB                5 days ago
day-6-redis-1             redis                                 8.4                 linux/amd64         4326c43a061b        139MB               2 weeks ago
```

### Stop, Start, Restart, Pause e Unpause Services

**Pause**

```bash
$ docker compose -f giropops-stack.yaml pause            
[+] pause 2/2
 ✔ Container day-6-redis-1           Paused                              0.0s
 ✔ Container day-6-giropops-senhas-1 Paused 
 ```


**Unpause**

```bash
$ docker compose -f giropops-stack.yaml unpause
[+] unpause 2/2
 ✔ Container day-6-giropops-senhas-1 Unpaused                            0.0s
 ✔ Container day-6-redis-1           Unpaused 
 ```


**Restart**

```bash
$ docker compose -f giropops-stack.yaml restart
[+] restart 0/2
 ⠼ Container day-6-giropops-senhas-1 Restarting                         10.5s
 ⠼ Container day-6-redis-1           Restarting   
 ```


**Stop**

```bash
$ docker compose -f giropops-stack.yaml stop                      [+] stop 2/2
 ✔ Container day-6-giropops-senhas-1 Stopped                             10.3s
 ✔ Container day-6-redis-1           Stopped                             0.4s
```

**Start**

```bash
$ docker compose -f giropops-stack.yaml start            
[+] start 2/2
 ✔ Container day-6-redis-1           Started                               0.1s
 ✔ Container day-6-giropops-senhas-1 Started                               0.2s
 ```


## Volumes

```yaml
services:
  giropops-senhas:
    image: pauloneves/giropops-senhas-assinada:1.0
    restart: unless-stopped
    environment:
      - REDIS_HOST=redis
    ports:
      - "5000:5000"
    depends_on:
      - redis
    networks:
      - giropops-net
    volumes:
      - strigus:/strigus
  redis:
    image: redis:8.4
    restart: unless-stopped
    ports:
      - "6379:6379"
    networks:
      - giropops-net

networks:
  giropops-net:
    driver: bridge
volumes:
  strigus:
```

**-p** permite definir o nome da stack (projeto)

```bash
 $ docker compose -p giropops -f giropops-stack.yaml up -d
[+] up 4/4
 ✔ Network giropops_giropops-net        Created                            0.0s
 ✔ Volume giropops_strigus              Created                            0.0s
 ✔ Container giropops-redis-1           Created                            0.0s
 ✔ Container giropops-giropops-senhas-1 Created                            0.0s
```

### listar volumes

```bash
$ docker volume ls   
DRIVER    VOLUME NAME
local     0efc66f56126bee45bb86ecc5c2be960f4b51f931dc11e0a2b12c9f5cfc66e19
local     1d1d365d2118647dc86d0ea169d5d20651aaf7bb27177c8e077fd123877ca462
local     2a52dc1355e5f8e142c36e99148de8075496ee8a84af669c17b4d7e386f3b546
...
local     giropops_strigus
local     nexus-data
local     portainer_data
```

## Build de imagem no Compose

```yaml
services:
  giropops-senhas:
    build: 
      context: ../giropops-senhas
      dockerfile: Dockerfile
    restart: unless-stopped
    environment:
      - REDIS_HOST=redis
    ports:
      - "5000:5000"
    depends_on:
      - redis
    networks:
      - giropops-net
    volumes:
      - strigus:/strigus
  redis:
    image: redis:8.4
    restart: unless-stopped
    ports:
      - "6379:6379"
    networks:
      - giropops-net

networks:
  giropops-net:
    driver: bridge
volumes:
  strigus:
```

```bash
$ docker compose -p giropops -f giropops-stack.yaml up -d
[+] Building 14.4s (19/19) FINISHED                                                                                                                                        
 => [internal] load local bake definitions                                                                                                                            0.0s
 => => reading from stdin 593B                                                                                                                                        0.0s
 => [internal] load build definition from Dockerfile                                                                                                                  0.0s
 => => transferring dockerfile: 756B                                                                                                                                  0.0s
 => [internal] load metadata for cgr.dev/chainguard/python:latest                                                                                                     1.5s
 => [internal] load metadata for cgr.dev/chainguard/python:latest-dev                                                                                                 1.4s
 => [internal] load .dockerignore                                                                                                                                     0.0s
 => => transferring context: 2B                                                                                                                                       0.0s
 => [builder 1/5] FROM cgr.dev/chainguard/python:latest-dev@sha256:b397f616ddbef7e70cb5f4af8a01ffea05d2ffa28601b04d672e041b1eec3872                                   8.7s
 => => resolve cgr.dev/chainguard/python:latest-dev@sha256:b397f616ddbef7e70cb5f4af8a01ffea05d2ffa28601b04d672e041b1eec3872                                           0.0s
 => => sha256:b814c6a9e538f9cf8730dd700f470b441dae270a0d12115e1df561fd1d178fb7 2.51kB / 2.51kB                                                                        0.0s
 => => sha256:b397f616ddbef7e70cb5f4af8a01ffea05d2ffa28601b04d672e041b1eec3872 1.01kB / 1.01kB                                                                        0.0s
 => => sha256:7b9bb751c8fdac4de38f3403d1e3b3716b5fcfc21350dd9f4f78ee49ff4c2aca 2.82kB / 2.82kB                                                                        0.0s
 => => sha256:991ddfab8b6049ed7afec441be419c15e77454b24e9dad58d0451ece8ad6d648 141.33MB / 141.33MB                                                                    6.2s
 => => sha256:cc6c6f121c14af505301622af6480517a58d914239b03a4b3feab2f4a156a018 23.64MB / 23.64MB                                                                      2.5s
 => => sha256:e3ceb9789d4ef42352879b1c1ebeb4a051dd0c3f986f9dcdff41eb5005e80409 25.20MB / 25.20MB                                                                      3.1s
 => => sha256:63189bbbe249b599530380451d4fbb35e348e99158373ef1e4e7b76f31508183 18.14MB / 18.14MB                                                                      3.2s
 => => sha256:c6de73b30d7245d2f904999753c401329ef45ae38fa601146c5ebc5fc59f4e23 13.00MB / 13.00MB                                                                      3.6s
 => => sha256:3d94ddcfd69ab78fd11b7b1e4281c04c3f55380cedc8704e0cb0dedf9f7d1512 9.20MB / 9.20MB                                                                        3.8s
 => => sha256:490039afa741fe7601f71c51dd752cdc247edbecd42aecc7de60804f8f8e200c 5.74MB / 5.74MB                                                                        4.0s
 => => sha256:a15b4f664c3e88749e6331806d363169b5aa0278364b8bdda162e859d53cbff5 2.51MB / 2.51MB                                                                        4.3s
 => => sha256:6450178657daa36fd99d6e424ac9a828ebad9a644802897b3197022176a409d2 2.03MB / 2.03MB                                                                        4.4s
 => => sha256:42d9b1cd68e23bd177b1a8249a973f241ec5fd898fd4ae744117e4a49cc03df3 16.63MB / 16.63MB                                                                      5.1s
 => => sha256:bce93430c781bc6a26a1c9973cc95e431f8d7426e688b7879dd463cb720899a0 238.92kB / 238.92kB                                                                    4.7s
 => => extracting sha256:991ddfab8b6049ed7afec441be419c15e77454b24e9dad58d0451ece8ad6d648                                                                             1.0s
 => => extracting sha256:cc6c6f121c14af505301622af6480517a58d914239b03a4b3feab2f4a156a018                                                                             0.1s
 => => extracting sha256:e3ceb9789d4ef42352879b1c1ebeb4a051dd0c3f986f9dcdff41eb5005e80409                                                                             0.2s
 => => extracting sha256:63189bbbe249b599530380451d4fbb35e348e99158373ef1e4e7b76f31508183                                                                             0.1s
 => => extracting sha256:c6de73b30d7245d2f904999753c401329ef45ae38fa601146c5ebc5fc59f4e23                                                                             0.2s
 => => extracting sha256:3d94ddcfd69ab78fd11b7b1e4281c04c3f55380cedc8704e0cb0dedf9f7d1512                                                                             0.1s
 => => extracting sha256:490039afa741fe7601f71c51dd752cdc247edbecd42aecc7de60804f8f8e200c                                                                             0.1s
 => => extracting sha256:a15b4f664c3e88749e6331806d363169b5aa0278364b8bdda162e859d53cbff5                                                                             0.1s
 => => extracting sha256:6450178657daa36fd99d6e424ac9a828ebad9a644802897b3197022176a409d2                                                                             0.1s
 => => extracting sha256:42d9b1cd68e23bd177b1a8249a973f241ec5fd898fd4ae744117e4a49cc03df3                                                                             0.2s
 => => extracting sha256:bce93430c781bc6a26a1c9973cc95e431f8d7426e688b7879dd463cb720899a0                                                                             0.0s
 => [internal] load build context                                                                                                                                     0.0s
 => => transferring context: 429B                                                                                                                                     0.0s
 => [stage-1 1/6] FROM cgr.dev/chainguard/python:latest@sha256:292095443fbb5bb18a7fced80b04f6f330e1873e0be0e21d360f0ecd4dbedcf0                                       1.9s
 => => resolve cgr.dev/chainguard/python:latest@sha256:292095443fbb5bb18a7fced80b04f6f330e1873e0be0e21d360f0ecd4dbedcf0                                               0.0s
 => => sha256:292095443fbb5bb18a7fced80b04f6f330e1873e0be0e21d360f0ecd4dbedcf0 1.01kB / 1.01kB                                                                        0.0s
 => => sha256:86ee21ec36403fa91456d4fcac0ed8087e5f7b5b4a5dfd4ee92c1734b3da2c4d 2.82kB / 2.82kB                                                                        0.0s
 => => sha256:2044b06e10106c3a0b1bb73242810055214ee42c53afeaf73908bbbb8ceb6eb1 12.52MB / 12.52MB                                                                      0.8s
 => => sha256:2f76c816474a96e8215bf2d18516968824ad4040d39365bf708586e488fcf7aa 3.21MB / 3.21MB                                                                        0.4s
 => => sha256:f6787bd56ad48a1afb89f229ada27195fd2b66942b7470a357604fbb655f1ba3 2.50kB / 2.50kB                                                                        0.0s
 => => sha256:02fc55a9c757fa6bcfe11d95fb21dbe82723d434ab5cb91cfedf674fb3eb8267 2.84MB / 2.84MB                                                                        0.5s
 => => sha256:33f56615ef90602f8b5f5660e68d8324b9007d76293acdf5d6adf95cfb3627e8 1.22MB / 1.22MB                                                                        0.9s
 => => sha256:2ccd8e311840cbc8b01c60f51e94ab154d4a6d31865f8d0a1904620d1de6d587 1.76MB / 1.76MB                                                                        0.9s
 => => extracting sha256:2044b06e10106c3a0b1bb73242810055214ee42c53afeaf73908bbbb8ceb6eb1                                                                             0.2s
 => => sha256:e3619391d2403ba4d46a90dcbef94ad7807541fdd8b53356d29cabf4dd81d482 528.99kB / 528.99kB                                                                    1.2s
 => => sha256:76efa7341361d723963c07f13f079d6ccd2eacc1610bee9b0a1c19ea02765f9b 884.35kB / 884.35kB                                                                    1.3s
 => => extracting sha256:02fc55a9c757fa6bcfe11d95fb21dbe82723d434ab5cb91cfedf674fb3eb8267                                                                             0.0s
 => => sha256:21690b6db2076c44176b796863c5668386f35f2f09fd8efd3f15caf8925d3688 578.51kB / 578.51kB                                                                    1.3s
 => => extracting sha256:2f76c816474a96e8215bf2d18516968824ad4040d39365bf708586e488fcf7aa                                                                             0.0s
 => => extracting sha256:33f56615ef90602f8b5f5660e68d8324b9007d76293acdf5d6adf95cfb3627e8                                                                             0.0s
 => => extracting sha256:2ccd8e311840cbc8b01c60f51e94ab154d4a6d31865f8d0a1904620d1de6d587                                                                             0.0s
 => => extracting sha256:e3619391d2403ba4d46a90dcbef94ad7807541fdd8b53356d29cabf4dd81d482                                                                             0.0s
 => => sha256:ea0d0f23a42862227e821ec47b1bce9918b389505c6b97fa86e73380d91082b3 366.93kB / 366.93kB                                                                    1.5s
 => => extracting sha256:76efa7341361d723963c07f13f079d6ccd2eacc1610bee9b0a1c19ea02765f9b                                                                             0.0s
 => => extracting sha256:21690b6db2076c44176b796863c5668386f35f2f09fd8efd3f15caf8925d3688                                                                             0.0s
 => => sha256:688cf4b97b765a507c7f19dbab792e9496504e84e4fbd550098a4a653b9dc421 60.92kB / 60.92kB                                                                      1.7s
 => => sha256:1b5f37798f135b7df20b1aae0747dffb823b16839cf15cc3359a8be9e3feea31 1.11MB / 1.11MB                                                                        1.7s
 => => extracting sha256:ea0d0f23a42862227e821ec47b1bce9918b389505c6b97fa86e73380d91082b3                                                                             0.0s
 => => extracting sha256:1b5f37798f135b7df20b1aae0747dffb823b16839cf15cc3359a8be9e3feea31                                                                             0.0s
 => => extracting sha256:688cf4b97b765a507c7f19dbab792e9496504e84e4fbd550098a4a653b9dc421                                                                             0.0s
 => [stage-1 2/6] WORKDIR /app                                                                                                                                        0.0s
 => [builder 2/5] WORKDIR /app                                                                                                                                        0.2s
 => [builder 3/5] RUN python -m venv /app/venv                                                                                                                        2.0s
 => [builder 4/5] COPY requirements.txt .                                                                                                                             0.0s
 => [builder 5/5] RUN /app/venv/bin/pip install --no-cache-dir -r requirements.txt                                                                                    1.3s
 => [stage-1 3/6] COPY --from=builder /app/venv /app/venv                                                                                                             0.2s
 => [stage-1 4/6] COPY app.py .                                                                                                                                       0.0s
 => [stage-1 5/6] COPY templates templates/                                                                                                                           0.0s
 => [stage-1 6/6] COPY static static/                                                                                                                                 0.0s
 => exporting to image                                                                                                                                                0.1s
 => => exporting layers                                                                                                                                               0.1s
 => => writing image sha256:2afb879f6e928fcf8c3bb4e98e47682613949cb91997852cdfaebb21926d3eb0                                                                          0.0s
 => => naming to docker.io/library/giropops-giropops-senhas                                                                                                           0.0s
 => resolving provenance for metadata file                                                                                                                            0.0s
[+] up 4/4
 ✔ Image giropops-giropops-senhas       Built                                                                                                                         14.5s
 ✔ Network giropops_giropops-net        Created                                                                                                                       0.0s
 ✔ Container giropops-redis-1           Created                                                                                                                       0.0s
 ✔ Container giropops-giropops-senhas-1 Created                                                                                                                       0.0s
```

## Scale - Escalar services

```bash
$ docker compose -p giropops -f giropops-stack.yaml up -d --scale giropops-senhas=3                                                                                  1 ↵
[+] up 5/5
 ✔ Network giropops_giropops-net        Created                                                                                                                        0.0s
 ✔ Container giropops-redis-1           Created                                                                                                                        0.0s
 ✔ Container giropops-giropops-senhas-1 Created                                                                                                                        0.0s
 ✔ Container giropops-giropops-senhas-2 Created                                                                                                                        0.0s
 ✔ Container giropops-giropops-senhas-3 Created                                                                                                                        0.0s
Error response from daemon: failed to set up container networking: driver failed programming external connectivity on endpoint giropops-giropops-senhas-1 (167f5f243f333c32347cadb8e4d1fcb0698f8492cbc63cbbaa01f202a4077c9f): Bind for 0.0.0.0:5000 failed: port is already allocated
```

```yaml
services:
  giropops-senhas:
    build: 
      context: ../giropops-senhas
      dockerfile: Dockerfile
    restart: unless-stopped
    environment:
      - REDIS_HOST=redis
    ports:
      - "5000:5000"
    depends_on:
      - redis
    networks:
      - giropops-net
    volumes:
      - strigus:/strigus
  redis:
    image: redis:8.4
    restart: unless-stopped
#    ports:
#      - "6379:6379"    # nao preciso expoer a porta do redis, pois ele só será acessado internamente pelos outros serviços
    networks:
      - giropops-net

networks:
  giropops-net:
    driver: bridge
volumes:
  strigus:
```

```bash
$ docker compose -p giropops -f giropops-stack.yaml up -d --scale redis=3          
[+] up 4/4
 ✔ Container giropops-redis-3           Created                     0.0ss
 ✔ Container giropops-redis-2           Created                     0.0ss
 ✔ Container giropops-redis-1           Recreated                   0.4ss
 ✔ Container giropops-giropops-senhas-1 Recreated                   10.1s
```

## Reservando e Definindo recursos como CPU e memória

**Define que o serviço giropops-senhas depende do redis**

```yaml
    depends_on:
      - redis
```

**Define os recursos minimos e máximos do serviço giropops**

```yaml
    deploy:
      resources:
        reservations:
          cpus: '0.25'
          memory: 256M
        limits:
          cpus: '0.5'
          memory: 256M
```

```yaml
services:
  giropops-senhas:
    build: 
      context: ../giropops-senhas
      dockerfile: Dockerfile
    restart: unless-stopped
    environment:
      - REDIS_HOST=redis
    ports:
      - "5000:5000"
    depends_on:
      - redis
    networks:
      - giropops-net
    volumes:
      - strigus:/strigus
    deploy:
      resources:
        reservations:
          cpus: '0.25'
          memory: 256M
        limits:
          cpus: '0.5'
          memory: 256M
  redis:
    image: redis:8.4
    restart: unless-stopped
#    ports:
#      - "6379:6379"    # nao preciso expoer a porta do redis, pois ele só será acessado internamente pelos outros serviços
    networks:
      - giropops-net


networks:
  giropops-net:
    driver: bridge
volumes:
  strigus:
```

```bash
$ docker inspect giropops-giropops-senhas-1                              
[
    {
        "Id": "42588896d129abe0f45d31248cb3a9315ffa04b7ef5df6ee6e8ee422756db53d",
        "Created": "2026-02-17T12:13:30.116244118Z",
        "Path": "python",
        "Args": [
            "-m",
            "flask",
            "run"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 1346149,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2026-02-17T12:13:30.519374756Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        ...
        "HostConfig": {
            "Binds": [
                "giropops_strigus:/strigus:rw"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "giropops_giropops-net",
            "PortBindings": {
                "5000/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "5000"
                    }
                ]
            },
            ...

            "Memory": 268435456,
            "NanoCpus": 500000000,
            "CgroupParent": "",
            ...
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": null,
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 268435456,
            ...
        },

        "Mounts": [
            {
                "Type": "volume",
                "Name": "giropops_strigus",
                "Source": "/var/lib/docker/volumes/giropops_strigus/_data",
                "Destination": "/strigus",
                "Driver": "local",
                "Mode": "rw",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "42588896d129",
            "Domainname": "",
            "User": "65532",
            "AttachStdin": false,
            "AttachStdout": true,
            "AttachStderr": true,
            "ExposedPorts": {
                "5000/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "REDIS_HOST=redis",
                "PATH=/app/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/bin:/usr/sbin:/sbin:/bin",
                "SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt",
                "FLASK_APP=app.py",
                "FLASK_RUN_HOST=0.0.0.0"
            ],
            "Cmd": null,
            "Image": "giropops-giropops-senhas",
            "Volumes": null,
            "WorkingDir": "/app",
            "Entrypoint": [
                "python",
                "-m",
                "flask",
                "run"
            ],
            ...
        },
        "NetworkSettings": {
            "SandboxID": "0b34e08dcd69c68d0063a9f81551f2e6ddfe026db134309185d8979549c9666b",
            "SandboxKey": "/var/run/docker/netns/0b34e08dcd69",
            "Ports": {
                "5000/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "5000"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "5000"
                    }
                ]
            },
            "Networks": {
                "giropops_giropops-net": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "giropops-giropops-senhas-1",
                        "giropops-senhas"
                    ],
                    "DriverOpts": null,
                    "GwPriority": 0,
                    "NetworkID": "aee7ae409c54863abfe9dfe27f47bb95f1e6af9fa431871dc0d2070649212364",
                    "EndpointID": "868022bac47ad06da1202b949847d6230e022d4d88f1134bb46d5cd0f5fd32ef",
                    "Gateway": "192.168.80.1",
                    "IPAddress": "192.168.80.5",
                    "MacAddress": "fa:2d:de:cc:9f:8a",
                    "IPPrefixLen": 20,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": [
                        "giropops-giropops-senhas-1",
                        "giropops-senhas",
                        "42588896d129"
                    ]
                }
            }
        }
    }
]
```

## Health check

```yaml
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
```

```yaml
services:
  giropops-senhas:
    build: 
      context: ../giropops-senhas
      dockerfile: Dockerfile
    restart: unless-stopped
    environment:
      - REDIS_HOST=redis
    ports:
      - "5000:5000"
    depends_on:
      - redis
    networks:
      - giropops-net
    volumes:
      - strigus:/strigus
    deploy:
      resources:
        reservations:
          cpus: '0.25'
          memory: 256M
        limits:
          cpus: '0.5'
          memory: 256M
  redis:
    image: redis:8.4
#   command: ["redis-server", "--appendonly", "yes"]     # habilita o modo de persistência do Redis, garantindo que os dados sejam salvos mesmo após reinicializações
    restart: unless-stopped
#    ports:
#      - "6379:6379"    # nao preciso expoer a porta do redis, pois ele só será acessado internamente pelos outros serviços
    networks:
      - giropops-net
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

networks:
  giropops-net:
    driver: bridge
volumes:
  strigus:
```

**Recriar Stack**

```bash
$ docker compose -p giropops -f giropops-stack.yaml up -d --scale redis=1
[+] up 2/2
 ✔ Container giropops-redis-1           Running                     0.0ss
 ✔ Container giropops-giropops-senhas-1 Recreated 
```

```bash
 docker container ls
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS                         PORTS                                                      NAMES
c144410d2a1d   giropops-giropops-senhas        "python -m flask run"    16 seconds ago   Up 16 seconds                  0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp                giropops-giropops-senhas-1
2012c27127e3   redis:8.4                       "docker-entrypoint.s…"   16 seconds ago   Up 16 seconds (healthy)        6379/tcp                                                   giropops-redis-1
```

```yaml
services:
  giropops-senhas:
    build: 
      context: ../giropops-senhas
      dockerfile: Dockerfile
    restart: unless-stopped
    environment:
      - REDIS_HOST=redis
    ports:
      - "5000:5000"
    depends_on:
      - redis
    networks:
      - giropops-net
    volumes:
      - strigus:/strigus
    deploy:
      resources:
        reservations:
          cpus: '0.25'
          memory: 256M
        limits:
          cpus: '0.5'
          memory: 256M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
  redis:
    image: redis:8.4
#   command: ["redis-server", "--appendonly", "yes"]     # habilita o modo de persistência do Redis, garantindo que os dados sejam salvos mesmo após reinicializações
    restart: unless-stopped
#    ports:
#      - "6379:6379"    # nao preciso expoer a porta do redis, pois ele só será acessado internamente pelos outros serviços
    networks:
      - giropops-net
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

networks:
  giropops-net:
    driver: bridge
volumes:
  strigus:
```

**Recriar Stack**

```bash
$ docker compose -p giropops -f giropops-stack.yaml up -d --scale redis=1
[+] up 2/2
 ✔ Container giropops-redis-1           Running                     0.0ss
 ✔ Container giropops-giropops-senhas-1 Recreated 
```

```bash
$ docker container ls                                                    
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS                            PORTS                                                      NAMES
75c48609927a   giropops-giropops-senhas        "python -m flask run"    13 seconds ago   Up 2 seconds (health: starting)   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp                giropops-giropops-senhas-1
2012c27127e3   redis:8.4                       "docker-entrypoint.s…"   2 minutes ago    Up 2 minutes (healthy)            6379/tcp                                                   giropops-redis-1
```

**Deu erro porque não temos curl na imagem**

```bash
$ docker container ls
CONTAINER ID   IMAGE                           COMMAND                  CREATED         STATUS                         PORTS                                                      NAMES
75c48609927a   giropops-giropops-senhas        "python -m flask run"    2 minutes ago   Up 2 minutes (unhealthy)       0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp                giropops-giropops-senhas-1
2012c27127e3   redis:8.4                       "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes (healthy)         6379/tcp                                                   
```

## Docker Compose Avançado

### Contextos no build

```yaml
  giropops-senhas:
    build: 
      context: ../giropops-senhas
      dockerfile: Dockerfile
```

### Environment

```yaml
    environment:
      - REDIS_HOST=redis
    env_file:
      - .env
```

### Volumes

```yaml

   volumes:
      - type: volume
        source: strigus
        target: /strigus
```

Define um volume chamado strigus usando o driver local, mas com opções para fazer um bind mount. Isso significa que, em vez de criar um volume gerenciado pelo Docker, ele faz um mapeamento direto para a pasta /home/paulo/docker do host. Assim, tudo que for salvo no volume strigus dentro do container será armazenado diretamente nessa pasta do sistema operacional, permitindo fácil acesso e persistência dos dados fora do container.

```yaml
  volumes:
    strigus:
      driver: local
      driver_opts:
        type: "none"
        device: /home/paulo/docker
        o: "bind"
```

### Réplicas

```yaml
    deploy:
      replicas: 2
```

### Labels

```yaml
      labels:
        - "io.linuxtips.description=Giropops application service"
        - "io.linuxtips.version=1.0"
        - "com.docker.stack.namespace=giropops"
```

### Update-config

O parâmetro `update_config` no Docker Compose define como as atualizações de containers devem ocorrer durante um processo de deploy. Ele permite configurar a quantidade de containers atualizados em paralelo (`parallelism`) e o tempo de espera entre cada atualização (`delay`).
Por exemplo, ao atualizar uma aplicação, você pode controlar para que apenas dois containers sejam atualizados ao mesmo tempo, com um intervalo de 10 segundos entre cada atualização.

```yaml
    update_config:
      parallelism: 2
      delay: 10s
```

### Restart Policy

```yaml
    restart_policy:
      condition: on-failure
      delay: 5s
      max_attempts: 3
      window: 60s
```

### Attach USD Device

```yaml
    devices:
      - "/dev/usb:/dev/usb"   # mapeia o dispositivo de leitura de senhas para dentro do container, permitindo que a aplicação acesse o hardware necessário para funcionar
```

### DNS

```yaml
   dns:
     - 8.8.8.8
     - 4.4.4.4
```

### Network Avançado (Subnet)

```yaml
networks:
  giropops-net:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: "172.20.0.0/16"
```

### Inspect

```bash
$ docker inspect 396e8 --format '{{ json .Config.Labels }}' | jq       
{
  "com.docker.compose.config-hash": "2a0f56aaff6ad6d809dcf09278de9e4e8b27aba636f517dcf7b6b8cd4477ec27",
  "com.docker.compose.container-number": "1",
  "com.docker.compose.depends_on": "redis:service_started:false",
  "com.docker.compose.image": "sha256:2afb879f6e928fcf8c3bb4e98e47682613949cb91997852cdfaebb21926d3eb0",
  "com.docker.compose.oneoff": "False",
  "com.docker.compose.project": "giropops",
  "com.docker.compose.project.config_files": "/home/paulo/workspace/linuxtips/PICK-2026/day-6/giropops-stack.yaml",
  "com.docker.compose.project.working_dir": "/home/paulo/workspace/linuxtips/PICK-2026/day-6",
  "com.docker.compose.replace": "giropops-senhas-1",
  "com.docker.compose.service": "giropops-senhas",
  "com.docker.compose.version": "5.0.2",
  "com.docker.stack.namespace": "giropops",
  "dev.chainguard.image.title": "python",
  "dev.chainguard.package.main": "",
  "io.linuxtips.description": "Giropops application service",
  "io.linuxtips.version": "1.0",
  "org.opencontainers.image.authors": "Chainguard Team https://www.chainguard.dev/",
  "org.opencontainers.image.created": "2026-02-13T14:04:42Z",
  "org.opencontainers.image.source": "https://github.com/chainguard-images/images/tree/main/images/python",
  "org.opencontainers.image.title": "python",
  "org.opencontainers.image.url": "https://images.chainguard.dev/directory/image/python/overview",
  "org.opencontainers.image.vendor": "Chainguard"
}
```