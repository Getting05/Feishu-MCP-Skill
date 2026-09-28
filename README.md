# Feishu MCP Skill

A reusable authoring skill for creating and editing high-quality Feishu/Lark Docs and Wiki pages through MCP/plugin tools.

The repository currently contains one skill:

- `SKILL.md` — Feishu Document Authoring

The skill is designed for workflows such as:

- translating academic papers into Feishu
- writing technical notes and project documentation
- editing existing Feishu Wiki/Docs pages
- preserving equations with native equation rendering
- using native heading/list/table/code/image blocks instead of plain-text approximations
- verifying document structure after writing

## Main principles

1. **Native blocks first** — use headings, equations, tables, lists, code blocks, images, and links as native Feishu structures whenever possible.
2. **Equations must render properly** — use Feishu equation elements/blocks with LaTeX/KaTeX content rather than Unicode/plain-text formulas.
3. **Do not dump plain text first** — parse the source structure before writing and construct the correct blocks from the beginning.
4. **Use the Docx/OpenAPI when needed** — convenience paragraph tools are only for simple prose; rich technical content should use lower-level APIs that preserve semantics.
5. **Read back and verify** — after substantial edits, re-read the document and confirm equations, hierarchy, tables, figures, and ordering are correct.

## Skill entry point

The complete instructions are in [`SKILL.md`](./SKILL.md).

The YAML frontmatter in `SKILL.md` describes when the skill should be activated:

```yaml
---
name: feishu-document-authoring
description: Create, translate, edit, and restructure Feishu/Lark Docs and Wiki pages using native document blocks and rich-text elements. Use this skill whenever working with Feishu documents, especially technical papers, formulas, tables, figures, code, and long structured content.
---
```

## Intended tool split

The skill assumes the Feishu MCP/plugin provides capabilities such as reading documents, updating blocks, appending content, and calling the Feishu Docx/OpenAPI.

A useful division of responsibility is:

- **MCP/plugin tools**: expose Feishu capabilities and individual API operations.
- **Tool descriptions**: state what each individual tool should and should not be used for.
- **`SKILL.md`**: defines the end-to-end authoring workflow and quality standard.

## Example

For a paper containing

```text
ĉ_smooth(q,t)=q_t−q_{t−1}
```

the skill instructs the agent to write a native equation element containing:

```latex
\hat{c}_{\mathrm{smooth}}(q,t)=q_t-q_{t-1}
```

rather than leaving the expression as plain text.

## Repository

This repository is intended to evolve together with the Feishu MCP workflow. Additional conventions for paper translation, experiment reports, and project documentation can be added to `SKILL.md` or separated into additional skills later.
