# 权限、结果与排错

## 两层权限

飞书接口授权和目标资源授权是两件事。此 MCP 只获取应用的 `tenant_access_token`：应用需先获相应 tenant scope 并发布生效，还需对目标知识库父节点、文档或文件夹有读/编辑权限。用户本人能打开某页，不保证应用可以在该页下新建子页面。`get_feishu_capabilities` 展示已提供的权限清单与接口元数据；它不是实时资源权限检查。

| 现象 | 处理 |
| --- | --- |
| `131006`、`node permission denied, tenant needs edit permission` | 应用缺少该 Wiki 父节点编辑权限。确认父节点及知识库成员/应用授权；不要持续重试或改写到别处。 |
| `99991672`、缺少 `base:app:read` 等 scope | 这是**应用身份**权限不足。用户身份的 `base:*` 授权不自动转给应用。请求启用相应 tenant 权限并发布；旧 Bitable 操作可查是否有 `bitable:app` 支持的对应 API。 |
| 明确提示 `user OAuth` 或接口目录只列 `user` token | 此服务不支持该接口。说明身份限制；不要把 `useUAT`、user token 或其他 MCP 的配置参数传入。 |
| `not found`、`131005` 等资源错误 | 先区分 Wiki `node_token`、Docx `document_id`、表格 `app_token`、`table_id`，并确认目标是否向应用开放。不要仅凭前缀判断类型。 |
| 接口不在目录、字段为 `null` | 查官方文档确认 URL、方法、参数、token 支持后，再考虑领域工具；`null` 不是“无需权限”。 |
| 内容写入超时或响应不确定 | 先读回确认。Wiki 页面创建后已返回 `node` 时不要再创建；继续相同的 Markdown 写入时复用返回的 `client_token`。 |

`list_feishu_events` 只有在 Worker `/feishu/events` 回调、校验密钥及飞书事件订阅均配置后才可能读到真实事件。空结果不意味着没有发生变更。分页由 `cursor`/`limit` 控制；读取事件本身不代表消费确认。

## 响应和大小限制

`call_feishu_api` 与领域工具成功时通常保留飞书原始 `{code,msg,data}`；领域接口的二进制下载会返回 `{base64,content_type,status}`。MCP 的 `isError: true` 表示工具失败。目录内 339 条接口是发现入口，不能视为 339 项线上成功验证。一次上传或响应上限 8 MiB；分片、分页、异步导出和复制任务要按官方文档处理。

此服务当前未对 `/mcp` 增加客户端身份校验。连接方如果能调用它，就可能使用应用权限执行写入或权限操作。将其作为对外共享入口时，应由部署方配置访问控制；Skill 不应索取或输出飞书应用密钥。
