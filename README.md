# module-dynamic-load

该项目演示一个基于 Java 的模块化动态加载框架。框架在启动时会扫描指定目录下的模块 JAR，当发现文件变更后自动加载最新版本，从而实现应用的无停机升级。

## 项目结构

- **module-dynamic-load**：核心实现，提供模块容器、模块管理器、加载器以及业务代理工厂等组件。
- **demo-user-interface**：示例接口模块，定义业务接口 `GameService` 等。
- **demo-user**：接口实现模块，在 Manifest 中声明模块元信息，并通过 `@ModuleService` 向外暴露服务。

核心模块默认使用 Spring 进行组件管理，在 `META-INF/modules` 目录下放置需要加载的模块 JAR。业务模块只需在 Spring 配置中扫描 `@ModuleService` 注解即可被框架识别并注册。

## 代码架构概览

- `ModuleContainer` 负责在容器启动时加载模块并在资源更新后重新加载，同时在销毁时释放所有模块资源【F:module-dynamic-load/src/main/java/yunnex/mdl/ModuleContainer.java†L3-L8】。
- `ModuleLoader` 根据模块配置激活模块，使其具备提供服务的能力【F:module-dynamic-load/src/main/java/yunnex/mdl/ModuleLoader.java†L12-L19】。
- `ModuleContext` 统一提供 `ModuleManager`、`ModuleLoader` 等运行时组件，用于模块的安装、卸载和状态监听【F:module-dynamic-load/src/main/java/yunnex/mdl/ModuleContext.java†L11-L45】。
- 示例实现 `GameServiceImpl` 通过 `@ModuleService` 注解对外提供服务【F:demo-user/src/main/java/yunnex/game/service/impl/GameServiceImpl.java†L8-L16】。

整体关系如下：

```
+----------------+      +--------------------+      +------------------+
| demo-user.jar  |----->| module-dynamic-load |----->| 应用通过代理访问 |
| (实现模块)      |      | (核心容器和管理)    |      | ModuleService    |
+----------------+      +--------------------+      +------------------+
         ^                        ^
         |                        |
+------------------+     +-----------------------+
| demo-user-interface |--| Spring 及自定义配置  |
| (接口定义)        |     | (META-INF/spring/*) |
+------------------+     +-----------------------+
```

## 使用方式

1. 在项目的 Spring 配置文件中引入自定义命名空间 `mdl`，并配置 `<mdl:container>` 指定模块存放目录和接口包路径。
2. 在业务模块中使用 `@ModuleService` 标记实现类，并打包为 JAR（Manifest 中包含模块名称、版本等元信息）。
3. 将生成的 JAR 放入配置的模块目录，启动应用即可自动加载。模块更新后替换 JAR，框架会检测到变更并重新加载。

此示例项目在 `module-dynamic-load/src/test` 中提供了简单的测试用例，可用于观察模块在运行时被重复调用与热更新的效果。

