# RFC: 友链服务

| 创建日期     | 2023-01-05           |
| :----------- | :------------------- |
| 目标完成版本 | 2.x                  |
| 参考问题     | None                 |
| 当前状态     | Pending              |
| 所有者       | wibus <wibus@qq.com> |

# Summary 概要

实现一个较为全面且智能的友链服务，让友链审核、展示、申请等功能尽量高效地运作

# Motivation 动机

  根据其他程序 / 程序内插件等所提供的功能上看，对于审核机制较为弱小，此服务为的是提供一个尽量完善的工作机制，提高用户体验效率，尽量达到最全面的友链服务。

# Detailed design 详细设计

## 交互流程

![](https://user-images.githubusercontent.com/62133302/210674345-cfe4a12f-4bff-43f6-9518-1db33cb1f24f.jpg)

## 审核逻辑

TBD.

## 友链状态

```ts
export enum FriendStatus {
  Passed,
  Rejected,
  Invalid,
  Trashy
}
```

目前主要分为：通过、拒绝、失效、垃圾

拒绝 / 失效 / 垃圾将会存档信息，定时删除（optional）

当被标注为拒绝 / 失效 / 垃圾后在数据删除之前重新申请，可通过数据库的指定字段检索，自动填充

## 数据库设计

|   name   |     key      | optional |                     usage                      |
| :------: | :----------: | :------: | :--------------------------------------------: |
|   名称   |     name     |    ❌     |                    博客名称                    |
|   链接   |     link     |    ❌     |                用于跳转友链博客                |
|   描述   |     desc     |    ✅     | 用于展示对方博客的内容或者是对方博客的个性签名 |
|   昵称   |   nickname   |    ✅     |           用于记录对方博主的昵称称呼           |
|   头像   |    avatar    |    ❌     |      用于展示对方博客的图标或者是博主头像      |
|   邮箱   |    email     |    ✅     |                  用于联系博主                  |
|   状态   |    status    |    ❌     |             [友链状态](#友链状态)              |
|  友链组  |    group     |    ✅     |          用于对友链的不同情况进行分类          |
| 友链检测 |  autoCheck   |    -     |  用于**自动检测**对方博客是否添加己方博客友链  |
| 订阅地址 |     feed     |    ✅     |                                                |
| 订阅类型 |   feedType   |    ✅     |                                                |
| 订阅内容 | feedContents |    ✅     |                                                |

## 服务活动

```ts
export enum FriendsEvents {
  FriendsGetList = "friends.get.list",
  FriendCreate = "friend.create",
  FriendGet = "friend.get",
  FriendPutByMaster = "friend.put.auth",
  FriendPatchByMaster = "friend.patch.auth",
  FriendPutWithToken = "friend.put.token",
  FriendDeleteByMaster = "friend.delete.auth",
  FriendDeleteWithToken = "friend.delete.token",
  FriendAnalyseFeed = "friend.analyse.feed",
  FriendAnalyseAutoCheck = "friend.analyse.autoCheck",
  FriendCheckAlive = "friend.check.alive"
}
```



# Drawbacks 缺点

- 实现审核逻辑较为复杂，且准确度并不高，其检索逻辑很有可能会使用到一定量的算力，需要继续改进
