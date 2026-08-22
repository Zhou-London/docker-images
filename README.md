<img src="https://capsule-render.vercel.app/api?type=waving&height=400&text=Containers&fontAlign=80&fontAlignY=40&color=gradient" />

<p align="center">
  <img alt="Docker" src="https://img.shields.io/badge/Docker-images-2496ED?logo=docker&logoColor=white" />
  <img alt="Ubuntu 24.04" src="https://img.shields.io/badge/Ubuntu-24.04%20LTS-E95420?logo=ubuntu&logoColor=white" />
  <img alt="C++23" src="https://img.shields.io/badge/C%2B%2B-23-00599C?logo=cplusplus&logoColor=white" />
  <img alt="Arrow / Parquet" src="https://img.shields.io/badge/Apache-Arrow%20%2F%20Parquet-419EBF?logo=apache&logoColor=white" />
</p>

Dockerfiles for the C++ development and deployment containers, plus the working
tree they are mounted into.

## Images

| Image | Dockerfile | Purpose |
|---|---|---|
| `cpp-dev` | [`Dockerfiles/cpp-dev/Dockerfile`](Dockerfiles/cpp-dev/Dockerfile) | The C++ development container — compilers, debuggers, build tools, and the trading-stack libraries. This is the one to work in. |
| `orderbook-deploy` | [`Dockerfiles/orderbook-deploy/Dockerfile`](Dockerfiles/orderbook-deploy/Dockerfile) | Intended runtime image for the orderbook service. **Currently byte-identical to `cpp-dev`** — a placeholder that still has to be slimmed to a runtime-only stage. |

Both start from `ubuntu:24.04`, set `TZ=Asia/Shanghai` and `LANG=C.UTF-8`, and
land in `/work` with a bash shell.

### Lakehouse

Images for the Iceberg lakehouse in `apps/lakehouse` (MinIO + Lakekeeper +
Postgres + DuckDB, orchestrated by that project's compose file):

| Image | Dockerfile | Purpose |
|---|---|---|
| `lakehouse/minio` | [`Dockerfiles/lakehouse-minio/Dockerfile`](Dockerfiles/lakehouse-minio/Dockerfile) | Object store; Parquet data and Iceberg metadata. |
| `lakehouse/mc` | [`Dockerfiles/lakehouse-mc/Dockerfile`](Dockerfiles/lakehouse-mc/Dockerfile) | One-shot MinIO provisioning (bucket, STS user). |
| `lakehouse/postgres` | [`Dockerfiles/lakehouse-postgres/Dockerfile`](Dockerfiles/lakehouse-postgres/Dockerfile) | Lakekeeper's metadata database. |
| `lakehouse/lakekeeper` | [`Dockerfiles/lakehouse-lakekeeper/Dockerfile`](Dockerfiles/lakehouse-lakekeeper/Dockerfile) | Iceberg REST catalog. |
| `lakehouse/duckdb` | [`Dockerfiles/lakehouse-duckdb/Dockerfile`](Dockerfiles/lakehouse-duckdb/Dockerfile) | Query engine, catalog init job, and smoke test. |

### What's inside

- **Toolchain** — `build-essential`, `g++`, `clang`, `clangd`, `clang-format`,
  `clang-tidy`, `gdb`, `lldb`, `cmake`, `ninja-build`, `make`, `pkg-config`
- **Core libraries** — Boost, OpenSSL, libcurl, fmt, spdlog, TBB, nlohmann-json,
  Protobuf, ZeroMQ (`libzmq3` + `cppzmq`)
- **Compression** — zlib, zstd, lz4, snappy, bzip2
- **Testing** — GoogleTest, GoogleMock, Google Benchmark
- **Columnar data** — Apache Arrow (core, dataset, acero, flight) and Parquet,
  installed from Apache's own APT repository keyed to the Ubuntu codename

## Usage

```bash
docker build -t cpp-dev Dockerfiles/cpp-dev
```

```bash
docker run --rm -it -v "$PWD/apps:/work" cpp-dev
```

Inside the container, a project builds the usual way:

```bash
cmake -S /work/nlib -B /work/nlib/build -DCMAKE_BUILD_TYPE=Release && cmake --build /work/nlib/build -j
```

## Repository layout

```
Dockerfiles/    the images (this is all that is tracked)
apps/           projects mounted at /work — gitignored, each its own repository
volumes/        persistent service data (QuestDB) — gitignored
```

`apps/` and `volumes/` are deliberately untracked: the projects under `apps/`
are separate repositories ([nlib](https://github.com/Zhou-London/nlib),
[orderbook](https://github.com/Zhou-London/nqbook),
[lakehouse](https://github.com/Zhou-London/nqlake),
[util](https://github.com/Zhou-London/nq-util)), and `volumes/` holds
database state that has no business in git.

`util` is the exception to the mount: its tools run on the host, not in these
images. It sits under `apps/` because it is part of the same stack, not
because a container ever mounts it.

## Releases

### 2026-08-22

- **`jq` in `lakehouse-duckdb`.** The image is where the lakehouse runs its
  bootstrap and smoke-test scripts, and `lakekeeper-init` now reads the
  warehouse's stored storage profile back to compare MinIO's endpoint against
  the configured one — parsing that JSON with `grep` was what limited the
  script to "does this warehouse exist" in the first place.

### 2026-08-16

- **The five `lakehouse-*` images**, the container half of the Iceberg
  lakehouse in `apps/lakehouse`: MinIO and its `mc` provisioning client,
  Postgres 17 for catalog metadata, Lakekeeper v0.13.1 as the Iceberg REST
  catalog, and DuckDB v1.5.5 as the query engine.
- Four are a pinned `FROM` and the reason for the pin. `lakehouse-duckdb` is a
  real build on `ubuntu:24.04` — the distroless upstream image has no shell,
  and the stack runs bootstrap and smoke-test scripts inside this container. It
  bakes in the `httpfs` and `iceberg` extensions so a running container never
  needs `extensions.duckdb.org`.

### 2026-08-15

- **`cpp-dev`**, the C++23 development image: toolchain, Boost, Arrow and
  Parquet from Apache's own APT repository, ZeroMQ, Protobuf, the compression
  libraries, and GoogleTest/Benchmark.
- Fixed `clangd`, which had been misspelled `cland` in the `apt-get install`
  list and so had never actually been installed — a typo there fails only on
  the next rebuild, which is why package names now get checked against the
  Ubuntu 24.04 archive before they land.
- **`orderbook-deploy`** exists as a placeholder and is byte-identical to
  `cpp-dev`. Slimming it to a runtime-only stage is still to do.
