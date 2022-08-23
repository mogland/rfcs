# RFC: 插件系统

- 创建日期: 2022-08-23
- 目标完成版本: 2.x
- 参考问题: https://github.com/nx-space/core/issues/176, https://github.com/nx-space/core/issues/196, https://github.com/nx-space/core/issues/222, https://github.com/nx-space/core/issues/211
- 当前状态：Pending
- 所有者：*wibus <wibus@qq.com>*

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

# Basic example 

```js
import { Plugin } from '@nx-space/plugin-system';

export default class MyPlugin extends Plugin {
 // ...
}
```

# Motivation 动机

由于有很多功能我们核心团队都非常想实现，但是大多数都非常零散，如果将它们分别独立于一个模块里，未免有点浪费，因此使用一个个插件实现是一个非常正确的选择。

# Detailed design 详细设计

TBD.

# Drawbacks 缺点

- 实现成本较高，似乎尚没有前人在 NestJS 中实现类似的功能，对于 API 的设计较为复杂，需要让方法提供扩展点

# Alternatives 选择

曾经想过将项目重构，不再使用 NestJS，但是 NestJS 提供了很多方便的功能，反而使用 Express 等底层平台，如何建立起一个标准的系统，是一个更大的难题

# Adoption strategy 采用的策略

如果实现这个提案，我们需要先决定插件 API，再进行其他的操作，对于这个突破性的变化，将会强化用户的功能体验，优质的开发文档更会强化 NEXT 的生态。

# Unresolved questions 未解决的问题

- [ ] 插件 API 的合理设计
- [ ] 服务端如何实现插件扩展 / 如何实现可使用的扩展点