# Wind.Stock.ReaderRuleManager.Service

> Reader Rule（智读规则）管理服务，用于集中维护 Prompt、规则映射、规则条件和操作日志，并为 SmartRead、DataProcess 等业务系统提供统一的规则查询能力。

---

## 1. 项目简介

`Wind.Stock.ReaderRuleManager.Service` 是智读规则配置中心。系统负责将业务编码、规则条件和 Prompt 进行集中管理，支持后台页面维护、Excel 批量导入导出、外部系统查询、SSO 登录认证和操作日志审计。

服务默认配置：

| 项目 | 内容 |
|---|---|
| 应用名 | `Wind.Stock.ReaderRuleManager.Service` |
| Maven artifactId | `Wind.Stock.ReaderRuleManager.Service` |
| 默认端口 | `8090` |
| Context Path | `/readerRule` |
| 启动类 | `com.wind.stock.readerRule.ReaderRuleApplication` |
| 默认 profile | `${PROFILE:local}` |

---

## 2. 核心能力

- Prompt 管理：提示语分页查询、批量查询、新增、修改、删除、Excel 导入导出。
- Prompt 同步：从 PMP Prompt 平台同步 Prompt 名称、描述和内容。
- 规则管理：规则分页查询、新增、修改、删除、上下架、Excel 导入导出。
- 对外查询：为 SmartRead、DataProcess 等系统提供规则和 Prompt 查询接口。
- 数据同步：支持将原始映射规则同步到正式规则表。
- 登录认证：支持 SSO 扫码登录、回调验签、登录态缓存和登出。
- 操作日志：通过 AOP 记录关键管理操作，支持查询、导出、删除和定时清理。
- 静态页面：内置 SmartReadRuleManager 前端静态资源。

---

## 3. 技术栈

| 技术 | 用途 |
|---|---|
| Java 17 | 开发语言 |
| Spring Boot 3.4.0 | 应用框架 |
| Spring MVC | REST API |
| Spring AOP | 操作日志切面 |
| Spring Scheduling | 定时任务 |
| Spring Retry | 重试能力 |
| MyBatis / MyBatis-Plus | 数据访问 |
| PageHelper | 分页查询 |
| MySQL | 业务数据存储 |
| Druid | 数据库连接池 |
| Redis | 登录态、token 等缓存 |
| Apollo Client | 配置中心 |
| Knife4j / SpringDoc / Swagger | 接口文档 |
| Apache POI | Excel 导入导出 |
| Log4j2 | 日志框架 |
| Lombok | 简化 Java Bean |
| OkHttp | HTTP 调用 |

---

## 4. 项目结构

```text
Wind.Stock.ReaderRuleManager.Service
├── README.md
├── src
│   ├── pom.xml
│   ├── page
│   │   └── SmartReadRuleManager              # 前端页面构建产物
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── com/wind/stock/readerRule
│       │   │       ├── ReaderRuleApplication.java
│       │   │       ├── common                 # 公共配置、常量、认证、工具
│       │   │       └── rulemanage             # 规则管理业务模块
│       │   │           ├── controller         # REST 接口
│       │   │           ├── service            # 业务逻辑
│       │   │           ├── mapper             # MyBatis Mapper
│       │   │           ├── bean               # 请求/响应/实体对象
│       │   │           ├── aop                # 操作日志切面
│       │   │           ├── interceptor        # token 过滤器
│       │   │           ├── job                # 定时任务
│       │   │           └── utils              # 工具类
│       │   └── resources
│       │       ├── application*.yml           # 环境配置
│       │       ├── mapper                     # MyBatis XML
│       │       ├── sql                        # DDL/DML 脚本
│       │       ├── template                   # Excel 模板
│       │       ├── static                     # 静态页面资源
│       │       └── config                     # Redis / Expo / Eagle 配置
│       └── test
│           └── java                           # 测试代码
├── design
│   ├── api                                   # 接口文档
│   ├── architecture                          # 架构说明和架构图
│   └── storage                               # 数据库设计、SQL、原始数据
├── config
├── data
├── demo
├── deploy
└── lib
```

---

## 5. 核心模块说明

### 5.1 启动与基础配置

`ReaderRuleApplication` 是 Spring Boot 启动入口，启用了异步、重试、定时任务、Apollo 配置和 MyBatis Mapper 扫描。

关键配置：

- `server.port=8090`
- `server.servlet.context-path=/readerRule`
- `spring.profiles.active=${PROFILE:local}`
- `app.id=Wind.Stock.ReaderRuleManager.Service`

### 5.2 登录与权限

登录相关接口在 `SysLoginController` 中，主要包括：

- 生成 SSO 扫码地址。
- SSO 回调验签。
- 缓存登录用户和 token。
- 查询登录状态。
- 登出。

`TokenFilter` 负责拦截需要登录态的请求。部分对外服务接口和静态资源被配置为排除路径，例如 `/ruleManage/getRuleManageList`、`/ruleManage/getPromptList`、同步接口、导入导出接口、Swagger 文档等。

### 5.3 Prompt 管理

`MappingPromptService` 负责 Prompt 数据维护，核心数据表为 `tb_mapping_prompt`。

主要能力：

- Prompt 分页查询和批量查询。
- Prompt 新增、修改、删除。
- Prompt Excel 导入导出。
- 从 PMP 平台同步 Prompt 描述。
- 查询 Prompt 相关下拉选项。

### 5.4 规则管理

`RuleManageService` 负责规则主表和规则条件子表维护，核心数据表为 `tb_rule_manage` 和 `tb_rule_manage_sub`。

主要能力：

- 规则分页查询。
- 对外规则查询。
- 规则新增、修改、删除。
- 规则上下架。
- 规则 Excel 导入导出。
- 从映射规则表同步规则。

规则保存和修改时会校验 `promptId` 是否存在于 Prompt 映射表中。

### 5.5 操作日志

`LogAspect` 通过 `@OperateLogAop` 记录关键管理操作，`OperateLogService` 提供日志查询、导出和删除能力，核心数据表为 `sys_operate_log`。

定时任务 `TaskJob` 每天 1 点清理 30 天前的操作日志。

### 5.6 Excel 导入导出

系统使用 Apache POI 处理 Excel 文件，模板位于：

- `src/src/main/resources/template/prompt_template.xlsx`
- `src/src/main/resources/template/rule_manage.xlsx`
- `src/src/main/resources/template/operateLog_template.xlsx`

导入接口使用 `multipart/form-data` 上传文件。

---

## 6. 主要接口分组

完整字段、请求示例和响应示例见 `design/api/规则管理-接口文档_整理版.md`。

### 6.1 登录与认证

| 接口 | 用途 |
|---|---|
| `GET /` | 跳转 SSO 登录 |
| `GET /generateSSO` | 生成 SSO 扫码地址 |
| `GET /auth/ssoCallBack` | SSO 回调处理 |
| `POST /login` | 查询登录状态 |
| `GET /loginOut` | 登出 |
| `GET /health` | 健康检查 |

### 6.2 Prompt 管理

| 接口 | 用途 |
|---|---|
| `POST /ruleManage/promptPageList` | Prompt 分页查询 |
| `POST /ruleManage/promptBatchList` | Prompt 批量查询 |
| `POST /ruleManage/getPromptList` | 对外查询 Prompt |
| `POST /ruleManage/promptSave` | 新增 Prompt |
| `POST /ruleManage/promptUpdate` | 修改 Prompt |
| `POST /ruleManage/promptDelete` | 删除 Prompt |
| `POST /ruleManage/exportPrompt` | 导出 Prompt |
| `POST /ruleManage/importPrompt` | 导入 Prompt |
| `POST /ruleManage/syncPromptDesc` | 同步 Prompt 描述 |
| `POST /ruleManage/getPromptDescByIds` | 按 Prompt ID 查询描述 |
| `GET /ruleManage/getComboEntity` | 查询下拉选项 |

### 6.3 规则管理

| 接口 | 用途 |
|---|---|
| `POST /ruleManage/getRuleManagePageList` | 规则分页查询 |
| `POST /ruleManage/getRuleManageList` | 对外查询规则 |
| `POST /ruleManage/ruleManageSave` | 新增规则 |
| `POST /ruleManage/ruleManageUpdate` | 修改规则 |
| `POST /ruleManage/ruleManageDelete` | 删除规则 |
| `POST /ruleManage/ruleManageUpdateStatus` | 规则上下架 |
| `POST /ruleManage/exportRuleManage` | 导出规则 |
| `POST /ruleManage/importRuleManage` | 导入规则 |
| `POST /ruleManage/syncRuleManage` | 同步规则 |

### 6.4 操作日志

| 接口 | 用途 |
|---|---|
| `POST /log/pageList` | 操作日志分页查询 |
| `GET /log/menus` | 查询操作菜单 |
| `GET /log/operateAction` | 查询操作类型 |
| `POST /log/exportLog` | 导出操作日志 |
| `POST /log/delete` | 删除操作日志 |

---

## 7. 数据存储

数据库设计见 `design/storage/MySql 数据存储设计方案.md`，初始化脚本见：

- `design/storage/SQL脚本/DDL_20260709.sql`
- `design/storage/SQL脚本/DML_20260709.sql`
- `src/src/main/resources/sql/DDL_20260709.sql`
- `src/src/main/resources/sql/DML_20260709.sql`

核心表：

| 表名 | 说明 |
|---|---|
| `tb_mapping_prompt` | Prompt 映射表 |
| `tb_rule_manage` | 规则主表 |
| `tb_rule_manage_sub` | 规则条件子表 |
| `tb_mapping_rule_manage` | 原始映射规则表 |
| `sys_operate_log` | 操作日志表 |

---

## 8. 外部依赖

| 依赖 | 说明 |
|---|---|
| MySQL | 规则、Prompt 和日志数据 |
| Redis | 登录态和 token 缓存 |
| Apollo | 应用配置中心 |
| SSO | 管理端登录认证 |
| PMP Prompt 平台 | Prompt 描述同步来源 |
| SmartRead / DataProcess | 规则和 Prompt 查询调用方 |

---

## 9. 本地启动

进入 Maven 工程目录：

```bash
cd src
```

编译：

```bash
mvn clean package
```

本地启动：

```bash
mvn spring-boot:run -Dspring-boot.run.profiles=local
```

或使用环境变量指定 profile：

```bash
set PROFILE=local
mvn spring-boot:run
```

启动后默认访问：

| 地址 | 说明 |
|---|---|
| `http://localhost:8090/readerRule/health` | 健康检查 |
| `http://localhost:8090/readerRule/index.html` | 管理页面 |
| `http://localhost:8090/readerRule/swagger-ui.html` | Swagger UI |
| `http://localhost:8090/readerRule/doc.html` | Knife4j 文档 |

本地能否完整启动取决于 MySQL、Redis、Apollo、SSO 等环境配置是否可用。

---

## 10. 配置说明

主要配置文件：

| 文件 | 说明 |
|---|---|
| `src/src/main/resources/application.yml` | 基础配置，包含端口、context-path、profile |
| `src/src/main/resources/application-local.yml` | local 环境配置 |
| `src/src/main/resources/application-test.yml` | test 环境配置 |
| `src/src/main/resources/application-trial.yml` | trial 环境配置 |
| `src/src/main/resources/application-njprod.yml` | 南京生产环境配置 |
| `src/src/main/resources/application-shprod.yml` | 上海生产环境配置 |
| `src/src/main/resources/application.properties` | Apollo bootstrap 和静态资源配置 |

配置项关注点：

- `spring.datasource.*`：MySQL 连接配置。
- `redis.*` / `wind.redis.*`：Redis 配置。
- `apollo.*`：Apollo 地址和命名空间配置。
- `sso.*`：SSO 登录、回调、token 和白名单配置。
- `springdoc.*` / `knife4j.*`：接口文档配置。
- `spring.servlet.multipart.*`：Excel 上传大小限制。

### 10.1 MySQL 配置

MySQL 连接配置位于各环境 `application-*.yml` 的 `spring.datasource` 节点。密码属于敏感信息，README 中不展开明文，实际值以对应环境配置或 Apollo 配置为准。

| 环境 | 配置文件 | JDBC URL | 用户名 | 驱动 |
|---|---|---|---|---|
| local | `application-local.yml` | `jdbc:mysql://10.102.16.65:3306/wind_alicereader?useUnicode=true&characterEncoding=utf-8&autoReconnect=true` | `root` | `com.mysql.cj.jdbc.Driver` |
| test | `application-test.yml` | `jdbc:mysql://10.26.31.56:3306/alicereader_db?useUnicode=true&characterEncoding=utf-8&autoReconnect=true` | `rchost_app` | `com.mysql.cj.jdbc.Driver` |
| trial | `application-trial.yml` | `jdbc:mysql://10.10.14.47:3306/alicereader_db?useUnicode=true&characterEncoding=utf-8&autoReconnect=true` | `rchost_app` | `com.mysql.cj.jdbc.Driver` |
| njprod | `application-njprod.yml` | `jdbc:mysql://10.10.14.47:3306/alicereader_db?useUnicode=true&characterEncoding=utf-8&autoReconnect=true` | `rchost_app` | `com.mysql.cj.jdbc.Driver` |
| shprod | `application-shprod.yml` | `jdbc:mysql://10.10.14.47:3306/alicereader_db?useUnicode=true&characterEncoding=utf-8&autoReconnect=true` | `rchost_app` | `com.mysql.cj.jdbc.Driver` |

相关实现：

- 数据源连接池使用 Druid。
- `AppDataSource` 在非 dev 环境创建物理连接时会尝试通过密码服务刷新数据库密码。
- MyBatis Mapper XML 位于 `src/src/main/resources/mapper`。

### 10.2 Redis 配置

Redis 配置在不同环境中存在两种写法：

- `redis.*`：local、test、trial、njprod、shprod 等环境使用。
- `wind.redis.*`：dev 环境使用。

| 环境 | 配置文件 | Redis 节点 | AppId | 账号/数据源 | 客户端名 | namespace |
|---|---|---|---|---|---|---|
| local | `application-local.yml` | `10.102.16.126:6379;10.102.16.127:6379;10.102.16.129:6379` | `2267` | `dev_redis` | `DocChatService` | `airesearch` |
| test | `application-test.yml` | `10.26.73.32:7001;10.26.73.33:7001;10.26.73.34:7001` | `2267` | `comm_redis` | `DocChatService` | `airesearch` |
| trial | `application-trial.yml` | `10.10.159.98:7000...10.10.159.152:7003` | `2267` | `comm_redis` | `DocChatService` | `airesearch` |
| njprod | `application-njprod.yml` | `10.10.159.98:7000...10.10.159.152:7003` | `2267` | `comm_redis` | `DocChatService` | `airesearch` |
| shprod | `application-shprod.yml` | `10.10.159.98:7000...10.10.159.152:7003` | `2267` | `comm_redis` | `DocChatService` | `airesearch` |

其他 Redis 相关配置：

- `maxConns`：连接数上限，当前多数环境为 `16`。
- `define.filePath` / `wind.redis.redis-define.path`：Redis 定义文件路径，通常指向 `config/redis/RedisDefine.xml`。
- Redis 主要用于 SSO 登录态、token 和用户会话缓存。

### 10.3 敏感配置说明

以下配置在本地 yml 中可能存在明文值，提交文档或对外传递时应避免展开：

- `spring.datasource.password`
- `apollo.accesskey.secret`
- `sso.public-key` / `sso.private-key`
- SSO 默认账号密码、管理员账号密码

---

## 11. 文档索引

| 文档 | 说明 |
|---|---|
| `design/architecture/业务架构说明.md` | 面向业务、产品、测试的架构说明 |
| `design/architecture/架构图-规则管理2.png` | 规则管理架构图 |
| `design/api/规则管理-接口文档_整理版.md` | 接口文档 |
| `design/storage/MySql 数据存储设计方案.md` | MySQL 数据存储设计 |
| `design/storage/SQL脚本/DDL_20260709.sql` | DDL 脚本 |
| `design/storage/SQL脚本/DML_20260709.sql` | DML 脚本 |

---

## 12. 测试与验证

执行测试：

```bash
cd src
mvn test
```

建议重点验证：

- Prompt 新增、修改、删除、导入导出。
- 规则新增、修改、删除、上下架、导入导出。
- 规则查询接口是否只返回上架规则。
- `/ruleManage/getRuleManageList`、`/ruleManage/getPromptList` 等对外接口的免登录访问行为。
- 规则保存时 `promptId` 不存在的业务校验。
- Prompt 被规则引用时的删除限制。
- 操作日志是否按菜单、动作、状态、时间范围正确查询和导出。
- SSO 登录、回调验签、登录状态查询和登出。
