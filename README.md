# PICK-2026

**[Day 1 - Containers](day-1/README.md)**
- O que é um container linux
- Containers vs VMs
- O que é o Docker
- Descomplicando Namespaces
- Descomplicando cgroups
  - Instala cgroup tools
  - Criar cgroups
  - Associar um processo
  - Limitar CPU
  - Limitar memória
- Copy-On-Write
- Docker Internals
- Instalando o Docker Engine no Linux
- Primeiros comandos
- Criando e gerenciando os primeiros containers
- Visualizando métricas e a utilização de recursos pelos containers
- Visualizando e inspecionando imagens e containers
- Criando um container Dettached e o Docker exec

**[Day 2 - Imagens do Container](day-2/README.md)**
  - O que são imagens de container ?
  - O meu primeiro Dockerfile
	- Exemplo de Dockerfile com Ubuntu e Nginx
	- Build da imagem
  - Conhecendo mais parâmetros no Dockerfile
  - Dockerfile e Entrypoint
  - Adicionando HEALTHCHECK ao nosso Dockerfile
  - Descomplicando o meu Dockerfile
  - Desafio prático
  - Multistage
	- Dockerfile sem multistage
	- Dockerfile com multistage
	- Descomplicando o meu Dockerfile com multistage
  - ENV e ARG no Dockerfile
	    - Comando ENV
	    - Comando ARG
  - Volumes
  - Pull, Push e Dockerhub
  - Pull
  - Login
  - Logout
  - Tag
  - Push
  - Search
  - History
  - Registry privado
  - Glossário Dockerfile
  - Timeline: Criação de uma Imagem Docker

**[Day 3 - Otimizando Imagens](day-3/README.md)**
  - Giropops Senhas
    - Clone do projeto
    - Instalar o pip
    - Criar um ambiente virtual do python3
    - Instalar as dependências da nossa aplicação
    - Executar a aplicação Giropops Senhas
    - Instalar o REDIS
    - Reexecutar a aplicação
  - Containerizando a aplicação Giropops Senhas
    - Build
    - Run
  - Optimizando Imagem
    - 2ª versão (versão slim)
    - 3ª versão (alpine)
  - Optimizando mais ainda a nossa imagem com Distroless
    - Chainguard Wolfi
    - Distroless Google
    - Comparativo
  - Verificar vulnerabilidade com Trivy
  - Assinando imagens com cosign
    - Install cosign
    - Gerar par de chaves
    - Assinando imagens de containers
    - Verificando e validando assinatura

**[Day 4 - Volumes](day-4/README.md)**
   - O que são volumes
   - Tipos de volumes
   - Particularidades entre volumes e Containers
   - Volume do tipo Bind
   - Listar volumes
   - Volume do tipo Volume

**[Day 5 - Networks](day-5/README.md)**
  
**[Day 6 - Docker Compose](day-6/README.md)**
  - Primeiro Docker Compose
  - Giropops Senhas no compose
    - Compose file
    - Apply compose file
  - Docker Compose- Comandos Adicionais
    - Listar stacks docker compose
    - Obter logs de um serviço
    - Obter estatisticas dos serviços
    - Executar um comando dentro de um container
    - Listar imagens dos serviços
    - Stop, Start, Restart, Pause e Unpause Services
  - Volumes
    - listar volumes
  - Build de imagem no Compose
  - Scale - Escalar services
  - Reservando e Definindo recursos como CPU e memória
  - Health check
  - Docker Compose Avançado
    - Contextos no build
    - Environment
    - Volumes
    - Réplicas
    - Labels
    - Update-config
    - Restart Policy
    - Attach USD Device
    - DNS
    - Network Avançado (Subnet)
    - Inspect

   