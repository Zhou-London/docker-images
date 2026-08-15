# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

The container definitions for the C++ trading stack, and the directory those
containers mount. Only `Dockerfiles/` is tracked — see [README.md](README.md).

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

## Commits

One image change per commit, imperative subject. `.DS_Store` is gitignored and
untracked — keep it that way.
