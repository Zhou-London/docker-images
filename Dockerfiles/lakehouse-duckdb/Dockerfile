# Query engine and init client of the lakehouse. Built on Ubuntu instead of
# the official distroless duckdb/duckdb image because the compose stack also
# runs shell scripts in it (catalog bootstrap, smoke test), which need bash,
# curl, and jq.
#
# The iceberg and httpfs extensions are installed at build time so the
# container works without network access to extensions.duckdb.org.
FROM ubuntu:24.04

ENV TZ=Asia/Shanghai \
    LANG=C.UTF-8 \
    DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    jq \
    unzip \
    && rm -rf /var/lib/apt/lists/*

ARG DUCKDB_VERSION=1.5.5

RUN arch="$(dpkg --print-architecture)" \
    && curl -fsSL -o /tmp/duckdb.zip \
       "https://github.com/duckdb/duckdb/releases/download/v${DUCKDB_VERSION}/duckdb_cli-linux-${arch}.zip" \
    && unzip /tmp/duckdb.zip -d /usr/local/bin \
    && rm /tmp/duckdb.zip \
    && duckdb -c "INSTALL httpfs; INSTALL iceberg;"

WORKDIR /work
CMD ["duckdb"]
