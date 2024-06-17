+++
title = "Matrix 的 Elements 客户端"
weight = 100
template = "docs/with_menu.html"
aliases = ["/docs/matrix-concepts", "/docs/technical/"]
[extra]
updated = "2023-02-08T08:00:00Z"
meta_description = """
Matrix 依靠homeserver将客户端连接在一起。应用程序服务是
应用服务是将 Matrix 和第三方平台连接在一起的软件。
Matrix 规范定义了所有这些软件之间的交互。
"""
+++

{{ howitworks() }}

Matrix 的工作原理与电子邮件类似，但用于即时通讯。人们需要使用客户端来编写和接收信息，而他们需要提供者在他们的 homeserver 上为他们提供一个账户。

{{ figure(
    img="./federation.svg",
    caption="连接到 homeserver 的客户端模式，同时服务器联合在一起")
}}

让我们来看看这些都是什么。

## homeserver

homeserver 是一款托管 Matrix 用户账户的软件。它绑定到一个单一的域名，该域名不会随时间而改变。服务器上的账户有一个由本地部分（用户名）和服务器部分组成的标识符，服务器部分是 homeserver 的（虚荣）域名。典型的标识符如下

```文本
@username:example.com
```

上述模式中用户的Matrix ID 如下所示。

{{ figure(
    img="./federation_matrix_ids.svg",
    caption="连接到联合 homeserver 的客户端模式。所有用户都有一个Matrix ID")
}}

你可以在[生态系统 > 服务器](/ecosystem/servers)中找到现有 homeserver 实现的列表。其中大部分都是开源的，因此你可以探索它们是如何工作的。

{{ page_card(
    path="/ecosystem/servers",
    title="Servers",
    summary="发现所有家庭服务器实施方案")
}}

### 客户端

homeserver 之间通过服务器-服务器/联盟应用程序接口进行通信。API 进行通信，但它们也以标准方式与客户端通信：客户端-服务器API 进行通信。

客户端是一种软件，可使用 Matrix 账户发送和接收来自特定 homeserver 的事件。客户端本身只与它们使用的账户的 homeserver 进行对话。如果客户端使用 `@alice:example.com` 账户，就只能与 `example.com` 对话。

最常见的客户端是面向用户的客户端。在即时通讯中，这些客户端将聊天室显示为信息时间轴，用户可以加入、离开、编辑信息

从这里可以了解到更多的客户端，[生态系统 > 客户端](/ecosystem/clients)。

{{ page_card(
    path="/ecosystem/clients",
    title="Clients",
    summary="了解所有 Matrix 客户端。")
}}

如果你更想编写自己的客户端，为用户带来新体验，你可能需要依赖现有的 SDK（请参阅[生态系统 > SDKs](/ecosystem/sdks)）。这些工具包将承担Matrix的繁重工作，让你可以专注于你想要构建的用户体验。
最后，如果你有兴趣了解客户端和服务器之间的交互，请访问客户端和服务器之间的交互感兴趣，请访问[Matrix 规范的客户端-服务器部分](https://spec.matrix.org/latest/client-server-api/)。

{{ page_card(
    path="/ecosystem/sdks",
    title="SDKs",
    summary="浏览 SDK，编写自己的客户端。")
}}

## 桥接器和机器人

许多 Matrix 机器人都是非人类客户端。它们可以使用与普通客户端相同的`SDK`但它们并不显示用户界面来显示正在发生的事情，而是通过监听事件、解析事件并发送自动回复。

### 简单机器人和高级机器人

RSS 机器人就是简单机器人的一个很好的例子：它完全在 Matrix 之外订阅一个完全独立于 Matrix 之外的 RSS 源，每当在源中看到一个新项目，它就会在特定的 Matrix 聊天室里发布一条信息，并附上项目的名称。这是一种非常简单的机器人。

但有时你需要从更全局的角度来了解在 homeserver 上发生的事件，以便采取行动。例如，如果你想编写一个反垃圾邮件模块、你希望能读取来自公共聊天室的每一条信息，检测其模式，并发出警报或采取相应措施。

要使用机器人来实现这一点，你需要邀请机器人进入你希望监控的每个聊天室。应用程序服务可以监控所有未加密事件（发送/编辑/删除的消息、人员加入或离开聊天室）。

#### 桥接器

有时，你需要做的比 "千里眼" 还多：你需要自动创建用户和聊天室。一个典型的用例是[桥接器](/ecosystem/bridges)。桥接器允许你将 Matrix 社区连接到 IRC、Discord 或 Slack 等第三方平台。这些社区上的用户在 Matrix 上显示为本地用户，在第三方平台上也是如此。

- 桥接器在 Matrix 上创建的用户模仿第三方平台上的用户。在第三方平台上的用户称为 "幽灵"。
- 通过桥接器在第三方平台上创建的模仿 Matrix 用户的用户称为 "傀儡"。

{{ figure(
    img="./bridge.svg",
    caption="matrix.org 和 slack.com 之间的聊天室桥接模式")
}}

为此，桥接器需要能够在 Matrix 上创建用户和冒充用户、并控制聊天室。为了限制滥用的风险，桥接器可以仅限于控制一个命名空间。

要了解桥接概念的高层次视图，并查看 Matrix 可以与哪些平台桥接，请访问本网站的 [Ecosystem > Bridges](/ecosystem/bridges)。
如果你有兴趣编写自己的桥接器，这种情况下，你可以查看 [Ecosystem > SDKs](/ecosystem/bridges) 中的现有 SDK。[生态系统 > SDKs](/ecosystem/sdks) 以及 [规范的应用程序服务部分](https://spec.matrix.org/latest/application-service-api/)。

{{ page_card(
    path="/ecosystem/bridges",
    title="Bridges",
    summary="了解您可以连接到哪些平台以及如何连接。")
}}

## 规范

我们已经多次提到[Matrix规范](https://spec.matrix.org)
Matrix 规范是一份描述，Matrix 生态系统各组成部分（homeserver、客户端、应用服务之间的交互。
对于特定功能，实现细节可能会有所不同，但 Matrix 的目标是实现一致的行为，避免各方之间的协商。

该规范是开放的、版本化的，可在以下网址自由浏览 [spec.matrix.org](https://spec.matrix.org) 自由浏览。因此任何人都可以通过 Matrix Spec Change 提出扩展建议或参与管理和决策过程。

Matrix Spec Change (MSC) 是一份文件，说明作者希望如何修改Matrix规范，以引入生态系统各组成部分之间的交互变化的文件。
此类文件会被公开讨论、当作者认为他们已经解决了所有重要问题时[规范核心小组](/about#the-spec-core-team) 的注意，并开始正式审查程序。

- [Matrix规范流程](https://spec.matrix.org/proposals/) 
- [spec.matrix.org](https://spec.matrix.org)