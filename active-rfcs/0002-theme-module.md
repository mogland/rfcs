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

目前的主题都是独立的程序，这样的好处是可以自由的开发，但是也带来了一些问题：

- 开发者需要自己实现一些基础功能，如：用户系统、评论系统、文章系统等
- 主题之间的切换需要重新安装
- 开发周期长，需要自己实现一些基础功能
- ...more

使用模板引擎解析的方式，可以解决上述问题，缩短开发周期，提高开发效率。

# Detailed design 详细设计

## Model 定义

```ts
export class ThemeDto {
  id: string;
  name: string;
  path: string;
  active?: boolean;
  package?: string;
  version?: string;
  config?: string;
}
```

- **id**: 在「插件系统」中我们有提到插件的唯一id，而到主题我们也需要制作这么一个id，需要确保其唯一性

我们推荐的 id 形式为 `theme.<custom>.<author_name>.<?others>`，如 `theme.g.wibus-wee`, `theme.tiny.wibus-wee.pro` 

此 id 并不会作为其他用途，仅做唯一性检验用途。

- **name**: 主题名字
- **active**: 是否启动，需要保证 active 只有一种主题
- **package**: 记录 package.json 中的相关字段（ `description`, `mog` 等）
- **version**: 主题版本
- **config**: 主题配置，此处需要具体讨论
- **path**: 主题文件夹名字

## 主题目录结构

- **packages.json**: 主题的配置文件，类似于 npm 的 `packages.json`，用于描述主题的基本信息，如：主题名字、版本、作者、描述等
- **config.yaml**: 主题的配置文件，用于描述主题的配置项，如：头像源、评论系统等
- **i18n.yaml**: 主题的国际化文件，用于描述主题的国际化信息
- **assets**: 主题的静态资源文件，如：css、js、图片等
- **plugins**: 主题的插件文件，用于注入方法到主题中，如：moment.js 等方法类库
- **index.ejs**: 主页
- **post.ejs**: 文章页
- **page.ejs**: 页面页
- **archive.ejs**: 归档页
- **category.ejs**: 分类 & 标签页
- **friends.ejs**: 友链页
- **404.ejs**: 404页
- **page-*.ejs**: 自定义页面，如：关于页 等, `*` 为自定义的页面路径

## 服务活动

```ts
export enum ThemesEvents {
  Ping = 'themes.ping',

  ThemesGetAll = 'themes.get.all',
  ThemeGetInfo = 'theme.get.info',
  ThemeActiveByMaster = 'theme.active.auth',
  ThemeDeleteByMaster = 'theme.delete.auth',
  ThemeGetConfig = 'theme.get.config',
  ThemeGetConfigItem = 'theme.get.config.item',
  ThemeUpdateConfig = 'theme.update.config.auth',
  ThemeUpdateConfigItem = 'theme.update.config.item.auth',
  // spread theme install hook
  ThemeBeforeInstall = 'theme.beforeInstall',
  ThemeAfterInstall = 'theme.afterInstall',

  ThemeBeforeActivate = 'theme.beforeActivate',
  ThemeAfterActivate = 'theme.afterActivate',

  ThemeBeforeDeactivate = 'theme.beforeDeactivate',
  ThemeAfterDeactivate = 'theme.afterDeactivate',
  
  ThemeBeforeUninstall = 'theme.beforeUninstall',
  ThemeAfterUninstall = 'theme.afterUninstall',
}
```

## 主题配置

我们考虑使用 YAML 定义主题配置

```yaml
id: "theme.tiny.wibus-wee"
language: zh # 语言, 要与 i18n.yaml 中设置的语言一致
# name: Tiny # name 使用 package.json 中的值
configs:
  - name: "头像源"
    key: "avatar_source" 
    # key is optional, if not set, it will be the same as name
    # But I recommend you to set it if your name contains special characters or use Chinese
    type: "select"
    data:
      - name: "Gravatar"
        # key: "Gravatar"
        value: "https://cn.gravatar.com/avatar/"
      - name: "国内源"
        key: "China"
        value: "https://cdn.v2ex.com/gravatar/"
    value: "Gravatar"
```

- **name**: 配置展示名
- **key**: 配置储存名（取值的时候使用此名）

key 是可选的，如果不去设置就与 name 是一样的。但是当 name 中涉及到一些特殊符号或者涉及中文字符的时候，强制需要填入 key

- **type**: 配置组件
- **data**: 传入配置组件的值（内部的定义与上方相似）
- **value**: 默认值，会传入配置组件，可选

## 配置组件

所有的 key 一般情况下除特殊要求，皆为可选字段，如果不设置 key 就与 name 一样。但是当 name 中涉及到一些特殊符号或者涉及中文字符的时候，**强制需要填入 key。**

- **input**: 输入框

  ```yaml
  type: "input"
  value: "https://cn.gravatar.com/avatar/"
  ```

传入的 data 为一个对象，对象中包含 value 一个必须的字段，key 为可选字段，如果不设置 key 就与 name 一样。

- **switch**: 开关

  ```yaml
  type: "switch"
  value: true
  ```

传入的 data 为一个对象，对象中包含 value 一个必须的字段，要求 value 为布尔值。

- **color**: 颜色选择器

  ```yaml
  type: "color"
  value: "#000000"
  ```

传入的 data 为一个对象，对象中包含 value 一个必须的字段，要求 value 为颜色值。

- **slider**: 滑块

  ```yaml
  type: "slider"
  value: 50
  ```

传入的 data 为一个对象，对象中包含 value 一个必须的字段，要求 value 为数字。

- **radio**: 单选框

  ```yaml
  type: "radio"
  data:
    - name: "Gravatar"
      value: "https://cn.gravatar.com/avatar/"
    - name: "国内源"
      key: "China"
      value: "https://cdn.v2ex.com/gravatar/"
  value: "Gravatar"
  ```

传入的 data 为一个数组，数组中的每一项都是一个对象，对象中包含 name, value 两个必须的字段。key 为可选字段，如果不设置 key 就与 name 一样。

value 为传入的值，value 会传入配置组件，需要与 data 中的 key 对应。

- **checkbox**: 多选框

  ```yaml
  type: "checkbox"
  data:
    - name: "Gravatar"
      value: "https://cn.gravatar.com/avatar/"
    - name: "国内源"
      key: "China"
      value: "https://cdn.v2ex.com/gravatar/"
  value: "Gravatar"
  ```

传入的 data 为一个数组，数组中的每一项都是一个对象，对象中包含 name, value 两个必须的字段。key 为可选字段，如果不设置 key 就与 name 一样。

value 为传入的值，value 会传入配置组件，需要与 data 中的 key 对应。

- **textarea**: 多行文本框

  ```yaml
  type: "textarea"
  value: "https://cn.gravatar.com/avatar/"
  ```

传入的 data 为一个对象，对象中包含 value 一个必须的字段，key 为可选字段，如果不设置 key 就与 name 一样。

- **select**: 下拉选择框

  ```yaml
  type: "select"
  data:
    - name: "Gravatar"
      value: "https://cn.gravatar.com/avatar/"
    - name: "国内源"
      key: "China"
      value: "https://cdn.v2ex.com/gravatar/"
  value: "Gravatar"
  ```

传入的 data 为一个数组，数组中的每一项都是一个对象，对象中包含 name, value 两个必须的字段。key 为可选字段，如果不设置 key 就与 name 一样。

value 为传入的值，value 会传入配置组件，需要与 data 中的 key 对应。

## 主题安装、更新、卸载

在主题安装、更新、卸载时，会触发主题的安装、更新、卸载活动传播，活动传播的执行顺序如下：

1. 主题安装前钩子 `theme.beforeInstall`
2. 主题安装钩子 `theme.install`
3. 主题安装后钩子 `theme.afterInstall`
4. 主题更新前钩子 `theme.beforeUpdate`
5. 主题更新钩子 `theme.update`
6. 主题更新后钩子 `theme.afterUpdate`
7. 主题卸载前钩子 `theme.beforeUninstall`
8. 主题卸载钩子 `theme.uninstall`
9. 主题卸载后钩子 `theme.afterUninstall`

全部活动都会在 notification_service 中启动监听并对外广播，可以通过监听这些活动来实现主题的安装、更新、卸载提醒。

### 验证主题合法性

在主题安装、更新、卸载时，会对主题的合法性进行验证，验证的内容如下：

1. 主题是否存在（利用 `fs.existsSync`）
2. 主题是否已经安装 （已经安装的都存于数据库）
3. 主题 id 是否合法（只能包含字母、数字、下划线，且与已安装的主题 id 不重复）
4. 配置文件是否存在且合法

若主题不合法，需要提示用户主题已损坏，需要重新安装。

### 安装、更新、卸载主题方式

安装、更新、卸载主题的方式有两种：

1. 通过主题管理页面安装、更新主题
2. 手动安装、更新主题

事实上，进入主题管理页面时，需要检查配置文件中的主题是否在主题目录中存在，如果不存在，需要提示用户主题已损坏，需要重新安装。

## i18n 方案

我们考虑注入一个 i18n 的函数 `_i(<key>)` ，这个函数可以根据传入的 key 来获取对应的 value。

### 定义 i18n key

我们考虑依然使用 yaml 文件来定义 i18n `i18n.yaml`

```yaml
# i18n.yaml
en:
  hello: 'Hello'
  world: 'World'
zh:
  hello: '你好'
  world: '世界'
```

按照上面的定义，我们可以通过 `_i('hello')` 来获取对应的 value。

```ejs
<%- _i('hello') %> <%- _i('world') %>
```

## 主题模板扩展

我们考虑单独储存一个 `plugins` 文件夹，用于存放主题模板扩展的插件。

### 定义主题模板扩展插件

我们将会遍历 `plugins` 文件夹下的所有文件，将其作为主题模板扩展插件，将其挂载到主题模板中。

### 简单实现

```typescript
const app: FastifyAdapter = new FastifyAdapter()

app.getInstance().decorateReply('render', function (view, options) {
  const plugins = fs.readdirSync(path.resolve(__dirname, 'plugins'))
  const pluginMap = {}
  plugins.forEach((plugin) => {
    const pluginPath = path.resolve(__dirname, 'plugins', plugin)
    const pluginModule = require(pluginPath)
    pluginMap[pluginModule.name] = pluginModule
  })

  const template = fs.readFileSync(path.resolve(__dirname, 'views', view), 'utf-8')
  const compiled = ejs.compile(template, {
    filename: path.resolve(__dirname, 'views', view),
    async: true,
  })
  
  const result = compiled({
    ...options,
    ...pluginMap,
  })

  this.send(result)
})

app.setViewEngine({
  engine: {
    compile: (template, options) => {
      return ejs.compile(template, options)
    },
    render: (compiled, options) => {
      return compiled(options)
    },
  },
  templates: path.resolve(__dirname, 'views'),
})

app.useStaticAssets(path.resolve(__dirname, 'public'))
```

```js
// plugins/time.js
module.exports = {
  name: 'time',
  time: () => {
    return new Date().toLocaleString()
  },
}
```

```ejs
<!-- views/index.ejs -->
<%- time() %>
```

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
