# Docker Build Images

这个仓库提供两个 GitHub Actions 手动工作流，用来构建并导出离线 Docker 镜像包。

保留文件：

- `.github/workflows/docker-single.yml`
- `.github/workflows/docker-combo.yml`
- `Dockerfile.single`
- `Dockerfile.combo`

## 工作流入口

### Docker Build - Single

基于一个完整镜像继续安装可选依赖。适合直接构建：

- `node:22-alpine`
- `node:22-bookworm-slim`
- `python:3.12-slim`
- `python:3.12-alpine`
- `python:3.13-slim`
- 任意自定义镜像

常用填写：

- `single_image_preset`: 选择常用镜像，或选 `custom`
- `single_image`: 自定义镜像名；填写后会覆盖 preset
- `package_profile`: 默认 `auto`
- `python_packages`: 需要额外安装的 pip 包
- `node_packages`: 需要额外安装的 npm 全局包
- `install_packages`: 需要额外安装的系统包
- `build_arm64`: 默认关闭；开启后同时打包 `linux/arm64`

### Docker Build - Combo

选择基础 Linux 镜像和最终运行时镜像。最终安装依赖时使用运行时镜像自己的包管理器，避免把 Alpine、Debian、RHEL 系文件系统硬拷贝到一起导致 ABI/动态库问题。

常用填写：

- `base_image_preset` / `base_image`: 基础镜像信息
- `runtime_image_preset` / `runtime_image`: 最终运行时镜像
- `package_profile`: 默认 `auto`
- `install_packages`: 需要额外安装的系统包
- `build_arm64`: 默认关闭；开启后同时打包 `linux/arm64`

## 自动生成输出名称

`image_name`、`image_tag`、`release_tag` 都可以留空。

Single 模式会从 `single_image` 自动推导：

- `node:22-alpine` -> `image_name=node`、`image_tag=22-alpine`、`release_tag=node-22-alpine`
- `python:3.12-slim` -> `image_name=python`、`image_tag=3.12-slim`、`release_tag=python-3.12-slim`
- `docker.elastic.co/elasticsearch/elasticsearch:8.14.3` -> `image_name=elasticsearch`、`image_tag=8.14.3`

Combo 模式会从 `base_image + runtime_image` 自动推导：

- `ubuntu:24.04 + python:3.12-slim` -> `image_name=ubuntu-python`、`image_tag=24.04-3.12-slim`
- `alpine:3.20 + node:22-alpine` -> `image_name=alpine-node`、`image_tag=3.20-22-alpine`

GitHub Actions 原生表单不会在选择镜像后把输入框视觉上改成这些值；这些值是在工作流运行时计算出来的。需要自定义时，直接填写对应输入即可覆盖自动值。

## package_profile

- `auto`: 默认值；根据 `python_packages` / `node_packages` 自动安装常见编译依赖
- `none`: 不安装系统预设包
- `runtime`: 安装 `ca-certificates`、`curl`、`bash`
- `python-build`: 安装 Python 原生扩展常见编译依赖
- `node-build`: 安装 Node 原生扩展常见编译依赖
- `python-node-build`: 同时安装 Python 和 Node 编译依赖

如果 `python:3.12+` 安装 pip 包失败，优先保持 `package_profile=auto`，或者手动选 `python-build`。如果 `node:22-alpine` 安装 npm 包失败，手动选 `node-build`。

## 输出

工作流会按平台生成并上传：

- GitHub Release 附件：`*.tar.gz`
- GitHub Actions Artifact：`docker-images-<image_tag>`

默认只构建 `linux/amd64`。开启 `build_arm64` 后，会同时构建 `linux/amd64` 和 `linux/arm64`。

平台标签会追加到镜像 tag 后面，例如：

- `node:22-alpine-amd64`
- `node:22-alpine-arm64`

离线导入：

```bash
gzip -d node-22-alpine-amd64.tar.gz
docker load -i node-22-alpine-amd64.tar
```
