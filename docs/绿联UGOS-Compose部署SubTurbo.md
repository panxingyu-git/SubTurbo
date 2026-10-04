# 绿联 UGOS Pro 使用 Docker Compose 部署 SubTurbo

适用设备：绿联 NAS，地址 `192.168.50.216`，项目目录 `/volume1/docker/subboost`。

这份说明按 UGOS Pro 常见界面编写。系统升级后菜单可能显示为“Docker 管理”“容器项目”或“项目”。

## 先确认一个关键点

仓库里的品牌修改只存在于源码中。Compose 如果填写官方镜像 `ghcr.io/subboost/subboost:latest`，启动后仍会显示官方品牌，不会显示 `SubTurbo`。

要使用 `SubTurbo` 品牌，需要先把这个仓库构建成镜像，再在 Compose 的 `SUBBOOST_IMAGE` 中填写你自己的镜像地址。最省事的做法是使用 GitHub Actions 构建并推送到 GHCR；如果暂时只想验证数据库和网络，也可以先用官方镜像测试。

## 一、准备目录和文件

在绿联文件管理器中打开：

```text
/volume1/docker/subboost
```

在这里放置两个文件：

```text
docker-compose.yml
.env
```

不要把 `.env` 上传到 GitHub。它包含数据库密码和加密密钥。

## 二、UGOS 界面创建项目

1. 打开 UGOS，进入 **Docker**。
2. 打开 **项目**（有些版本叫“容器项目”）。
3. 点击 **创建项目**。
4. 项目名称填写 `subboost`。
5. 项目路径选择 `/volume1/docker/subboost`。
6. 选择 **编辑 Compose** 或 **从文件导入**，使用项目中的 `local/docker-compose.image.yml` 内容。
7. 在项目环境变量区域导入 `.env`，或者直接在项目目录创建 `.env` 后选择读取环境变量文件。
8. 保存后点击 **部署/启动**。

如果界面同时要求选择 Compose 文件，使用 `docker-compose.image.yml`，不要选择开发用的 `docker-compose.yml`；前者直接拉取镜像，不会在 NAS 上编译 Node.js 项目。

## 三、`.env` 填写示例

下面的值可以直接作为格式参考。请自行替换所有密码和随机密钥：

```dotenv
SUBBOOST_IMAGE=ghcr.io/subboost/subboost:latest
SUBBOOST_PORT=3002

POSTGRES_DB=subboost
POSTGRES_USER=subboost
POSTGRES_PASSWORD=改成一串只含字母和数字的强密码
DATABASE_URL=postgresql://subboost:改成一串只含字母和数字的强密码@db:5432/subboost

ENCRYPTION_KEY=至少32字节的随机字符串
JWT_SECRET=至少32字节的随机字符串
CRON_SECRET=至少32字节的随机字符串
LOCAL_SETUP_TOKEN=至少32字节的随机字符串

TRUST_PROXY_HEADERS=false
APP_URL=http://192.168.50.216:3002
```

密码中先不要使用 `@`、`#`、`:`、`/` 等字符，否则 `DATABASE_URL` 需要额外进行 URL 编码。`DATABASE_URL` 中的主机名必须是 `db`，不能填 `localhost`。

如果使用自建的 SubTurbo 镜像，只需要把 `SUBBOOST_IMAGE` 改成自己的镜像地址，例如：

```dotenv
SUBBOOST_IMAGE=ghcr.io/panxingyu-git/subturbo:latest
```

GHCR 私有镜像需要在 NAS 的 Docker 设置中先登录 GitHub Container Registry；公开镜像不需要登录。

## 四、部署后检查

在 UGOS 的项目详情中确认三个容器都为运行状态：

- `db`：数据库
- `app`：SubTurbo 网页服务
- `cron`：定时刷新任务

浏览器打开：

```text
http://192.168.50.216:3002
```

第一次打开时使用 `LOCAL_SETUP_TOKEN` 完成本地管理员初始化，然后设置管理员账号和密码。初始化完成后不要再把 setup token 发给别人。

如果 `app` 反复重启，先查看项目日志，重点检查 `DATABASE_URL`、四个密钥以及 `db` 是否已经变成运行状态。首次启动数据库可能需要几十秒。

## 五、升级和回滚

在 UGOS 项目页面点击 **停止** 后，再替换 Compose 文件或 `.env`，然后点击 **重新部署/启动**。不要删除 `db` 容器对应的持久化卷，否则会删除订阅和账号数据。

升级前备份：

- `.env`
- Docker 卷 `subboost-local-db`

公网访问时，把 `3002` 交给带 HTTPS 的反向代理，并将 `APP_URL` 改为实际的 HTTPS 地址。不要直接把 NAS 的 `3002` 端口裸露到公网。

## 参考

- 绿联 UGOS Pro Docker/Compose 界面示例（第三方图文）：<https://zhuanlan.zhihu.com/p/708706484>
- 绿联官方支持中心：<https://www.ugnas.com/support>

