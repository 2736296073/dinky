# Dinky 自定义开发指南

## 项目配置

### Git Remote 配置

本项目已配置 Fork + Upstream Remote 方案：

```
origin   → https://github.com/2736296073/dinky.git (你的 Fork)
upstream → https://github.com/DataLinkDC/dinky.git (原始仓库)
当前分支 → custom-dev
```

### 分支结构

```
upstream/dev (社区最新代码)
     ↓ 同步
origin/dev (你的 Fork 主分支，保持与上游同步)
     ↓ 
origin/custom-dev (你的自定义开发分支)
```

---

## 自定义开发策略

### 推荐方案：模块化扩展

创建独立的自定义模块，最小化对原有代码的修改：

```
dinky/
├── dinky-core/
├── dinky-admin/
├── dinky-web/
├── dinky-custom/           # 新增：自定义模块
│   ├── pom.xml
│   └── src/main/java/org/dinky/custom/
│       ├── function/       # 自定义函数
│       ├── connector/      # 自定义连接器
│       ├── alert/          # 自定义告警
│       └── service/        # 自定义服务扩展
└── pom.xml                 # 添加模块
```

**优点：**
- 合并上游代码冲突最少
- 可以独立打包部署
- 升级方便

---

## 日常开发流程

### 同步上游代码

```bash
# 1. 同步上游最新代码
git fetch upstream

# 2. 更新 dev 分支（保持与上游同步）
git checkout dev
git merge upstream/dev
git push origin dev

# 3. 合并到自定义分支
git checkout custom-dev
git merge dev
```

### 开发与提交

```bash
# 4. 开发自定义功能...
git add .
git commit -m "[custom] 添加xxx功能"
git push origin custom-dev
```

---

## Docker 本地部署

### 启动服务

```bash
cd deploy/docker
docker-compose -f docker-compose.dev.yml up -d
```

### 服务地址

| 服务 | 地址 |
|------|------|
| Dinky Web UI | http://localhost:8888 |
| Flink JobManager | http://localhost:8081 |

### 默认登录

- 用户名: `admin`
- 密码: `admin`

### 常用命令

```bash
# 查看服务状态
docker-compose -f docker-compose.dev.yml ps

# 查看日志
docker logs docker-dinky-1

# 停止服务
docker-compose -f docker-compose.dev.yml down

# 重启服务
docker-compose -f docker-compose.dev.yml restart
```

---

## 已解决的问题

### 1. Flink TaskManager 无法连接 JobManager

**原因：** 使用 host 网络模式时，容器无法通过名称互相访问。

**解决：** 修改 `.env` 中的 `FLINK_PROPERTIES`：
```properties
FLINK_PROPERTIES="jobmanager.rpc.address: localhost"
```

### 2. Flink SQL 客户端无法连接

**原因：** 缺少必要的依赖包。

**解决：** 添加以下 jar 包到 Dinky lib 目录：
- `jline-3.21.0.jar`
- `flink-sql-gateway-1.17.2.jar`
- `flink-sql-client-1.17.2.jar`

### 3. S3/OSS 存储配置导致服务崩溃

**原因：** 配置了 S3/OSS 存储但缺少相关依赖。

**解决：** 如需使用 S3/OSS，添加 `flink-s3-fs-presto` 等依赖；或使用本地存储。

---

## 编译项目

### 环境要求

- Java 8 或 Java 11
- Maven 3.8+
- Node.js 18+
- npm 10+

### 编译命令

使用 Java 8 编译（推荐，与 Flink 1.17 镜像兼容）：

```bash
export JAVA_HOME=~/.sdkman/candidates/java/8.0.422-zulu
mvn clean install -DskipTests -P prod,fast,flink-single-version,aliyun,flink-1.17,web
```

使用 Java 11 编译：

```bash
export JAVA_HOME=~/.sdkman/candidates/java/11.0.19-zulu
mvn clean install -DskipTests -P prod,jdk11,flink-single-version,aliyun,flink-1.17,web
```

编译结果位于 `build/dinky-release-1.17-1.3.0-SNAPSHOT.tar.gz`

---

## Profile 说明

| Profile | 说明 |
|---------|------|
| `prod` | 生产环境打包 |
| `fast` | 跳过代码检查（JDK 8 必须勾选） |
| `jdk11` | 使用 JDK 11 |
| `flink-single-version` | 单版本打包 |
| `aliyun` | 使用阿里云 Maven 镜像加速 |
| `flink-1.17` | 指定 Flink 1.17 版本 |
| `web` | 包含前端资源打包 |
| `npm-huawei` | 使用华为 npm 镜像 |
| `npm-taobao` | 使用淘宝 npm 镜像 |