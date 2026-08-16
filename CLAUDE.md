# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

The container definitions for the trading stack — the C++ development and
deployment images, and the images of the Iceberg lakehouse — plus the directory
those containers mount. Only `Dockerfiles/` is tracked — see
[README.md](README.md).

## Layout

```
Dockerfiles/<image>/Dockerfile   one directory per image
apps/                            gitignored; each subdirectory is its own repository
volumes/                         gitignored; persistent service data (QuestDB)
```

`apps/` and `volumes/` are in `.gitignore`. Never `git add -f` anything under
them — a change to a project in `apps/` belongs in that project's own
repository, not here.

## Working on an image

Verify a change by building it; an unbuilt Dockerfile edit is unverified.

```bash
docker build -t cpp-dev Dockerfiles/cpp-dev
```

A package-name typo in the long `apt-get install` list fails the build only when
someone rebuilds, which can be months later — `cland` for `clangd` survived
several commits that way. Check names against the Ubuntu 24.04 archive before
adding them.

`orderbook-deploy` is currently a copy of `cpp-dev`. It is meant to become a
runtime-only image (no compilers, debuggers, or `-dev` packages); do not "fix"
the duplication by making the two share a base until that split is designed.

## The `lakehouse-*` images

These are built by `docker compose` from `apps/lakehouse`, not by hand, and
that compose file names them `lakehouse/<service>`. A change here is only
verified by bringing that stack up (`make up && make smoke` there).

Four of the five are a pinned `FROM` and a comment explaining what the service
does and why the tag is pinned there — that is deliberate. The pin is the
content: an upstream tag moving under the stack is exactly the failure these
images exist to prevent, so never replace one with `:latest`, and record the
reason in the comment when you move a pin.

`lakehouse-duckdb` is the one real build. It is on `ubuntu:24.04` rather than
the distroless `duckdb/duckdb` image because the compose stack runs shell
scripts in it (catalog bootstrap, smoke test) that need `bash` and `curl`, and
it installs the `httpfs` and `iceberg` extensions at build time so a container
never has to reach `extensions.duckdb.org` at run time. Keep both properties.

## Commits

One image change per commit, imperative subject. `.DS_Store` is gitignored and
untracked — keep it that way.
