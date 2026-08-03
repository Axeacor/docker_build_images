# Universal Docker Build

这个仓库提供一个 GitHub Actions 手动工作流，用来构建并导出离线 Docker 镜像包。

保留文件：

- `.github/workflows/universal-docker-matrix.yml`
- `Dockerfile.single`
- `Dockerfile.combo`

## 构建模式

### single

基于一个完整镜像继续安装可选依赖，适合直接构建：

- `node:22-alpine`
- `node:22-bookworm-slim`
- `python:3.12-slim`
- `python:3.12-alpine`
- `python:3.13-slim`
- 任意自定义镜像

常用填写：

- `build_mode`: `single`
- `single_image_preset`: 选择常用镜像，或选 `custom`
- `single_image`: 自定义镜像名；填写后会覆盖 preset
- `package_profile`: 默认 `auto`
- `python_packages`: 需要额外安装的 pip 包
- `node_packages`: 需要额外安装的 npm 全局包
- `install_packages`: 需要额外安装的系统包

### combo

以一个运行时镜像作为最终镜像，并记录一个基础 Linux 镜像来源。最终安装依赖时使用运行时镜像自己的包管理器，避免把 Alpine、Debian、RHEL 系文件系统硬拷贝到一起导致 ABI/动态库问题。

常用填写：

- `build_mode`: `combo`
- `base_image_preset` / `base_image`: 基础镜像信息
- `runtime_image_preset` / `runtime_image`: 最终运行时镜像

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

平台标签会追加到镜像 tag 后面，例如：

- `universal-runtime:latest-amd64`
- `universal-runtime:latest-arm64`

离线导入：

```bash
gzip -d universal-runtime-latest-amd64.tar.gz
docker load -i universal-runtime-latest-amd64.tar
```
