
# Day 2 - Imagens do Container


## Índice

- [Day 2 - Imagens do Container](#day-2---imagens-do-container)
  - [Índice](#índice)
  - [O que são imagens de container ?](#o-que-são-imagens-de-container-)
  - [O meu primeiro Dockerfile](#o-meu-primeiro-dockerfile)
    - [Exemplo de Dockerfile com Ubuntu e Nginx](#exemplo-de-dockerfile-com-ubuntu-e-nginx)
    - [Build da imagem](#build-da-imagem)
  - [Conhecendo mais parâmetros no Dockerfile](#conhecendo-mais-parâmetros-no-dockerfile)
  - [Dockerfile e Entrypoint](#dockerfile-e-entrypoint)
  - [Adicionando HEALTHCHECK ao nosso Dockerfile](#adicionando-healthcheck-ao-nosso-dockerfile)
  - [Descomplicando o meu Dockerfile](#descomplicando-o-meu-dockerfile)
  - [Desafio prático](#desafio-prático)


## O que são imagens de container ?

Imagens de container são ficheiros imutáveis que contêm tudo o que é necessário para executar uma aplicação: código, bibliotecas, dependências, variáveis de ambiente e ficheiros de configuração. Funcionam como um modelo para criar containers, garantindo que a aplicação funcione de forma consistente em qualquer ambiente. Cada imagem pode ser composta por várias camadas, optimizando o armazenamento e a partilha entre diferentes containers.

![Imagem de Container](/images/Docker_Image_Layers_01.png)


## O meu primeiro Dockerfile

Um ficheiro Dockerfile é um documento de texto que contém instruções para construir uma imagem de container. Cada linha define um passo do processo, como a base do sistema, instalação de pacotes, cópia de ficheiros e configuração do ambiente.

**Notas especiais:**
- Cada RUN representa uma camada
- A ordem das instruções afecta o cache e o tamanho final da imagem
- COPY e ADD também criam novas camadas
- CMD define o comando padrão, mas pode ser sobrescrito na execução
- ENV define variáveis de ambiente disponíveis no container
- EXPOSE apenas documenta a porta, não a publica
- LABEL permite adicionar metadados à imagem
- É recomendado minimizar o número de camadas para imagens mais leves


### Exemplo de Dockerfile com Ubuntu e Nginx

**nome do ficheiro:** Dockerfile

**Exemplo simples da aula**
```dockerfile
FROM ubuntu:24.04

# Install nginx
RUN apt update && apt install -y nginx 
# expose port 80
EXPOSE 80

# command to run nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]
```


### Build da imagem

```bash
$ docker image build -t meu-nginx:1.0 .
+] Building 8.9s (7/7) FINISHED                                                                                              docker:default
 => [internal] load build definition from Dockerfile                                                                                     0.0s
 => => transferring dockerfile: 217B                                                                                                     0.0s
 => [internal] load metadata for docker.io/library/ubuntu:24.04                                                                          1.7s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                            0.0s
 => [internal] load .dockerignore                                                                                                        0.0s
 => => transferring context: 2B                                                                                                          0.0s
 => [1/2] FROM docker.io/library/ubuntu:24.04@sha256:cd1dba651b3080c3686ecf4e3c4220f026b521fb76978881737d24f200828b2b                    0.0s
 => => resolve docker.io/library/ubuntu:24.04@sha256:cd1dba651b3080c3686ecf4e3c4220f026b521fb76978881737d24f200828b2b                    0.0s
 => => sha256:cd1dba651b3080c3686ecf4e3c4220f026b521fb76978881737d24f200828b2b 6.69kB / 6.69kB                                           0.0s
 => => sha256:a4453623f2f8319cfff65c43da9be80fe83b1a7ce689579b475867d69495b782 424B / 424B                                               0.0s
 => => sha256:493218ed0f404132311952996fea8ce85e50c49f5a717f26f25c52a25fcb2e56 2.30kB / 2.30kB                                           0.0s
 => [2/2] RUN apt update && apt install -y nginx                                                                                         6.9s
 => exporting to image                                                                                                                   0.1s 
 => => exporting layers                                                                                                                  0.1s 
 => => writing image sha256:83a80dbac42e13fb6946fcb9b5b1daf8b7be7516cf9c25eeef30057c1af66c16                                             0.0s 
 => => naming to docker.io/library/meu-nginx:1.0                                                                                         0.0s 

 $
 $ docker image ls
                                                                                                                         i Info →   U  In Use
 IMAGE                           ID             DISK USAGE   CONTENT SIZE   EXTRA  
meu-nginx:1.0                   83a80dbac42e        143MB             0B        
mysql:8                         4f1413420360        545MB             0B    U   
mysql:8.0                       33037edcac9b        444MB             0B    U   
openzipkin/zipkin:latest        ad5bf93e3f50        165MB             0B    U   
pauloneves/opsdeck-api:0.0.1    91c84a05acbc        380MB             0B        
pauloneves/opsdeck-api:0.0.2    b3e2bcc720d9        380MB             0B        
portainer/portainer-ce:latest   1a0fb356ea35        294MB             0B    U   
sonatype/nexus3:latest          8716903d1912        629MB             0B    U   
```

```bash
$ docker container run -d -p 8080:80 --name meu-nginx meu-nginx:1.0
be2d823037be1841e2f30a31bc038432e0ef2a822c5cba60f72b0bbf92f57023
$
$ $ docker container ls                                              
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS           PORTS                                               NAMES
be2d823037be   meu-nginx:1.0                   "nginx -g 'daemon of…"   38 seconds ago   Up 37 seconds    0.0.0.0:8080->80/tcp, [::]:8080->80/tcp          meu-nginx
80f8e6004515   kindest/node:v1.27.3            "/usr/local/bin/entr…"   3 weeks ago      Up 3 days        127.0.0.1:34611->6443/tcp                girus-control-plane
$ 
$ curl localhost:8080                     
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
$
```


## Conhecendo mais parâmetros no Dockerfile


```dockerfile
FROM ubuntu:24.04
RUN apt-get update && \
	apt-get install -y nginx && \
	apt-get clean && rm -rf /var/lib/apt/lists/*
COPY index.html /var/www/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
WORKDIR /var/www/html
ENV APP_VERSION=1.0.0
```

Este Dockerfile cria uma imagem baseada no Ubuntu, instala o Nginx, copia um ficheiro `index.html` para o diretório web e expõe a porta 80.



```bash
$ docker image build -t meu-nginx:2.0 .
[+] Building 0.5s (9/9) FINISHED                                                                                               docker:default
 => [internal] load build definition from Dockerfile                                                                                     0.0s
 => => transferring dockerfile: 288B                                                                                                     0.0s
 => [internal] load metadata for docker.io/library/ubuntu:24.04                                                                          0.4s
 => [internal] load .dockerignore                                                                                                        0.0s
 => => transferring context: 2B                                                                                                          0.0s
 => [1/4] FROM docker.io/library/ubuntu:24.04@sha256:cd1dba651b3080c3686ecf4e3c4220f026b521fb76978881737d24f200828b2b                    0.0s
 => [internal] load build context                                                                                                        0.0s
 => => transferring context: 32B                                                                                                         0.0s
 => CACHED [2/4] RUN apt-get update &&  apt-get install -y nginx &&  apt-get clean && rm -rf /var/lib/apt/lists/*                        0.0s
 => CACHED [3/4] COPY index.html /var/www/html/index.html                                                                                0.0s
 => CACHED [4/4] WORKDIR /var/www/html                                                                                                   0.0s
 => exporting to image                                                                                                                   0.0s
 => => exporting layers                                                                                                                  0.0s
 => => writing image sha256:2c8112c3c359ea2dde18f85b999ed11b8eb0b9a578b98e73527a85eb816b1e9a                                             0.0s
 => => naming to docker.io/library/meu-nginx:2.0                                                                                         0.0s

```

**Testando a minha nova imagem**

```bash
$ docker container rm -f meu-nginx 
meu-nginx
$
$ docker container run -d -p 8080:80 --name meu-nginx meu-nginx:2.0
d74cb20d0e9293130599bad2b37a16993008f8aa5887f55d33bf2d07629c34cd
$ curl localhost:8080
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Olá Mundo!</title>
</head>
<body>
    <h1>Olá Mundo!</h1>
</body>
</html>
```

## Dockerfile e Entrypoint

```dockerfile
FROM ubuntu:24.04
LABEL maintainer="email@email.com"
LABEL version="1.0"
LABEL description="Imagem com Nginx baseada em Ubuntu"
LABEL org.opencontainers.image.source="https://github.com/paulozagaloneves/PICK-2026"
LABEL org.opencontainers.image.licenses="MIT"
RUN apt-get update && \
	apt-get install -y nginx && \
	apt-get clean && rm -rf /var/lib/apt/lists/*
COPY index.html /var/www/html/index.html
EXPOSE 80
WORKDIR /var/www/html
ENV APP_VERSION=1.0.0
# comando principal
ENTRYPOINT ["nginx"]
# parametros
CMD ["-g", "daemon off;"]
```

```bash
$ docker image build -t meu-nginx:3.0 .                            
[+] Building 1.0s (10/10) FINISHED                                                                                      docker:default
 => [internal] load build definition from Dockerfile                                                                              0.0s
 => => transferring dockerfile: 577B                                                                                              0.0s
 => [internal] load metadata for docker.io/library/ubuntu:24.04                                                                   0.9s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                     0.0s
 => [internal] load .dockerignore                                                                                                 0.0s
 => => transferring context: 2B                                                                                                   0.0s
 => [1/4] FROM docker.io/library/ubuntu:24.04@sha256:cd1dba651b3080c3686ecf4e3c4220f026b521fb76978881737d24f200828b2b             0.0s
 => [internal] load build context                                                                                                 0.0s
 => => transferring context: 32B                                                                                                  0.0s
 => CACHED [2/4] RUN apt-get update &&  apt-get install -y nginx &&  apt-get clean && rm -rf /var/lib/apt/lists/*                 0.0s
 => CACHED [3/4] COPY index.html /var/www/html/index.html                                                                         0.0s
 => CACHED [4/4] WORKDIR /var/www/html                                                                                            0.0s
 => exporting to image                                                                                                            0.0s
 => => exporting layers                                                                                                           0.0s
 => => writing image sha256:fc118a38f845448b3ff9a2afe718d2e59a2ed5142f8054cc157e662efc550784                                      0.0s
 => => naming to docker.io/library/meu-nginx:3.0                                                                                  0.0s
 $
 ```

 ```bash
 $  docker image ls 
                                                                                       i Info →   U  In Use
IMAGE                           ID             DISK USAGE   CONTENT SIZE   EXTRA
ibmcom/db2:latest               2ec8bf76e622       2.79GB             0B    U   
meu-nginx:1.0                   83a80dbac42e        143MB             0B        
meu-nginx:2.0                   2c8112c3c359       84.6MB             0B    U   
meu-nginx:3.0                   fc118a38f845       84.6MB             0B        
mysql:8                         4f1413420360        545MB             0B    U   
mysql:8.0                       33037edcac9b        444MB             0B    U   
openzipkin/zipkin:latest        ad5bf93e3f50        165MB             0B    U   
pauloneves/opsdeck-api:0.0.1    91c84a05acbc        380MB             0B        
pauloneves/opsdeck-api:0.0.2    b3e2bcc720d9        380MB             0B        
portainer/portainer-ce:latest   1a0fb356ea35        294MB             0B    U   
sonatype/nexus3:latest          8716903d1912        629MB             0B    U   
$
```

## Adicionando HEALTHCHECK ao nosso Dockerfile

**ADD**    - Copia ficheiros ou diretórios para a imagem, podendo também extrair ficheiros de um arquivo .tar e fazer download de URLs remotas.
**COPY**   - Copia ficheiros ou diretórios do contexto de build para a imagem, de forma simples e direta.

**Diferença:**
O comando COPY é recomendado para cópias simples de ficheiros locais, pois é mais previsível e seguro. O comando ADD oferece funcionalidades extra, como descompactar arquivos .tar e aceitar URLs, mas pode introduzir comportamentos inesperados. Por isso, prefira COPY, excepto quando realmente precisa das funcionalidades adicionais do ADD.

**HEALTHCHECK**  - Permite definir um teste de saúde para o container, verificando periodicamente se a aplicação está a funcionar corretamente. Se o teste falhar, o Docker marca o container como 'unhealthy'.

**Exemplo:**
```
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:80 || exit 1
```


```dockerfile
FROM ubuntu:24.04
LABEL maintainer="email@email.com"
LABEL version="1.0"
LABEL description="Imagem com Nginx baseada em Ubuntu"
LABEL org.opencontainers.image.source="https://github.com/paulozagaloneves/PICK-2026"
LABEL org.opencontainers.image.licenses="MIT"
RUN apt-get update && \
	apt-get install -y nginx curl && \
	apt-get clean && rm -rf /var/lib/apt/lists/*
COPY index.html /var/www/html/index.html
# Apenas um exemplo
ADD https://github.com/paulozagaloneves/kvm-compose/releases/download/0.3.7/kvm-compose_0.3.7_linux_amd64.tar.gz /usr/bin/kvm-compose
EXPOSE 80
WORKDIR /var/www/html
ENV APP_VERSION=1.0.0
# comando principal
ENTRYPOINT ["nginx"]
# parametros
CMD ["-g", "daemon off;"]
HEALTHCHECK --timeout=2s CMD curl -f localhost || exit 1
```

**build**

```bash
$  docker image build -t meu-nginx:4.0 .
[+] Building 9.8s (12/12) FINISHED                                                                                                                  docker:default
 => [internal] load build definition from Dockerfile                                                                                                          0.0s
 => => transferring dockerfile: 787B                                                                                                                          0.0s
 => [internal] load metadata for docker.io/library/ubuntu:24.04                                                                                               0.9s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                                                 0.0s
 => [internal] load .dockerignore                                                                                                                             0.0s
 => => transferring context: 2B                                                                                                                               0.0s
 => CACHED [1/5] FROM docker.io/library/ubuntu:24.04@sha256:cd1dba651b3080c3686ecf4e3c4220f026b521fb76978881737d24f200828b2b                                  0.0s
 => [4/5] ADD https://github.com/paulozagaloneves/kvm-compose/releases/download/0.3.7/kvm-compose_0.3.7_linux_amd64.tar.gz /usr/bin/kvm-compose               0.5s
 => [internal] load build context                                                                                                                             0.0s
 => => transferring context: 32B                                                                                                                              0.0s
 => [2/5] RUN apt-get update &&  apt-get install -y nginx curl &&  apt-get clean && rm -rf /var/lib/apt/lists/*                                               8.6s
 => [3/5] COPY index.html /var/www/html/index.html                                                                                                            0.0s
 => [4/5] ADD https://github.com/paulozagaloneves/kvm-compose/releases/download/0.3.7/kvm-compose_0.3.7_linux_amd64.tar.gz /usr/bin/kvm-compose               0.0s
 => [5/5] WORKDIR /var/www/html                                                                                                                               0.0s 
 => exporting to image                                                                                                                                        0.1s 
 => => exporting layers                                                                                                                                       0.1s 
 => => writing image sha256:9d3eafcec4f6a94cc5512b7a388777db683753ca885baf27dc726e354ba34531                                                                  0.0s 
 => => naming to docker.io/library/meu-nginx:4.0                                                                                                              0.0s
 $
 ```

**executando**

```bash
$  docker container run -d -p 8888:80 --name meu-nginx-4 meu-nginx:4.0
75fa23c8e7e5e47424b04c942b5cd06bf9b5f674811ea872f32de94796a69ae3
$
$ docker container ls
docker container ls
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS                             PORTS                                                      NAMES
75fa23c8e7e5   meu-nginx:4.0                   "nginx -g 'daemon of…"   27 seconds ago   Up 26 seconds (health: starting)   0.0.0.0:8888->80/tcp, [::]:8888->80/tcp                    meu-nginx-4
d74cb20d0e92   meu-nginx:2.0                   "nginx -g 'daemon of…"   33 minutes ago   Up 33 minutes                      0.0.0.0:8080->80/tcp, [::]:8080->80/tcp                    meu-nginx
$
$ docker container ls
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS                          PORTS                                                      NAMES
fbf4dde0c42c   meu-nginx:4.0                   "nginx -g 'daemon of…"   32 seconds ago   Up 31 seconds (healthy)         0.0.0.0:8888->80/tcp, [::]:8888->80/tcp                    meu-nginx-4
d74cb20d0e92   meu-nginx:2.0                   "nginx -g 'daemon of…"   37 minutes ago   Up 37 minutes                   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp                    meu-nginx
$
$ docker container exec -ti meu-nginx-4 bash
root@fbf4dde0c42c:/var/www/html# ls -l /usr/bin/kvm-compose 
-rw------- 1 root root 1847876 Feb  5 17:10 /usr/bin/kvm-compose
root@fbf4dde0c42c:/var/www/html# 
```


**Versão Optimizada**

```dockerfile
FROM ubuntu:24.04
LABEL maintainer="email@email.com"
LABEL version="1.0"
LABEL description="Imagem com Nginx baseada em Ubuntu"
LABEL org.opencontainers.image.source="https://github.com/paulozagaloneves/PICK-2026"
LABEL org.opencontainers.image.licenses="MIT"
RUN apt-get update && \
	apt-get install -y nginx curl && \
	apt-get clean && rm -rf /var/lib/apt/lists/*
COPY index.html /var/www/html/index.html
# Apenas um exemplo, faz download mas nao extrai o arquivo tar.gz. Apenas extrai se o arquivo for local
# ADD https://github.com/paulozagaloneves/kvm-compose/releases/download/0.3.7/kvm-compose_0.3.7_linux_amd64.tar.gz /usr/bin/kvm-compose
# Para download e extrair o arquivo tar.gz, é necessário usar o comando RUN para executar os comandos de download e extracao
RUN curl -L -o /tmp/kvm-compose.tar.gz https://github.com/paulozagaloneves/kvm-compose/releases/download/0.3.7/kvm-compose_0.3.7_linux_amd64.tar.gz \
 && tar -xzvf /tmp/kvm-compose.tar.gz -C /usr/bin \
 && chmod +x /usr/bin/kvm-compose \
 && rm /tmp/kvm-compose.tar.gz
EXPOSE 80
WORKDIR /var/www/html
ENV APP_VERSION=1.0.0
# comando principal
ENTRYPOINT ["nginx"]
# parametros
CMD ["-g", "daemon off;"]
HEALTHCHECK --timeout=2s CMD curl -f localhost || exit 1
```

**build**

```bash
$  docker image build -t meu-nginx:5.0 .     
[+] Building 1.5s (11/11) FINISHED                                                                                                                                                                   docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                                           0.0s
 => => transferring dockerfile: 1.27kB                                                                                                                                                                         0.0s
 => [internal] load metadata for docker.io/library/ubuntu:24.04                                                                                                                                                0.8s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                                                                                                  0.0s
 => [internal] load .dockerignore                                                                                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                                                                                0.0s
 => [1/5] FROM docker.io/library/ubuntu:24.04@sha256:cd1dba651b3080c3686ecf4e3c4220f026b521fb76978881737d24f200828b2b                                                                                          0.0s
 => [internal] load build context                                                                                                                                                                              0.0s
 => => transferring context: 32B                                                                                                                                                                               0.0s
 => CACHED [2/5] RUN apt-get update &&  apt-get install -y nginx curl &&  apt-get clean && rm -rf /var/lib/apt/lists/*                                                                                         0.0s
 => CACHED [3/5] COPY index.html /var/www/html/index.html                                                                                                                                                      0.0s
 => [4/5] RUN curl -L -o /tmp/kvm-compose.tar.gz https://github.com/paulozagaloneves/kvm-compose/releases/download/0.3.7/kvm-compose_0.3.7_linux_amd64.tar.gz  && tar -xzvf /tmp/kvm-compose.tar.gz -C /usr/b  0.6s
 => [5/5] WORKDIR /var/www/html                                                                                                                                                                                0.0s
 => exporting to image                                                                                                                                                                                         0.0s
 => => exporting layers                                                                                                                                                                                        0.0s
 => => writing image sha256:f6e0087e5a988f22eb39582be125dc55d25e9970d2b5091f2cdc63935d781e70                                                                                                                   0.0s
 => => naming to docker.io/library/meu-nginx:5.0                                                                                                                                                               0.0s
$ 
```

**testando**
```bash
$ docker container run -d -p 8889:80 --name meu-nginx-5 meu-nginx:5.0 
9f046d62f709a14266abad3105fa3b18647828f64c0d6f11ad699ddc7546f113
$
$ docker container exec -ti meu-nginx-5 bash                         
root@9f046d62f709:/var/www/html# ls -l /usr/bin/kvm-compose
-rwxr-xr-x 1 1001 1001 4514084 Feb  5 17:10 /usr/bin/kvm-compose
root@9f046d62f709:/var/www/html# 
$
```

## Descomplicando o meu Dockerfile


```dockerfile
FROM ubuntu:24.04
LABEL maintainer="email@email.com"
LABEL version="1.0"
LABEL description="Imagem com Nginx baseada em Ubuntu"
LABEL org.opencontainers.image.source="https://github.com/paulozagaloneves/PICK-2026"
LABEL org.opencontainers.image.licenses="MIT"
RUN apt-get update && \
	apt-get install -y nginx curl && \
	apt-get clean && rm -rf /var/lib/apt/lists/*
COPY index.html /var/www/html/index.html
# Apenas um exemplo, faz download mas nao extrai o arquivo tar.gz. Apenas extrai se o arquivo for local
# ADD https://github.com/paulozagaloneves/kvm-compose/releases/download/0.3.7/kvm-compose_0.3.7_linux_amd64.tar.gz /usr/bin/kvm-compose
# Para download e extrair o arquivo tar.gz, é necessário usar o comando RUN para executar os comandos de download e extracao
RUN curl -L -o /tmp/kvm-compose.tar.gz https://github.com/paulozagaloneves/kvm-compose/releases/download/0.3.7/kvm-compose_0.3.7_linux_amd64.tar.gz \
 && tar -xzvf /tmp/kvm-compose.tar.gz -C /usr/bin \
 && chmod +x /usr/bin/kvm-compose \
 && rm /tmp/kvm-compose.tar.gz
EXPOSE 80
WORKDIR /var/www/html
ENV APP_VERSION=1.0.0
# comando principal
ENTRYPOINT ["nginx"]
# parametros
CMD ["-g", "daemon off;"]
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:80 || exit 1
```

**FROM**ubuntu:24.04: Usa a versão 24.04 do Ubuntu como base.
**LABEL**: Adiciona metadados úteis como maintainer, versão, descrição, origem e licença.
**RUN** apt-get update && apt-get install -y nginx curl ...: Instala o Nginx e o curl, limpa a cache do apt para reduzir o tamanho da imagem.
**COPY** index.html ...: Copia um ficheiro local para o diretório web do Nginx.
**ADD** Copia ficheiros, diretórios, ficheiros TAR ou ficheiros remotos e os adiciona ao filesystem do container.
**RUN** curl ... && tar ... && chmod ... && rm ...: Faz download de um ficheiro .tar.gz, extrai para /usr/bin, torna executável e remove o ficheiro temporário. Esta abordagem é eficiente e reduz camadas.
**EXPOSE** 80: Documenta que a aplicação escuta na porta 80.
**WORKDIR**/var/www/html: Define o diretório de trabalho padrão.
**ENV** APP_VERSION=1.0.0: Define uma variável de ambiente.
**ENTRYPOINT** ["nginx"]: Define o processo principal do container.
**CMD** ["-g", "daemon off;"]: Parâmetros para o Nginx rodar em foreground.
**HEALTHCHECK**: Adiciona um teste de saúde simples usando curl.
**USER** Determina qual utilizador será utilizado na imagem. Por default é o root.


## Desafio prático

Criar uma imagem e container com imagem base debian e executabdo NGINX.


**Dockerfile-desafio**

```dockerfile
FROM debian:13
LABEL maintainer="email@email.com"
LABEL version="1.0"
LABEL description="Imagem com Nginx baseada em debian"
LABEL org.opencontainers.image.source="https://github.com/paulozagaloneves/PICK-2026"
LABEL org.opencontainers.image.licenses="MIT"
RUN apt-get update && \
	apt-get install -y nginx curl && \
	apt-get clean && rm -rf /var/lib/apt/lists/*
EXPOSE 80
ENTRYPOINT ["nginx"]
# parametros
CMD ["-g", "daemon off;"]
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:80 || exit 1
```


**build**

```bash
$ $ docker image build -f Dockerfile-desafio -t debian-nginx:1.0 .
[+] Building 10.6s (7/7) FINISHED                                                                                       docker:default
 => [internal] load build definition from Dockerfile-desafio                                                                      0.0s
 => => transferring dockerfile: 595B                                                                                              0.0s
 => [internal] load metadata for docker.io/library/debian:13                                                                      1.3s
 => [auth] library/debian:pull token for registry-1.docker.io                                                                     0.0s
 => [internal] load .dockerignore                                                                                                 0.0s
 => => transferring context: 2B                                                                                                   0.0s
 => [1/2] FROM docker.io/library/debian:13@sha256:2c91e484d93f0830a7e05a2b9d92a7b102be7cab562198b984a84fdbc7806d91                2.0s
 => => resolve docker.io/library/debian:13@sha256:2c91e484d93f0830a7e05a2b9d92a7b102be7cab562198b984a84fdbc7806d91                0.0s
 => => sha256:404d07a1110c5965e2a8c39c66a2985de4d977c8bf66033309c13007322dcf3e 451B / 451B                                        0.0s
 => => sha256:ef235bf1a09a237b896b69935c8c8d917c9c6a78b538724911414afc0a96763c 49.29MB / 49.29MB                                  1.3s
 => => sha256:2c91e484d93f0830a7e05a2b9d92a7b102be7cab562198b984a84fdbc7806d91 8.93kB / 8.93kB                                    0.0s
 => => sha256:aca3e110b8fee2a2acdf8cbe6cef1cfebef52e5257da87fbd41920d3411c1aed 1.02kB / 1.02kB                                    0.0s
 => => extracting sha256:ef235bf1a09a237b896b69935c8c8d917c9c6a78b538724911414afc0a96763c                                         0.6s
 => [2/2] RUN apt-get update &&  apt-get install -y nginx curl &&  apt-get clean && rm -rf /var/lib/apt/lists/*                   7.1s
 => exporting to image                                                                                                            0.2s
 => => exporting layers                                                                                                           0.1s
 => => writing image sha256:820f6b8fb803f4536afc5f3773b6565ebcf9d0f2f415f309ec190446437e060f                                      0.0s 
 => => naming to docker.io/library/debian-nginx:1.0                                                                               0.0s 
 $
```

**Executar e validar**

```bash
$ docker container run -d -p 9999:80 --name debian-nginx debian-nginx:1.0 
132dfe28c904159dd3236c666d63201791c9ce8118df87c25136db0997e3c7c5

$ curl localhost:9999                                                    
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
$
$
$ docker container ls                                                    
CONTAINER ID   IMAGE                           COMMAND                  CREATED             STATUS                         PORTS                                                      NAMES
132dfe28c904   debian-nginx:1.0                "nginx -g 'daemon of…"   37 seconds ago      Up 36 seconds (healthy)        0.0.0.0:9999->80/tcp, [::]:9999->80/tcp                    debian-nginx
e7a731fa5aa1   meu-nginx:5.0                   "nginx -g 'daemon of…"   13 minutes ago      Up 13 minutes (healthy)        0.0.0.0:8889->80/tcp, [::]:8889->80/tcp                    meu-nginx-5
fbf4dde0c42c   meu-nginx:4.0                   "nginx -g 'daemon of…"   40 minutes ago      Up 40 minutes (healthy)        0.0.0.0:8888->80/tcp, [::]:8888->80/tcp                    meu-nginx-4
d74cb20d0e92   meu-nginx:2.0                   "nginx -g 'daemon of…"   About an hour ago   Up About an hour               0.0.0.0:8080->80/tcp, [::]:8080->80/tcp                    meu-nginx
$
```