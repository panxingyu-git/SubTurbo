# SubTurbo 自用部署说明

本目录是官方 SubBoost 源码。目标是部署一套仅供个人使用的订阅转换和聚合服务。

## 推荐方式

在 NAS 上使用 Docker Compose 的官方多架构镜像部署，不需要在 NAS 上安装 Node.js 或编译源码。

项目已提供：

- `local/docker-compose.image.yml`：PostgreSQL、应用和定时更新容器
- `local/local.env.example`：生产环境变量模板
- 官方安装脚本：会生成密钥、拉取镜像并启动服务

## 开始前准备

1. NAS 支持 Docker 和 Docker Compose，并能访问 GitHub Container Registry（GHCR）。
2. NAS 至少准备一个持久化存储位置；数据库卷不能放在临时目录。
3. 为服务预留一个局域网端口，例如 `3000`。
4. 准备强管理员密码，并准备保存数据库密码、加密密钥、JWT 密钥和定时任务密钥。
5. 只导入你有权使用的订阅链接。订阅链接等同于密码，不要提交到 Git 或发给别人。

官方镜像提供 `linux/amd64` 和 `linux/arm64` 架构，常见 Intel/AMD 和 ARM NAS 都可以使用。

## NAS 部署

官方一键脚本适合 Linux SSH 终端。进入 NAS 上的部署目录后执行：

```bash
curl -fsSL https://github.com/SubBoost/subboost/releases/latest/download/install.sh | bash
```

脚本会询问访问地址和端口，并在默认目录 `/opt/subboost` 生成配置。完成后使用输出的首次初始化链接创建管理员。

如果 NAS 使用图形化容器管理器，也可以使用 `local/docker-compose.image.yml`，复制 `local/local.env.example` 为 `.env`，填写全部必需变量，再执行：

```bash
docker compose --env-file .env -f local/docker-compose.image.yml up -d
```

其中 `SUBBOOST_IMAGE` 可使用 `ghcr.io/subboost/subboost:latest`，也可以固定到具体 release 版本。`DATABASE_URL` 的主机名应为 Compose 服务名 `db`，而不是 `localhost`。

公网访问时必须通过反向代理配置 HTTPS；个人自用建议先只绑定局域网，不开放端口到公网。

## 备份与维护

- 数据库和 `.env` 必须一起备份；缺少 `.env` 中的 `ENCRYPTION_KEY` 将无法解密已保存的订阅。
- 安装脚本部署后可使用 `subboost status`、`subboost doctor`、`subboost logs`、`subboost backup` 和 `subboost update`。
- 升级前先备份，升级后检查登录、订阅刷新和生成的配置链接。

## 许可证

个人自用不需要把修改发布到公网。仍需保留仓库中的 `LICENSE` 和版权信息；如果以后让其他人通过网络使用修改后的版本，再按 AGPL-3.0 提供对应源码。
