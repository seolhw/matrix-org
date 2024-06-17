+++
title = "聊天室和事件"
weight = 200
template = "docs/with_menu.html"
[extra]
updated = "2023-02-23T08:00:00Z"
meta_description = """
Matrix 依靠聊天室在服务器和客户端之间分配事件。聊天室
有一个基于功率级别的层次结构。每个homeserver都有自己的
聊天室的副本。
"""
+++

服务器上的用户可以向*聊天室*发送*事件*。事件是一个特定的 json对象，描述用户要做的事情（加入聊天室、发送信息、更新特定值......）。

一个事件的数据如下

```json
{
  "content": {
    "body": "This is an example text message",
    "format": "org.matrix.custom.html",
    "formatted_body": "<b>This is an example text message</b>",
    "msgtype": "m.text"
  },
  "event_id": "$143273582443PhrSn:example.org",
  "origin_server_ts": 1432735824653,
  "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
  "sender": "@example:example.org",
  "type": "m.room.message",
  "unsigned": {
    "age": 1234
  }
}
```

就即时通讯而言，客户端显示聊天室的方式与 Slack 、Discord 或 IRC 频道非常相似。这些聊天室中的大部分事件都是消息。
聊天室有一个唯一的技术标识符，以及零个或多个人类可读的别名。
别名由聊天室名称和服务器部分组成，有时也被称为 "地址"。
一个典型的聊天室别名是

```
#goodfriends:example.com
```

从技术角度看，聊天室是一系列 json 对象。下面的模式表示的是一个具有简化事件的聊天室。客户端可以读取、解析并正确显示事件。

{{ figure(
    img="./events.svg",
    caption="简化事件的 #goodfriends:example.com 聊天室")
}}

### 本地副本

homeserver是联合的：
Matrix 规范定义了 [Sever-Server API](https://spec.matrix.org/latest/server-server-api/)（又称联盟 API）来描述服务器之间的交互。
每当用户进入某个聊天室时，其 homeserver 都需要拥有该聊天室的本地副本。

例如，如果 `@alice:example.com` 是来自 `example.com` 的第一个用户尝试加入 `#goodfriends:matrix.org`，那么她的 homeserver 就会向 `Matrix.org` 获取聊天室副本。然后，`example.com`和`matrix.org`会保持
保持对该聊天室的同步。

{{ figure(
    img="./room_federated.svg",
    caption="三台 homeserver 保持聊天室本地副本同步")
}}

### 管理与权限

每当 homeserver 接收到新事件，它就会负责解析这些事件、对事件进行检查，并采取相应的措施（例如，向其他参与的 homeserver 发送信息，或将其他参与的 homeserver 发送的信息）。
homeserver的预期行为[Matrix规范](https://spec.matrix.org)对 homeserver 的预期行为作了全面描述。

如果认为每个人都有自己的本地聊天室。这岂不是意味着任何人都可以成为聊天室的管理员，并做出令人讨厌的事情？
幸运的是，不会。聊天室里的人的权限是由以下方面定义的

权力等级。

权力等级定义了聊天室的等级制度。聊天室里的所有行动都需要最低权限级别。在聊天室里发布信息需要有 0 级，编辑别人的信息通常需要有 50 级的权限，更改聊天室地址通常需要有 100 级的权限。

有关权限等级的更多详情，请查阅社区管理文档的相关章节。
社区管理文档的相关部分。

{{ page_card(path="/docs/communities/moderation/#power-levels",
    title="权力级别",
    summary="了解实际中如何使用权力级别")
}}

### 在创建聊天室时

权力等级是一个通常介于 0 和 100 之间的整数，与聊天室中的某个账户绑定。根据规范建议，当某人创建了一个聊天室，他的账户在该聊天室中的权限等级为 100。
如果 Alice 在 example.com 上创建了`#goodfriends`聊天室，那么她的账户`@alice:example.com`将在所有副本上获得100级的权限，无论该副本是在 example.com 上，还是在 ergaster.com 上。

{{ figure(
    img="./room_creation.svg",
    caption="Alice 创建聊天室并自动获得 100 级能量")
}}

### 聊天室加入时

当其他人加入聊天室时，无论他们是与创建者在同一 homeserver 上，还是在其他 homeserver 默认情况下他们都会被分配到 0 级权限。
如果 `ergaster.org` 上的 Bob 加入`#goodfriends:example.com`，他的服务器将向 Alice 的`example.com` 请求该聊天室的本地副本，并与之保持同步。

{{ figure(
    img="./room_join.svg",
    caption="Bob 加入聊天室并自动获得 0 级权限")
}}

如果卡罗尔从她的 homeserver `matrix.org` 加入，她也将获得 0 级别的权限

{{ figure(
    img="./room_federated.svg",
    caption="卡罗尔加入聊天室后，也会自动获得 0 级权限，聊天室的每个本地副本都是完全相同的。")
}}

### 更改聊天室的本地副本

现在我们来看看 Walter。Walter 是 example.com 的 homeserver 管理员，Alice 使用该 homeserver 创建了 #goodfriends:example.com 聊天室。

加入聊天室时，Walter 的默认权限级别为 0，和其他人一样。

{{ figure(
    img="./walter_joins.svg",
    caption="Walter 加入聊天室并获得 0 级权限：他是普通用户")
}}

如果 walter 在 homeserver 数据库中做了手脚，变更了本地的聊天室的本地副本，他的变更将不会传播，都会是 0 级权限，当其他 homeserver 想更新它们的聊天室本地副本时，它们会拒绝 walter 的更改。

### 冒充他人

一些 homeserver（如 Synapse）有一个非标准的[管理 API](https://matrix-org.github.io/synapse/latest/usage/administration/admin_api/index.html)。
尤其是[让某人成为聊天室管理员的 API](https://matrix-org.github.io/synapse/latest/admin_api/rooms.html#make-room-admin-api)。
这并不意味着 homeserver 管理员不能随意接管聊天室。

当 Walter 调用 "设置聊天室管理员" API 时，Synapse 将控制 Alice 的账户，授予 Walter 与她相同的 "权力等级"。Alice 的权力等级是 100，她可以将 Walter 提升到与她相同的 "权力等级"。这是有效的
因此，当其他 homeserver 更新它们的聊天室的本地副本时，就会接受这一变更。

{{ figure(
    img="./walter_escalate_ok.svg",
    caption="Alice 的账户被 "突触 "用来为沃尔特提供相同的权力等级")
}}

如果沃尔特是 `ergaster.org` 的 homeserver 管理员呢？每个 ergaster.org 的用户都拥有 0 级权限。如果 Walter 调用此 API，他的 homeserver 就只能控制权力级别为 0 的用户，而不能提升他的权力级别，无法对其进行晋升。

{{ figure(
    img="./walter_escalate_ko.svg",
    caption="Walter 无法控制任何账户来提升自己的权力等级")
}}
