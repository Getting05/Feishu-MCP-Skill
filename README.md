# Feishu MCP Skill

这是 [Getting05/FeishuMCP](https://github.com/Getting05/FeishuMCP) 的配套 Skill，帮助 AI 在飞书知识库、云文档、云空间和多维表格中选择正确的 MCP 工具、保留文档结构，并识别应用身份的权限边界。入口为 [SKILL.md](SKILL.md)，Skill 名称为 `feishu-mcp`（旧版为 `feishu-document-authoring`）。

MCP 服务地址：`https://feishumcp.chengetting.workers.dev/mcp`。先在支持 Streamable HTTP MCP 的客户端连接该服务，再让客户端加载本 Skill。Skill 本身不安装另一套飞书 CLI，也不要求把 App ID/Secret 写进 Skill 文件。MCP Worker 的应用凭据由部署方配置。

可处理的典型请求：

- “在这个 Wiki 页面下新建一篇周报，并写入标题、列表和表格。”
- “找到文档中的旧结论，改成新结论，保留其他段落。”
- “列出知识库子页面，再把其中一篇重命名。”
- “创建一份云空间文档，读取并追加 Markdown 内容。”
- “查询可用的 Bitable/Sheets/Drive 接口，核对参数后调用。”
- “把技术论文翻译到飞书，保留公式、图表、代码和标题层级。”

`SKILL.md` 放通用流程；[工具与示例](references/tools.md) 列出 26 个工具和真实参数形式；[原生文档写作](references/rich-documents.md) 处理论文、公式和图片；[权限与排错](references/access-and-errors.md) 说明应用权限与常见错误。使用时只需读取当前任务相关的参考页。

服务只使用应用身份令牌。部分 API 仅支持用户身份；即使接口允许应用调用，目标资源也必须授权给应用。当前知识库父节点若返回 `131006`，需先为应用补充该节点的编辑权限；新版 Base 若返回 `99991672`，需开通对应的 **tenant** scope。Skill 不会把用户身份授权视为应用授权。

两个参考项目使用了不同的工具实现： [cso1z/Feishu-Skill](https://github.com/cso1z/Feishu-Skill) 用 `feishu-tool` CLI，[whatevertogo/FeiShuSkill](https://github.com/whatevertogo/FeiShuSkill) 用飞书官方 MCP。本仓库借鉴其模块化参考文档和示例组织方式，所有工具名与参数均以 Getting05 的 MCP 为准。
