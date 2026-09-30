---
name: feishu-mcp
description: 使用 Getting05 的 Feishu MCP 以应用身份读取、创建和编辑飞书知识库页面、云文档、多维表格及其他云空间资源。用户提供飞书链接或要求操作飞书文档、Wiki、表格、画板、评论或权限时使用；不适用于飞书消息、任务、通讯录及需要用户 OAuth 的操作。
---

# Feishu MCP

使用本 Skill 时，调用用户已连接的 [Feishu MCP](https://github.com/Getting05/FeishuMCP) 工具。服务地址是 `https://feishumcp.chengetting.workers.dev/mcp`。Skill 只指导工具选择，不负责安装另一套飞书 CLI，也不要求用户提供 App Secret。若工具不可用，先检查客户端是否已连接此 MCP；不要将其他飞书 MCP 的工具名或参数套用到这里。

## 身份与边界

- 服务统一使用 `tenant_access_token`。它只能访问应用获授权的资源；用户在飞书中有权限，不代表应用有权限。`get_feishu_capabilities` 显示的是权限和接口证据，不保证某个资源实测可访问。
- 新建知识空间、部分搜索与订阅接口仅支持用户令牌，此服务会拒绝已知的用户专属接口。不要尝试传 `useUAT`、切换 OAuth 或调用 `feishu-tool` 来绕过。
- 当前服务重点支持云文档、Wiki、Base/Bitable、画板、Sheets、Slides、Drive、Mindnote；没有消息、群聊、任务或通讯录工具。先确认 `tools/list` 中有目标工具，再行动。
- 只有用户授权的资源和操作才可写入。删除、转移权限等高影响操作先确定准确的资源 ID 和官方接口契约；不把模糊的名称匹配直接用于写入。

## 选择工具

| 任务 | 首选工具 |
| --- | --- |
| 读取 Wiki/Docx、查找某段文字 | `resolve_feishu_url`、`read_feishu_document`、`find_feishu_blocks` |
| 在已有 Wiki 页面下创建 Docx，可附带内容 | `create_feishu_wiki_page` |
| 列子页面；重命名、复制或移动节点 | `list_feishu_wiki_children`、`manage_feishu_wiki_page` |
| 新建云文档、文件夹、电子表格、多维表格或幻灯片 | `create_feishu_file` |
| 向 Wiki/Docx 添加标题、列表、表格、代码等 | `append_feishu_markdown` |
| 仅修改一个现有文本块；只加简单段落 | `update_feishu_text_block`；`append_feishu_paragraph` |
| 评论、权限、媒体、复杂块、表格记录及其他开放接口 | `search_feishu_api` → `call_feishu_api`，必要时使用对应 `feishu_*_api` |
| 读取已接收的事件 | `list_feishu_events`；先确认事件回调已配置 |

具体参数和其余工具分组见 [工具与示例](references/tools.md)。涉及公式、图片或学术论文时，读取 [原生文档写作](references/rich-documents.md)；遇到权限或接口错误时，读取 [权限与排错](references/access-and-errors.md)。

## 文档与知识库工作流

1. **定位资源。** `/wiki/<node_token>` 是知识库节点；`/docx/<document_id>` 是底层文档。可直接把这两类链接传给高层工具。调用 Docx 原始接口前，用 `resolve_feishu_url` 取得 `document_id`。不要假设 token 都有固定前缀。
2. **读取已有内容。** 编辑现有文档时先读 `read_feishu_document`；需要精确修改时用 `find_feishu_blocks` 定位 `block_id`。该读取工具主要返回文本和块 ID；复杂样式、嵌套结构或公式需用 Docx API 查看完整块。
3. **选择保留结构的写法。** 新增标题、列表、表格、代码优先用 Markdown 工作流；简单纯文本段落才用 `append_feishu_paragraph`。修改现有段落可用 `update_feishu_text_block`，但它会替换整块内联文本，可能丢失粗体、链接等样式；需要保留样式时用 Docx 块接口做定点更新。
4. **核对结果。** 较大范围的写入后读回目标文档，确认内容、顺序和块类型。`append_feishu_markdown` 保留旧内容；结构重写前检查旧块，避免重复。

创建知识库子页面时优先给 `create_feishu_wiki_page` 一个**父页面** URL、标题和可选 `markdown`。已有 `space_id` 但没有父页面时可用 `create_feishu_wiki_document`。若飞书返回父节点编辑权限不足，停止对该父节点重复尝试，并说明应用需要该节点的编辑权限。

Markdown 转换单次最多 1000 个块；长内容按章节分段写入。该工作流会拒绝图片，不会自动上传外部图片；图片需按官方媒体上传和 `replace_image` 流程操作。若新页面已创建而内容写入失败，返回值会包含 `node` 与 `client_token`：先读回该页面，后续同一写入重试复用该 token，勿重复创建页面。

## 扩展 API 工作流

先用 `search_feishu_api` 搜关键词、scope 或接口 ID，再用 `endpoint_id` 取得参数及官方文档链接。核对 `accessTokens`、请求方法、路径参数、查询参数、请求体与目标资源权限。`null` 表示元数据未知，不表示无限制。目录包含未做过真实租户验证的路由。

优先用 `call_feishu_api` 传 `endpoint_id`、`parameters` 及可选 `body`/`upload`。目录未覆盖或需要官方文档中的新字段时，用领域工具 `feishu_docx_api`、`feishu_wiki_api`、`feishu_bitable_api` 等，传**从 `/open-apis` 之后开始**的精确 `path`。GET 不带 body；JSON body 和 multipart upload 二选一。分页、异步任务、导出或分片上传要根据返回的游标与任务 ID 继续，不把第一步响应当最终结果。工具返回 `isError` 或飞书非零 `code` 时，先解释真实错误，再决定是否重试。

请求新功能、权限或事件时，不根据 scope 名称臆造接口。先用目录及官方文档确认该 API 支持应用身份；`base:*` 出现在 user 授权列表中不等于 tenant 已有授权。
