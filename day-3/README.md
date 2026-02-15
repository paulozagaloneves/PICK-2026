# Day 3 - Otimizando Imagens

## Índice

- [Day 3 - Otimizando Imagens](#day-3---otimizando-imagens)
  - [Índice](#índice)
  - [Giropops Senhas](#giropops-senhas)
    - [Clone do projeto](#clone-do-projeto)
    - [Instalar o pip](#instalar-o-pip)
    - [Criar um ambiente virtual do python3](#criar-um-ambiente-virtual-do-python3)
    - [Instalar as dependências da nossa aplicação](#instalar-as-dependências-da-nossa-aplicação)
    - [Executar a aplicação Giropops Senhas](#executar-a-aplicação-giropops-senhas)
    - [Instalar o REDIS](#instalar-o-redis)
    - [Reexecutar a aplicação](#reexecutar-a-aplicação)
  - [Containerizando a aplicação Giropops Senhas](#containerizando-a-aplicação-giropops-senhas)
    - [Build](#build)
    - [Run](#run)
  - [Optimizando Imagem](#optimizando-imagem)
    - [2ª versão (versão slim)](#2ª-versão-versão-slim)
    - [3ª versão (alpine)](#3ª-versão-alpine)
  - [Optimizando mais ainda a nossa imagem com Distroless](#optimizando-mais-ainda-a-nossa-imagem-com-distroless)
    - [Chainguard Wolfi](#chainguard-wolfi)
    - [Distroless Google](#distroless-google)
    - [Comparativo](#comparativo)
  - [Verificar vulnerabilidade com Trivy](#verificar-vulnerabilidade-com-trivy)
  - [Assinando imagens com cosign](#assinando-imagens-com-cosign)
    - [Install cosign](#install-cosign)
    - [Gerar par de chaves](#gerar-par-de-chaves)
    - [Assinando imagens de containers](#assinando-imagens-de-containers)
    - [Verificando e validando assinatura](#verificando-e-validando-assinatura)

## Giropops Senhas

### Clone do projeto

Efetuar clone do projeto Giropops Senhas do repositório do @badtuxx

```bash
$ git clone git@github.com:badtuxx/giropops-senhas.git
```

### Instalar o pip

Se ainda não tiver instalado, instalar o pip

```bash
$ sudo apt install pip -y
...
# validar versao
$ $ pip --version
pip 25.1.1 from /usr/lib/python3/dist-packages/pip (python 3.13)
$
```

### Criar um ambiente virtual do python3

Se ao executar o **pip install** obter o erro abaixo será necessário criar um virtual environment do python3.

```bash
$ pip install -r requirements.txt
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.
    
    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
    sure you have python3-full installed.
    
    If you wish to install a non-Debian packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.
    
    See /usr/share/doc/python3.13/README.venv for more information.

note: If you believe this is a mistake, please contact your Python installation or OS distribution provider. You can override this, at the risk of breaking your Python installation or OS, by passing --break-system-packages.
hint: See PEP 668 for the detailed specification.
$
```

Criar o virtual environmento na raiz do seu home.

Nota: Pode definir um diretorio à sua escolha para o venv.

```bash
$ python3 -m venv ~/venv
```

### Instalar as dependências da nossa aplicação

```bash
$ ~/venv/bin/pip install -r requirements.txt   
Collecting Flask==3.0.3 (from -r requirements.txt (line 1))
  Downloading flask-3.0.3-py3-none-any.whl.metadata (3.2 kB)
Collecting redis==4.5.4 (from -r requirements.txt (line 2))
  Downloading redis-4.5.4-py3-none-any.whl.metadata (8.3 kB)
Collecting prometheus-client==0.16.0 (from -r requirements.txt (line 3))
  Downloading prometheus_client-0.16.0-py3-none-any.whl.metadata (22 kB)
Collecting Werkzeug>=3.0.0 (from Flask==3.0.3->-r requirements.txt (line 1))
  Downloading werkzeug-3.1.5-py3-none-any.whl.metadata (4.0 kB)
Collecting Jinja2>=3.1.2 (from Flask==3.0.3->-r requirements.txt (line 1))
  Downloading jinja2-3.1.6-py3-none-any.whl.metadata (2.9 kB)
Collecting itsdangerous>=2.1.2 (from Flask==3.0.3->-r requirements.txt (line 1))
  Downloading itsdangerous-2.2.0-py3-none-any.whl.metadata (1.9 kB)
Collecting click>=8.1.3 (from Flask==3.0.3->-r requirements.txt (line 1))
  Downloading click-8.3.1-py3-none-any.whl.metadata (2.6 kB)
Collecting blinker>=1.6.2 (from Flask==3.0.3->-r requirements.txt (line 1))
  Downloading blinker-1.9.0-py3-none-any.whl.metadata (1.6 kB)
Collecting MarkupSafe>=2.0 (from Jinja2>=3.1.2->Flask==3.0.3->-r requirements.txt (line 1))
  Downloading markupsafe-3.0.3-cp313-cp313-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (2.7 kB)
Downloading flask-3.0.3-py3-none-any.whl (101 kB)
Downloading redis-4.5.4-py3-none-any.whl (238 kB)
Downloading prometheus_client-0.16.0-py3-none-any.whl (122 kB)
Downloading blinker-1.9.0-py3-none-any.whl (8.5 kB)
Downloading click-8.3.1-py3-none-any.whl (108 kB)
Downloading itsdangerous-2.2.0-py3-none-any.whl (16 kB)
Downloading jinja2-3.1.6-py3-none-any.whl (134 kB)
Downloading markupsafe-3.0.3-cp313-cp313-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (22 kB)
Downloading werkzeug-3.1.5-py3-none-any.whl (225 kB)
Installing collected packages: redis, prometheus-client, MarkupSafe, itsdangerous, click, blinker, Werkzeug, Jinja2, Flask
Successfully installed Flask-3.0.3 Jinja2-3.1.6 MarkupSafe-3.0.3 Werkzeug-3.1.5 blinker-1.9.0 click-8.3.1 itsdangerous-2.2.0 prometheus-client-0.16.0 redis-4.5.4
$ 
```

### Executar a aplicação Giropops Senhas

```bash
$ ~/venv/bin/flask run --host=0.0.0.0
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://192.168.1.205:5000
Press CTRL+C to quit
[2026-02-10 18:33:52,331] ERROR in app: Exception on / [GET]
Traceback (most recent call last):
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/redis/connection.py", line 698, in connect
    sock = self.retry.call_with_retry(
        lambda: self._connect(), lambda error: self.disconnect(error)
    )
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/redis/retry.py", line 46, in call_with_retry
    return do()
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/redis/connection.py", line 699, in <lambda>
    lambda: self._connect(), lambda error: self.disconnect(error)
            ~~~~~~~~~~~~~^^
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/redis/connection.py", line 955, in _connect
    for res in socket.getaddrinfo(
               ~~~~~~~~~~~~~~~~~~^
        self.host, self.port, self.socket_type, socket.SOCK_STREAM
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    ):
    ^
  File "/usr/lib/python3.13/socket.py", line 977, in getaddrinfo
    for res in _socket.getaddrinfo(host, port, family, type, proto, flags):
               ~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
socket.gaierror: [Errno -2] Name or service not known

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/flask/app.py", line 1473, in wsgi_app
    response = self.full_dispatch_request()
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/flask/app.py", line 882, in full_dispatch_request
    rv = self.handle_user_exception(e)
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/flask/app.py", line 880, in full_dispatch_request
    rv = self.dispatch_request()
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/flask/app.py", line 865, in dispatch_request
    return self.ensure_sync(self.view_functions[rule.endpoint])(**view_args)  # type: ignore[no-any-return]
           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^
  File "/workspace/linuxtips/PICK-2026/giropops-senhas/app.py", line 43, in index
    senhas = r.lrange("senhas", 0, 9)
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/redis/commands/core.py", line 2715, in lrange
    return self.execute_command("LRANGE", name, start, end)
           ~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/redis/client.py", line 1255, in execute_command
    conn = self.connection or pool.get_connection(command_name, **options)
                              ~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/redis/connection.py", line 1442, in get_connection
    connection.connect()
    ~~~~~~~~~~~~~~~~~~^^
  File "/home/paulo/workspace/linuxtips/PICK-2026/venv/lib/python3.13/site-packages/redis/connection.py", line 704, in connect
    raise ConnectionError(self._error_message(e))
redis.exceptions.ConnectionError: Error -2 connecting to redis-service:6379. Name or service not known.
192.168.1.205 - - [10/Feb/2026 18:33:52] "GET / HTTP/1.1" 500 -
192.168.1.205 - - [10/Feb/2026 18:33:52] "GET /favicon.ico HTTP/1.1" 404 -

```

### Instalar o REDIS

Vamos instalar um container docker com o REDIS

```bash
$ docker container run --name redis-giropops -p 6379:6379 -d redis:8.4
```

### Reexecutar a aplicação

```bash
$ export REDIS_HOST=127.0.0.1
$ ~/venv/bin/flask run --host=0.0.0.0 
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://192.168.1.205:5000
Press CTRL+C to quit
192.168.1.205 - - [10/Feb/2026 18:47:12] "GET / HTTP/1.1" 200 -
192.168.1.205 - - [10/Feb/2026 18:47:12] "GET /static/css/output.css HTTP/1.1" 200 -
192.168.1.205 - - [10/Feb/2026 18:47:12] "GET /static/linuxtips-logo.png HTTP/1.1" 200 -
192.168.1.205 - - [10/Feb/2026 18:47:12] "GET /static/js/main.js HTTP/1.1" 200 -
$ 
```

## Containerizando a aplicação Giropops Senhas

### Build

```Dockerfile
FROM python:3.14-trixie
WORKDIR /app
COPY requirements.txt .
COPY app.py .
COPY templates templates/
COPY static static/
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 5000

CMD ["flask", "run", "--host=0.0.0.0"]
```

**Build e validando a imagem**

```bash
$ docker image build -t giropops-senhas:1.0 .
$ 
$  docker image ls                         
                                                                         i Info →   U  In Use
IMAGE                                       ID             DISK USAGE   CONTENT SIZE   EXTRA
debian-nginx:1.0                            820f6b8fb803        150MB             0B    U   
giropops-senhas:1.0                         3e9d35ccab40       1.13GB             0B    U   
hello:1.0                                   fb05d4f6f072        880MB             0B    U  
```

### Run

**Criando um container redis**

```bash
$ docker container run -d --name redis -p 6379:6379 redis:8.4
```

**Iniciando a aplicação**

-e REDIS_HOST=192.168.1.205   => cria uma variável de ambiente REDIS_HOST no container

```bash
$ docker container run -d --name giropops-senhas -p 5000:5000 -e REDIS_HOST=192.168.1.205 giropops-senhas:1.0
```

**Verificando Vulnerabilidades**

```bash
$ docker scout quickview giropops-senhas:1.0    
    ✓ Image stored for indexing
    ✓ Indexed 619 packages

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

 Target             │  giropops-senhas:1.0  │    0C     1H     7M   148L     2?  
   digest           │  3e9d35ccab40         │                                    
 Base image         │  python:3.14          │    0C     1H     7M   148L     2?  
 Updated base image │  python:alpine        │    0C     0H     1M     1L         
                    │                       │           -1     -6   -147     -2  

What's next:
    View vulnerabilities → docker scout cves giropops-senhas:1.0
    View base image update recommendations → docker scout recommendations giropops-senhas:1.0
    Include policy results in your quickview by supplying an organization → docker scout quickview giropops-senhas:1.0 --org <organization>
```

**Camadas**

```bash
$ docker history giropops-senhas:1.0        
IMAGE          CREATED             CREATED BY                                      SIZE      COMMENT
3e9d35ccab40   About an hour ago   CMD ["flask" "run" "--host=0.0.0.0"]            0B        buildkit.dockerfile.v0
<missing>      About an hour ago   EXPOSE [5000/tcp]                               0B        buildkit.dockerfile.v0
<missing>      About an hour ago   RUN /bin/sh -c pip install --no-cache-dir -r…   16.5MB    buildkit.dockerfile.v0
<missing>      About an hour ago   COPY static static/ # buildkit                  101kB     buildkit.dockerfile.v0
<missing>      About an hour ago   COPY templates templates/ # buildkit            5.78kB    buildkit.dockerfile.v0
<missing>      About an hour ago   COPY app.py . # buildkit                        2.52kB    buildkit.dockerfile.v0
<missing>      About an hour ago   COPY requirements.txt . # buildkit              51B       buildkit.dockerfile.v0
<missing>      About an hour ago   WORKDIR /app                                    0B        buildkit.dockerfile.v0
<missing>      6 days ago          CMD ["python3"]                                 0B        buildkit.dockerfile.v0
<missing>      6 days ago          RUN /bin/sh -c set -eux;  for src in idle3 p…   36B       buildkit.dockerfile.v0
<missing>      6 days ago          RUN /bin/sh -c set -eux;   savedAptMark="$(a…   77.5MB    buildkit.dockerfile.v0
<missing>      6 days ago          ENV PYTHON_SHA256=a97d5549e9ad81fe17159ed02c…   0B        buildkit.dockerfile.v0
<missing>      6 days ago          ENV PYTHON_VERSION=3.14.3                       0B        buildkit.dockerfile.v0
<missing>      6 days ago          RUN /bin/sh -c set -eux;  apt-get update;  a…   17.9MB    buildkit.dockerfile.v0
<missing>      6 days ago          ENV PATH=/usr/local/bin:/usr/local/sbin:/usr…   0B        buildkit.dockerfile.v0
<missing>      7 days ago          RUN /bin/sh -c set -ex;  apt-get update;  ap…   656MB     buildkit.dockerfile.v0
<missing>      7 days ago          RUN /bin/sh -c set -eux;  apt-get update;  a…   185MB     buildkit.dockerfile.v0
<missing>      7 days ago          RUN /bin/sh -c set -eux;  apt-get update;  a…   60.2MB    buildkit.dockerfile.v0
<missing>      8 days ago          # debian.sh --arch 'amd64' out/ 'trixie' '@1…   120MB     debuerreotype 0.17
```

## Optimizando Imagem

### 2ª versão (versão slim)

```Dockerfile
FROM python:3.14-slim-trixie
WORKDIR /app
COPY requirements.txt .
COPY app.py .
COPY templates templates/
COPY static static/
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 5000

CMD ["flask", "run", "--host=0.0.0.0"]
```

```bash
$ docker image build -t giropops-senhas:2.0 . 
```

```bash
$ docker image ls                            
                                                                                                                                                                                                                                           i Info →   U  In Use
IMAGE                                       ID             DISK USAGE   CONTENT SIZE   EXTRA
debian-nginx:1.0                            820f6b8fb803        150MB             0B    U   
giropops-senhas:1.0                         3e9d35ccab40       1.13GB             0B    U   
giropops-senhas:2.0                         4e49dc4f2552        136MB             0B     
$
```

**Conclusão:** Nesta primeira versão de otimização já reduziu bastante. Reduziu cerca de 90%

**Verificando Vulnerabilidades**

```bash
$ docker scout quickview giropops-senhas:2.0
    ✓ Image stored for indexing
    ✓ Indexed 133 packages

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

 Target             │  giropops-senhas:2.0  │    0C     0H     1M    21L  
   digest           │  4e49dc4f2552         │                             
 Base image         │  python:3-slim        │    0C     0H     1M    21L  
 Updated base image │  python:alpine        │    0C     0H     1M     1L  
                    │                       │                        -20  

What's next:
    View vulnerabilities → docker scout cves giropops-senhas:2.0
    View base image update recommendations → docker scout recommendations giropops-senhas:2.0
    Include policy results in your quickview by supplying an organization → docker scout quickview giropops-senhas:2.0 --org <organization>
```

**History**

```bash
$ docker history giropops-senhas:2.0        
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
4e49dc4f2552   23 minutes ago   CMD ["flask" "run" "--host=0.0.0.0"]            0B        buildkit.dockerfile.v0
<missing>      23 minutes ago   EXPOSE [5000/tcp]                               0B        buildkit.dockerfile.v0
<missing>      23 minutes ago   RUN /bin/sh -c pip install --no-cache-dir -r…   16.5MB    buildkit.dockerfile.v0
<missing>      23 minutes ago   COPY static static/ # buildkit                  101kB     buildkit.dockerfile.v0
<missing>      23 minutes ago   COPY templates templates/ # buildkit            5.78kB    buildkit.dockerfile.v0
<missing>      23 minutes ago   COPY app.py . # buildkit                        2.52kB    buildkit.dockerfile.v0
<missing>      23 minutes ago   COPY requirements.txt . # buildkit              51B       buildkit.dockerfile.v0
<missing>      23 minutes ago   WORKDIR /app                                    0B        buildkit.dockerfile.v0
<missing>      6 days ago       CMD ["python3"]                                 0B        buildkit.dockerfile.v0
<missing>      6 days ago       RUN /bin/sh -c set -eux;  for src in idle3 p…   36B       buildkit.dockerfile.v0
<missing>      6 days ago       RUN /bin/sh -c set -eux;   savedAptMark="$(a…   36.6MB    buildkit.dockerfile.v0
<missing>      6 days ago       ENV PYTHON_SHA256=a97d5549e9ad81fe17159ed02c…   0B        buildkit.dockerfile.v0
<missing>      6 days ago       ENV PYTHON_VERSION=3.14.3                       0B        buildkit.dockerfile.v0
<missing>      6 days ago       RUN /bin/sh -c set -eux;  apt-get update;  a…   3.81MB    buildkit.dockerfile.v0
<missing>      6 days ago       ENV PATH=/usr/local/bin:/usr/local/sbin:/usr…   0B        buildkit.dockerfile.v0
<missing>      8 days ago       # debian.sh --arch 'amd64' out/ 'trixie' '@1…   78.6MB    debuerreotype 0.17
```

### 3ª versão (alpine)

```dockerfile
FROM python:3.14-alpine
WORKDIR /app
COPY requirements.txt .
COPY app.py .
COPY templates templates/
COPY static static/
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 5000
#ENV REDIS_HOST=192.168.1.205
CMD ["flask", "run", "--host=0.0.0.0"]
```

```bash
$ docker image build -t giropops-senhas:3.0 . 
```

```bash
$ docker image ls 
                                                                                                                                                                                                                                           i Info →   U  In Use
IMAGE                                       ID             DISK USAGE   CONTENT SIZE   EXTRA
debian-nginx:1.0                            820f6b8fb803        150MB             0B    U   
giropops-senhas:1.0                         3e9d35ccab40       1.13GB             0B    U   
giropops-senhas:2.0                         4e49dc4f2552        136MB             0B        
giropops-senhas:3.0                         4792436f6d73       64.1MB             0B
```

**Conclusão:** Nesta versão de otimização reduziu mais ainda

**Verificando Vulnerabilidades**

```bash
$ docker scout quickview giropops-senhas:3.0 
    ✓ Image stored for indexing
    ✓ Indexed 50 packages

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

 Target     │  giropops-senhas:3.0  │    0C     0H     1M     1L  
   digest   │  4792436f6d73         │                             
 Base image │  python:3-alpine      │    0C     0H     1M     1L  

What's next:
    View vulnerabilities → docker scout cves giropops-senhas:3.0
    Include policy results in your quickview by supplying an organization → docker scout quickview giropops-senhas:3.0 --org <organization>

```

## Optimizando mais ainda a nossa imagem com Distroless

### Chainguard Wolfi

```dockerfile
FROM cgr.dev/chainguard/python:latest-dev AS builder
WORKDIR /app

RUN python -m venv /app/venv
COPY requirements.txt .
RUN /app/venv/bin/pip install --no-cache-dir -r requirements.txt

FROM cgr.dev/chainguard/python:latest
WORKDIR /app
COPY --from=builder /app/venv /app/venv
COPY app.py .
COPY templates templates/
COPY static static/

ENV PATH="/app/venv/bin:$PATH"
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
EXPOSE 5000

ENTRYPOINT ["python", "-m", "flask", "run"]
```

```bash
$ docker image build -t giropops-senhas:4.0 . 
```

```bash
$ docker image ls                            
                                                                         i Info →   U  In Use
IMAGE                                       ID             DISK USAGE   CONTENT SIZE   EXTRA
debian-nginx:1.0                            820f6b8fb803        150MB             0B    U   
giropops-senhas:1.0                         3e9d35ccab40       1.13GB             0B        
giropops-senhas:2.0                         4e49dc4f2552        136MB             0B        
giropops-senhas:3.0                         4792436f6d73       64.1MB             0B    U   
giropops-senhas:4.0                         c96b83ea7bcb         83MB             0B
```

**Conclusão:** Nesta versão Wolfi aumentou um pouco comparado com a imagem alpine. Porem como podem ver abaixo a imagem tem 0 vulnerabilidades.

**Verificando Vulnerabilidades**

```bash
$ docker scout quickview giropops-senhas:4.0
    ✓ Image stored for indexing
    ✓ Indexed 63 packages

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

 Target   │  giropops-senhas:4.0  │    0C     0H     0M     0L  
   digest │  c96b83ea7bcb         │                             

What's next:
    Include policy results in your quickview by supplying an organization → docker scout quickview giropops-senhas:4.0 --org <organization>
```

**History**

```bash
$ docker history giropops-senhas:4.0
IMAGE          CREATED             CREATED BY                                      SIZE      COMMENT
fc7abfe606f1   About an hour ago   ENTRYPOINT ["python" "-m" "flask" "run"]        0B        buildkit.dockerfile.v0
<missing>      About an hour ago   EXPOSE [5000/tcp]                               0B        buildkit.dockerfile.v0
<missing>      About an hour ago   ENV FLASK_RUN_HOST=0.0.0.0                      0B        buildkit.dockerfile.v0
<missing>      About an hour ago   ENV FLASK_APP=app.py                            0B        buildkit.dockerfile.v0
<missing>      About an hour ago   ENV PATH=/app/venv/bin:/usr/local/sbin:/usr/…   0B        buildkit.dockerfile.v0
<missing>      About an hour ago   COPY static static/ # buildkit                  101kB     buildkit.dockerfile.v0
<missing>      About an hour ago   COPY templates templates/ # buildkit            5.78kB    buildkit.dockerfile.v0
<missing>      About an hour ago   COPY app.py . # buildkit                        2.52kB    buildkit.dockerfile.v0
<missing>      About an hour ago   COPY /app/venv /app/venv # buildkit             19.1MB    buildkit.dockerfile.v0
<missing>      22 hours ago        WORKDIR /app                                    0B        buildkit.dockerfile.v0
<missing>      2 days ago          apko                                            123kB     python by Chainguard
<missing>      2 days ago          apko                                            2.78MB    python by Chainguard
<missing>      2 days ago          apko                                            1.07MB    python by Chainguard
<missing>      2 days ago          apko                                            1.4MB     python by Chainguard
<missing>      2 days ago          apko                                            1.65MB    python by Chainguard
<missing>      2 days ago          apko                                            1.64MB    python by Chainguard
<missing>      2 days ago          apko                                            1.81MB    python by Chainguard
<missing>      2 days ago          apko                                            3.48MB    python by Chainguard
<missing>      2 days ago          apko                                            6.93MB    python by Chainguard
<missing>      2 days ago          apko                                            7.18MB    python by Chainguard
<missing>      2 days ago          apko                                            35.7MB    python by Chainguard
```

### Distroless Google

```dockerfile
FROM cgr.dev/chainguard/python:latest-dev AS builder
WORKDIR /app

RUN python -m venv /app/venv
COPY requirements.txt .
RUN /app/venv/bin/pip install --no-cache-dir -r requirements.txt

FROM gcr.io/distroless/python3-debian12
WORKDIR /app
COPY --from=builder /app/venv /app/venv
COPY app.py .
COPY templates templates/
COPY static static/

ENV PATH="/app/venv/bin:$PATH"
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
EXPOSE 5000

ENTRYPOINT ["python", "-m", "flask", "run"]
```

```bash
$ docker image build -t giropops-senhas:5.0 . 
```

```bash
$ docker image ls                           
                                                                        i Info →   U  In Use
IMAGE                                       ID             DISK USAGE   CONTENT SIZE   EXTRA
debian-nginx:1.0                            820f6b8fb803        150MB             0B    U   
giropops-senhas:1.0                         3e9d35ccab40       1.13GB             0B        
giropops-senhas:2.0                         4e49dc4f2552        136MB             0B        
giropops-senhas:3.0                         4792436f6d73       64.1MB             0B        
giropops-senhas:4.0                         fc7abfe606f1         83MB             0B        
giropops-senhas:5.0                         13d4f71ec41e       72.2MB             0B   
$
```

**Conclusão:** Nesta versão Wolfi aumentou um pouco comparado com a imagem alpine. Porem como podem ver abaixo a imagem tem 0 vulnerabilidades.

**Verificando Vulnerabilidades**

```bash
$ docker scout quickview giropops-senhas:5.0  
    ✓ Image stored for indexing
    ✓ Indexed 65 packages

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

 Target     │  giropops-senhas:5.0                │    0C     0H     0M     0L  
   digest   │  13d4f71ec41e                       │                             
 Base image │  distroless/static-debian12:latest  │    0C     0H     0M     0L  

What's next:
    Include policy results in your quickview by supplying an organization → docker scout quickview giropops-senhas:5.0 --org <organization>
```

**History**

```bash
$ docker history giropops-senhas:5.0
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
13d4f71ec41e   7 minutes ago   ENTRYPOINT ["python" "-m" "flask" "run"]        0B        buildkit.dockerfile.v0
<missing>      7 minutes ago   EXPOSE [5000/tcp]                               0B        buildkit.dockerfile.v0
<missing>      7 minutes ago   ENV FLASK_RUN_HOST=0.0.0.0                      0B        buildkit.dockerfile.v0
<missing>      7 minutes ago   ENV FLASK_APP=app.py                            0B        buildkit.dockerfile.v0
<missing>      7 minutes ago   ENV PATH=/app/venv/bin:/usr/local/sbin:/usr/…   0B        buildkit.dockerfile.v0
<missing>      7 minutes ago   COPY static static/ # buildkit                  101kB     buildkit.dockerfile.v0
<missing>      7 minutes ago   COPY templates templates/ # buildkit            5.78kB    buildkit.dockerfile.v0
<missing>      7 minutes ago   COPY app.py . # buildkit                        2.52kB    buildkit.dockerfile.v0
<missing>      7 minutes ago   COPY /app/venv /app/venv # buildkit             19.1MB    buildkit.dockerfile.v0
<missing>      23 hours ago    WORKDIR /app                                    0B        buildkit.dockerfile.v0
<missing>      N/A                                                             38B       
<missing>      N/A                                                             6.9MB     
<missing>      N/A                                                             8.34MB    
<missing>      N/A                                                             5.17MB    
<missing>      N/A                                                             55.7kB    
<missing>      N/A                                                             209kB     
<missing>      N/A                                                             445kB     
<missing>      N/A                                                             113kB     
<missing>      N/A                                                             120kB     
<missing>      N/A                                                             1.05MB    
<missing>      N/A                                                             26.1kB    
<missing>      N/A                                                             251kB     
<missing>      N/A                                                             407kB     
<missing>      N/A                                                             226kB     
<missing>      N/A                                                             43.5kB    
<missing>      N/A                                                             155kB     
<missing>      N/A                                                             616kB     
<missing>      N/A                                                             524kB     
<missing>      N/A                                                             376kB     
<missing>      N/A                                                             67.2kB    
<missing>      N/A                                                             1.66MB    
<missing>      N/A                                                             321kB     
<missing>      N/A                                                             374kB     
<missing>      N/A                                                             1.86MB    
<missing>      N/A                                                             94.5kB    
<missing>      N/A                                                             94kB      
<missing>      N/A                                                             126kB     
<missing>      N/A                                                             2.31MB    
<missing>      N/A                                                             291kB     
<missing>      N/A                                                             5.91MB    
<missing>      N/A                                                             12.8MB    
<missing>      N/A                                                             236kB     
<missing>      N/A                                                             346B      
<missing>      N/A                                                             497B      
<missing>      N/A                                                             0B        
<missing>      N/A                                                             64B       
<missing>      N/A                                                             0B        
<missing>      N/A                                                             149B      
<missing>      N/A                                                             0B        
<missing>      N/A                                                             82.1kB    
<missing>      N/A                                                             1.47MB    
<missing>      N/A                                                             22.9kB    
<missing>      N/A                                                             271kB    
```

### Comparativo

| Aplicacao       | Versao | Tipo              | Size   | Vulnerabilidades |
| --------------- | ------ | ----------------- | ------ | ---------------- |
| giropops-senhas | 1.0    | Standard          | 1.13GB | 0C  1H  7M  148L |
| giropops-senhas | 2.0    | Multistage        | 136MB  | 0C  0H  1M  21L  |
| giropops-senhas | 3.0    | Alpine            | 64.1MB | 0C  0H  1M  1L   |
| giropops-senhas | 4.0    | Wolfi             | 83MB   | 0C  0H  0M  0L   |
| giropops-senhas | 5.0    | Google Distroless | 72.2MB | 0C  0H  0M  0L   |

## Verificar vulnerabilidade com Trivy

- [Trivy](https://trivy.dev)
- [Install](https://trivy.dev/docs/latest/getting-started/installation/)

```bash
$ trivy image giropops-senhas:1.0 > scans/trivy-v1.0.txt
```

```
Report Summary

┌──────────────────────────────────────────────────────────────────────────────────┬────────────┬─────────────────┬─────────┐
│                                      Target                                      │    Type    │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ giropops-senhas:1.0 (debian 13.3)                                                │   debian   │      1416       │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/blinker-1.9.0.dist-info/METADATA          │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/click-8.3.1.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/flask-3.0.3.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/itsdangerous-2.2.0.dist-info/METADATA     │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/jinja2-3.1.6.dist-info/METADATA           │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/markupsafe-3.0.3.dist-info/METADATA       │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/pip-25.3.dist-info/METADATA               │ python-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/prometheus_client-0.16.0.dist-info/METAD- │ python-pkg │        0        │    -    │
│ ATA                                                                              │            │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/redis-4.5.4.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/werkzeug-3.1.5.dist-info/METADATA         │ python-pkg │        0        │    -    │
└──────────────────────────────────────────────────────────────────────────────────┴────────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)
```

```bash
$ trivy image giropops-senhas:2.0 > scans/trivy-v2.0.txt
```

```
Report Summary

┌──────────────────────────────────────────────────────────────────────────────────┬────────────┬─────────────────┬─────────┐
│                                      Target                                      │    Type    │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ giropops-senhas:2.0 (debian 13.3)                                                │   debian   │       67        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/blinker-1.9.0.dist-info/METADATA          │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/click-8.3.1.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/flask-3.0.3.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/itsdangerous-2.2.0.dist-info/METADATA     │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/jinja2-3.1.6.dist-info/METADATA           │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/markupsafe-3.0.3.dist-info/METADATA       │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/pip-25.3.dist-info/METADATA               │ python-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/prometheus_client-0.16.0.dist-info/METAD- │ python-pkg │        0        │    -    │
│ ATA                                                                              │            │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/redis-4.5.4.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/werkzeug-3.1.5.dist-info/METADATA         │ python-pkg │        0        │    -    │
└──────────────────────────────────────────────────────────────────────────────────┴────────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)
```

```bash
$ trivy image giropops-senhas:3.0 > scans/trivy-v3.0.txt
```

```
Report Summary

┌──────────────────────────────────────────────────────────────────────────────────┬────────────┬─────────────────┬─────────┐
│                                      Target                                      │    Type    │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ giropops-senhas:3.0 (alpine 3.23.3)                                              │   alpine   │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/blinker-1.9.0.dist-info/METADATA          │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/click-8.3.1.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/flask-3.0.3.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/itsdangerous-2.2.0.dist-info/METADATA     │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/jinja2-3.1.6.dist-info/METADATA           │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/markupsafe-3.0.3.dist-info/METADATA       │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/pip-25.3.dist-info/METADATA               │ python-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/prometheus_client-0.16.0.dist-info/METAD- │ python-pkg │        0        │    -    │
│ ATA                                                                              │            │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/redis-4.5.4.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.14/site-packages/werkzeug-3.1.5.dist-info/METADATA         │ python-pkg │        0        │    -    │
└──────────────────────────────────────────────────────────────────────────────────┴────────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)
```

```bash
$ trivy image giropops-senhas:4.0 > scans/trivy-v4.0.txt
```

```
Report Summary

┌──────────────────────────────────────────────────────────────────────────────────┬────────────┬─────────────────┬─────────┐
│                                      Target                                      │    Type    │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ giropops-senhas:4.0 (wolfi 20230201)                                             │   wolfi    │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/blinker-1.9.0.dist-info/METADATA           │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/click-8.3.1.dist-info/METADATA             │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/flask-3.0.3.dist-info/METADATA             │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/itsdangerous-2.2.0.dist-info/METADATA      │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/jinja2-3.1.6.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/markupsafe-3.0.3.dist-info/METADATA        │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/pip-26.0.1.dist-info/METADATA              │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/prometheus_client-0.16.0.dist-info/METADA- │ python-pkg │        0        │    -    │
│ TA                                                                               │            │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/redis-4.5.4.dist-info/METADATA             │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/werkzeug-3.1.5.dist-info/METADATA          │ python-pkg │        0        │    -    │
└──────────────────────────────────────────────────────────────────────────────────┴────────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)
```

```bash
$ trivy image giropops-senhas:5.0 > scans/trivy-v5.0.txt
```

```
Report Summary

┌──────────────────────────────────────────────────────────────────────────────────┬────────────┬─────────────────┬─────────┐
│                                      Target                                      │    Type    │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ giropops-senhas:5.0 (debian 12.13)                                               │   debian   │       92        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/blinker-1.9.0.dist-info/METADATA           │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/click-8.3.1.dist-info/METADATA             │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/flask-3.0.3.dist-info/METADATA             │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/itsdangerous-2.2.0.dist-info/METADATA      │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/jinja2-3.1.6.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/markupsafe-3.0.3.dist-info/METADATA        │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/pip-26.0.1.dist-info/METADATA              │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/prometheus_client-0.16.0.dist-info/METADA- │ python-pkg │        0        │    -    │
│ TA                                                                               │            │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/redis-4.5.4.dist-info/METADATA             │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ app/venv/lib/python3.14/site-packages/werkzeug-3.1.5.dist-info/METADATA          │ python-pkg │        0        │    -    │
└──────────────────────────────────────────────────────────────────────────────────┴────────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)
```

 

## Assinando imagens com cosign

**Cosign** é uma ferramenta open source da sigstore usada para assinar, verificar e armazenar assinaturas de imagens de container. Ela permite garantir a autenticidade e integridade das imagens, ajudando a proteger contra alterações ou ataques.

Assinatura de imagens é o processo de aplicar uma assinatura digital a uma imagem de container. Isso permite que usuários e sistemas verifiquem se a imagem foi criada por uma fonte confiável e se não foi modificada desde a assinatura. Assim, aumenta a segurança no uso e distribuição de containers.

- [Github](https://github.com/sigstore/cosign)
- [Sigstore.dev](https://www.sigstore.dev/)
- [Installation](https://docs.sigstore.dev/cosign/system_config/installation/)

### Install cosign

```bash
$ curl -O -L "https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64"
sudo mv cosign-linux-amd64 /usr/local/bin/cosign
sudo chmod +x /usr/local/bin/cosign
$
$ cosign version                                                                             
  ______   ______        _______. __    _______ .__   __.
 /      | /  __  \      /       ||  |  /  _____||  \ |  |
|  ,----'|  |  |  |    |   (----`|  | |  |  __  |   \|  |
|  |     |  |  |  |     \   \    |  | |  | |_ | |  . `  |
|  `----.|  `--'  | .----)   |   |  | |  |__| | |  |\   |
 \______| \______/  |_______/    |__|  \______| |__| \__|
cosign: A tool for Container Signing, Verification and Storage in an OCI registry.

GitVersion:    v3.0.4
GitCommit:     6832fba4928c1ad69400235bbc41212de5006176
GitTreeState:  clean
BuildDate:     2026-01-09T21:17:16Z
GoVersion:     go1.25.5
Compiler:      gc
Platform:      linux/amd64
$
```

Configure Completion
```bash
$ cosign completion zsh > "${fpath[1]}/_cosign"
$ 
# restart do shell
$ source ~/.zshrc 
```

### Gerar par de chaves

```bash
$ cosign generate-key-pair                     
Enter password for private key: 
Enter password for private key again: 
Private key written to cosign.key
Public key written to cosign.pub
$
```

### Assinando imagens de containers

[Signing](https://docs.sigstore.dev/cosign/signing/signing_with_containers/)

O **cosign** assina a imagem no Dockerhub, então precisamos primeiro efetuar push da imagem para o DockerHub

```bash
$ docker image build -t pauloneves/giropops-senhas-assinada:1.0  .
$ docker push pauloneves/giropops-senhas-assinada:1.0
```

**Assinando**

```bash
$ cosign sign --key cosign.key pauloneves/giropops-senhas-assinada:1.0
WARNING: Image reference pauloneves/giropops-senhas-assinada:1.0 uses a tag, not a digest, to identify the image to sign.
    This can lead you to sign a different image than the intended one. Please use a
    digest (example.com/ubuntu@sha256:abc123...) rather than tag
    (example.com/ubuntu:latest) for the input to cosign. The ability to refer to
    images by tag will be removed in a future release.

Enter password for private key: 
$
```

### Verificando e validando assinatura

```bash
$ cosign verify --key cosign.pub pauloneves/giropops-senhas-assinada:1.0

Verification for index.docker.io/pauloneves/giropops-senhas-assinada:1.0 --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - Existence of the claims in the transparency log was verified offline
  - The signatures were verified against the specified public key

[{"critical":{"identity":{"docker-reference":"index.docker.io/pauloneves/giropops-senhas-assinada:1.0"},"image":{"docker-manifest-digest":"sha256:b33d1df7b9b6c001e944bac796da4a1548ce7660359f3ff01cb0aece98c6e1a3"},"type":"https://sigstore.dev/cosign/sign/v1"},"optional":{}},{"critical":{"identity":{"docker-reference":"index.docker.io/pauloneves/giropops-senhas-assinada:1.0"},"image":{"docker-manifest-digest":"sha256:b33d1df7b9b6c001e944bac796da4a1548ce7660359f3ff01cb0aece98c6e1a3"},"type":"https://sigstore.dev/cosign/sign/v1"},"optional":{}}]
```