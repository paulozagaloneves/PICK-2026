# Day 1 - Containers

## Índice

- [Day 1 - Containers](#day-1---containers)
  - [Índice](#índice)
  - [O que é um container linux](#o-que-é-um-container-linux)
  - [Containers vs VMs](#containers-vs-vms)
  - [O que é o Docker](#o-que-é-o-docker)
  - [Descomplicando Namespaces](#descomplicando-namespaces)
  - [Descomplicando cgroups](#descomplicando-cgroups)
    - [**Instala cgroup tools**](#instala-cgroup-tools)
    - [**Criar cgroups**](#criar-cgroups)
    - [**Associar um processo**](#associar-um-processo)
    - [**Limitar CPU**](#limitar-cpu)
    - [**Limitar memória**](#limitar-memória)
  - [Copy-On-Write](#copy-on-write)
  - [Docker Internals](#docker-internals)
  - [Instalando o Docker Engine no Linux](#instalando-o-docker-engine-no-linux)
  - [Primeiros comandos](#primeiros-comandos)
  - [Criando e gerenciando os primeiros containers](#criando-e-gerenciando-os-primeiros-containers)
  - [Visualizando métricas e a utilização de recursos pelos containers](#visualizando-métricas-e-a-utilização-de-recursos-pelos-containers)
  - [Visualizando e inspecionando imagens e containers](#visualizando-e-inspecionando-imagens-e-containers)
  - [Criando um container Dettached e o Docker exec](#criando-um-container-dettached-e-o-docker-exec)

## O que é um container linux

Um container Linux é uma unidade de software que empacota código e as suas dependências, permitindo que as aplicações sejam executadas de forma isolada e consistente em diferentes ambientes. 
Utiliza recursos do kernel Linux (namespaces e cgroups) para criar ambientes isolados sem a necessidade de virtualização completa.

Um container é como uma “mini-caixa” isolada que contém:

- a aplicação
- as dependências
- as configurações
- o runtime (ex: JVM, .NET)

👉 Mas partilha o kernel do sistema operativo, por isso é:

- muito mais leve que uma VM
- muito mais rápido a arrancar

  Palavra-chave **ISOLAMENTO**.

**Namespaces**
   - filesystem    : isolamento de filesystem
   - processos     : isolamento de processos
   - network       : isolamento de rede
   - users         : isolamento de utilizadores

**cgroups**
cgroups (control groups) é uma funcionalidade do kernel Linux que permite limitar, contabilizar e isolar o uso de recursos de hardware (CPU, memória, I/O de disco, rede) por grupos de processos.

**Para que serve:**

**Limitar recursos:** Define limites máximos de CPU, memória, etc. que um grupo de processos pode utilizar
**Priorização:** Garante que certos processos tenham acesso prioritário a recursos
**Contabilização:** Monitoriza quanto de cada recurso está a ser utilizado
**Isolamento:** Assegura que um contentor não consome todos os recursos do sistema, prejudicando outros
**Exemplo prático em containers:**

Um container pode ser limitado a usar apenas 0,5 CPU (50% de um núcleo) e 128Mi de memória
Se o container tentar usar mais recursos, o cgroups impede e mantém dentro dos limites definidos
Isto garante que múltiplos containers possam coexistir no mesmo host sem interferir uns com os outros

   - CPU         ex: 0.5
   - Memória     ex: 128Mi

**História**
   - chroot       - versão simplória de isolamento. Apenas isolamento de filesystem.
   - Jails        - vieram no FreeBSD permitiam isolamento de filesystem e de processos.        - 
   - CGroups      - Criado pelo Google como parte do kernel linux para utilização de containers em seu Datacenter
   - LXC          - Em 2008 developers da Virtuozzo, IBM e Google iniciaram o projeto LXC que trazia CGroups, namespaces e chroot.
   - Docker       - Em 2013 os containers popularizaram com o Docker

## Containers vs VMs

![VM vs Container](/DescomplicandoDocker/images/AppsConatinersVSAppsVMs.png)

**Containers:**

- Partilham o kernel do sistema operativo do host
- Apenas isolam a aplicação e as suas dependências
- Arranque em segundos
- Mais leves (MBs)
- Menor consumo de recursos

**Máquinas Virtuais:**

- Cada VM tem o seu próprio sistema operativo completo
- Virtualizam o hardware completo
- Arranque em minutos
- Mais pesadas (GBs)
- Maior consumo de recursos (CPU, memória, disco)

**Analogia:**

**VM:** É como ter várias casas independentes, cada uma com a sua própria estrutura completa, canalizações e eletricidade
**Container:** É como ter apartamentos no mesmo edifício, partilhando a estrutura principal (kernel), mas com espaços isolados

**Quando usar:**

**Containers:** Microserviços, aplicações cloud-native, ambientes de desenvolvimento, CI/CD
**VMs:** Isolamento completo, sistemas operativos diferentes, aplicações legadas, requisitos de segurança muito elevados

## O que é o Docker

Docker é uma plataforma open-source que permite criar, empacotar, distribuição e execução de aplicações em containers. 
Fornece um conjunto de ferramentas e uma interface padronizada que torna os containers mais acessíveis e fáceis de utilizar.

![Docker Workflow](/DescomplicandoDocker/images/WorkflowDocker.png)

**Componentes principais:**
- **Docker Engine**: Motor que executa e gere os containers
- **Docker Images**: Modelos imutáveis que contêm a aplicação e dependências
- **Docker Hub**: Repositório público de imagens
- **Dockerfile**: Ficheiro de instruções para construir imagens

**História do Docker:**

- **2013**: Solomon Hykes apresenta o Docker na PyCon, revolucionando o mundo dos containers ao torná-los acessíveis
- **2014**: Lançamento do Docker 1.0 e criação da Docker Inc.
- **2015**: Criação da OCI (Open Container Initiative) para padronização de containers
- **2016**: Docker Swarm lançado como orquestrador nativo
- **2017**: Kubernetes torna-se o padrão de orquestração, Docker adiciona suporte nativo
- **2020**: Docker Hub introduz limitações de rate para pulls
- **Atualidade**: Docker continua a ser a ferramenta mais popular para desenvolvimento com containers, embora alternativas como Podman ganhem terreno

## Descomplicando Namespaces

**Nosso primeiro container**

```bash
$ uname -a
Linux discovery 6.12.63+deb13-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.12.63-1 (2025-12-30) x86_64 GNU/Linux
```

**Instalar debootstrap**

```bash
$ sudo apt update -y
$ sudo apt install debootstrap -y
```

**Usar debootstrap para criar base do container (Debian)**

```bash
$ sudo debootstrap stable ./debian http://deb.debian.org/debian
I: Target architecture can be executed
I: Retrieving InRelease 
I: Checking Release signature
I: Valid Release signature (key id 41587F7DB8C774BCCF131416762F67A0B2C39DE4)
I: Retrieving Packages 
I: Validating Packages 
I: Resolving dependencies of required packages...
I: Resolving dependencies of base packages...
I: Checking component main on http://deb.debian.org/debian...
...
I: Configuring fdisk...
I: Configuring ifupdown...
I: Configuring libc-bin...
I: Base system installed successfully.
```

**Validar filesystem criado**

```bash
$ ls /debian
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

**Criar namespace**

```bash
$ unshare --help


Usage:
 unshare [options] [<program> [<argument>...]]

Run a program with some namespaces unshared from the parent.

Options:
 -m, --mount[=<file>]      unshare mounts namespace
 -u, --uts[=<file>]        unshare UTS namespace (hostname etc)
 -i, --ipc[=<file>]        unshare System V IPC namespace
 -n, --net[=<file>]        unshare network namespace
 -p, --pid[=<file>]        unshare pid namespace
 -U, --user[=<file>]       unshare user namespace
 -C, --cgroup[=<file>]     unshare cgroup namespace
 -T, --time[=<file>]       unshare time namespace

 --mount-proc[=<dir>]      mount proc filesystem first (implies --mount)
 --mount-binfmt[=<dir>]    mount binfmt filesystem first (implies --user and --mount)
 -l, --load-interp <file>  load binfmt definition in the namespace (implies --mount-binfmt)
 --propagation slave|shared|private|unchanged
                           modify mount propagation in mount namespace
 -R, --root <dir>          run the command with root directory set to <dir>
 -w, --wd <dir>            change working directory to <dir>

 -S, --setuid <uid>        set uid in entered namespace
 -G, --setgid <gid>        set gid in entered namespace
 --map-user <uid>|<name>   map current user to uid (implies --user)
 --map-group <gid>|<name>  map current group to gid (implies --user)
 -r, --map-root-user       map current user to root (implies --user)
 -c, --map-current-user    map current user to itself (implies --user)
 --map-auto                map users and groups automatically (implies --user)
 --map-users <inneruid>:<outeruid>:<count>
                           map count users from outeruid to inneruid (implies --user)
 --map-groups <innergid>:<outergid>:<count>
                           map count groups from outergid to innergid (implies --user)

 -f, --fork                fork before launching <program>
 --kill-child[=<signame>]  when dying, kill the forked child (implies --fork)
                             defaults to SIGKILL

 --setgroups allow|deny    control the setgroups syscall in user namespaces
 --keep-caps               retain capabilities granted in user namespaces

 --monotonic <offset>      set clock monotonic offset (seconds) in time namespaces
 --boottime <offset>       set clock boottime offset (seconds) in time namespaces

 -h, --help                display this help
 -V, --version             display version

For more details see unshare(1).
```

```bash
$ unshare --mount --pid --net --uts --ipc --map-root-user --user --fork chroot ./debian bash
# ps -ef        
Error, do this: mount -t proc proc /proc
# mount -t proc none /proc
ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 17:56 ?        00:00:00 bash
root           4       1  0 18:00 ?        00:00:00 ps -ef
```

```bash
# ip address
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
#
```

```bash
# mount -t sysfs none /sys
# mount -t tmpfs none /tmp
```

**Sair do Namespace**
```bash
# CTFL+D
ou
# exit
```


**Mostrar

```bash
$ lsns
       NS TYPE   NPROCS   PID USER  COMMAND
4026531834 time      105  2403 paulo java -Xms128m -Xmx128m -XX:+ExitOnOutOfMemoryError -cp . -Dlog4j2.disable.jmx=true org.springframework.boot.loader.PropertiesLauncher
4026531835 cgroup    103  6159 paulo /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
4026531836 pid        66  6159 paulo /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
4026531837 user       64  2403 paulo java -Xms128m -Xmx128m -XX:+ExitOnOutOfMemoryError -cp . -Dlog4j2.disable.jmx=true org.springframework.boot.loader.PropertiesLauncher
4026531838 uts       103  6159 paulo /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
4026531839 ipc        90  6159 paulo /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
4026531840 net        87  6159 paulo /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
4026531841 mnt        66  6159 paulo /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
4026532600 pid         1 49768 paulo /usr/share/code/code --type=zygote
...
4026534021 mnt         1  8096 paulo ./girus-server
4026534023 pid         1  8096 paulo ./girus-server
4026534024 cgroup      1  8096 paulo ./girus-server
4026534037 user        3 49610 paulo /usr/share/code/code --type=zygote
4026534038 user        2 78597 paulo unshare --mount --pid --net --uts --ipc --map-root-user --user --fork chroot ./debian bash
4026534039 mnt         2 78597 paulo unshare --mount --pid --net --uts --ipc --map-root-user --user --fork chroot ./debian bash
4026534040 uts         2 78597 paulo unshare --mount --pid --net --uts --ipc --map-root-user --user --fork chroot ./debian bash
4026534042 ipc         2 78597 paulo unshare --mount --pid --net --uts --ipc --map-root-user --user --fork chroot ./debian bash
4026534043 pid         1 78598 paulo └─bash
4026534044 net         2 78597 paulo unshare --mount --pid --net --uts --ipc --map-root-user --user --fork chroot ./debian bash
$
```

## Descomplicando cgroups

### **Instala cgroup tools**

```bash
$ sudo apt install cgroup-tools -y
Installing:                     
  cgroup-tools

Installing dependencies:
  libcgroup3

Summary:
  Upgrading: 0, Installing: 2, Removing: 0, Not Upgrading: 0
  Download size: 149 kB
  Space needed: 608 kB / 67,2 GB available

Get:1 http://deb.debian.org/debian trixie/main amd64 libcgroup3 amd64 3.1.0-2+b2 [62,1 kB]
Get:2 http://deb.debian.org/debian trixie/main amd64 cgroup-tools amd64 3.1.0-2+b2 [87,1 kB]
Fetched 149 kB in 0s (7400 kB/s)        
Selecting previously unselected package libcgroup3:amd64.
(Reading database ... 368046 files and directories currently installed.)
Preparing to unpack .../libcgroup3_3.1.0-2+b2_amd64.deb ...
Unpacking libcgroup3:amd64 (3.1.0-2+b2) ...
Selecting previously unselected package cgroup-tools.
Preparing to unpack .../cgroup-tools_3.1.0-2+b2_amd64.deb ...
Unpacking cgroup-tools (3.1.0-2+b2) ...
Setting up libcgroup3:amd64 (3.1.0-2+b2) ...
Setting up cgroup-tools (3.1.0-2+b2) ...
Processing triggers for libc-bin (2.41-12+deb13u1) ...
Processing triggers for man-db (2.13.1-1) ...

```

### **Criar cgroups**

```bash
$ cgcreate --help
Usage: cgcreate [-h] [-f mode] [-d mode] [-s mode] [-t <tuid>:<tgid>] [-a <agid>:<auid>] -g <controllers>:<path> [-g ...]
Create control group(s)
  -a <tuid>:<tgid>              Owner of the group and all its files
  -d, --dperm=mode              Group directory permissions
  -f, --fperm=mode              Group file permissions
  -g <controllers>:<path>       Control group which should be added
  -h, --help                    Display this help
  -s, --tperm=mode              Tasks file permissions
  -t <tuid>:<tgid>              Owner of the tasks file
  -b                            Ignore default systemddelegate hierarchy
  -c, --scope                   Create a delegated systemd scope
  -p, --pid=pid                 Task pid to use to create systemd scope
  -S, --setdefault              Set this scope as the default scope delegate hierarchy
```

**Criar cgroups v1**
```bash
# cgcreate -g cpu,memory,blkio,devices,freezer:giropops
#
```

**Criar cgroups v2**
```bash
$ sudo mkdir /sys/fs/cgroup/giropops
$ cat /sys/fs/cgroup/cgroup.controllers
cpuset cpu io memory hugetlb pids rdma misc
$ ls /sys/fs/cgroup/giropops 
cgroup.controllers      cpu.idle                         cpu.stat.local            hugetlb.2MB.events.local  memory.high          memory.swap.high        pids.events
cgroup.events           cpu.max                          cpu.weight                hugetlb.2MB.max           memory.low           memory.swap.max         pids.events.local
cgroup.freeze           cpu.max.burst                    cpu.weight.nice           hugetlb.2MB.numa_stat     memory.max           memory.swap.peak        pids.max
cgroup.kill             cpu.pressure                     hugetlb.1GB.current       hugetlb.2MB.rsvd.current  memory.min           memory.zswap.current    pids.peak
cgroup.max.depth        cpuset.cpus                      hugetlb.1GB.events        hugetlb.2MB.rsvd.max      memory.numa_stat     memory.zswap.max        rdma.current
cgroup.max.descendants  cpuset.cpus.effective            hugetlb.1GB.events.local  io.max                    memory.oom.group     memory.zswap.writeback  rdma.max
cgroup.pressure         cpuset.cpus.exclusive            hugetlb.1GB.max           io.pressure               memory.peak          misc.current
cgroup.procs            cpuset.cpus.exclusive.effective  hugetlb.1GB.numa_stat     io.stat                   memory.pressure      misc.events
cgroup.stat             cpuset.cpus.partition            hugetlb.1GB.rsvd.current  io.weight                 memory.reclaim       misc.events.local
cgroup.subtree_control  cpuset.mems                      hugetlb.1GB.rsvd.max      memory.current            memory.stat          misc.max
cgroup.threads          cpuset.mems.effective            hugetlb.2MB.current       memory.events             memory.swap.current  misc.peak
cgroup.type             cpu.stat                         hugetlb.2MB.events        memory.events.local       memory.swap.events   pids.current

$ ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 16:36 ?        00:00:02 /sbin/init
root           2       0  0 16:36 ?        00:00:00 [kthreadd]
root           3       2  0 16:36 ?        00:00:00 [pool_workqueue_release]
...
paulo      93551   93550  0 18:28 pts/1    00:00:00 bash
root       94289       2  0 18:29 ?        00:00:00 [kworker/11:0-mm_percpu_wq]
root       94317       2  0 18:29 ?        00:00:00 [kworker/1:0-events]
root       95089       2  0 18:30 ?        00:00:00 [kworker/5:3-events]
root       95119       2  0 18:30 ?        00:00:00 [kworker/14:1-events]
root       95252       2  0 18:30 ?        00:00:00 [kworker/u65:15]
paulo      95468   86508  0 18:30 ?        00:00:00 /app/extra/msedge --type=utility --utility-sub-type=asset_store.mojom.AssetStoreService --lang=en-US --service-sandbox-type=asset_store_service --render-node-override=/dev/dri/renderD128 --crashpad-handler-pid=19 --enable-crash-r
root       95731       2  0 18:30 ?        00:00:00 [kworker/3:0-mm_percpu_wq]
root       95993       2  0 18:31 ?        00:00:00 [kworker/4:0]
root       96020       2  0 18:31 ?        00:00:00 [kworker/u64:5-comp_1.0.1]
root       96820       2  0 18:32 ?        00:00:00 [kworker/6:1-mm_percpu_wq]
root       96831       2  0 18:32 ?        00:00:00 [kworker/15:1]
root       96856       2  0 18:32 ?        00:00:00 [kworker/10:1-mm_percpu_wq]
paulo      97212   78859 99 18:32 pts/2    00:00:00 ps -ef
$ # 93551
$ # a
```

### **Associar um processo**

**cgroups v1**
```bash
$ # cgclassify -g cpu,memory,blkio,devices,freezer:giropops <PID>
$ cgclassify -g cpu,memory,blkio,devices,freezer:giropops 93551
```

**cgroups v2**
```bash
$ # echo <PID> | sudo tee /sys/fs/cgroup/giropops/cgroup.procs
$ echo 93551 | sudo tee /sys/fs/cgroup/giropops/cgroup.procs
$ cat /sys/fs/cgroup/giropops/cgroup.procs 
93551
$
```

### **Limitar CPU**

**Obter a quantidade de processadores**

```bash
$ etconf _NPROCESSORS_ONLN                                                                                                                                                                                                                                                      130 ↵
16
$
```


**cgroups v1**
```bash
$ cgset -r cpu.cfs_quota_us=1000 giropops
```

**cgroups v2**

**1% CPU**

```bash
$ cat /sys/fs/cgroup/giropops/cpu.max
max 100000
$ echo "1000 100000" | sudo tee /sys/fs/cgroup/giropops/cpu.max
1000 100000
```

25% CPU

```bash
$ cat /sys/fs/cgroup/giropops/cpu.max
max 100000
$ echo "25000 100000" | sudo tee /sys/fs/cgroup/giropops/cpu.max
25000 100000
$
``` 

**Validar limite de memória**

**No nosso namespace**


```bash
$ unshare --mount --pid --net --uts --ipc --map-root-user --user --fork chroot ./debian bash
root@discovery:/# mount -t proc proc /proc
root@discovery:/# mount -t tmpfs none /tmp 
root@discovery:/# mount -t sysfs none /sys
root@discovery:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 18:28 ?        00:00:00 bash
root           5       1  0 18:29 ?        00:00:00 ps -ef

# com 1% de CPU

root@discovery:/# dd if=/dev/zero of=/tmp/catota.img bs=8k count=256K
262144+0 records in
262144+0 records out
2147483648 bytes (2.1 GB, 2.0 GiB) copied, 164.739 s, 13.0 MB/s
root@discovery:/# rm /tmp/catota.img 

# com 25% de CPU

root@discovery:/# dd if=/dev/zero of=/tmp/catota.img bs=8k count=256K
262144+0 records in
262144+0 records out
2147483648 bytes (2.1 GB, 2.0 GiB) copied, 3.50681 s, 612 MB/s
root@discovery:/# 
```



### **Limitar memória**

**cgroups v1**
```bash
$ cgset -r memory.limit_in_bytes=48M giropops
$ cgget -r memory.limit_in_bytes giropops
$ 
```



**cgroups v2**
```bash
$ echo 48M | sudo tee /sys/fs/cgroup/giropops/memory.max
$ cat /sys/fs/cgroup/giropops/memory.max 
50331648
$
```


## Copy-On-Write

O mecanismo de Copy-On-Write (COW) é uma técnica utilizada em sistemas operativos e containers para optimizar a utilização de recursos. Quando vários processos ou containers partilham o mesmo recurso (como ficheiros ou blocos de disco), acedem inicialmente à mesma cópia. Só quando um deles tenta modificar o recurso é que é criada uma nova cópia exclusiva para esse processo, evitando duplicações desnecessárias e poupando memória e armazenamento. Esta abordagem é fundamental para a eficiência de imagens de containers e sistemas de ficheiros modernos.

No mecanismo Copy-On-Write (COW), a nova camada contém apenas as modificações feitas pelo processo/container. Ela não é uma cópia completa da base; apenas os dados alterados são armazenados na nova camada. O restante continua a ser lido da camada base (read-only). Assim, economiza-se espaço e recursos, pois só o que muda é duplicado.

O Copy-on-Write (CoW) é uma estratégia de otimização em computação que adia a duplicação de dados até que uma modificação seja realmente necessária. 

**Como funciona ?**


1. **Compartilhamento Inicial:** Quando um recurso (como memória ou um ficheiro) é "copiado", o sistema não cria uma nova cópia física. Em vez disso, cria apenas uma nova referência (ponteiro) que aponta para os dados originais.
2. **Estado de Leitura:** Enquanto todos os utilizadores apenas lerem os dados, eles partilham a mesma instância, o que poupa memória e tempo de processamento.
3. **A Cópia Real:** No momento em que um utilizador tenta escrever (modificar) os dados, o sistema interseta a operação, cria uma cópia privada desses dados específicos e só então aplica a alteração nessa nova cópia. 

O **Docker** utiliza CoW para gerir camadas de imagens; múltiplos containers partilham a mesma imagem base, gravando apenas as suas alterações individuais numa camada fina no topo.

![Docker Image Layers](/DescomplicandoDocker//images/Docker_Image_Layers.png)


## Docker Internals

![Docker Internals](/DescomplicandoDocker//images/Docker_Internals.png)

## Instalando o Docker Engine no Linux

Reference: https://docs.docker.com/engine/install/ubuntu/

```bash
$ curl -fsSL https://get.docker.com | bash
curl -fsSL https://get.docker.com | bash
# Executing docker install script, commit: f381ee68b32e515bb4dc034b339266aff1fbc460
+ sudo -E sh -c 'apt-get -qq update >/dev/null'
+ sudo -E sh -c 'DEBIAN_FRONTEND=noninteractive apt-get -y -qq install ca-certificates curl >/dev/null'
+ sudo -E sh -c 'install -m 0755 -d /etc/apt/keyrings'
+ sudo -E sh -c 'curl -fsSL "https://download.docker.com/linux/debian/gpg" -o /etc/apt/keyrings/docker.asc'
+ sudo -E sh -c 'chmod a+r /etc/apt/keyrings/docker.asc'
+ sudo -E sh -c 'echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian trixie stable" > /etc/apt/sources.list.d/docker.list'
+ sudo -E sh -c 'apt-get -qq update >/dev/null'
+ sudo -E sh -c 'DEBIAN_FRONTEND=noninteractive apt-get -y -qq install docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-ce-rootless-extras docker-buildx-plugin docker-model-plugin >/dev/null'
Using systemd to manage Docker service
+ sudo -E sh -c systemctl enable --now docker.service
...
+ sudo -E sh -c 'docker version'
Client: Docker Engine - Community
 Version:           29.2.1
 API version:       1.53
 Go version:        go1.25.6
 Git commit:        a5c7197
 Built:             Mon Feb  2 17:17:31 2026
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          29.2.1
  API version:      1.53 (minimum version 1.44)
  Go version:       go1.25.6
  Git commit:       6bc6209
  Built:            Mon Feb  2 17:17:31 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v2.2.1
  GitCommit:        dea7da592f5d1d2b7755e3a161be07f43fad8f75
 runc:
  Version:          1.3.4
  GitCommit:        v1.3.4-0-gd6d73eb8
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

```


**Configurando Docker como rootless**

```bash
$ dockerd-rootless-setuptool.sh install
[ERROR] Missing system requirements. Run the following commands to
[ERROR] install the requirements and run this tool again.

########## BEGIN ##########
sudo sh -eux <<EOF
# Install newuidmap & newgidmap binaries
apt-get install -y uidmap
EOF
########## END ##########

$ sudo sh -eux <<EOF
# Install newuidmap & newgidmap binaries
apt-get install -y uidmap
EOF
+ apt-get install -y uidmap
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libsubid5
The following NEW packages will be installed:
  libsubid5 uidmap
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 416 kB of archives.
After this operation, 691 kB of additional disk space will be used.
Get:1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
Get:2 https://deb.debian.org/debian trixie/main amd64 libsubid5 amd64 1:4.17.4-2 [222 kB]
Get:3 https://deb.debian.org/debian trixie/main amd64 uidmap amd64 1:4.17.4-2 [194 kB]
Fetched 416 kB in 0s (5645 kB/s)
Selecting previously unselected package libsubid5:amd64.
(Reading database ... 24274 files and directories currently installed.)
Preparing to unpack .../libsubid5_1%3a4.17.4-2_amd64.deb ...
Unpacking libsubid5:amd64 (1:4.17.4-2) ...
Selecting previously unselected package uidmap.
Preparing to unpack .../uidmap_1%3a4.17.4-2_amd64.deb ...
Unpacking uidmap (1:4.17.4-2) ...
Setting up libsubid5:amd64 (1:4.17.4-2) ...
Setting up uidmap (1:4.17.4-2) ...
Processing triggers for man-db (2.13.1-1) ...
Processing triggers for libc-bin (2.41-12+deb13u1) ...

$ dockerd-rootless-setuptool.sh install
[INFO] Creating /home/paulo/.config/systemd/user/docker.service
[INFO] starting systemd service docker.service
+ systemctl --user start docker.service
+ sleep 3
+ systemctl --user --no-pager --full status docker.service
● docker.service - Docker Application Container Engine (Rootless)
     Loaded: loaded (/home/paulo/.config/systemd/user/docker.service; disabled; preset: enabled)
     Active: active (running) since Thu 2026-02-05 17:56:41 WET; 3s ago
 ...
Feb 05 17:56:41 Docker-Server-01 systemd[907]: Started docker.service - Docker Application Container Engine (Rootless).
+ DOCKER_HOST=unix:///run/user/1000/docker.sock /usr/bin/docker version
Client: Docker Engine - Community
 Version:           29.2.1
 API version:       1.53
 Go version:        go1.25.6
 Git commit:        a5c7197
 Built:             Mon Feb  2 17:17:31 2026
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          29.2.1
  API version:      1.53 (minimum version 1.44)
  Go version:       go1.25.6
  Git commit:       6bc6209
  Built:            Mon Feb  2 17:17:31 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v2.2.1
  GitCommit:        dea7da592f5d1d2b7755e3a161be07f43fad8f75
 runc:
  Version:          1.3.4
  GitCommit:        v1.3.4-0-gd6d73eb8
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
 rootlesskit:
  Version:          2.3.6
  ApiVersion:       1.1.1
  NetworkDriver:    slirp4netns
  PortDriver:       builtin
  StateDir:         /run/user/1000/dockerd-rootless
 slirp4netns:
  Version:          1.2.1
  GitCommit:        09e31e92fa3d2a1d3ca261adaeb012c8d75a8194
+ systemctl --user enable docker.service
Created symlink '/home/paulo/.config/systemd/user/default.target.wants/docker.service' → '/home/paulo/.config/systemd/user/docker.service'.
[INFO] Installed docker.service successfully.
[INFO] To control docker.service, run: `systemctl --user (start|stop|restart) docker.service`
[INFO] To run docker.service on system startup, run: `sudo loginctl enable-linger paulo`

[INFO] Creating CLI context "rootless"
Successfully created context "rootless"
[INFO] Using CLI context "rootless"
Current context is now "rootless"

[INFO] Make sure the following environment variable(s) are set (or add them to ~/.bashrc):
export PATH=/usr/bin:$PATH

[INFO] Some applications may require the following environment variable too:
export DOCKER_HOST=unix:///run/user/1000/docker.sock

```


**(Opcional) Configurando variáveis de ambiente**
```bash
export DOCKER_HOST=unix:///run/user/1000/docker.sock
``` 

## Primeiros comandos


```bash
# listando containers
$ docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# executando um container
$ docker container run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
17eec7bbc9d7: Pull complete 
ea52d2000f90: Download complete 
Digest: sha256:05813aedc15fb7b4d732e1be879d3252c1c9c25d885824f6295cab4538cb85cd
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
$
```

**Listando todos os containers**

```bash
$ docker container ls -a
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
b18aed2c21d2   hello-world   "/hello"   2 minutes ago   Exited (0) 2 minutes ago             loving_johnson
```

## Criando e gerenciando os primeiros containers

-i - interação
-t - terminal

```bash
$ docker container run -ti ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
a3629ac5b9f4: Pull complete 
1baf05536e37: Download complete 
Digest: sha256:cd1dba651b3080c3686ecf4e3c4220f026b521fb76978881737d24f200828b2b
Status: Downloaded newer image for ubuntu:latest
root@4800fabda6d6:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@4800fabda6d6:/# uname -a
Linux 4800fabda6d6 6.12.63+deb13-cloud-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.12.63-1 (2025-12-30) x86_64 x86_64 x86_64 GNU/Linux
root@4800fabda6d6:/# cat /proc/info       
cat: /proc/info: No such file or directory
root@4800fabda6d6:/# ps -ef        
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 18:11 pts/0    00:00:00 /bin/bash
root          12       1  0 18:12 pts/0    00:00:00 ps -ef
root@4800fabda6d6:/# cat /etc/issue
Ubuntu 24.04.3 LTS \n \l

root@4800fabda6d6:/#   <CTRL + D>
exit
$ 
$ docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
$
```

**Executando e saindo sem matar o container**

```bash
$ docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
paulo@Docker-Server-01:~$ docker container run -ti ubuntu
root@3efef89d96a8:/# uptime
 18:14:25 up 23 min,  0 user,  load average: 0.01, 0.01, 0.00
root@3efef89d96a8:/# cat /etc/issue
Ubuntu 24.04.3 LTS \n \l

root@3efef89d96a8:/#   <CTRL + p + q>
$
$ docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
3efef89d96a8   ubuntu    "/bin/bash"   26 seconds ago   Up 26 seconds             strange_booth

# Entrar no container acima (ATTACH)
$ docker container attach 3efef89d96a8
root@3efef89d96a8:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 18:14 pts/0    00:00:00 /bin/bash
root          12       1  0 18:17 pts/0    00:00:00 ps -ef
root@3efef89d96a8:/#
```


**Atribuindo nome**

```bash
$ docker container run --name toskao -ti ubuntu
root@5b43c0877f29:/# uptime
 18:19:27 up 28 min,  0 user,  load average: 0.08, 0.03, 0.01
root@5b43c0877f29:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 18:19 pts/0    00:00:00 /bin/bash
root          10       1  0 18:19 pts/0    00:00:00 ps -ef
root@5b43c0877f29:/#  <CTRL + p + q>

$ docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
5b43c0877f29   ubuntu    "/bin/bash"   32 seconds ago   Up 31 seconds             toskao

$ docker container attach toskao
root@5b43c0877f29:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 18:19 pts/0    00:00:00 /bin/bash
root          11       1  0 18:21 pts/0    00:00:00 ps -ef
root@5b43c0877f29:/#  <CTRL + p  + q>
exit
```

**Iniciando e Parando**

```bash
$ docker container ls -a
CONTAINER ID   IMAGE         COMMAND       CREATED          STATUS                      PORTS     NAMES
5b43c0877f29   ubuntu        "/bin/bash"   3 minutes ago    Exited (0) 29 seconds ago             toskao
3efef89d96a8   ubuntu        "/bin/bash"   8 minutes ago    Exited (0) 3 minutes ago              strange_booth
4800fabda6d6   ubuntu        "/bin/bash"   11 minutes ago   Exited (0) 10 minutes ago             compassionate_mirzakhani
b18aed2c21d2   hello-world   "/hello"      18 minutes ago   Exited (0) 18 minutes ago             loving_johnson

$ docker container start toskao
toskao

$ docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS          PORTS     NAMES
5b43c0877f29   ubuntu    "/bin/bash"   4 minutes ago   Up 20 seconds             toskao
$ docker container stop toskao
toskao
$
```

**Pausando e unpause**

```bash
$ docker container start toskao
toskao
$ docker container pause toskao
toskao
$ docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                       PORTS     NAMES
5b43c0877f29   ubuntu    "/bin/bash"   10 minutes ago   Up About a minute (Paused)             toskao
$ 
# pause
$ docker container unpause toskao
toskao
$ docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS         PORTS     NAMES
5b43c0877f29   ubuntu    "/bin/bash"   15 minutes ago   Up 6 minutes             toskao
$ 
```


**Remover o container**

```bash
# parar o container
$ docker container stop toskao
toskao
$ docker container rm toskao
toskao

$ docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ docker container ls -a
CONTAINER ID   IMAGE         COMMAND       CREATED          STATUS                      PORTS     NAMES
3efef89d96a8   ubuntu        "/bin/bash"   23 minutes ago   Exited (0) 19 minutes ago             strange_booth
4800fabda6d6   ubuntu        "/bin/bash"   26 minutes ago   Exited (0) 25 minutes ago             compassionate_mirzakhani
b18aed2c21d2   hello-world   "/hello"      33 minutes ago   Exited (0) 33 minutes ago             loving_johnson

```


## Visualizando métricas e a utilização de recursos pelos containers

```bash
$ docker container run --name toskao -ti ubuntu
root@0d35e44b3264:/# <CTRL + p + q>
$ docker container run --name toskao2 -ti ubuntu
root@0d35e44b3264:/# <CTRL + p + q>
$ docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
0d35e44b3264   ubuntu    "/bin/bash"   7 seconds ago        Up 6 seconds                  toskao2
303950dd22a9   ubuntu    "/bin/bash"   About a minute ago   Up About a minute             toskao
$ 
```

```bash
$ docker container stats
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O         BLOCK I/O   PIDS 
0d35e44b3264   toskao2   0.00%     908KiB / 3.83GiB    0.02%     746B / 126B     0B / 0B     1 
303950dd22a9   toskao    0.00%     904KiB / 3.83GiB    0.02%     1.17kB / 126B   0B / 0B     1 
```

```bash
$ docker container stats --no-stream
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O         BLOCK I/O   PIDS
0d35e44b3264   toskao2   0.00%     908KiB / 3.83GiB    0.02%     746B / 126B     0B / 0B     1
303950dd22a9   toskao    0.00%     904KiB / 3.83GiB    0.02%     1.17kB / 126B   0B / 0B     1
$ 
```


**Visualizando o que está rodando dentro do container**

```bash
$ docker container top toskao2
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                5126                5100                0                   18:47               pts/0               00:00:00            /bin/bash
$
```

**Visualizando logs**
```bash
$  docker container logs --details toskao
root@303950dd22a9:/# dd if=/dev/zero of=catota.img bs=8k count=256k
262144+0 records in
 262144+0 records out
 2147483648 bytes (2.1 GB, 2.0 GiB) copied, 2.00172 s, 1.1 GB/s
root@303950dd22a9:/# dd if=/dev/zero of=catota.img bs=8k count=256K
262144+0 records in
 262144+0 records out
 2147483648 bytes (2.1 GB, 2.0 GiB) copied, 1.85021 s, 1.2 GB/s
root@303950dd22a9:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
 root           1       0  0 18:45 pts/0    00:00:00 /bin/bash
 root          11       1  0 18:55 pts/0    00:00:00 ps -ef
root@303950dd22a9:/# dd if=/dev/zero of=catota.img bs=8k count=2560K
dd: error writing 'catota.img': No space left on device
 2241817+0 records in
 2241816+0 records out
 18364960768 bytes (18 GB, 17 GiB) copied, 15.8248 s, 1.2 GB/s

$
```


**Prune: Remover todos os containers parados**

```bash
$ docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
0d35e44b32646527b49d789309f99e5019cff8d414504e749306f5439bcb2ea2
303950dd22a909fa8045b2bd2a35c1445c0db1a53f3593db879ffbd87f7b76bf
3efef89d96a8254de6d18b14235cc8393e4c7d404d05369b92a386c35f93f664
4800fabda6d6df1cb88d20384965dc5af34fe315b2317d3407d67d43236c7bbb
b18aed2c21d2dcd9b3004edd731221876ce8dc68fde7f5889e99ed87d070bf2f

Total reclaimed space: 36.86kB

$ docker container ls -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ 
```

## Visualizando e inspecionando imagens e containers

```bash
$ docker image ls
                                                                         i Info →   U  In Use
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
debian:latest        2c91e484d93f        186MB         52.5MB        
hello-world:latest   05813aedc15f       25.9kB         9.52kB        
ubuntu:latest        cd1dba651b30        119MB         31.7MB
$
```

```bash
$ docker run -it debian
root@94660a32f130:/# 

$ docker image ls
                                                                        i Info →   U  In Use
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
debian:latest        2c91e484d93f        186MB         52.5MB    U   
hello-world:latest   05813aedc15f       25.9kB         9.52kB        
ubuntu:latest        cd1dba651b30        119MB         31.7MB 
```

```bash
$ docker image rm hello-world:latest 
Untagged: hello-world:latest
Deleted: sha256:05813aedc15fb7b4d732e1be879d3252c1c9c25d885824f6295cab4538cb85cd
$ 
```

```bash
$ docker image rm debian:latest
Error response from daemon: conflict: unable to delete debian:latest (must be forced) - container 94660a32f130 is using its referenced image 2c91e484d93f

$ docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
94660a32f130   debian    "bash"    9 minutes ago   Up 9 minutes             confident_hugle
$
```

**Inspecionar um recurso**

```bash
$ $ docker container inspect confident_hugle 
[
    {
        "Id": "94660a32f1307a19abfe26cfb70a02afe6a8a2ac776009be4dd28a80bdff5cd0",
        "Created": "2026-02-05T19:23:49.02712247Z",
        "Path": "bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 6319,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2026-02-05T19:23:49.078685788Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
...
"ImageManifestDescriptor": {
            "mediaType": "application/vnd.oci.image.manifest.v1+json",
            "digest": "sha256:aca3e110b8fee2a2acdf8cbe6cef1cfebef52e5257da87fbd41920d3411c1aed",
            "size": 1021,
            "annotations": {
                "com.docker.official-images.bashbrew.arch": "amd64",
                "org.opencontainers.image.base.name": "scratch",
                "org.opencontainers.image.created": "2026-02-02T00:00:00Z",
                "org.opencontainers.image.revision": "2b04ad858740e348d347477c0e0b27539eb323a3",
                "org.opencontainers.image.source": "https://github.com/debuerreotype/docker-debian-artifacts.git",
                "org.opencontainers.image.url": "https://hub.docker.com/_/debian",
                "org.opencontainers.image.version": "trixie"
            },
            "platform": {
                "architecture": "amd64",
                "os": "linux"
            }
        }
    }
]
```

```bash
$ docker image inspect 2c91e484d93f
[
    {
        "Id": "sha256:2c91e484d93f0830a7e05a2b9d92a7b102be7cab562198b984a84fdbc7806d91",
        "RepoTags": [
            "debian:latest"
        ],
        "RepoDigests": [
            "debian@sha256:2c91e484d93f0830a7e05a2b9d92a7b102be7cab562198b984a84fdbc7806d91"
        ],
        "Comment": "debuerreotype 0.17",
        "Created": "2026-02-02T00:00:00Z",
        "Config": {
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "bash"
            ]
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 49303357,
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:d16566b6c5a6bd5c45da221a2fc94937d8d503c920c8627f154a2f8a57537da4"
            ]
        },
        "Metadata": {
            "LastTagTime": "2026-02-05T19:22:15.261009057Z"
        },
        "Descriptor": {
            "mediaType": "application/vnd.oci.image.index.v1+json",
            "digest": "sha256:2c91e484d93f0830a7e05a2b9d92a7b102be7cab562198b984a84fdbc7806d91",
            "size": 8933
        },
        "Identity": {
            "Pull": [
                {
                    "Repository": "docker.io/library/debian"
                }
            ]
        }
    }
]
$
```


## Criando um container Dettached e o Docker exec

-d - Dettached 

```bash
$ docker run -d --name toskao-deb -it debian
49400b8ebfab6a5c81fbaee6c061b2132b626522f017e701f2db1639c814b0a1
$
$ docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
49400b8ebfab   debian    "bash"    22 seconds ago   Up 21 seconds             toskao-deb
```


```bash
$ docker container run -d --name meu-nginx nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
bae5a1799a80: Pull complete 
4f4efe02d542: Pull complete 
7b6cb8ccac7b: Pull complete 
f73400a233fd: Pull complete 
0c8d55a45c0d: Pull complete 
46bf3a120c8e: Pull complete 
47cd406a84ef: Pull complete 
2e02dba24409: Download complete 
a5d78d617315: Download complete 
Digest: sha256:b17697e86d0c02378716277d09f45b946f8709aaa12c708e30fdd4736f536af1
Status: Downloaded newer image for nginx:latest
4db8e657ae350a7ace2b4c75a2fdc9cb1bfeffeb4e6c5e4464ea29732f53e734
$ 
$ docker container ls
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
4db8e657ae35   nginx     "/docker-entrypoint.…"   49 seconds ago   Up 49 seconds   80/tcp    meu-nginx
49400b8ebfab   debian    "bash"                   2 minutes ago    Up 2 minutes              toskao-deb
$
$ docker image ls
                                                                           i Info →   U  In Use
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
debian:latest   2c91e484d93f        186MB         52.5MB    U   
nginx:latest    b17697e86d0c        240MB         65.8MB    U   
ubuntu:latest   cd1dba651b30        119MB         31.7MB  
```



**Attach num nginx**

```bash
 docker container attach meu-nginx 

<CTRL + C>

^C2026/02/05 19:49:57 [notice] 1#1: signal 2 (SIGINT) received, exiting
2026/02/05 19:49:57 [notice] 30#30: exiting
2026/02/05 19:49:57 [notice] 29#29: exiting
2026/02/05 19:49:57 [notice] 30#30: exit
2026/02/05 19:49:57 [notice] 29#29: exit
2026/02/05 19:49:57 [notice] 31#31: exiting
2026/02/05 19:49:57 [notice] 32#32: exiting
2026/02/05 19:49:57 [notice] 31#31: exit
2026/02/05 19:49:57 [notice] 32#32: exit
2026/02/05 19:49:57 [notice] 1#1: signal 17 (SIGCHLD) received from 29
2026/02/05 19:49:57 [notice] 1#1: worker process 29 exited with code 0
2026/02/05 19:49:57 [notice] 1#1: signal 29 (SIGIO) received
2026/02/05 19:49:57 [notice] 1#1: signal 17 (SIGCHLD) received from 30
2026/02/05 19:49:57 [notice] 1#1: worker process 30 exited with code 0
2026/02/05 19:49:57 [notice] 1#1: worker process 31 exited with code 0
2026/02/05 19:49:57 [notice] 1#1: worker process 32 exited with code 0
2026/02/05 19:49:57 [notice] 1#1: exit
$
$ docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
49400b8ebfab   debian    "bash"    6 minutes ago   Up 6 minutes             toskao-deb

# morreu o container meu-nginx
$
```


**EXEC: executar dentro do container**

```bash
$ docker container start meu-nginx 
meu-nginx

$ docker container exec -ti meu-nginx ls /
bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
$
$ docker container exec -ti meu-nginx curl localhost
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

```bash
$ docker container exec -ti meu-nginx bash
root@4db8e657ae35:/# curl localhost
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

root@4db8e657ae35:/# cat /proc/1/cmdline 
nginx: master process nginx -g daemon off;

root@4db8e657ae35:/#  cat /proc/38/cmdline 
bash

root@4db8e657ae35:/# 
```



**-p - Publish**


```bash
$ docker container run -d -p 8080:80 --name meu-nginx nginx
039f7b9afd63ff5d8e880c3c35b6f4e8f57d37bef1c0b6e8d05c876fc0aa482b
$ docker container ls
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
039f7b9afd63   nginx     "/docker-entrypoint.…"   8 seconds ago   Up 7 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   meu-nginx

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
```

```bash
$ docker container exec -ti meu-nginx bash
root@039f7b9afd63:/# echo OPAAAAAA > /usr/share/nginx/html/index.html 
root@039f7b9afd63:/# 
exit

$ curl localhost:8080
OPAAAAAA
$
```



**PULL**

```bash
$ docker pull almalinux:10
10: Pulling from library/almalinux
ba7004570691: Pull complete 
b443ed4dfc64: Download complete 
4b9babb8d0a9: Download complete 
Digest: sha256:3ea6bed76e47c1a816ed7e1ed7be8661efcf6984bec90bcad5ec73b66b6754ce
Status: Downloaded newer image for almalinux:10
docker.io/library/almalinux:10
$
$ docker image ls
                                                                      i Info →   U  In Use
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
almalinux:10    3ea6bed76e47        273MB           72MB        
debian:latest   2c91e484d93f        186MB         52.5MB        
nginx:latest    b17697e86d0c        240MB         65.8MB    U   
ubuntu:latest   cd1dba651b30        119MB         31.7MB 
$
```


**CREATE**

```bash
$ docker container create --name meu-debian -it debian
1328a6184c31f707a3097bf67402077d575c9f6b8b2786194680e06b0a1a4570

$ docker container ls
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                     NAMES
039f7b9afd63   nginx     "/docker-entrypoint.…"   17 minutes ago   Up 17 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   meu-nginx

$ docker container ls -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                     NAMES
1328a6184c31   debian    "bash"                   14 seconds ago   Created                                                   meu-debian
039f7b9afd63   nginx     "/docker-entrypoint.…"   17 minutes ago   Up 17 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   meu-nginx

$ docker container start meu-debian 
meu-debian

$ docker container ls 
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                     NAMES
1328a6184c31   debian    "bash"                   32 seconds ago   Up 4 seconds                                              meu-debian
039f7b9afd63   nginx     "/docker-entrypoint.…"   18 minutes ago   Up 18 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   meu-nginx
```