FROM golang:1.25 AS builder

# Use bash as the default shell (Definir o shell a ser usado para os comandos RUN, CMD e ENTRYPOINT)
# Note: The official Golang image uses /bin/sh by default, which is a symlink to dash in Debian-based images. 
# If you want to use bash, you can specify it in the SHELL instruction.
#SHELL ["/bin/bash"]

WORKDIR /app
COPY . ./
RUN go mod init hello
RUN go build -o /app/hello

FROM alpine:3.23.3
COPY --from=builder /app/hello /app/hello
ENV APP="hello_world"
ARG GIROPOPS="strigus"
ENV GIROPOPS=${GIROPOPS}

RUN echo "Hello, ${GIROPOPS} - ${APP}!" 

# Expose the volume for data persistence
VOLUME /app/dados
CMD ["/app/hello"]
