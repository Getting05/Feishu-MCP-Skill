# 工具与调用示例

以下名称来自 Getting05/FeishuMCP 线上 1.1.0 的 26 个 MCP 工具。客户端显示的命名空间前缀可能不同；以 `tools/list` 返回的名称和输入 schema 为准。

## 常用操作

### 在 Wiki 父页面下创建并写入

```json
{"name":"create_feishu_wiki_page","arguments":{"parent_wiki_url_or_token":"https://example.feishu.cn/wiki/PARENT_NODE","title":"项目周报","markdown":"# 进展\n\n- 已完成联调\n\n| 项目 | 状态 |\n| --- | --- |\n| API | 完成 |"}}
```

返回的 `node.node_token` 标识 Wiki 页面，`document_id`/`node.obj_token` 标识底层 Docx。创建前确认应用可编辑父节点；创建后如内容写入失败，保存返回的页面信息，先读回再恢复。

### 修改已有文档

```json
{"name":"read_feishu_document","arguments":{"url_or_token":"https://example.feishu.cn/wiki/NODE"}}
{"name":"find_feishu_blocks","arguments":{"url_or_token":"https://example.feishu.cn/wiki/NODE","query":"旧段落"}}
{"name":"update_feishu_text_block","arguments":{"url_or_token":"https://example.feishu.cn/wiki/NODE","block_id":"BLOCK_ID","text":"完整的新段落"}}
{"name":"append_feishu_markdown","arguments":{"url_or_token":"https://example.feishu.cn/wiki/NODE","markdown":"## 新章节\n\n追加内容。"}}
```

`update_feishu_text_block` 替换目标块全部内联文本。`append_feishu_paragraph` 接收 `paragraphs` 字符串数组，适合纯文本；`append_feishu_markdown` 支持 `parent_block_id`、`index`（默认 -1）和可选的幂等 `client_token`。

### 新建云空间文档或其他文件

```json
{"name":"create_feishu_file","arguments":{"file_type":"docx","title":"会议纪要"}}
{"name":"create_feishu_file","arguments":{"file_type":"sheet","title":"数据表","folder_token":"FOLDER_TOKEN"}}
```

`file_type` 可选 `docx`、`sheet`、`bitable`、`slides`、`mindnote`、`folder`。`mindnote` 仅能在可编辑 Wiki 空间中创建；Wiki 模式传 `space_id`，必要时加 `parent_node_token`；云空间可传 `folder_token`。不能混用 Wiki 与文件夹位置参数。Slides 云空间创建目前不接收 `folder_token`。

## 26 个工具分组

| 组 | 工具 | 用途 |
| --- | --- | --- |
| 发现与目录 | `get_feishu_capabilities`、`search_feishu_api`、`call_feishu_api` | 身份与权限证据、查接口、按目录调用 |
| Wiki | `list_feishu_wiki_spaces`、`list_feishu_wiki_children`、`create_feishu_wiki_document`、`create_feishu_wiki_page`、`manage_feishu_wiki_page` | 空间与节点管理 |
| Docx | `resolve_feishu_url`、`read_feishu_document`、`find_feishu_blocks`、`update_feishu_text_block`、`append_feishu_paragraph`、`append_feishu_markdown` | URL 解析、读取、定点编辑和追加 |
| 云空间 | `create_feishu_file` | 创建云文档、表格、文件夹等 |
| 事件 | `list_feishu_events` | 读取已验签且持久化的事件；需先配置回调 |
| 原始领域 API | `feishu_base_api`、`feishu_bitable_api`、`feishu_board_api`、`feishu_docs_api`、`feishu_docx_api`、`feishu_drive_api`、`feishu_mindnote_api`、`feishu_sheets_api`、`feishu_slides_api`、`feishu_wiki_api` | 调用对应飞书 OpenAPI |

`manage_feishu_wiki_page` 的 `action` 为 `rename`、`copy` 或 `move`：重命名提供 `title`；复制/移动提供 `target_parent_wiki_url_or_token`。列子节点时使用 `page_token` 翻页。复制等接口可能返回异步任务 ID，按目录中的查询任务 API 跟进。

## 其他资源的操作顺序

- **Bitable / Base：** 先确认 `app_token` 和 `table_id`，读取表与字段定义，再查询或修改记录。旧版多维表格走 `feishu_bitable_api`；新版 Base v3 走 `feishu_base_api`，但后者可能还缺 tenant `base:*` 权限。字段值结构以对应 API 文档为准，不把显示名称当成稳定 ID。
- **Sheets：** 先查电子表格元信息与工作表 ID，再读取或写入指定区域。区分云空间文件 token、spreadsheet token 和 sheet ID。
- **Drive、Docs、Wiki 评论与权限：** 先读取目标文件/节点及现有成员或设置，再调用目录中有文档佐证的增删改接口。权限变更、移动和删除会影响他人访问，需使用用户明确指定的资源与对象。
- **画板、幻灯片、思维笔记：** 查目录中的对应领域接口和素材规则；`board:whiteboard:node:update` 有 scope 不代表存在已验证的原位更新 API。画板创建节点的 POST 不能当作修改现有节点。
- **事件：** `list_feishu_events` 只读取已进入服务端收件箱的事件；未配置飞书回调和订阅时先处理配置问题。

## 按目录调用其他接口

```json
{"name":"search_feishu_api","arguments":{"query":"wiki:node:move"}}
{"name":"search_feishu_api","arguments":{"endpoint_id":"检索得到的接口 ID"}}
{"name":"call_feishu_api","arguments":{"endpoint_id":"检索得到的接口 ID","parameters":{"space_id":"SPACE_ID","node_token":"NODE_TOKEN"},"body":{"target_space_id":"TARGET_SPACE_ID","target_parent_token":"TARGET_NODE"}}}
```

上面最后一条是参数结构示意；调用前以实际接口 ID 返回的 schema 和官方文档核对方法及请求体。`parameters` 是路径/查询参数，`body` 是 JSON 请求体。`call_feishu_api` 是单次请求，不自动翻页或等待异步任务。

原始领域工具统一接收 `method`、`path`、可选 `body` 或 `upload`：

```json
{"name":"feishu_wiki_api","arguments":{"method":"GET","path":"/wiki/v2/spaces/SPACE_ID/nodes?page_size=50"}}
```

`upload` 结构为 `{field, filename, content_type, base64, fields?}`；上传和下载单次上限 8 MiB，下载结果以 base64 返回。`feishu_slides_api` 也接收 `/slides_ai/` 路径，`feishu_docs_api` 也接收 `/docs_ai/`。使用官方文档的准确 URL、字段和值；不要照搬另一个飞书 MCP 的 `path`/`params`/`data`/`useUAT` 结构。
