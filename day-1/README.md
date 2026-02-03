# Day 1 - Containers

## Índice

- [Day 1 - Containers](#day-1---containers)
  - [Índice](#índice)
  - [O que é um container linux](#o-que-é-um-container-linux)
  - [Containers vs VMs](#containers-vs-vms)
  - [O que é o Docker](#o-que-é-o-docker)
  - [Descomplicando Namespaces](#descomplicando-namespaces)
  - [Descomplicando CGroups](#descomplicando-cgroups)
    - [**Instala CGroup tools**](#instala-cgroup-tools)
    - [**Criar cgroups**](#criar-cgroups)
    - [**Associar um processo**](#associar-um-processo)
    - [**Limitar CPU**](#limitar-cpu)
    - [**Limitar memória**](#limitar-memória)
  - [Copy-On-Write](#copy-on-write)
  - [Docker Internals](#docker-internals)

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

  alavra-chave **ISOLAMENTO** de recursos.

**Namespaces**
   - filesystem    : isolamento de filesystem
   - processos     : isolamento de processos
   - network       : isolamento de rede
   - users         : isolamento de utilizadores

**cgroups  **
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

![VM vs Container](/images/AppsConatinersVSAppsVMs.png)

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

![Docker Workflow](/images/WorkflowDocker.png)

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

## Descomplicando CGroups

### **Instala CGroup tools**

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

![Docker Image Layers](/images/Docker_Image_Layers.png)


## Docker Internals

![Docker Internals](/images/Docker_Internals.png)