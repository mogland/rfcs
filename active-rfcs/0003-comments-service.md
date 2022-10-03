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
- 评论表情解析
- 评论 Markdown 解析
- ...

## 评论状态

```ts
enum CommentStatus {
  Pending = 'pending', // 待审核
  Approved = 'approved', // 已通过
  Spam = 'spam', // 垃圾评论
  Trash = 'trash', // 回收站
}
```

## 评论种类

```ts
export enum CommentType {
  Post = 'post',
  Page = 'page',
}
```

## Model 数据模型

- 评论者
- 内容
- 评论时间
- 评论状态
- 评论种类
- 评论所属文章或页面
- 评论的父子评论
- 评论的 Reaction
- ...

## 监听活动

评论服务需要监听 Mog 的活动，以便于在用户评论时，可以及时的将评论推送到评论服务中。

```ts
export enum CommentEvents {
  CommentsGetAll = 'comments.get.all',
  CommentsGetByPostId = 'comments.get.by.postid',
  CommentsGetByPostIdWithMaster = 'comments.get.by.postid.auth',
  CommentCreate = 'comment.create',
  CommentCreateByMaster = 'comment.create.auth',
  CommentPatch = 'comment.patch',
  CommentDelete = 'comment.delete',
  CommentReply = 'comment.reply',
  CommentAddRecaction = 'comment.add.reaction',
  CommentRemoveRecaction = 'comment.remove.reaction',
  CommentRecactionGetList = 'comment.reaction.get.list',
}
```

### `comments.get.all` 获取所有评论

获取所有评论，它需要管理员权限，可以获取任意状态的评论。

...

### `comments.get.by.postid` 获取文章或页面的评论

...

# Drawbacks 缺点

- 代码大小和复杂性将会变大，为了实现更多的内容我们需要更健壮的代码。
- 由于独立启动了另一个服务，内存占用会变大

# Alternatives 选择

其实本身是可以放入 core Gateway 中的，只需要做一个独立module即可，就不需要再新建多一个服务，这样反而会让内存占用变大

# Adoption strategy 采用的策略

如果我们实现这个提案，现有的开发者将如何使用它？这是一个突破性的变化吗？这将如何影响生态系统中的其他项目?

TBD.

# Unresolved questions 未解决的问题

- [ ] Model 与 Dto 的定义
- [ ] 因为内存占用会变大，所以需要考虑付出与回报
- [ ] 公开接口定义
- [ ] 活动监听事件定义
- [ ] 与扩展点一同考虑
