+++
title = "Matrix 即时通讯"
weight = 100
template = "docs/with_menu.html"
aliases = ["/docs/", "/docs/chat_basics/"]
[extra]
updated = "2023-06-16T16:00:00Z"
meta_description = """
Matrix 是一种开放式协议，许多应用程序都可以使用它来进行安全的、去中心化的
通信。使用 Matrix 的第一步就从这里开始。
"""
+++

## Matrix 是什么？

Matrix 的工作原理有点像电子邮件，但它是即时和安全的：

- 你需要在提供者处注册一个账户
- 无论你的服务提供者是谁，你都可以与使用其他服务提供者的人通话
- 就像你可以用同一个电子邮件账户使用 Outlook 或 Thunderbird 一样，你也可以用不同的 Matrix
  账户使用 Outlook 或 Thunderbird 的方式一样，你也可以为同一个 Matrix 账户使用不同的 Matrix 应用程序。

有几种应用程序，但为了简单起见，我们将使用 Element。
因为它是市场上功能最全的 Matrix 应用程序之一。

一旦你对基本操作比较熟悉，如果你想使用其他
应用程序，请访问本网站的 [客户端](/ecosystem/clients) 部分。

{{ page_card(
    title="客户端",
    path="/ecosystem/clients",
    summary="当你熟悉了基础知识后，可以前往了解 Matrix 客户端，选择最适合你需求的客户端。")
}}

## 创建 Matrix 账户

你可以使用任何你想要的提供者。专家甚至可以设置自己的提供者，但这并不是必须的。值得注意的是时，你可以使用第三方工具将聊天室所有权迁移到你的新账户。

Matrix.org 基金会是一个公共提供者，每个人都可以在上面免费注册一个账户。对于你的第一步，最简单的方法就是在那里注册一个账户。

要注册账户，你需要使用一个应用程序。在我们的例子中，选择了 Element 客户端，但实际上，你可以使用任何客户端，也可以选择某个客户端后，迁移到其他客户端。

访问 [app.element.io](https://app.element.io)，点击 "创建账户"。
你将进入以下页面。

{{ 
    figure(
        img="../element-io-sign-up.png",
        caption="app.element.io 的登录页")
}}


为简单起见，你可以使用 Google、Facebook、Apple、GitHub 或 GitLab 的账户直接登录。登录成功后，会自动创建一个 Matrix 账户。这被称为"社交登录"。

如果你比较注重隐私，也可以在下面的表格中输入用户名、密码和电子邮件进行注册。

你可能会遇到验证码，并被要求接受 Matrix.org 的隐私条款和用户协议。

接受条款后，最后会出现一个屏幕，要求你确认电子邮件地址。你可以放心地关闭此窗口。

检查收件箱，然后单击链接验证电子邮件地址。该链接将将带你进入 Matrix 网络应用程序 Element 的主页，以参与 Matrix 对话。

{{ figure(
    img="../element-landing-page.png"
    caption="登录成功后，进入到 loading 页面")
}}

你现在拥有一个账户，正在使用网页版的 Element。我们建议下载[桌面版Element](https://element.io/download)、这样可以更方便地跟踪 Matrix 链接。如果你不想下载桌面应用程序，可以继续使用web。

现在，你可以选择创建一个私人群聊来进行测试，然后邀请朋友，或者加入公共社群参与已有的对话。
