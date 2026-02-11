#FROM python:3.14-slim-trixie
#FROM python:3.14-alpine AS builder
#FROM gcr.io/distroless/python3-debian12 AS builder
FROM cgr.dev/chainguard/python:latest-dev AS builder
WORKDIR /app

RUN python -m venv /app/venv
COPY requirements.txt .
RUN /app/venv/bin/pip install --no-cache-dir -r requirements.txt

#FROM cgr.dev/chainguard/python:latest
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

#ENV REDIS_HOST=192.168.1.205
#CMD ["/app/venv/bin/flask", "run", "--host=0.0.0.0"]
ENTRYPOINT ["python", "-m", "flask", "run"]
