# RFC: 插件系统

- 创建日期: 2022-08-23
- 目标完成版本: 2.x
- 参考问题: https://github.com/nx-space/core/issues/176, https://github.com/nx-space/core/issues/196, https://github.com/nx-space/core/issues/222, https://github.com/nx-space/core/issues/211
- 当前状态：Pending
- 所有者：_wibus <wibus@qq.com>_

# Summary 概要

在 NEXT 中实现一个类似于 Typecho, WordPress 等博客程序可热插拔的插件系统。

## 目标

一般

- 插件管理：可视化的插件管理界面，可以安装、卸载、更新、禁用、启用等操作。
- 插件仓库：可以查看所有插件的信息，可以搜索插件并安装。应当建立起适合的发布审核机制，保护插件的版本安全。
- 插件框架：提供开发脚手架，方便开发者快速开发插件，提供插件开发文档。

服务端插件扩展

- 后端 API 使用插件进行扩展，给予插件单独的路由
- 允许插件调度数据库进行数据处理与操作
- 允许插件通过已全局化的模块，允许插件注入其余的独立模块
- 允许插件对请求上下文进行操作，如设置请求头，设置响应头等
- 允许插件调用其他插件的方法

后台插件扩展

- 后台插件扩展，给予插件单独的路由，允许插件新增路由
- 允许插件对已存在的页面修改，实现在原有基础上的功能扩展

> ⚠️ 此提案的插件系统，并不属于模块插件化，模块插件化并未开始考虑

# Basic example

```js
import { Plugin } from "@nx-space/plugin-system";

export default class MyPlugin extends Plugin {
  // ...
}
```

# Motivation 动机

由于有很多功能我们核心团队都非常想实现，但是大多数都非常零散，如果将它们分别独立于一个模块里，未免有点浪费，因此使用一个个插件实现是一个非常正确的选择。

使用插件可以额外实现的功能包括且不限于

- 使用 **MWeb** 等编辑器推送文章
- 实现核心团队并不打算实现的功能
- 集成统计系统至前端、私有文章的特殊处理、可移植到各处的功能
- 后台文章管理功能增强、后台编辑器进化
- 使用微信或 QQ 等工具管理后台
- 文章可以在服务端实现短代码解析
- 发生操作活动时，可以请求 webhook url

在实现了插件系统后，有利于社区的生态建造，用户可以根据自己的需要开发插件拓展功能，使用者也可以更加方便地使用插件来满足自己的需求。

# Detailed design 详细设计

## 术语

- **Inject Point**
  - 可扩展功能的方法接口扩展注入点。
  - 注入点应该处于基本处理 --> 输出处理结果之间应用。
  - 注入点无法脱离基本处理方法单独执行，仅作为功能上的补充，对于扩展点的配置将会独立放入 Configs 表中的 plugin 字段。
  - 注入点中载入的处理方法间应看情况建立无/依赖/冲突关系。
  - 注入点载入的处理方法应当按照插件被激活时间倒序排列处理。
- **Plugin**
  - Inject Point（扩展注入点）的应用方法。
  - 对其中的处理方法应当更加精细区分，达到禁用/启动某一功能的程度
  - 激活依赖方法的控制权、插件依赖模块加载权交由用户
  - 插件应支持自己的独立模型进行配置

## 后端

### 启动时

启动时插件应当交由 PluginModule 负责加载，它的行为包括且理论不限于：

- 验证 Plugin 的配置定义文件是否存在且合法
- 验证 Plugin 内部使用的依赖是否安全且合法
- （非目标内）验证 Plugin 与其余 Plugin 是否存在冲突行为
- 将 Plugin 方法注册至 EventManager

EventManager 负责插件与 Core 自带方法的活动管理，它的行为包括且理论不限于：

- 在 core 的控制器中启动了 `@EventInject()` 的方法被执行时调度已启动且适用于此方法的插件
- 对 App 全局请求监听，对全部请求或返回中注入执行方法

EventManager 的实现可以参考 [minimajs](https://github.com/lorry2018/minimajs)

**配置定义**

**manifest.yml**

```yaml
name: Test
description: Description
version: 0.0.1
author: "?"
require: ">=1.5.3" # 最大支持的后端版本
dependencies:
  - Test-2: "1.0.0"
homepage: "?"
displayName: "?"
pluginSignKey: org.wibuswee.plugin.tests
```

- `version`: 指定插件版本号
- `requires: >=1.5.3` 表示后端版本必须大于 1.5.3
- `pluginDependencies`: 如果依赖了其他插件则使用`pluginSignKey: version`的格式[可选]。
- `homepage`: 插件的主页[可选]。
- `displayName`:插件的显示名称。
- `pluginSignKey`: 识别插件的唯一密钥，防止出现同名插件问题
- `description`: 详细介绍[可选]。

**启动注入点**

注入装饰器应当处于控制器方法之上：

```ts
@Get("/")
@EventInject() // I'm here
async method(args) {
  return this.service.method(args);
}
```

**版本号规范**

| Code status                               | Stage         | Rule                                         | Example version |
| ----------------------------------------- | ------------- | -------------------------------------------- | --------------- |
| First release                             | New product   | 从 1.0.0 开始                                | 1.0.0           |
| Backward compatible bug fixes             | Patch release | 增加第三位数字                               | 1.0.1           |
| Backward compatible new features          | Minor release | 增加中间数字并将最后一位重置为零             | 1.1.0           |
| Changes that break backward compatibility | Major release | 增加第一位数字并将中间和最后一位数字重置为零 | 2.0.0           |

**自定义模型**

TBD.

**活动注入**

使用 `@EventInject()` 标记方法，编译时自动注入函数，装饰器应当接受一个参数，该参数是字符串，表示活动的名称，活动管理器会根据该名称调度活动。

```typescript
enum EventInjectMethod {
  GET_REQUEST = "request", // 接受请求时
  PUSH_RESPONSE = "response", // 发送响应时
  POST_CREATE = "create:post", // 在创建文章时调用
  POST_UPDATE = "update:post", // 在更新文章时调用
  POST_DELETE = "delete:post", // 在删除文章时调用
  POST_FIND = "find:post", // 在查询文章时调用
  PAGE_CREATE = "create:page", // 在创建页面时调用
  // ...
}
```

**在插件中的插件生命周期**

插件的生命周期是从插件安装时开始，到插件卸载时结束。只需要 `start`, `stop`, `install`, `uninstall` 四个方法即可。

```typescript
class TestPlugin extends Plugin {
  constructor(options) {
    super(options);
  }
  override install(): void {}
  override start(): void {}
  override stop(): void {}
  override uninstall(): void {}
}
```

**插件定义处理方法**

PluginModule 自动识别插件类中所有的公开方法，并获取jsDoc中标注的注入点，并将其与插件方法绑定。EventManager 会根据注入点调度活动，传入的参数有差异。但接口定义是相同的

```typescript
interface ExecuteEvent {
  name: EventInjectMethod;
  args: any[];
}
```

- `name`: 活动名称
- `args`: 活动参数，参数格式参考 执行传入 args 类型 的说明。

```typescript
class TestPlugin extends Plugin {
  /**
   * @Inject create:posts
   * @Inject update:posts
   */
  public async requestWebHooks(args) {
    reg(args.text);
  }
  private async reg(str) {}
}
```

**执行传入 args 类型**

- **Posts** 模块: Return `PostModel[] | PostModel`
- **Pages** 模块: Return `PageModel[] | PageModel`
- **Category** 模块: Return `CategoryModel[] | CategoryModel`
- **Links** 模块: Return `LinkModel[] | LinkModel`
- **Comments** 模块: Return `CommentModel[] | CommentModel`

### 后台

TBD.

# Drawbacks 缺点

- 实现成本较高，似乎尚没有前人在 NestJS 中实现类似的功能
- 对于 API 的设计较为复杂，需要控制器提供的方法提供扩展点

曾经想过将项目重构，不再使用 NestJS，但是 NestJS 提供了很多方便的功能，反而使用 Express 等底层平台，如何建立起一个标准的系统，是一个更大的难题

# Unresolved questions 未解决的问题

- [ ] 自定义模型的配置存放位置（表内/单独表）
- [ ] 插件 API 的合理设计
- [ ] 服务端如何实现插件扩展 / 如何实现可使用的扩展点
