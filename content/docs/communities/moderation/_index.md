+++
title = "社区管理"
template = "docs/with_menu.html"
weight = 200
[extra]
emoji = "🛡️"
tile = "我想部署社区安全工具"
updated = "2023-01-25T06:43:00Z"
meta_description = """
Matrix 可借助审核工具确保社区安全。Mjolnir 是
是社区管理者在 Matrix 上打击滥用行为的推荐解决方案。
Matrix.
"""
+++

## 权力等级

Matrix 有一套基本的角色系统，有时也称为 "权力等级"。它是一个通常是从 0 到 100 的数字。高级用户可以决定使用不同的范围，但为了简单起见，我们在此不做介绍。

默认情况下，Matrix 有三种角色：

| 功率级别 | 角色   |
| -------- | ------ |
| 0        | 用户   |
| 50       | 主持人 |
| 100      | 管理员 |

建议使用默认值：

- **用户**可以在聊天室中发送信息、媒体文件、回复或编辑自己的信息。
- **主持人**还可以更改聊天室名称、地址、主题，将用户从聊天室中删除（临时或永久删除）、编辑他人的信息并向聊天室中的所有人发送通知（@room 向聊天室里的所有人发送通知）
- **管理员**还可以更改历史记录的可见性（人们是否能看到他们加入或不加入之前的信息）、启用聊天室加密、从服务器中移除整个聊天室，并将其他人提升为主持人或管理员。

社区中的大多数人只是普通用户。通常情况下指定主持人来处理临时性的管理问题。请注意如果你希望保护信任与安全团队的匿名性，以确保他们的安全，那么这可能会成为一个问题。团队的匿名性以确保他们的安全。主持人机器人 mjolnir 可以很好地解决这个问题。

如果你要接管一个社区，而该社区之前是由在 Matrix 方面拥有高级权限的人管理的社区，那么他们的角色可能就不一样了。角色可能会有所不同。在这种情况下，我们建议你发送电子邮件至[support@matrix.org](mailto:support@matrix.org) 寻求帮助。

## 你社区的安全卫士

Matrix 内置的工具非常适合小型群组的管理，目前还很有限：删除用户（暂时或永久）只能在聊天室级别上实现。

幸运的是，有一种工具可以提高你的管理水平：一个安全卫士机器人，它可以同时对所有聊天室执行你的管理决定。

这就是 mjolnir 的真正用途：聊天室的保安。因为由于 Matrix 的分散性，你需要添加机器人来看守每个聊天室的门。你还需要赋予它管理员的角色，这样它才能正常工作。


### 获取雷神之锤

目前，要为你的社区获取魔力宝贝，你需要花钱请人为你运行 mjolnir，或者[在你自己的基础设施上运行它](https://github.com/matrix-org/mjolnir/tree/main/docs)
如果你有开发经验的话，就能很方便的使用它，不过我们正在开发一种更方便的服务，让人们可以更轻松地使用 mjolnir 机器人。

#### 在你的聊天室中进行设置

完成 mjolnir 的技术设置后，你需要
- 确保你的管理人员在你社区的控制室中
- 邀请 mjolnir 保安机器人进入你的所有聊天室
- 让 mjolnir 机器人成为所有聊天室的管理员（这样它就可以执行管理决定）。

为此，你需要邀请每个聊天室的管理机器人。方法如下
打开右侧面板，打开成员列表并点击
邀请，或者在底部的信息栏中输入以下信息：

```
/invite @yourMjolnirBot:example.com
```

机器人将加入，然后你可以在成员列表中找到它，并更改它的角色，从而晋升它为管理员，或者在底部的信息栏中键入以下信息：

```
/op @yourMjolnirBot:example.com 100
```

你应该花点时间考虑一下要赋予 Mjolnir 什么权限。常见的设置是赋予 Mjolnir 用户 100 级权限。这就赋予了 Mjolnir 踢人或禁止他人进入其保护的聊天室的权限。这将产生一些影响。首先，任何有权限使用 Mjolnir 的人在它保护的聊天室里都有 100 级的权限。同时，Mjolnir 也是这些社区聊天室的实际拥有者。有关更高级的设置，请阅读[涵盖权力等级的规范](https://spec.matrix.org/v1.5/client-server-api/#mroompower_levels)。

## 获取审核报告

默认情况下，审核报告会发送给报告人的homeserver管理员。如果你是homeserver管理员，可以选择将报告发送到 Mjolnir 管理室。这些报告看起来像这样

![](./mjolnir_report.png)

下面还有一些按钮，你可以用它们对报告采取行动。

稍后我们将讨论审核操作。如果你想设置这些报告，请参考文档 [此处](https://github.com/matrix-org/mjolnir#enabling-readable-abuse-reports)。

### 执行审核

### 编辑特定信息

编辑消息有两种方法。如果使用的是报告，可以点击报告下方的 {{ mjolnirbutton(text="🗍 Redact") }} 按钮。你也可以通过在 mjolnir 控制室发送信息来编辑特定信息。当你还没有收到关于某条信息的报告，但又想编辑它时，这种方法非常有用。

要编辑特定信息，你需要找到它的链接。将鼠标悬停在要删除的信息上，点击"..."，然后点击 "共享"，就能找到该链接。

![](./share_message.png)

这将弹出一个窗口，你可以从中复制该链接。

![](./copy_permalink.png)

获得永久链接后，就可以在 mjolnir 的编辑命令中使用它了

```
!mjolnir redact 事件永久链接 
```

例如

```
!mjolnir redact https://matrix.to/#/!yOatELRSQXzfQMmxjH:matrix.org/$F76L2figPEC240TFaUkHoKPxxhJ3P54vP4hi14Sd8xw?via=matrix.org&via=t2bot.io
```

Mjolnir 的一个重要功能是保护主持人个人免遭报复。如果你使用客户端用户界面编辑了一条消息，该编辑会显示你的用户 ID。而使用 Mjolnir 时，则会显示 Mjolnir 的 ID。这有助于减少对主持人审核行为的直接报复。你的管理室将记录谁采取了行动。

### 编辑用户的最后一条信息

攻击性用户有时会在被发现前尝试发送垃圾信息。查找这些邮件的永久链接会很繁琐，而且会使效率特别低。

如果发现了这样的用户，就有可能在全局或特定区域编辑他们最近的 n 条消息。为此，请获取用户的 Matrix ID 并按照以下模式发出命令：

```
mjolnir redact <用户 ID> [聊天室别名/ID] [限制］ 
```

例如，要编辑`@john:example.com`在
中的 `@john:example.com` 的最后 100 条信息：

```
!mjolnir redact @john:example.com #matrix:matrix.org 100
```

或者在全局范围内编辑来自 `@john:example.com` 的最后 100 条信息：

```
!mjolnir redact @john:example.com 100
```

请注意，该命令将删除用户的最后一条信息，但不会采取任何措施阻止用户发布信息。采取任何措施阻止他们发布更多辱骂性信息。大多数情况下，你会希望将此人暂时或永久地从你的社区中删除。


### 暂时删除某人（踢）

如果启用了审核报告并点击{{ mjolnirbutton(text="⚽️ Kick") }} 按钮，mjolnir 就会将用户踢出被举报的聊天室。

如果没有启用审核报告，也可以将用户踢出特定聊天室或使用以下命令全局踢出用户。

```
mjolnir kick <glob> [聊天室别名/ID] [原因］
```

例如

```
!mjolnir kick @john:example.com #matrix:matrix.org 与他人的不恰当互动。
```

### 绝对删除某人（禁言）

有些人可能在他们的信息被删节后仍不罢休，并被踢。在这种情况下，你需要能将他们从你的社区中永久删除。


如果启用了报告功能，并点击 {{ mjolnirbutton(text="🚫 Ban") }} 按钮，mjolnir 将禁止该用户禁止该用户进入它所保护的所有聊天室。用户不仅会被禁止进入被举报的聊天室还将被禁止进入整个社区。

如果没有启用审核报告，也可以通过以下命令禁止用户
使用以下命令

```
!mjolnir ban <list shortcode> <user|room|server> <glob> [reason] (禁言原因)
```

例如，如果要将 `@john:example.com` 从你的社区中永久删除，你可以在 mjolnir 的控制室中发出以下命令。

```
!mjolnir ban coc user @john:example.com User keeps insulting people
```

这是一个复杂的命令，让我们试着理解一下它的作用。

- !mjolnir ban` 告诉 mjolnir 我们想让它执行禁言命令
- `coc` 告诉 mjolnir 我们要把这个人加入禁言名单，他的简称是 coc 下面我们将进一步探讨什么是禁言列表。
- `user` 告诉 mjolnir 它需要封禁一个用户。这不是严格的强制要求，但为了避免在封禁单个用户时封禁整个服务器，添加它是个不错的做法。
- `@john:example.com` 是我们要封禁的用户的 Matrix ID。
- `用户不断侮辱他人` 是我们要封禁该用户的原因。他们的客户端就能显示这条信息。

然后，Mjolnir 会将我们要封禁的用户添加到我们指定的封禁列表中。Mjolnir 会持续关注一个或多个禁言列表，并将这些列表中的所有用户从其保护的所有聊天室中禁言。

注意，如果你是第一次尝试封禁某人，这条命令很可能会失败。禁止某人。事实上，你需要先创建一个或多个禁言列表，然后再添加用户或服务器。有关如何创建禁言列表的更多信息，请参阅下面的 创建禁言列表 部分。


### 绝对删除服务器

Matrix 是一个联合网络。这意味着人们可以建立新的服务器专门危害某些社区。这使得他们可以创建几乎无限的用户数量，从而可以轻而易举地突袭其他社区。

幸运的是，所有这些恶意用户都有一个共同点：他们来自同一个homeserver。这就意味着，如果你完全阻止了恶意homeserver、所有恶意用户都会被同时禁止。

要做到这一点，你可以发出用于禁止单个用户的命令的变体。
该命令是

```
!mjolnir ban <list shortcode> <user|room|server> <glob> [reason] 理由
```

你可能已经猜到了，这一次你要封禁的不是一个用户，而是一个服务器。
例如，禁用 `maliciousdomain.tld` 域：

```
!mjolnir ban spam server maliciousdomain.tld
```

Mjolnir 将把该域添加到 "垃圾邮件" 禁用列表中，每当来自该homeserver的成员试图加入你的社区时，他们就会被禁止。

### 创建禁言列表

Mjolnir 允许你创建多个禁言列表，以便日后查看。这些如果你把它们公开，第三方也可以监视它们。

在 mjolnir 的控制室中创建列表的命令是
如下

```
mjolnir list create <shortcode> <alias localpart> 
```

- `shortcode` 是这个列表的简短名称。它应该简短易因为在禁用用户时可能会经常输入。
- `alias localpart` 是 mjolnir 为该列表创建的地址的本地部分。如果你想与其他社区共享你的禁言名单，这很有用。

例如，假设 mjolnir 部署的服务器是 `example.com`、
下面的命令将创建一个禁言列表，其短代码为 `spam` ，地址`#my-community-spam-ban-list:example.com`。

```
!mjolnir list create spam my-community-spam-ban-list
```

你只需创建一次禁言列表，然后就可以在该禁言列表中添加任意数量的用户和服务器。你也可以创建任意数量的禁言列表。
禁言列表。

社区通常会创建两个禁言列表：一个针对垃圾邮件，另一个针对违反行为准则的行为。

如果没有指定短代码，你也可以配置默认的禁言列表。
默认禁言列表。例如，要使用默认短代码为 spam，请执行以下命令：

```
!mjolnir default spam
```

最后，从技术角度看，禁言列表只是一个普通的 Matrix 聊天室充满了审核隐藏信息，技术上称为事件。因此可以给它一个公共地址，让它像其他聊天室一样可以公开访问。


### 订阅禁言名单

禁言列表是一种巧妙的机制，它允许管理团队出于不同的动机禁言用户（例如，一个列表针对 "垃圾邮件"，一个列表用于 "coc"）。这种当多个社区希望共同合作时，这种区别就很有用。

并非所有社区都有类似的行为准则，但很多社区会对垃圾邮件的定义达成一致。能订阅另一个社区的垃圾邮件列表意味着你自己的社区将受到保护，不会受到其他社区已经遇到的垃圾邮件发送者的攻击。同时遵守不同的行为准则。

要订阅公共禁止列表，你需要检索该列表的地址。该列表在技术上只不过是一个 Matrix 聊天室，其地址采用通常的 `#room_name:server.tld` 格式。

然后，要让 mjolnir 跟随这个列表，你需要在它的控制室里发出以下命令

```
!mjolnir watch <room alias/ID>
```

例如，要订阅由 Matrix.org 基金会维护的 `#matrix-org-hs-tos-bl:matrix.org` ban list 的禁言列表，你需要发出以下命令

```
!mjolnir watch #matrix-org-hs-tos-bl:matrix.org
```