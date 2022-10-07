# 评论服务

| 创建日期     | 2022-09-28                                        |
| :-- | :-- |
| 目标完成版本 | 2.x                                                       |
| 参考问题    | None |
| 当前状态     | Pending                                                      |
| 所有者       | wibus-wee <wibus@qq.com>                 |

# Summary 概要

实现 Mog 的评论服务，可以作为一个独立评论系统存在

# Motivation 动机

随着 2.x 的开始动工，我们意识到有一些服务是可以单独取出来对其扩展，将其变成一个更强大的服务，其中便有评论服务。

评论是博客中的一大重要部分，它是博客中为数不多的交互部分，我们应该慎重构建相关代码，因而才有了这篇 RFC

# Detailed design 详细设计

## 特别功能

因为这是一个独立的评论服务，所以我们可以在这个服务中实现一些特别的功能，比如：

- 评论 Reaction 功能（点赞，踩，笑脸，哭脸，生气，惊讶等）
- 评论短代码解析
- 评论表情解析，表情默认采用 [GitHub Emoji](https://api.github.com/emojis) 或者是 [Microsoft Emoji](https://github.com/microsoft/fluentui-emoji) 支持，可采用[missive/emoji-mart](https://github.com/missive/emoji-mart) 支持库。
- 评论 Markdown 解析
- 评论的部分 HTML 标签支持，Markdown 语法支持，表情拓展支持（可选择拓展图片或者颜文字）
- 多种评论方式，多种评论展示方式，可选择单选或者多选。

## 评论状态

```ts
enum CommentStatus {
  Pending = 'pending', // 待审核
  Approved = 'approved', // 已通过
  Spam = 'spam', // 垃圾评论
  Trash = 'trash', // 回收站
  Private = 'private', // 私密评论
}
```

|         评论状态         |                         评论状态说明                         |
| :----------------------: | :----------------------------------------------------------: |
| 待审核（需开启审核功能） |                评论进入后台审核，前台不做展示                |
|          已通过          |                         正常展示评论                         |
|         垃圾评论         | 后台对评论内容进行状态操作，评论会进入垃圾评论中，不做展示，可恢复为正常评论 |
|          回收站          | 后台对评论进行删除，删除的评论会暂时移动至回收站，可恢复为正常评论，在指定时间之后会彻底删除，可选择暂存时间或者接删除，不进入回收站暂存 |
|         私密评论         | 私密评论可选择以私密状态进行展示，也可以选择不展示（此种状态建议网站回复的相关通知功能完善时启用） |


## 评论种类

|              评论种类               |      评论种类说明      |
| :---------------------------------: | :--------------------: |
|               页评论                | 文章或者独立页面的评论 |
| 段、句评论（需开启文章句段评论支持) | 指定段落或者指定句评论 |

## Model 数据模型

数据库字段可分类为三大类：

1. 评论者：昵称、邮箱、网站网址、头像、IP地址、使用浏览器、是否文章创建者（与用户系统配合使用）。

   | 字段名称 |                           字段说明                           |
   | :------: | :----------------------------------------------------------: |
   |    id    |            唯一，字段标识符，用于定位具体评论内容            |
   |   name   |                             昵称                             |
   |  email   |                             邮箱                             |
   | website  |                           网站网址                           |
   |  avatar  | 可为空，为空状态下可选择通过邮箱检索不同头像源，若非空，则为指定链接，首次填写之后下次可为空，下次可通过数据库进行检索（暂不建议使用，规则未完善） |
   |    ip    |                      记录评论者 IP 地址                      |
   |  agent   |                   记录评论者浏览器相关信息                   |

2. 评论属性：评论内容、评论时间、状态、种类、所属页面、所属句段。

   | 字段名称  |             字段说明             |
   | :-------: | :------------------------------: |
   |  comment  |          评论的具体内容          |
   |   time    | 评论的时间，以时间戳形式进行记录 |
   |  status   |    详见 [评论状态](#评论状态)    |
   |   kind    |    详见 [评论种类](#评论种类)    |
   |   page    |             所属页面             |
   | paragraph |             所属段落             |

3. 交互属性：评论父子关系，评论的 Reaction。

   | 字段名称 |              字段说明              |
   | :------: | :--------------------------------: |
   |  parent  |         评论所属的父类评论         |
   | reaction | 数组存储，记录评论的 reaction 记录 |

## 监听活动

评论服务需要监听 Mog 的活动，以便于在用户评论时，可以及时的将评论推送到评论服务中。

```ts
export enum CommentEvents {
  CommentsGetAll = 'comments.get.all',
  CommentsGetWithQuery = 'comments.get.query'
  CommentsGetJustMaster = 'comments.get.master.only'
  CommentsGetByPostId = 'comments.get.by.postid',
  CommentsGetByPostIdWithMaster = 'comments.get.by.postid.auth',
  CommentCreate = 'comment.create',
  CommentCreateByMaster = 'comment.create.auth',
  CommentPatch = 'comment.patch',
  CommentDelete = 'comment.delete',
  CommentReply = 'comment.reply',
  CommentGet = 'comment.get',
  CommentAddRecaction = 'comment.add.reaction',
  CommentRemoveRecaction = 'comment.remove.reaction',
  CommentRecactionGetList = 'comment.reaction.get.list',
}
```

### `comments.get.all` 获取所有评论

获取所有评论，它需要管理员权限，可以获取任意状态的评论。

### `comments.get.by.postid` 获取文章或页面的评论

获取文章或页面的评论，它不需要管理员权限，只能获取已发布的评论。但是如果 headers 携带了有效的 token，那么它可以获取任意状态的评论。

### `CommentsGetByPostIdWithMaster` 获取文章或页面的评论

获取文章或页面的评论，它需要管理员权限，默认返回任意状态的评论。

### `CommentsGetJustMaster` 获取所有作者的评论

获取所有作者的评论，它不需要管理员权限。

### `CommentsGetWithQuery` 获取符合条件的评论

获取符合条件的评论，在一定情况下它不需要管理员权限，有部分条件需要管理员权限。

## 评论方式与展示方式

1. 传统文章末尾展示。
2. 文章、页面指定句、段评论。

|                   展示类型                   |                         展示类型说明                         |
| :------------------------------------------: | :----------------------------------------------------------: |
|   传统页末展示（建议选择文章尾部评论支持）   |                       大多数的使用场合                       |
| 文章段、句跟随展示（需开启文章句段评论支持） | 类似于现在流行的小说软件当中的评论，可以明确地对部分句、段进行精准评论，方便知晓评论所指内容。 |
|           弹幕展示（任选评论支持）           | 1. 文章当中使用弹幕展示（不推荐，比较影响文章观感）。<br />2. 文章、页面开头或尾部留出一定空间以弹幕的形式随机展示当前文章、页面的弹幕。<br />3. 在网站某处选择指定地点以弹幕的形式随机展示当前网站的所有评论。 |
|   侧页评论支持（建议选择文章尾部评论支持）   | 可选择在页面侧边进行展示所有评论，适用于对于文章展示比较有要求，但是又比较希望有评论支持的网站。 |

# Drawbacks 缺点

- 代码大小和复杂性将会变大，为了实现更多的内容我们需要更健壮的代码。
- 由于独立启动了另一个服务，内存占用会变大

# Alternatives 选择

其实本身是可以放入 core Gateway 中的，只需要做一个独立module即可，就不需要再新建多一个服务，这样反而会让内存占用变大

# Adoption strategy 采用的策略

如果我们实现这个提案，现有的开发者将如何使用它？这是一个突破性的变化吗？这将如何影响生态系统中的其他项目?

TBD.

# Unresolved questions 未解决的问题

- [X] Model 与 Dto 的定义
- [ ] 因为内存占用会变大，所以需要考虑付出与回报
- [X] 公开接口定义
- [X] 活动监听事件定义
- [ ] 与扩展点一同考虑
