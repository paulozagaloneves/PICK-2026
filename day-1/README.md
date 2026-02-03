# Day 1 - Containers

## Índice
- [O que é um container linux](#o-que-é-um-container-linux)
  - [Namespaces](#namespaces)
  - [cgroups](#cgroups)
  - [História](#história)
- [Containers vs VMs](#containers-vs-vms)
- [O que é o Docker](#o-que-é-o-docker)

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



 Palavra-chave **ISOLAMENTO** de recursos.


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

