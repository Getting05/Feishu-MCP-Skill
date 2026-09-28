---
name: feishu-document-authoring
description: Create, translate, edit, and restructure Feishu/Lark Docs and Wiki pages using native document blocks and rich-text elements. Use this skill whenever working with Feishu documents, especially technical papers, formulas, tables, figures, code, and long structured content.
---

# Feishu Document Authoring

## Purpose

Use this skill whenever creating, translating, editing, restructuring, or formatting content inside Feishu/Lark Docs or Wiki documents.

The goal is not merely to insert text. The goal is to produce a document that looks and behaves like it was authored natively in Feishu.

## Core principle

Prefer native Feishu document structures over plain-text approximations.

Do not represent structured content as ordinary paragraphs when Feishu provides a native representation.

Prefer:

- headings → native heading blocks
- equations → native equation elements or equation blocks
- ordered/unordered lists → native list blocks
- tables → native table blocks
- code → native code blocks
- quotes → native quote blocks
- images → native image blocks
- links → rich-text links

Use plain-text paragraph helpers only for genuinely simple prose or when no structured API is available.

## Required workflow

When a user provides a Feishu Wiki or Doc URL and asks to create or modify content:

1. Resolve the Wiki URL to the underlying document/token when necessary.
2. Read the existing document before making non-trivial edits.
3. Inspect the current block hierarchy and identify the exact insertion or replacement location.
4. Parse the source content into semantic units before writing:
   - title
   - headings and subheadings
   - paragraphs
   - equations
   - lists
   - tables
   - figures/images
   - captions
   - code
   - references
5. Map each semantic unit to the best native Feishu representation.
6. Write the document using the simplest API that preserves those semantics.
7. Read the document again after substantial edits and verify the result.

For long documents, work section by section and verify incrementally.

## Do not use a plain-text-first workflow

If a document is known in advance to contain equations, headings, tables, figures, code, or other structured content, do **not** first dump the whole document as plain-text paragraphs and repair the formatting later.

Construct the correct native block structure from the beginning.

Bad workflow:

1. Append every paragraph as plain text.
2. Finish the whole document.
3. Discover formulas, headings, and tables render poorly.
4. Repair them one by one.

Preferred workflow:

1. Parse source structure.
2. Classify blocks.
3. Create native headings/equations/tables/images as content is written.
4. Verify rendering after each major section.

## Equation rendering

Mathematical expressions must be preserved semantically and rendered using Feishu's native equation representation whenever available.

Never use Unicode/plain-text approximations for mathematical expressions if equation elements are supported.

Bad:

```text
ĉ_smooth(q,t)=q_t−q_{t−1}
```

Preferred equation content:

```latex
\hat{c}_{\mathrm{smooth}}(q,t)=q_t-q_{t-1}
```

### Inline equations

Use inline equation elements for short symbols and expressions embedded inside prose.

Examples:

```latex
q_t
```

```latex
T_{\mathrm{world}\leftarrow\mathrm{base}}T_{\mathrm{base}\leftarrow i}
```

### Display equations

For important equations, long expressions, aligned equations, optimization objectives, matrices, sums, integrals, or piecewise functions, prefer a standalone/display equation block when the API supports it.

Example:

```latex
d_c=\begin{cases}
-d+\frac{1}{2}\eta, & d<0,\\
\frac{1}{2\eta}(-d+\eta)^2, & 0<d<\eta,\\
0, & \text{otherwise}.
\end{cases}
```

### Preserve notation exactly

Preserve:

- subscripts and superscripts
- hats, bars, tildes, dots, and primes
- Greek symbols
- sums, products, integrals, norms, and determinants
- matrices and vectors
- fractions
- cases/aligned environments
- SO(3), SE(3), and Lie-group notation
- transforms such as `T_{base \leftarrow i}`
- derivatives such as `\dot q`, `\ddot q`

Do not silently simplify mathematical notation into prose unless the user explicitly asks for a simplified explanation.

### Equation API strategy

If a high-level Feishu helper only accepts plain text, do not force equations through it.

Use the Feishu Docx/OpenAPI rich-text or block endpoint that supports an `equation` element. If the exact schema is unknown, inspect/search the Feishu API schema first.

After writing equations, re-read the affected blocks and verify that equations are actual equation elements rather than `text_run` content.

## Headings and hierarchy

Preserve semantic hierarchy using native heading blocks.

For academic papers, a typical hierarchy is:

```text
Title
Heading 1: 摘要
Heading 1: 1 引言
Heading 1: 2 相关工作
Heading 2: 2.1 逆运动学
Heading 2: 2.2 轨迹优化
Heading 1: 3 方法
Heading 2: 3.1 Solver
...
```

Do not represent section headings such as `III. PYROKI: MODULAR KINEMATIC OPTIMIZATION` as ordinary body text when a heading block is available.

Preserve the source section hierarchy unless the user explicitly requests restructuring.

## Tables

When the source contains a structured table, prefer a native Feishu table.

Do not flatten a table into a single paragraph such as:

```text
Method CPU GPU TPU Arm Hand Humanoid IK ...
```

Preserve:

- row and column semantics
- headers
- units
- percentages
- `mean ± std`
- check/cross indicators
- table numbers
- captions
- notes/footnotes

If native table creation is not available through the currently loaded convenience tool, use the lower-level Docx API if supported.

If the source itself contains an inconsistency between prose and a table, preserve both source values and annotate the inconsistency. Do not silently fix the paper.

## Figures and images

For figures in papers or technical documents:

1. Preserve the figure number.
2. Preserve or translate the caption as requested.
3. Keep the caption adjacent to the figure.
4. If the figure can be extracted or uploaded, insert it as a native image block rather than replacing it with a textual description.
5. Do not omit diagrams that carry methodological information.
6. If an image cannot be inserted, explicitly preserve the caption and state that the image itself was not inserted.

For documents derived from PDFs, inspect rendered pages when figures, diagrams, or tables contain information not represented correctly in extracted text.

## Lists, code, quotes, and links

Use native structures whenever possible:

- procedural steps → ordered list
- unordered items → bullet list
- shell/Python/YAML/JSON → code block
- quotations → quote block
- external references → rich-text link

Avoid using manually typed prefixes such as `1)`, `-`, or backticks as a substitute for native blocks when proper blocks are available.

## Technical paper translation

When translating an academic paper into Feishu:

- preserve the original organization
- preserve technical terminology
- preserve equations and notation
- translate figure and table captions
- preserve table values exactly
- preserve section numbering when useful
- normally keep bibliography entries in their original bibliographic language/form
- do not turn a full translation into a summary
- do not silently omit difficult paragraphs, figures, equations, or tables

For specialized terms, retain the English term on first occurrence when this helps precision, for example:

- 逆运动学（Inverse Kinematics, IK）
- 动作重定向（Motion Retargeting）
- Levenberg–Marquardt（LM）
- Jacobian
- Finite Scalar Quantization（FSQ）

When a source is supplied by the user, the translation must be grounded in that source. Do not silently replace source content with general knowledge.

## Block editing strategy

Before editing an existing document:

1. Read the document and obtain block IDs.
2. Locate the exact block(s) to edit.
3. Prefer targeted block updates over rewriting the entire document.
4. Preserve existing rich formatting when possible.
5. Be aware that replacing an entire text block may remove inline formatting/runs.
6. Use raw Docx APIs when high-level helper tools cannot preserve the required structure.

When making a structural rewrite, ensure old content is not accidentally duplicated.

## API preference

Use the simplest API that preserves the required semantics.

Preferred order:

1. High-level Feishu tool if it natively supports the desired structure.
2. Feishu Docx/OpenAPI for rich blocks, equation elements, tables, images, or formatting.
3. Plain-text paragraph helpers only as a fallback.

Do not choose an easier API if doing so materially degrades document quality.

If unsure how to construct a block or element:

1. search the Feishu API catalog/schema
2. inspect a known block of the desired type if available
3. make a small test update when safe
4. read the block back and verify its type and payload

## Verification checklist

After substantial document creation or editing, verify:

- headings are real headings
- equations are native equation elements/blocks
- tables are real tables when supported
- lists are real lists
- code is in code blocks
- figures are present when available
- captions are adjacent to figures/tables
- section order matches the intended structure
- no major source content was omitted
- no accidental duplicate content was introduced
- formulas retained symbols, subscripts, superscripts, and operators
- links resolve to the intended target

For equations specifically, read the edited block and confirm the API returns an `equation` element rather than only plain `text_run` content.

## Quality standard

A finished Feishu document should look like a document authored directly in Feishu, not raw text pasted through an API.

Before declaring the task complete, ask:

- Are headings actually headings?
- Are equations actually equations?
- Are tables actually tables?
- Are lists actually lists?
- Is code actually code?
- Are figures and captions handled correctly?
- Is the hierarchy readable?
- Did I verify the written result?

If the API supports a better representation and the answer to any relevant question is no, fix it before finishing.

## Tool-description guidance for Feishu MCP maintainers

If maintaining the Feishu MCP/plugin itself, add complementary constraints to tool descriptions.

For a plain paragraph helper:

```text
Use only for simple plain-text paragraphs. Do not use this function for equations, headings, tables, code blocks, images, or other structured document content when native Docx blocks/elements are available.
```

For the low-level Docx API tool:

```text
Prefer this API when creating or editing rich Feishu documents containing equations, tables, headings, lists, images, code blocks, or other structured block types that cannot be represented correctly through plain-text helpers.
```

The division of responsibility should be:

- tool descriptions: define what an individual tool is appropriate for
- this skill: define how to complete the end-to-end Feishu document task with good structure and rendering
