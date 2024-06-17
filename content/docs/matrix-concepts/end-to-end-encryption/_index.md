+++
title = "端到端加密实施指南"
weight = 900
template = "docs/with_menu.html"
aliases = ["/docs/guides/end-to-end-encryption-implementation-guide", "/docs/legacy/e2e-implementation/"]

[extra]
updated = "2023-02-08T08:00:00Z"
meta_description = """
本指南面向希望添加端到端加密支持的 Matrix 客户端作者。
端到端加密支持的 Matrix 客户端作者。强烈建议读者熟悉
Matrix 协议和访问令牌的使用。
"""
+++

## 在 Matrix 客户端中实现端到端加密

本指南面向希望添加端到端加密支持的 Matrix 客户端作者。
端到端加密支持的 Matrix 客户端作者。强烈建议读者熟悉
Matrix 协议和访问令牌的使用。

### Olm/Megolm 实现

Matrix 中的端到端加密基于 Olm 和 Megolm 加密棘轮。建议客户端作者从 [vodozemac](https://github.com/matrix-org/vodozemac/)入手，其中包含所需的所有加密方法的实现。 [libolm](https://gitlab.matrix.org/matrix-org/olm) 也可供使用。

### 设备

我们对 "设备 "有特殊的理解。作为用户，我可能有几个设备（一个桌面客户端、一些网络浏览器、一个安卓设备、一个 iPhone、等等）。
当我第一次使用客户端时，它应该将自己注册为一个新设备。如果注销并以不同用户身份再次登录，客户端必须注册为新设备。
重要的是，客户端必须为每个 "设备 "创建一组新的密钥（见下文）

设备的寿命取决于客户端。在网络客户端中，我们每次登录都会创建一个新设备。在移动客户端中，如果登录会话过期，重复使用设备也是可以接受的、
**前提是**用户是相同的。
**切勿在不同用户之间共享密钥，这是绝对不允许的，非常危险的**。

设备通过 `device_id`（在特定用户范围内是唯一的）来识别。默认情况下，"/登录 "和"/注册 "端点将自动生成一个 `device_id` 并在响应中返回；客户端也可自由生成自己的 `device_id` 或如上所述重复使用一个设备，在这种情况下
在这种情况下，客户端应在请求正文中传递 `device_id`。

设备的生命周期与 "访问令牌" 密切相关。在简单的情况下，每次登录都会创建一个新设备，因此`device_id`和`access_token`之间存在一对一的映射关系。
如果客户端在重复使用 `device_id` ，就会有多个 `access_token` 与给定的 `device_id` 相关联。

但我们仍然希望只有一个访问令牌同时处于活动状态（虽然我们并不希望在同一时间有多个访问令牌同时处于活动状态）。

### 端到端加密中使用的密钥

加密通信中涉及到许多密钥：以下是它们的摘要。

摘要如下。

#### Ed25519 指纹密钥对

Ed25519 是一种用于签署信息的公钥加密系统。在 Matrix 中每个设备都有一个用于识别该设备的 Ed25519 密钥对。密钥对的私有部分永远不会离开设备，但公钥部分会发布到 Matrix 网络。

#### Curve25519 身份配对密钥

Curve25519 是一种公钥加密系统，可用于建立共享密钥。在 Matrix 中，每台设备都有一个长期有效的 Curve25519 身份密钥，用于建立 Olm 会话。
用于与该设备建立 Olm 会话。同样，私钥永远不会离开设备，但公钥部分会用Ed25519 指纹密钥签名并发布到网络上。

理论上，我们应该不时地轮换 Curve25519 身份密钥，但我们还没有实现这一点。

#### Curve25519 一次性密钥

除了身份密钥外，每个设备还会创建一些 Curve25519 密钥对。这些密钥对也用于建立 Olm 会话，但只能使用一次。
私人部分同样保留在设备上。

启动时，Alice 会创建一些一次性密钥对，并将它们发布到她的 homeserver 上。
如果 Bob 想与 Alice 建立 Olm 会话，他需要认领 Alice 的一个一次性密钥，并创建一个自己的新密钥。这这两个密钥以及 Alice 和 Bob 的身份密钥将用于用于在 Alice 和 Bob 之间的 Olm 会话。

#### Megolm 加密密钥

Megolm 密钥用于加密群组信息（事实上，它是用来推导 AES-256 密钥的，而 AES-256 密钥则是用于加密群组信息的。
它由随机数据初始化。每次发送信息时，都要对 Megolm 密钥进行散列计算，以得出下一条信息的密钥。

因此，可以与用户共享 Megolm 密钥的当前状态，允许他们解密未来的信息，但不能解密过去的信息。


#### Ed25519 Megolm 签名密钥对

发送方创建 Megolm 会话时，也会创建另一个 Ed25519 签名密钥对。这对密钥用于签署通过 Megolm 会话发送的信息，以验证发件人的身份。
同样，密钥的私人部分仍保留在设备上。公开部分则与聊天室里的其他设备共享。

### 创建和注册设备密钥

此过程只在设备首次启动时发生一次。

它必须创建 Ed25519 指纹密钥对和 Curve25519 身份密钥对。这需要调用 libolm 中的 `olm_create_account` 来完成。创建的（base64编码）密钥可通过调用`olm_account_identity_keys`来获取。账户应存储起来，以备将来使用。

然后，它应将这些密钥发布到 homeserver，通过使用的 `device_keys` 属性来完成。

[/keys/upload](https://matrix.org/docs/spec/client_server/r0.4.0.html#post-matrix-client-r0-keys-upload)

为了按照[签署 JSON](https://matrix.org/docs/spec/appendices.html#signing-json)中所述，客户应调用 `olm_account_sign`。

### 创建和注册一次性密钥

客户端应跟踪 homeserver 为其存储了多少个一次性密钥，并在必要时对其进行注册。

这可通过检查 `/sync/` 中的 `device_one_time_keys_count` 属性来实现。

libolm 支持的最大活动密钥数由 `olm_account_max_number_of_one_time_keys` 返回。客户端应尽量保持这个数量的一半左右。

生成新的一次性密钥

- 调用 `olm_account_generate_one_time_keys` 生成新密钥。
- 调用 `olm_account_one_time_keys` 获取未发布的密钥。这返回一个 JSON 格式的对象，其中包含一个属性 `curve25519`，它本身就是键 id 与键值的映射对象。
    例如

    ```json
    {
      "curve25519": {
        "AAAAAA": "wo76WcYtb0Vk/pBOdmduiGJ0wIEjW4IBMbbQn7aSnTo",
        "AAAAAB": "LRvjo46L1X2vx69sS9QNFD29HWulxrmW11Up5AfAjgU"
      }
    }
    ```

- 每个密钥的签名方式应与之前的身份密钥相同有效负载，并使用/keys/upload中的 "one_time_keys "属性上传。
  [/keys/upload](https://matrix.org/docs/spec/client_server/r0.4.0.html#post-matrix-client-r0-keys-upload)端点的 `one_time_keys` 属性上传。

- 调用 `olm_account_mark_keys_as_published` 来告诉 olm 库不要在以后的调用中返回相同的密钥。
  调用 `olm_account_mark_keys_as_published` 来告诉 olm 库不要在以后调用 `olm_account_one_time_keys` 时返回相同的密钥。

### 配置使用加密的聊天室

要在聊天室中启用加密，客户端应发送类型为
m.room.encryption`，内容为`{ "algorithm"： "m.megolm.v1.aes-sha2" }`。

### 处理 `m.room.encryption` 状态事件

当客户端收到上述 `m.room.encryption` 事件时，应设置一个
标记，以表明在聊天室中发送的信息应被加密。

如果随后发生的`m.room.encryption`事件改变了配置，该标记不应***被清除。
配置。这是为了避免 MITM 可以简单地要求参与者禁用加密。
参与者禁用加密。简而言之：一旦在聊天室中启用加密，就永远无法禁用。
聊天室中启用加密后，就永远无法禁用。

事件应包含一个`算法`属性，用于定义加密时应使用的加密算法。
算法。目前这里只允许使用 `m.megolm.v1-aes-sha2` 算法。
允许使用。

该事件还可包含其他设置，用于说明在聊天室内发送的信息
例如，"rotation_period_ms "用于定义会话更换的频率。
会话更换频率）。详情请参见规范。

### 处理 `m.room.encrypted` 事件

加密事件的类型是 `m.room.encrypted`。它们有一个内容属性
算法"，它给出了正在使用的加密算法，以及与算法[m.room.encrypted]特定的其他
属性[^1]。

加密的有效载荷是一个 JSON 对象，其属性为`type`（给出解密事件类型）、`type`属性、`type`属性和`type`属性。
解密事件类型）和 `content`（提供解密内容）属性的 JSON 对象。根据
根据使用的算法，有效载荷可能包含额外的密钥。

目前有两种已定义的算法：


### `m.olm.v1.curve25519-aes-sha2

规范提供了 [该算法的详细信息
](https://matrix.org/docs/spec/client_server/r0.4.0.html#m-olm-v1-curve25519-aes-sha2)
和[有效载荷示例
](https://matrix.org/docs/spec/client_server/r0.4.0.html#m-room-encrypted)
.

事件内容的 `sender_key` 属性给出了发送者的 Curve25519 身份密钥。
的身份密钥。客户端应为每台与之通话的设备维护一份已知 Olm 会话列表。
建议按 Curve25519 身份密钥对其进行索引。
密钥编制索引。

每个收件人设备的 Olm 信息都是单独加密的。密码文本
是接收设备的 Curve25519 身份密钥的对象映射。
当然，接收客户端应在此对象中查找自己的身份密钥。
对象。(如果没有列出，说明信息不是为其发送的，客户端
就无法解密；客户端应显示错误或类似信息）。

这将产生一个具有 `type` 和 `body` 属性的对象。信息
类型为 "0 "的信息是 "预钥 "信息，用于在两个设备之间建立新的 Olm 会话
类型'1'是普通报文，一旦会话收到报文就会使用。
在会话中接收到信息后就会使用。

当收到信息（无论哪种类型）时，客户端应首先尝试
解密。这有两个步骤
步骤：

- 如果（且仅当）`type==0`，客户端应调用
  olm_matched_inbound_session"。这将返回一个
  标志，表明信息是否使用该会话加密。
- 客户端会调用包含会话、`type`和`body`的`olm_decrypt`。如果调用
  成功，它将返回事件的明文。

如果客户端无法使用任何已知会话解密消息（或还没有已知的会话
且***报文类型为 0、
**并且***`olm_matches_inbound_session`在任何现有会话中都不为真、
  那么客户端就可以尝试建立一个新会话。具体方法如下：

- 调用 `olm_create_inbound_session_from` 使用 olm 帐户和
  消息的 `sender_key` 和 `body`。
- 如果会话已成功建立：
    - 调用 `olm_remove_one_time_keys` 以确保相同的一次性密钥
      不能被重复使用。
    - 使用新会话调用 `olm_decrypt`。
    - 存储会话以备将来使用。

在此过程结束时，客户端将有望成功解密
有效负载。

除了 `type` 和 `content` 属性外，明文有效载荷还应包含许多其他属性。
还包含许多其他属性。每个属性的检查方法如下
如下[^2]。

- 发件人 发件人的用户 ID。客户端应检查是否与事件中的
  发件人"。

- 收件人"： 收件人的用户 ID。客户端应检查
  与本地用户 ID 一致。

- keys"：属性为 "ed25519 "的对象。客户端应检查
  值与发件人的指纹密钥相匹配。
  将事件标记为已验证 时，客户端应检查该属性的值是否与发件人的指纹密钥一致。

- recipient_keys"：属性为 "ed25519 "的对象。客户端应检查
  该属性的值是否与自己的指纹密钥一致。

### `m.megolm.v1.aes-sha2

规范提供了[该算法的详细信息
](https://matrix.org/docs/spec/client_server/r0.4.0.html#m-megolm-v1-aes-sha2)
和[有效载荷示例
](https://matrix.org/docs/spec/client_server/r0.4.0.html#m-room-encrypted)
.

使用此算法加密的事件应具有 `sender_key`、`session_id` 和 `ciphertext` 内容属性。
内容属性。如果 `room_id`、`sender_key` 和
会话_id "对应于已知的 Megolm 会话（见下文），则密文
可以通过将密文传入 `olm_group_decrypt` 来解密。

为避免重放攻击，客户端应牢记 Megolm
事件的 `message_index` 返回值。
会话中解密的每个事件的 megolm `message_index` 返回值。如果客户端解密的事件的 `message_index` 与它已用该事件解密的事件的 `message_index` 相同，则该事件将被解密。
相同的事件，则客户端应将该消息视为无效。
消息视为无效。不过，当一个事件被解密
时，必须注意不要将其标记为重放攻击。例如
例如，当客户端对事件进行解密后，事件从客户端的缓存中清除，然后
事件从客户端缓存中清除，然后客户端回填并重新解密事件时，就可能发生这种情况。处理这种情况的一种
处理这种情况的一种方法是，确保在客户端缓存被清除时适当地清除 `message_index`es 记录。
事件记录。另一种方法是
是记住事件的 `event_id` 和 `origin_server_ts` 以及它的
消息索引"。当客户端解密的事件的 `message_index
相匹配的事件时，客户端就可以比较
事件_id "和 "源服务器ts"、
如果这些字段相匹配，则应按正常方式解密信息。

客户端应检查发件人的指纹密钥是否与
keys.ed25519 "属性。
将事件标记为已验证。

### 处理 `m.room_key` 事件

这些事件包含用于解密其他信息的密钥数据。它们
发送到特定设备，因此它们会出现在对 `GET /_matrix/client/r0/sync'的
响应的 "to_device "部分。它们也将被加密，因此
这些事件是由其他客户端生成的。
其他客户端生成--请参阅启动 megolm 会话)。

聊天室_id "和 "m.room_key "事件的 "发送者密钥
和 `session_id` 能唯一标识一个 Megolm 会话。如果
它们不代表一个已知会话，则客户端应通过调用 `session_id` 启动一个新的入站
会话。
会话密钥

客户机应记住
加密的 `m.room_key` 事件有效载荷的 keys 属性的值，并将其与入站会话一起存储。在将事件标记为
在将事件标记为已验证时使用。

### 下载聊天室内用户的设备列表

在发送加密信息之前，有必要检索聊天室内每个用户的设备列表。
设备列表。这项工作可以主动进行，也可以推迟到
直到发送第一条信息。用户还需要这些信息来验证或阻止设备。
用户验证或阻止设备。

客户端应使用 [/keys/query
](https://matrix.org/docs/spec/client_server/r0.4.0.html#post-matrix-client-r0-keys-query)
端点，在请求的 `device_keys` 属性中传递聊天室成员的 ID。
属性中传递聊天室成员的 ID。

客户机必须首先检查由/keys/query返回的 "设备键 "对象上的签名。
对象上的签名。
[](https://matrix.org/docs/spec/client_server/r0.4.0.html#post-matrix-client-r0-keys-query)返回的 `DeviceKeys` 对象的签名。
为此，客户端应删除 `signatures` 和 `unsigned` 属性，将剩余部分格式化为 Canonical JSON。
并将结果传入 `olm_ed25519_verify`、
参数使用 Ed25519 密钥，`signature`参数使用相应的签名。
参数的相应签名。如果签名检查失败
处理。

客户端还必须检查对象中的 `user_id` 和 `device_id` 字段是否与顶层映射中的
对象中的`user_id`和`device_id`字段与顶层 map[^3] 中的字段相匹配。

客户机应检查`user_id`/`device_id`是否与它以前看到过的设备相对应。
之前看到过的设备。如果是，客户端**必须**检查 Ed25519 密钥
没有更改。同样，如果已更改，则不应在设备上执行进一步处理。
处理。

否则，客户机将存储有关该设备的信息。

### 发送加密信息事件

在聊天室内发送信息时 
配置为使用加密 时，客户端
首先检查它是否有一个活动的向外 Megolm 会话。如果没有，则首先 
创建一个会话 starting-a-megolm-session。如果对外会话
则应检查是否到了[轮转]（#rotating-megolm-sessions）的时候、
并创建一个新会话。

然后，客户端会按如下方式创建加密有效载荷：

```json
{
  "type": "<event type>",
  "content": "<event content>",
  "room_id": "<id of destination room>"
}
```

并调用 `olm_group_encrypt` 加密有效载荷。然后将其打包到
事件内容：

```json
{
  "algorithm": "m.megolm.v1.aes-sha2",
  "sender_key": "<our curve25519 device key>",
  "ciphertext": "<encrypted payload>",
  "session_id": "<outbound group session id>",
  "device_id": "<our device ID>"
}
```

最后，通过以下方式将加密事件发送到聊天室
`PUT /_matrix/client/r0/rooms/<room_id>/send/m.room.encrypted/<txn_id>`.

### 开始 Megolm 会话

首次在加密聊天室发送信息时，客户端应启动一个新的
出站 Megolm 会话。为避免不必要的 Megolm 会话激增，不应主动****。
不必要的 Megolm 会话。

要创建会话，客户端应调用 `olm_init_outbound_group_session`、
并存储出站会话的详细信息，以备将来使用。

然后，客户端应调用 `olm_outbound_group_session_id` 获取新会话的唯一 ID，并调用 `olm_outbound_group_session_id` 获取新会话的唯一 ID。
和 `olm_outbound_group_session_key` 获取当前棘轮键和索引。
当前棘轮键和索引。它应将这些详细信息存储为入站（inbound
会话，就像 
通过 m.room_key 事件接收这些信息。

然后，客户端必须与聊天室内的每个设备共享该会话的密钥。
聊天室。因此，如果尚未下载设备列表，则必须 下载。
这样做。然后，它应该创建一个唯一的 `m.room_key` 事件，并将其加密发送
使用 Olm加密后发送给聊天室里每个未被阻止的设备。
未被阻止的设备。

一旦所有密钥共享事件的内容都汇集在一起，事件
应通过
`PUT /_matrix/client/r0/sendToDevice/m.room.encrypted/<txnId>`.

### 轮流使用 Megolm 会话

Megolm 会话不能无限期重复使用。定义会话轮换频率的参数是
参数在聊天室的 `m.room.encryption` 状态事件中定义。
事件中定义。

一旦达到信息限制或时间限制，客户端应
开始新的会话，然后再发送信息。

### 使用 Olm 加密事件

Olm 不适用于加密聊天室事件，因为它需要为每个设备单独拷贝一份
而且接收设备只能对收到的信息解密一次。
解密一次。不过，它可用于为 Megolm 的密钥共享事件加密。
事件进行加密。

使用 Olm 加密事件时，客户端应

- 按照 [spec
  ](https://matrix.org/docs/spec/client_server/r0.4.0.html#m-olm-v1-curve25519-aes-sha2)中的说明构建加密有效载荷。
- 检查是否有现有的 Olm 会话；如果没有，则 启动一个新会话
。如果它有多个会话（由于在建立会话时可能会发生竞赛
  会话），则应使用最后收到信息的会话。
  收到信息的会话。

启动一个 Olm 会话

- 调用 `olm_encrypt` 加密有效载荷。
- 将有效载荷打包成一个 Olm `m.room.encrypted` 事件。

### 启动 Olm 会话

要与另一台设备启动新的 Olm 会话，客户端必须首先申请另一台设备的
一次性密钥。为此，客户端应请求
[请求。
](https://matrix.org/docs/spec/client_server/r0.4.0.html#post-matrix-client-r0-keys-claim)。

客户端应检查响应中已签名密钥对象上的签名。
响应。与检查设备密钥上的签名一样，客户端应删除
签名 "和（如果存在）"未签名 "属性，将剩余部分格式化为
格式化为 Canonical JSON，并将结果传入 `olm_ed25519_verify`，使用
Ed25519设备密钥作为`key`参数。

如果密钥对象通过验证，客户端应将
密钥和远程设备的 Curve25519 身份密钥一起传入
olm_create_outbound_session`。

### 处理成员变更

客户端应监控配置为使用加密来处理成员变更的聊天室。
成员变更。

当成员离开聊天室时，客户端应使任何活动的出站
Megolm 会话，以确保用户下次发送信息时使用新会话。
信息。

当有新成员加入聊天室时，客户端应首先 下载设备列表、
如果还没有的话。

让用户有机会阻止
[](https://matrix.org/docs/spec/client_server/r0.4.0.html#device-verification)
任何可疑设备后，客户端应与所有新成员共享出站
Megolm 会话的密钥。共享方式与
与 创建新会话 的方法相同，只是不需要启动新的 Megolm 会话。
不需要启动新的 Megolm 会话：由于 Megolm 棘轮装置的设计，新用户只能访问所有设备、
新用户只能从当前状态开始解密信息。
状态开始解密。建议的方法是维护一份等待会话密钥的成员名单，并共享这些成员的会话密钥。
会话密钥的成员名单，并在用户下一次发送信息时共享这些密钥。

### 处理新设备

当用户在新设备上登录时，有必要确保任何聊天室内的其他
聊天室中启用了加密功能的其他设备都知道新设备，这样它们就可以共享它们的对外会话。
这样它们就可以像与新成员一样共享它们的出站会话。
成员。

应实施的设备跟踪过程已记录在案规范。
[规范](https://matrix.org/docs/spec/client_server/r0.4.0.html#tracking-the-device-list-for-a-user)。

###阻止/验证设备

用户应能将属于其他用户的每个设备标记为
用户应可通过 中详细说明的程序，将属于另一用户的每台设备标记为 "已阻止 "或 "已验证"。
[规范](https://matrix.org/docs/spec/client_server/r0.4.0.html#sending-encrypted-attachments)。

当用户选择阻止一个设备时，这意味着不再与该设备共享加密信息。
信息。简而言之，应排除
在开始新的 Megolm 会话时共享聊天室密钥。任何活动的出站
设备共享密钥的活动外向 Megolm 会话也应
失效，以便不再通过它们发送信息。

### 将事件标记为 "已验证

一旦设备通过验证，就可以确认事件是否从特定设备发送。
从特定设备发送的事件。请参阅处理 m.room.encrypted
事件部分，了解如何针对每种算法进行验证。从
可以在用户界面中进行装饰，以显示这些事件是从经过验证的设备发送的。
从已验证设备发送的事件。

## 加密附件

homeserver不能读取加密聊天室中共享的文件。客户端
应执行[规范
](https://matrix.org/docs/spec/client_server/r0.4.0#sending-encrypted-attachments)。

目前，文件是使用 AES-CTR 加密的，而 AES-CTR 并不包含在
libolm。客户端必须依赖第三方库。

##密钥共享

当一个事件因缺少密钥而无法解密时，客户端可能想
向其他可能拥有密钥的客户端申请密钥。同样，客户端可能
如果能确认请求设备被允许查看密钥，则客户端可能希望用相关密钥回复密钥请求。
请求设备允许查看用该密钥加密的信息。

这些功能可通过 `m.room_key_request` 和 `m.forwarded_room_request` 来实现。
事件来实现。

m.forwarded_room_key`事件的`session_key`属性不同于`m.room_key`事件的`session_key`属性。
事件不同，因为它不包括原始发送者的 Ed25519 签名。
签名。应从
olm_export_inbound_group_session "中的 "信息索引 "中获取，并可使用`m.room_key`恢复会话。
可以使用 `olm_import_inbound_group_session` 恢复会话。

`forwarded_room_key`属性一开始是空的，但每次密钥被转发到其他设备时，之前的密钥就会被删除。
转发到另一个设备时，链中的前一个发送方就会被添加到列表末尾。
列表的末尾。请看下面的例子：

> -   A -\> B : m.room\_key
> -   B -\> C : m.forwarded\_room\_key
> -   C -\> D : m.forwarded\_room\_key

在信息 B -\> C 中，"forwarded_room_key "是空的，但是在信息 C -\> D 中
中却包含了 B 的 Curve25519 密钥。为了让 D 相信会话来自 A
D 就必须信任直接发送者 C 和这个链中的每一个条目。

为了安全地实现密钥共享，客户端必须不回复它们收到的每一个密钥请求。
请求。建议的策略是
自动共享密钥。请求
来自未经验证设备的请求应提示对话框，允许用户
验证设备、不经验证共享密钥或不共享密钥（并忽略以后的请求）。
（并忽略以后的请求）。客户端还应检查
来自其他用户设备的请求是否合法。这可以通过
跟踪与哪些用户共享了会话，以及在哪个 "信息索引 "上共享了会话。[^3]

密钥请求可以发送到当前用户的所有设备、会话的原始发送者以及聊天室里的其他设备。[^1]
会话的原始发送者和聊天室里的其他设备。当
客户端收到请求的密钥后，应向所有向其请求密钥的设备发送 `m.room_key_request` 事件，并设置`action_key_request`事件。
事件，并将 `action` 属性设置为
action "属性设置为`"cancel_request"，`request_id "属性设置为初始请求的 ID。

[^1]: 请注意，编辑后的事件内容为空，因此其
内容将没有`算法`属性。因此，客户端在检查
算法 "属性。

[^2]: 这些测试可防止攻击者将他人的 curve25519 密钥
作为自己的密钥，并随后声称发送了信息，但其实并没有。
的信息。

[^3]: 这可防止恶意或被入侵的homeserver将设备的密钥替换为他人的密钥。
设备的密钥。 