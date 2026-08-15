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
`orderbook`), and `volumes/` holds database state that has no business in git.
