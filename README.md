🤖 Telegram 双向转发客服机器人 (Cloudflare Worker)

本项目基于 Cloudflare Worker 和 Key-Value (KV) 存储构建，旨在为 Telegram 机器人提供一套高效的私聊用户与管理员群组话题之间的双向通信中继系统。它具备强大的反垃圾和自动化功能，极大地提升了客服效率。

✨ 核心功能亮点

图标

功能名称

描述

💬

私聊 - 话题双向中继

用户私聊消息转发至管理员群组的独立话题，管理员在话题内回复即可中继给用户。

🏷️

话题名动态管理

话题名根据用户昵称和 ID 自动生成并更新，方便识别和查找用户。

🛡️

人机验证 (Captcha)

新用户需通过问答验证才能开始使用，有效阻止恶意机器人和垃圾信息。

✏️

已编辑消息通知

用户修改消息时，自动在话题中通知管理员修改前后的内容。

❌

一键屏蔽与解封

管理员可通过话题卡片上的内联按钮，一键屏蔽/解除屏蔽用户。

🗑️

关键词自动屏蔽

可配置关键词列表和阈值，对发送垃圾内容的用户进行自动封禁。

⚙️

关键词自动回复

对常见问题设置关键词自动回复，减轻管理员日常压力。

🖼️🔗

内容类型精细过滤

精确控制转发的消息类型（如图片、链接、纯文本、频道转发），避免接收不必要的内容。

🛠️ 部署前准备

1. Telegram 设置

创建机器人：通过 @BotFather 创建新的机器人，并获取您的 Bot Token。

创建管理员群组：创建一个超级群组，并开启“话题模式/Forum”。

获取群组 ID：将机器人添加为该群组的管理员。群组 ID 通常以 -100 开头。

2. Cloudflare 环境

创建 Worker：在 Cloudflare 控制台创建一个新的 Worker 服务。

创建 KV 命名空间：创建一个 KV 存储空间，例如命名为 TG_BOT_KV，用于持久化存储用户状态、话题 ID 和屏蔽记录。

⚙️ 环境变量配置 (Worker Variables)

请在 Cloudflare Worker 的设置页面配置以下环境变量。

1. 核心与存储配置（必填）

变量名称

描述

示例值

BOT_TOKEN

您的 Telegram 机器人 Token。

123456:ABC-DEF_GHI-JKL

ADMIN_GROUP_ID

管理员群组 ID (必须是话题模式)。

-1001234567890

KV 绑定

绑定您创建的 KV 命名空间，命名为 TG_BOT_KV。

TG_BOT_KV

2. 验证与欢迎信息（可选）

变量名称

描述

默认值

VERIFICATION_ANSWER

人机验证的正确答案。建议将答案写在 Bot 简介中。

"3"

VERIFICATION_QUESTION

发送给新用户的验证问题。

(代码默认)

WELCOME_MESSAGE

用户发送 /start 时收到的欢迎语。

(代码默认)

3. 关键词过滤与反垃圾配置（可选）

变量名称

描述

格式说明

KEYWORD_RESPONSES

自动回复规则。

关键词1|关键词2===回复内容\n关键词3===回复内容2

BLOCK_KEYWORDS

自动屏蔽关键词。

关键词1|关键词2\n关键词3 (每一行或使用 `

BLOCK_THRESHOLD

关键词触发次数达到此阈值后，用户将被自动屏蔽。

5 (数字)

4. 消息类型转发开关（可选）

所有未设置或设置为非 false 的值，都默认为 启用 (true)。设置为 false 即可禁用。

变量名称

描述

默认值

ENABLE_IMAGE_FORWARDING

转发图片 (Photo/Picture) 类型的消息。

true

ENABLE_LINK_FORWARDING

转发包含 URL 或 text_link 实体的消息。

true

ENABLE_TEXT_FORWARDING

转发纯文本消息。

true

ENABLE_CHANNEL_FORWARDING

转发来自 Telegram 频道的转发内容。

true

🚀 部署步骤

将 src/index.js 中的代码复制到您的 Cloudflare Worker 编辑器中。

在 Worker 设置中配置好所有的环境变量和 KV 绑定。

点击 保存并部署。

设置 Webhook：使用以下 URL 调用 Telegram API，将您的 Worker URL 绑定到机器人。

# 替换 [您的BOT_TOKEN] 和 [您的Worker的URL]
[https://api.telegram.org/bot](https://api.telegram.org/bot)<您的BOT_TOKEN>/setWebhook?url=<您的Worker的URL>


示例：https://api.telegram.org/bot123456:ABC-DEF/setWebhook?url=https://my-tg-worker.workers.dev

💾 KV 存储结构（高级维护）

Worker 通过以下 Key 格式在 TG_BOT_KV 中存储关键数据：

KV Key 格式

存储内容

描述

user_state:<user_id>

用户的验证状态 (verified, pending_verification 等)。



user_topic:<user_id>

用户对应的管理员群组话题 ID。



topic_user:<topic_id>

话题 ID 对应的私聊用户 ID (反向查找)。



is_blocked:<user_id>

用户的屏蔽状态 ("true" 表示被屏蔽)。



block_count:<user_id>

自动屏蔽关键词的触发计数。



msg_data:<user_id>:<msg_id>

存储消息原始内容和时间，用于处理消息编辑。

