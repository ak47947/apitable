# APITable 本地开发环境启动指南

本文档整理了在 Mac 本地环境下，完整启动并运行 APITable 整个技术栈项目的详细流程。

## 1. 基础环境准备
请确保您的电脑已经安装以下基础设施：
- **Node.js**: `v16.15.0` (推荐使用 nvm 进行管理)
- **Java**: JDK 17 及以上（配合 Gradle 编译后端）
- **pnpm**: `v8.5.5` (使用时需注意与 Node 22+ Corepack 的冲突问题)
- **Docker** 桌面客户端或 OrbStack (用于运行数据库等中间件)

切换至正确的 Node 和 pnpm 版本环境：
```bash
nvm use 16.15.0
npm install -g pnpm@8.5.5
```

> [!WARNING]
> 如果您之前全局开启了 `corepack` 并尝试使用 Node.js 22 等高版本安装依赖，可能会由于核心插件不兼容导致由于 `URL.canParse` 报错，请执行 `corepack disable` 后再通过 npm 安装独立的 pnpm 8 版本。

## 2. 进行项目的依赖安装与全量构建
项目根目录下，使用构建命令全量安装和打包整个 Monorepo 体系（前后端语言包、组件等）：
```bash
make install
```
*此过程会同步下载 Java 依赖和通过 NX/PNPM 跨工作区联动构建 NPM 制品，需要耐心等待1~5分钟。*

## 3. 启动底层依赖中间件并注入数据表
依赖于在 [.env](file:///Users/lcw/work/code/apitable/.env) 中定义的配置信息，启动本地的 MySQL, Redis, RabbitMQ 与 MinIO 服务。

```bash
make dataenv
```
执行完毕后，可通过 `make dataenv-ps` 或 Docker 客户端查看五个核心容器是否正常 Running 或已结束 (init-db 会执行并自己结束出场)。在这个过程中，Liquibase（init-db） 会自动执行 SQL 文件完成数据库表结构的构建。MySQL 容器会按 `MYSQL_DATABASE` 创建数据库（若不存在），init-db 负责表结构与初始化数据的写入。

## 4. 启动核心应用服务栈
在您确保 **第 2，3 步** 执行无误并且中间件容器健康运行后，您需要新开 **3个独立终端界面** 分别启动以下核心服务。由于每个服务都支持热重载（Watch），他们将处于一直监听执行状态。

**终端 1：启动主后端 API 服务 (backend-server)**
```bash
make _run-local-backend-server
```

**终端 2：启动实时通讯协同服务 (room-server)**
必须保证在此 Node 生命周期内使用的是 `v16.15.0`:
```bash
source ~/.nvm/nvm.sh && nvm use 16.15.0 
make _run-local-room-server
```

**终端 3：启动前端界面服务 (web-server/Datasheet)**
必须保证在此 Node 生命周期内使用的是 `v16.15.0`:
```bash
source ~/.nvm/nvm.sh && nvm use 16.15.0 
make _run-local-web-server
```

## 5. 访问运行系统

当终端 3 的前端编译服务输出如 `> Ready on http://localhost:3000` 时，表明整个框架已可用。
接下来前往浏览器打开以下本地地址：

```
http://localhost:3000
```

> [!TIP]
> 开发者模式下，Next.js 初次访问时可能需数十秒实时编译页面 Bundle，此时浏览器可能会无响应或加载中，请耐心等待直到 UI 呈现即可。可以查看控制台的 webpack 输出监控进度。

## 6. 关闭开发环境

当您完成开发并希望关闭本地环境时，由于服务涉及到前端、后台不同进程以及 Docker 容器，请按以下步骤关闭系统以免造成端口占用：

1. **终止核心应用服务**：在前面启动了服务（backend-server、room-server、web-server）的各个终端界面中，按下 `Ctrl + C` 退出并终止对应的 Node.js 和 Java 监听进程。
2. **关闭依赖中间件容器**：在项目根目录下新开一个终端，运行以下命令关闭相关的开发依赖容器（这会安全地关闭 MySQL、Redis、MinIO、RabbitMQ，不会删除数据卷）：
   ```bash
   make dataenv-down
   ```
