# 主题模块

| 创建日期     | 2022-08-23                                                   |
| :-- | :-- |
| 目标完成版本 | 2.x                                                          |
| 参考问题:    | RFC-0001, https://github.com/nx-space/core/issues/193, https://github.com/nx-space/core/issues/178, https://github.com/nx-space/core/issues/162, https://github.com/nx-space/core/issues/178 |
| 当前状态     | Pending                                                      |
| 所有者       | wibus <wibus@qq.com>                                         |

# Summary 概要

实现如 Typecho, WordPress, Halo等博客系统的主题模块功能。

## 目标

- 主题应用方式，独立程序 / 模板引擎解析
- 主题开发
- 主题模板渲染方案
- 主题安装、更新、卸载
- 主题模板扩展
- 主题切换策略
- 主题配置
- 主题语言国际化

# Motivation 动机

我们为什么要这么做？期望是什么？结果是什么？

请重点解释其动机，以便如果该RFC不被接受，这种动机可以用于开发替代解决方案。换句话说，枚举您试图解决的约束条件

# Detailed design 详细设计

这是RFC的大部分内容。解释设计足够的细节，让熟悉 NEXT 的人理解，为熟悉的人来实现。这应该涉及到细节和极端情况，并包括如何使用该特性的示例。任何新的术语都应该在这里定义。

# Drawbacks 缺点

为什么我们不应该这样做呢?请考虑:

- 实现成本并不高，但是不合理的设计会使开发主题变得复杂
- 
- NEXT 其他生态兼容的成本 （是 Breaking Change ？）

选择任何道路都需要权衡。试着在这里辨认他们。

# Alternatives 选择

还考虑过其他的设计吗？不这样做的影响是什么？

# Adoption strategy 采用的策略

如果我们实现这个提案，现有的 NEXT 开发者将如何使用它？这是一个突破性的变化吗？这将如何影响 NEXT 生态系统中的其他项目?

# Unresolved questions 未解决的问题

可选，但建议初稿使用。设计的哪些部分仍然待定？
