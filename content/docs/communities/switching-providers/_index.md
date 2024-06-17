+++
title = "更换提供者"
template = "docs/with_menu.html"
weight = 300
[extra]
emoji = "🏡"
tile = "I want to get my own homeserver"
updated = "2023-01-30T14:00:00Z"
meta_description = """
Even after creating a community on a provider, it's possible to switch to
another one. This allows communities to get more sovereignty on their
conversations, and to connect them to their own systems.
"""
+++

## 为什么要更换提供者？

许多用户首先会在 Matrix.org 上创建一个 Matrix 账户，该账户由 Matrix.org 基金会运营。这是一个可靠、免费的提供者，但随着社区的发展社区可能希望托管自己的实例，以便更好地控制自己的数据和形象。

### 品牌

当社区迁移到自己的homeserver时，最明显的变化之一是基于域的品牌。事实上，用户标识符不再是 `@username:matrix.org`，聊天室地址格式为 `#roomname:matrix.org`。社区可以开始使用自己的域名创建用户账户和聊天室名称（例如，`@username:example.com` 和`#roomname:example.com`）。

|                        | 使用 matrix.org         | 使用 example.com         |
| ---------------------- | ----------------------- | ------------------------ |
| 用户 Matrix ID alice   | @alice:matrix.org       | @alice:example.com       |
| 聊天室地址 goodfriends | #goodfriends:matrix.org | #goodfriends:example.com |


### 聊天室和账户主权

当某人创建一个聊天室时，他将自动拥有该聊天室的最高权限等级。这个权力等级通常是 100，但自定义设置可以让这个数字更高。聊天室的最高权力等级是事实上的所有者，并可以指定共同所有人、主持人、编辑信息、将他人排除在对话之外等等。

这是一项很大的权力，但幸运的是，homeserver管理员可以冒充聊天室管理员来重新控制聊天室。请注意，这并不意味着homeserver管理员可以读取他们假冒的用户的信息：信息是端到端加密的，homeserver管理员无法读取这些信息。homeserver管理员无法获得解密密钥，因为这些密钥存在于用户设备上。


homeserver管理员可以使用 Synapse 的非标准 [Admin API](https://matrix-org.github.io/synapse/latest/usage/administration/admin_api/index.html)。尤其是 [Make Room Admin API](https://matrix-org.github.io/synapse/latest/admin_api/rooms.html#make-room-admin-api) 将其他用户提升为聊天室管理员。该 API 只有在以下情况下才会起作用其中一名用户在聊天室内。它将授予目标用户中的最高权限。

由拥有 `@xxx:matrix.org` 账户的用户管理的社区依赖于 matrix.org 的支持团队来升级问题并试图重新控制聊天室。Matrix.org 基金会财力有限，其支持团队在尽力而为的模式下工作。

有关聊天室所有权如何运作的高级解释，请访问技术文档中的
技术文档中的

{{ page_card(
    title="Rooms",
    path="/docs/matrix-concepts/rooms_and_events/",
    summary="了解聊天室如何通过联盟运作，以及谁能真正控制它们。")
}}


### 连接到你的系统

Matrix.org 基金会和 [t2bot.io](https://t2bot.io/)托管了一些公共桥接器，方便用户使用。当然，它们将公共服务与 Matrix.org homeerver 的桥接。

如果你想桥接私人服务，比如你公司的 Slack 工作区、你需要自行部署桥接器，因为 Matrix.org homeerver 不接受与新的私有系统桥接。

最后，依赖 Matrix.org 基金会托管的桥接会造成不必要的集中化：如果 Matrix.org homeerver 或 Matrix.org 基金会托管的桥接器出现故障，你就会暂时失去对该系统的访问。对该系统的访问。要了解原因，请阅读 [应用程序服务](/docs/matrix-concepts/elements-of-matrix/#appservice-bridges and-some-bots)和 [room](/docs/matrix-concepts/rooms_and_events/) 章节。概念文档中的 [room](/oc/matrix-concepts/rooms_and_events/) 部分。

可以桥接到 Matrix 的系统列表请参见
[桥接部分](/ecosystem/bridges)。

{{ page_card(
    title="Bridges",
    path="/ecosystem/bridges",
    summary="了解网桥的工作原理，了解 Matrix 可以与哪些平台桥接，以及如何部署自己的桥接。")
}}

## 选择新的提供者

Matrix.org 基金会乐意提供 Matrix.org homeerver，以提供便利。但强烈建议各组织拥有自己的homeserver。

### 付费托管

要获得自己的homeserver，最简单的方法是向提供者付费，让其为你托管一个Matrix 实例。你可以在 Matrix.org 基金会所知道的 [托管服务商列表] 中找到这样的服务商。[提供者列表](/ecosystem/hosting)中找到这样的提供者。

{{ page_card(
    title="Hosting",
    path="/ecosystem/hosting",
    summary="了解 Matrix.org 基金会熟知的托管服务提供商，并找到最适合您的托管服务提供商。")
}}


### 自行开发

想要或需要内部部署的用户可以选择[服务器实现](/ecosystem/servers)，然后自己安装或者从[基金会认识的提供者之一](/ecosystem/servers)获得内部部署支持计划。

{{ page_card(
    title="Servers",
    path="/ecosystem/servers",
    summary="浏览所有服务器实现，找到最适合您需要的您的需求以及如何自行安装。")
}}

## 转移所有权

转让所有权是 Matrix 的一大优势。你可以在 Matrix.org homeerver 上创建的账户开始你的社区，然后决定转移到另一个服务器上。

Matrix 还没有所有权的概念：它只依赖于[权力级别](/docs/communities/moderation/#power-levels) 来定义某人是否有权做某事。这意味着是该聊天室和空间的（共同）所有者。

无论何时，只要你把权力等级 100（管理员角色）时，你实际上就与他人分享了你的聊天室或空间的所有权。作为共同所有者，你们不能单方面收回给他人的东西。

想象一下，Alice 在 matrix.org 上的一个空间中为她的 WarmDrinks 社区创建了几个聊天室。然后，她决定将 "热饮" 作为一项严肃的事业来经营。她希望有一个 Matrix 实例专用于此。她依靠[托管服务提供者](/ecosystem/hosting/) 创建一个 `warm-drinks.com` homeerver。然后，她创建了用户 `@alice:warm-drinks.com`。

她可以在所有聊天室和空间中将 `@alice:warm-drinks.com` 提升到权力等级 100。和关于她在 matrix.org 上创建的热饮料的 Spaces，并降级她的 matrix.org 用户。现在她的新用户是社区的唯一所有者。
