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