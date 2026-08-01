---
name: java-springboot
description: 在创建、修改、重构、调试或审查 Java 与 Spring Boot 代码时，按仓库现有约定执行开发、质量检查和验证。适用于 REST API、Controller、Service、依赖注入、配置、事务、MyBatis/MyBatis-Plus、Spring Data JPA、Mapper、Repository、DTO、异常、日志、安全、单元测试、集成测试和 Java/Spring Boot 代码审查任务。
---

# Java 与 Spring Boot 开发规范

在不破坏现有架构和项目约定的前提下，编写清晰、可维护、可测试且可上线的 Java/Spring Boot 代码。

## 规范优先级

遵循更高优先级指令。在本 Skill 的建议与仓库约定不冲突时，再应用本 Skill：

1. 读取并遵循适用的 `AGENTS.md` 和用户明确要求。
2. 以仓库现有构建、静态检查、架构和相邻代码惯例为准。
3. 将本 Skill 作为仓库未规定部分的默认规范，不据此强行迁移架构、框架或命名体系。

## 执行流程

1. 阅读项目根目录及当前目录链上的 `AGENTS.md`、`README.md`、构建文件和静态检查配置。
2. 查看同模块相邻类，识别命名、分层、DTO、异常、日志、事务、持久化和测试惯例。
3. 根据任务范围读取下方对应的专题规范；涉及多个专题时读取全部相关文件。
4. 先确认行为边界和验证方式，再实施当前任务所需的最小改动。
5. 检查长方法、复杂嵌套、魔法值、异常、日志、空指针、安全、事务和测试影响。
6. 执行仓库已有的格式化、编译、静态检查和测试命令，修复本次改动导致的问题并重新验证。
7. 汇报改动、验证结果和仍存在的限制。无法执行某项验证时说明原因，不宣称其已通过。

若仓库配置了 Checkstyle、PMD、SpotBugs、Sonar、ArchUnit、Spotless 或其他团队规则，以现有配置为准。

## 按任务加载专题规范

- 涉及模块或包结构、分层、依赖、Spring Bean、配置、Controller 或 REST API 时，读取 [architecture-and-spring.md](references/architecture-and-spring.md)。
- 涉及方法设计、命名、注释、常量、空值、集合、`Optional`、异常或日志时，读取 [java-code-quality.md](references/java-code-quality.md)。代码实现和代码审查通常都需要读取此文件。
- 涉及 `@Transactional`、MyBatis、MyBatis-Plus、JPA、SQL、Repository、认证授权、文件路径或敏感数据时，读取 [persistence-and-security.md](references/persistence-and-security.md)。
- 修改代码、修复 Bug、增加测试或进行代码审查时，在完成前读取 [testing-and-review.md](references/testing-and-review.md) 并执行适用检查。

不要预先加载与任务无关的专题文件。

## 实施边界

- 不擅自改变现有模块和包结构，不替换框架，不引入完成任务不需要的依赖。
- 只修改完成当前任务所必需的代码，保留用户已有改动，避免顺手整理无关文件。
- 在实现或修复任务中，直接修复检查发现且属于本次范围的问题，然后重新验证。
- 在纯审查、解释或诊断任务中，只报告发现和依据；除非用户明确要求，否则不修改代码。
- 无法判断规则是否适用时，优先考察仓库中的实际用法，不机械套用通用最佳实践。
