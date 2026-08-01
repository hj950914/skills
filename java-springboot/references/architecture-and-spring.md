# 架构与 Spring 规范

## 目录

- 项目结构与分层
- 模块拆分默认结构
- 构建与依赖
- 依赖注入与 Spring Bean
- 配置管理
- Web 层

## 项目结构与分层

- 除非用户明确说出“重新拆分项目结构”，否则保持现有模块结构，不因本规范或普通重构任务改造成另一种架构。
- 让 Controller 负责协议转换、参数校验、身份上下文和响应组装，不承载核心业务规则。
- 让应用服务负责用例编排、事务边界和跨领域协作。
- 将核心业务规则优先放在领域对象、领域服务或职责明确的业务组件中。
- 让基础设施层负责数据库、缓存、消息、文件和第三方服务的具体实现。
- 只把真正跨业务复用的稳定能力放入公共模块，避免形成无关工具和常量的垃圾桶。
- 禁止为了“统一风格”将所有业务逻辑堆进一个 `@Service` 类。

## 模块拆分默认结构

只有用户明确说出“重新拆分项目结构”时，才启用本节规则。用户没有使用这句话时，不得主动新增、删除、重命名或重新划分 Maven/Gradle 模块；“拆分模块”“重构代码”“调整依赖”“整理包结构”等表述均不触发本节。

触发本节且用户没有指定其他目标结构时，默认创建以下五个模块，不自行简化为 `api/service/server`：

```text
project-root/
├── app-api/
├── app-biz/
├── app-bootstrap/
├── app-domain/
└── app-infra/
```

除非用户明确要求精简层级，否则保留这五个模块；不要因为当前代码较少就简化为含义模糊的 `service`、`server` 或 `core`。

### 模块职责

- `app-api`：Controller、协议转换、参数校验、请求响应 DTO、接口适配和对外契约。
- `app-biz`：应用服务、用例编排、事务边界、权限协调和跨领域协作。
- `app-domain`：领域模型、值对象、业务规则、领域服务以及由内层声明的端口或仓储接口。
- `app-infra`：数据库、缓存、消息、文件、第三方 Client、认证实现以及端口或仓储实现。
- `app-bootstrap`：启动类、Spring 装配、运行配置和最终可执行包；不承载业务规则。

### 命名规则

- 将模块目录固定命名为 `app-api`、`app-biz`、`app-bootstrap`、`app-domain`、`app-infra`。
- 让 Maven `artifactId` 或 Gradle project name 与模块目录保持一致，分别使用 `app-api`、`app-biz`、`app-bootstrap`、`app-domain`、`app-infra`。
- 不再为子模块追加根项目名称前缀，避免在 `common-iam` 等根目录下再次展示一组 `common-iam-*` 的视觉重复。
- 不将 `service`、`server`、`core`、`application` 或 `infrastructure` 作为新拆模块的默认名称；仅在用户明确指定其他结构时使用。
- 不创建 `common` 模块收纳暂时无法归类的代码；真正跨业务且稳定复用的能力才可进入公共模块。

### 依赖方向

- 保持 `app-domain` 独立，不依赖 `app-api`、`app-infra` 或 `app-bootstrap`。
- 让 `app-biz` 依赖 `app-domain`，不反向依赖外层实现。
- 让 `app-api` 调用 `app-biz`，不直接调用数据库 Mapper、Repository 实现或第三方 Client。
- 让 `app-infra` 实现内层声明的端口，并只为实现需要依赖 `app-biz` 或 `app-domain`。
- 让 `app-bootstrap` 依赖并装配 `app-api`、`app-biz` 和 `app-infra`；只在 `app-bootstrap` 生成可运行的 Spring Boot 包。

### 拆分完成标准

- 先按职责盘点现有类，再迁移代码；不要只创建目录或修改模块名称。
- 同步更新父构建文件、子模块依赖、包名、导入、测试、资源路径、启动配置和 README。
- 删除或停用已被替代的旧模块，避免新旧分层同时存在。
- 检查依赖方向，消除循环依赖和外层实现向内层泄漏。
- 执行完整编译和测试，确认最终可执行模块能够启动或至少完成打包。

## 构建与依赖

- 使用项目现有的 Maven 或 Gradle 构建方式。
- 优先使用 Spring Boot 官方 Starter 和现有依赖，不重复引入功能相同的库。
- 未经任务要求，不升级 Java、Spring Boot、Spring Cloud 或关键依赖版本。
- 不为简单功能引入重量级框架。
- 仅在项目已经使用 Lombok 时继续使用，不为构造器注入单独引入 Lombok。

## 依赖注入与 Spring Bean

- 对必需依赖使用构造器注入，并将依赖字段声明为 `private final`。
- 禁止使用字段注入。
- 已使用 Lombok 的项目可使用 `@RequiredArgsConstructor`。
- 按职责使用 `@Component`、`@Service`、`@Repository`、`@Controller` 和 `@RestController`。
- 默认保持 Bean 无状态，不在单例 Bean 中保存请求级可变数据。
- 除框架基础设施场景外，不在业务代码中直接通过 `ApplicationContext` 获取 Bean。
- 遇到循环依赖时重新划分职责，不使用 `@Lazy` 掩盖设计问题。

## 配置管理

- 沿用项目现有的 `application.yml` 或 `application.properties`，不无理由切换格式。
- 将一组相关配置用 `@ConfigurationProperties` 绑定为强类型对象。
- 对有单位的配置使用 `Duration`、`DataSize` 等明确类型，避免裸数字。
- 通过配置、环境变量或配置中心处理环境差异。
- 禁止硬编码或提交密钥、Token、密码、私钥和 Cookie 密钥。
- 只将 Profile 用于环境差异，不用它隐藏复杂业务分支。
- 新增配置时提供合理默认值、校验规则和必要说明。

## Web 层

- 保持 API 风格、路径、请求方法和响应结构与现有项目一致。
- 请求和响应使用 DTO，不直接暴露数据库实体、DO 或内部领域对象。
- 使用 `@Valid`、`@Validated` 和 Bean Validation 注解完成边界校验。
- 让 Controller 只做轻量编排，不直接调用 Mapper 或编写复杂业务判断。
- 使用统一的全局异常处理器返回一致的错误结构。
- 不向客户端暴露内部异常堆栈、数据库信息或敏感字段。
- 让分页、排序、时间格式和枚举值遵循项目已有约定。
- 对文件上传校验大小、类型、文件名和存储路径。
