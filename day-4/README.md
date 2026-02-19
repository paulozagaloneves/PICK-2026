# Day 4 - Volumes

## Índice

- [Day 4 - Volumes](#day-4---volumes)
  - [Índice](#índice)
  - [O que são volumes](#o-que-são-volumes)
  - [Tipos de volumes](#tipos-de-volumes)
  - [Particularidades entre volumes e Containers](#particularidades-entre-volumes-e-containers)
  - [Volume do tipo Bind](#volume-do-tipo-bind)
  - [Listar volumes](#listar-volumes)
  - [Volume do tipo Volume](#volume-do-tipo-volume)

## O que são volumes

Volumes no Docker são mecanismos de armazenamento persistente usados para guardar dados gerados e utilizados por containers. Permitem que informações não se percam quando um container é removido ou recriado, pois os dados ficam fora do sistema de ficheiros interno do container, sendo mantidos numa área gerida pelo Docker no anfitrião.

Principais pontos sobre volumes:

Permitem a persistência de dados entre execuções de containeres.

Facilitam a partilha de dados entre múltiplos containeres.

São a forma recomendada de armazenar dados no Docker, em vez de usar o sistema de ficheiros do próprio container.

Podem ser criados e geridos com comandos como docker volume create, docker volume ls, docker volume rm, etc.

São montados nos containeres usando a opção -v ou --mount.

Exemplo de utilização:
```bash
$ docker run -v meu_volume:/app/dados imagem
```

Assim, tudo o que for guardado em /app/dados dentro do container ficará persistente no volume chamado meu_volume, mesmo que o container seja removido.

## Tipos de volumes

Existem três tipos principais de volumes no Docker:

**1. Volumes** (geridos pelo Docker)
 - Criados e geridos pelo Docker.
 - Armazenados numa área específica do anfitrião (geralmente /var/lib/docker/volumes).
 - São a forma mais recomendada para persistência de dados.

**2. Bind** mounts (montagem de diretórios do anfitrião)
 - Mapeiam um diretório ou ficheiro específico do anfitrião para dentro do container.
 - Permitem maior controlo sobre o local de armazenamento, mas dependem da estrutura do anfitrião.

**3. tmpfs** mounts
 - Armazenam dados apenas na memória (RAM).
 - Os dados são voláteis e desaparecem quando o container é parado ou removido.
 - Úteis para dados temporários e sensíveis.

Resumo:

- **Volumes**: persistentes, geridos pelo Docker.
- **Bind mounts**: persistentes, controlados pelo utilizador, dependem do anfitrião.
- **tmpfs**: voláteis, armazenados em memória.

## Particularidades entre volumes e Containers

- O volume é inicializado quando o container é criado.
- Caso ocorra de já haver dados no diretório em que você está montando como volume, ou seja, se o diretório já existe e está "populado" na imagem base, aqueles dados serão copiados para o volume.
- Um volume pode ser reusado e compartilhado entre containers.
- Alterações em um volume são feitas diretamente no volume.
- Alterações em um volume não irão com a imagem quando você fizer uma cópia ou snapshot de um container.
- Volumes continuam a existir mesmo se você deletar o container

## Volume do tipo Bind

```bash
$ docker container run -ti --name testando-volumes --mount type=bind,source=/workspace/linuxtips/PICK-2026/giropops-senhas/,target=/giropops-senhas debian 
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
ef235bf1a09a: Already exists 
Digest: sha256:2c91e484d93f0830a7e05a2b9d92a7b102be7cab562198b984a84fdbc7806d91
Status: Downloaded newer image for debian:latest
root@3e3234a8da25:/# ls giropops-senhas/
Dockerfile  Dockerfile.extra  Dockerfile.wolfi  LICENSE  __pycache__  app.py  cosign.key  cosign.pub  requirements.txt  scans  static  tailwind.config.js  templates
root@3e3234a8da25:/# touch FUNCIONA
$ 
```

**montar volume como RO**

```bash
 docker container run -ti --name testando-volumes --mount type=bind,source=/workspace/linuxtips/PICK-2026/giropops-senhas/,target=/giropops-senhas,ro debian
root@a2104ecda4f4:/# ls
bin  boot  dev  etc  giropops-senhas  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@a2104ecda4f4:/# cd giropops-senhas/
root@a2104ecda4f4:/giropops-senhas# ls
Dockerfile        Dockerfile.wolfi  LICENSE      app.py      cosign.pub        scans   tailwind.config.js
Dockerfile.extra  FUNCIONA          __pycache__  cosign.key  requirements.txt  static  templates
root@a2104ecda4f4:/giropops-senhas# touch AAA
touch: cannot touch 'AAA': Read-only file system
root@a2104ecda4f4:/giropops-senhas# exit
exit
```

## Listar volumes

```bash
$ docker volume ls
DRIVER    VOLUME NAME
local     0efc66f56126bee45bb86ecc5c2be960f4b51f931dc11e0a2b12c9f5cfc66e19
local     1d1d365d2118647dc86d0ea169d5d20651aaf7bb27177c8e077fd123877ca462
local     2a52dc1355e5f8e142c36e99148de8075496ee8a84af669c17b4d7e386f3b546
local     2d716ddc206b1e642210ab97cdf5ce02dc517dc6f71e6f1e7894b31ae6201f8d
local     2f536d4bb4e1a3e6dfc91ce6b123fe92800580ce197bc1c2a9bb72f0bccfb2f7
...
local     nexus-data
local     portainer_data
```

## Volume do tipo Volume

```bash
$ docker volume create giropops
giropops
$
```

**inspect**

```bash
$ docker volume inspect giropops 
[
    {
        "CreatedAt": "2026-02-13T19:34:13Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/giropops/_data",
        "Name": "giropops",
        "Options": null,
        "Scope": "local"
    }
]
```

**Adicionar ficheiros num volume local**

```bash
$ sudo touch /var/lib/docker/volumes/giropops/_data/FUNCIONA
```

```bash
$ docker container run -ti --name testando-volumes --mount type=volume,source=giropops,target=/giropops-senhas debian
root@452dea0db556:/# ls
bin  boot  dev  etc  giropops-senhas  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@452dea0db556:/# cd giropops-senhas/
root@452dea0db556:/giropops-senhas# ls
FUNCIONA
root@452dea0db556:/giropops-senhas#
```