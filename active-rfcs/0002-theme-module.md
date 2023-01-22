# 主题模块

| 创建日期     | 2022-08-23                                                   |
| :----------- | :----------------------------------------------------------- |
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

## Model 定义

```ts

export class ThemeDto {
  @IsString()
  @IsNotEmpty()
  id: string;

  @IsString()
  @IsNotEmpty()
  name: string;

  @IsBoolean()
  @IsOptional()
  active?: boolean;

  @IsString()
  @IsOptional()
  package?: string;

  @IsString()
  @IsOptional()
  version?: string;

  @IsString()
  @IsOptional()
  config?: string;
}

```



# Drawbacks 缺点



# Alternatives 选择

在 NestJS 中实现主题系统可以通过以下几种方式实现：

1. 使用 NestJS 的中间件实现主题系统，在请求/响应生命周期中设置主题。
2. 使用 NestJS 的拦截器实现主题系统，在拦截器中设置主题。
3. 使用 NestJS 的管道实现主题系统，在管道中设置主题。
4. 使用 NestJS 的模块实现主题系统，在模块的配置阶段设置主题。
5. 在你的服务里面实现主题系统, 在请求进入服务之前设置主题。

而我认为是应该使用微服务来实现主题系统的。在微服务架构中，可以将主题系统作为一个独立的微服务来实现。具体来说，微服务都可以通过 API 调用来与其他微服务通信，这样可以更好地控制主题系统的复杂性并使其可扩展性更高。

在 NestJS 网关层实现主题系统可以让主题系统更加集中，并且能够更好地控制主题的应用。这样可以在网关层进行验证，授权，负载均衡，缓存等操作。

但是如果你的主题系统需要更复杂的功能，例如添加自定义样式或者权限管理，利用NestJS微服务载入独立网关来实现主题系统可能是更好的选择。这样可以使用独立网关来处理主题相关的请求，并且可以使用微服务来实现更复杂的业务逻辑。

# Adoption strategy 采用的策略

如果我们实现这个提案，现有的 NEXT 开发者将如何使用它？这是一个突破性的变化吗？这将如何影响 NEXT 生态系统中的其他项目?

# Unresolved questions 未解决的问题

可选，但建议初稿使用。设计的哪些部分仍然待定？
