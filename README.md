# Docker Publish Action

Build and publish Docker images to **Docker Hub** with a **safe, opinionated tagging strategy**, **multi-platform support**, and **zero boilerplate**.

> This action is designed to prevent common release mistakes such as accidentally pushing `latest` for prereleases.

## ✨ Features

- 🧱 **Build once, push multiple tags**
- 🏷 **Smart SemVer-based tagging**
- 🧬 **Multi-platform images**

  - `linux/amd64`
  - `linux/arm64`

- 🔄 Supports versions without `v` prefix (`v1.2.3` → `1.2.3`)
- ⚡ Uses Docker **Buildx + QEMU**
- 🧩 Composite action (transparent & easy to audit)
- 📦 Docker Hub compatible
- 🧠 Fail-fast safety checks
- ⚡ GitHub Actions cache for Docker layers
- 🧪 Dry-run support
- 🧾 Automatic job summary
- 🚫 Strict tag safety guarantees

> [!IMPORTANT]
>
> ## CI/CD Runner Requirement
>
> Recommended Runner: `ubuntu-latest`
>
> This action must be executed on a Linux GitHub Actions runner.
>
> ```yaml
> runs-on: ubuntu-latest
> ```
>
> **Why Linux runners?**
>
> - Docker image builds require **Linux kernel features**.
> - GitHub-hosted **macOS** and **Windows runners do not provide Docker Desktop**.
> - Docker Desktop (used locally on macOS/Windows) cannot run inside CI runners.
> - Linux runners provide a **native Docker daemon** required by **Buildx**.
>
> **What this means**
>
> - Builds run on Linux CI runners.
> - Built images run on Linux, macOS, and Windows via Docker Desktop or WSL2.
> - macOS / Windows runners are not supported for building.
>
> This is the standard and recommended setup for Docker-based CI/CD workflows.

## 🚀 Quick Start

```yaml
- uses: teneplaysofficial/docker-publish@v2
  with:
    image_repo: tenedev/release-hub
    version: v1.2.4
    docker_username: ${{ secrets.DOCKERHUB_USERNAME }}
    docker_password: ${{ secrets.DOCKERHUB_TOKEN }}
```

## 🏷 Tagging Strategy (Important)

The action determines tags **only from the version string**. All tags are derived strictly and exclusively from the provided version. No tags are inferred from Git history, branches, or commit metadata.

### 🟢 Stable Release (no `-`)

**Example**

```text
1.2.3
v1.2.4
```

**Tags pushed**

```text
:1.2.4
:latest
:1
```

### 🔴 Numeric Prerelease → `next`

**Example**

```text
1.2.3-1
1.2.3-34
```

**Tags pushed**

```text
:1.2.3-34
:next
```

### 🟡 Labeled Prerelease → label tag

**Example**

```text
1.2.3-beta.2
1.2.3-alpha
1.2.3-rc.1
```

**Tags pushed**

```text
:1.2.3-beta.2
:beta
```

```text
:1.2.3-alpha
:alpha
```

```text
:1.2.3-rc.1
:rc
```

## 🔐 Tag Safety Rules

- `latest` is pushed only for stable releases
- Major tags (e.g. `:1`) are only for stable releases
- Prereleases can never overwrite stable tags
- Invalid tag strategies fail the workflow before push

> These rules are enforced automatically and cannot be disabled.

## 🧬 Multi-Platform Support

By default, images are built for:

```text
linux/amd64
linux/arm64
```

## Image runtime support

These images run on:

- Linux servers (native)
- macOS (Docker Desktop)
- Windows (Docker Desktop / WSL2)

## 🧪 Dry-Run Mode

When `dry_run: true`:

- Image is built.
- Tags are generated and validated.
- Multi-platform build runs.
- Images are not pushed.
- Registry state is untouched.

> Ideal for CI validation and release previews.

## ⚡ Docker Layer Caching

This action uses GitHub Actions cache for Docker layers.

**Benefits**

- Faster rebuilds
- No external cache registry
- Works automatically across workflow runs

> No configuration required.

## ⚙️ Inputs

| Name              | Required | Default        | Description                            |
| ----------------- | -------- | -------------- | -------------------------------------- |
| `image_repo`      | ✅       | —              | Docker image repo (`username/repo`)    |
| `version`         | ✅       | —              | App version (`1.2.3`, `v1.2.3-beta.2`) |
| `docker_username` | ✅       | —              | Docker Hub username                    |
| `docker_password` | ✅       | —              | Docker Hub token/password              |
| `context_path`    | ❌       | `.`            | Docker build context                   |
| `dockerfile_path` | ❌       | `./Dockerfile` | Path to Dockerfile                     |
| `dry_run`         | ❌       | `false`        | Build only, do not push images         |
| `summary`         | ❌       | `true`         | Generate job summary                   |

## 🛑 Fail-Fast Behavior

The workflow intentionally fails if:

- `image_repo` is not in `namespace/repo` format.
- No Docker tags are generated.
- A prerelease attempts to publish `latest`.
- Tag generation results in an empty set.
- Docker build fails for any platform.

> This prevents broken or unsafe releases.

## 🧪 Full Example Workflow

```yaml
name: Docker Release

on:
  push:
    tags:
      - "v*"

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: teneplaysofficial/docker-publish@v2
        with:
          image_repo: tenedev/release-hub
          version: ${{ github.ref_name }}
          docker_username: ${{ secrets.DOCKERHUB_USERNAME }}
          docker_password: ${{ secrets.DOCKERHUB_TOKEN }}
```

> The action automatically strips the leading `v` from Git tags.

## 🧾 Job Summary Output

When `summary: true`, the action publishes a job summary including:

- Image name.
- Normalized version.
- Release type.
- Published tags.
- Target platforms.
- Execution mode (publish / dry-run).

> This improves traceability and auditability.

### Sample Job Summary

Below is an example of what appears in the GitHub Actions → Job Summary panel:

```md
## Docker Publish Summary

Image: `tenedev/release-hub`  
Version: `1.2.4`  
Strategy: `stable`  
Mode: publish

### Tags

- `tenedev/release-hub:1.2.4`
- `tenedev/release-hub:latest`
- `tenedev/release-hub:1`

### Platforms

- linux/amd64
- linux/arm64
```

**Prerelease (Dry-Run) Example**

```md
## Docker Publish Summary

Image: `tenedev/release-hub`  
Version: `1.3.0-rc.1`  
Strategy: `labeled`  
Mode: dry-run

### Tags

- `tenedev/release-hub:1.3.0-rc.1`
- `tenedev/release-hub:rc`

### Platforms

- linux/amd64
- linux/arm64
```

## 🛡 Why This Action?

Most Docker workflows:

- Push `latest` accidentally.
- Rebuild per tag.
- Don’t support ARM.
- Copy-paste huge YAML blocks.

This action:

- Encodes **safe defaults**.
- Keeps workflows **short**.
- Follows **real SemVer rules**.
- Scales cleanly across projects.

## 🔍 Security & Transparency

- Uses **official Docker GitHub Actions**.
- Secrets used only for authentication.
- No secrets exposed to build steps.
- No bundled binaries.
- No Node.js runtime.
- No compiled artifacts.
- Fully auditable YAML + Bash.

## 🙌 Contributing

Issues and PRs are welcome.
This action is intentionally **small, focused, and predictable**.
