# Universal Docker Workflow

这个目录提供一个可在 GitHub Actions 中调用的动态 `Dockerfile`，用于组合：

- Linux 基础镜像：`ubuntu`、`debian`、`alpine`、`rockylinux`、`almalinux`、`centos`
- 运行时/服务镜像：`python`、`node`、`elasticsearch`、`meilisearch`

工作流文件：

- `.github/workflows/universal-docker-matrix.yml`

动态 Dockerfile：

- `docker/Dockerfile.dynamic`

支持的典型组合示例：

- `ubuntu:22.04 + python:3.11-slim`
- `debian:12 + node:20-bookworm-slim`
- `rockylinux:9 + getmeili/meilisearch:v1.11`
- `ubuntu:24.04 + docker.elastic.co/elasticsearch/elasticsearch:8.14.3`

说明：

- `base_image` 用于准备系统层通用依赖。
- `runtime_image` 用于决定最终运行时环境。
- `install_packages` 可安装系统包。
- `python_packages` 可安装 pip 包。
- `node_packages` 可安装 npm 全局包。

注意：

- 某些服务镜像（如 `elasticsearch`）自身有固定入口与目录结构，虽然可叠加基础层文件，但不建议再安装太多运行时依赖。
- `alpine` 与 `glibc` 生态镜像组合时，部分二进制依赖可能不兼容，建议优先选 `debian/ubuntu` 作为通用基础镜像。