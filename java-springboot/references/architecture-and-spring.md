# 架构与 Spring 规范

## 目录

- 项目结构与分层
- 构建与依赖
- 依赖注入与 Spring Bean
- 配置管理
- Web 层

## 项目结构与分层

- 保持现有模块和包结构，不因本规范强行改造成另一种架构。
- 让 Controller 负责协议转换、参数校验、身份上下文和响应组装，不承载核心业务规则。
- 让应用服务负责用例编排、事务边界和跨领域协作。
- 将核心业务规则优先放在领域对象、领域服务或职责明确的业务组件中。
- 让基础设施层负责数据库、缓存、消息、文件和第三方服务的具体实现。
- 只把真正跨业务复用的稳定能力放入公共模块，避免形成无关工具和常量的垃圾桶。
- 禁止为了“统一风格”将所有业务逻辑堆进一个 `@Service` 类。

当项目已经采用 `bootstrap/api/biz/domain/infra/common` 结构时，遵循其现有职责：

- `bootstrap`：应用启动、Spring 装配和运行配置。
- `api`：接口定义、请求响应 DTO 和对外契约。
- `biz`：应用服务、用例编排和事务协调。
- `domain`：领域模型、业务规则、领域服务和仓储接口。
- `infra`：数据库、缓存、消息、第三方接口和仓储实现。
- `common`：稳定、通用且与具体业务无关的公共能力。

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
