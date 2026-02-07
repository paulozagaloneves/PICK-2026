FROM golang:1.25 AS builder
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
CMD ["/app/hello"]
