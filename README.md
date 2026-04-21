# DESIGN.md

一种用于向编码智能体描述视觉识别的格式规范。DESIGN.md 为智能体提供了一个持久化、结构化的设计系统理解。

## 格式

DESIGN.md 文件将机器可读的设计令牌（YAML 前置事项）与人类可读的设计原理（Markdown 正文）相结合。令牌提供精确值，正文解释这些值存在的原因以及如何应用它们。

```md
---
name: Heritage
colors:
  primary: "#1A1C1E"
  secondary: "#6C7278"
  tertiary: "#B8422E"
  neutral: "#F7F5F2"
typography:
  h1:
    fontFamily: Public Sans
    fontSize: 3rem
  body-md:
    fontFamily: Public Sans
    fontSize: 1rem
  label-caps:
    fontFamily: Space Grotesk
    fontSize: 0.75rem
rounded:
  sm: 4px
  md: 8px
spacing:
  sm: 8px
  md: 16px
---

## 概述

建筑极简主义与新闻庄重感相结合。UI 唤起一种高级哑光质感——如同高端大报或当代画廊。

## 颜色

调色板以高对比度中性色和单一强调色为根基。

- **主色 (#1A1C1E):** 深墨色，用于标题和核心文本。
- **次色 (#6C7278):** 精致石板灰，用于边框、说明文字、元数据。
- **第三色 (#B8422E):** "波士顿陶土"——交互的唯一驱动色。
- **中性色 (#F7F5F2):** 温暖石灰岩底色，比纯白色更柔和。
```

阅读此文件的智能体将生成一个 UI：使用 Public Sans 字体的深墨色标题、温暖石灰岩背景和波士顿陶土行动号召按钮。

## 快速开始

验证 DESIGN.md 是否符合规范，捕获损坏的令牌引用，检查 WCAG 对比度比率，并以结构化 JSON 形式展示发现结果——所有这些都是智能体可以执行的操作。

```bash
npx @google/design.md lint DESIGN.md
```

```json
{
  "findings": [
    {
      "severity": "warning",
      "path": "components.button-primary",
      "message": "textColor (#ffffff) on backgroundColor (#1A1C1E) has contrast ratio 15.42:1 — passes WCAG AA."
    }
  ],
  "summary": { "errors": 0, "warnings": 1, "info": 1 }
}
```

比较两个版本的设计系统，以检测令牌级别和正文回归：

```bash
npx @google/design.md diff DESIGN.md DESIGN-v2.md
```

```json
{
  "tokens": {
    "colors": { "added": ["accent"], "removed": [], "modified": ["tertiary"] },
    "typography": { "added": [], "removed": [], "modified": [] }
  },
  "regression": false
}
```

## 规范

完整的 DESIGN.md 规范位于 [`docs/spec.md`](https://github.com/google-labs-code/design.md/blob/main/docs/spec.md)。以下是精简参考。

### 文件结构

DESIGN.md 文件有两层：

1. **YAML 前置事项** — 机器可读的设计令牌，以文件顶部的 `---` 围栏分隔。
2. **Markdown 正文** — 人类可读的设计原理，按 `##` 章节组织。

令牌是规范值。正文提供如何应用它们的上下文。

### 令牌模式

```yaml
version: <string>          # 可选，当前: "alpha"
name: <string>
description: <string>      # 可选
colors:
  <token-name>: <Color>
typography:
  <token-name>: <Typography>
rounded:
  <scale-level>: <Dimension>
spacing:
  <scale-level>: <Dimension | number>
components:
  <component-name>:
    <token-name>: <string | token reference>
```

### 令牌类型

| 类型     | 格式                                                                                                              | 示例               |
| :------- | :---------------------------------------------------------------------------------------------------------------- | :----------------- |
| 颜色     | `#` + 十六进制 (sRGB)                                                                                             | `"#1A1C1E"`        |
| 尺寸     | 数字 + 单位 (`px`, `em`, `rem`)                                                                                   | `48px`, `-0.02em`  |
| 令牌引用 | `{path.to.token}`                                                                                                 | `{colors.primary}` |
| 排版     | 包含 `fontFamily`, `fontSize`, `fontWeight`, `lineHeight`, `letterSpacing`, `fontFeature`, `fontVariation` 的对象 | 见上方示例         |

### 章节顺序

章节使用 `##` 标题。它们可以省略，但存在的章节必须按此顺序出现：

| #    | 章节       | 别名       |
| :--- | :--------- | :--------- |
| 1    | 概述       | 品牌与风格 |
| 2    | 颜色       |            |
| 3    | 排版       |            |
| 4    | 布局       | 布局与间距 |
| 5    | 高度与深度 | 高度       |
| 6    | 形状       |            |
| 7    | 组件       |            |
| 8    | 注意事项   |            |

### 组件令牌

组件将名称映射到一组子令牌属性：

```yaml
components:
  button-primary:
    backgroundColor: "{colors.tertiary}"
    textColor: "{colors.on-tertiary}"
    rounded: "{rounded.sm}"
    padding: 12px
  button-primary-hover:
    backgroundColor: "{colors.tertiary-container}"
```

有效的组件属性：`backgroundColor`, `textColor`, `typography`, `rounded`, `padding`, `size`, `height`, `width`。

变体（悬停、激活、按下）以单独的组件条目表示，使用相关的键名。

### 未知内容的消费者行为

| 场景             | 行为             |
| :--------------- | :--------------- |
| 未知章节标题     | 保留；不报错     |
| 未知颜色令牌名称 | 如果值有效则接受 |
| 未知排版令牌名称 | 作为有效排版接受 |
| 未知组件属性     | 接受并警告       |
| 重复章节标题     | 错误；拒绝文件   |

## CLI 参考

### 安装

```bash
npm install @google/design.md
```

或直接运行：

```bash
npx @google/design.md lint DESIGN.md
```

所有命令都接受文件路径或 `-` 作为标准输入。输出默认为 JSON。

### `lint`

验证 DESIGN.md 文件的结构正确性。

```bash
npx @google/design.md lint DESIGN.md
npx @google/design.md lint --format json DESIGN.md
cat DESIGN.md | npx @google/design.md lint -
```

| 选项       | 类型     | 默认值 | 描述                                    |
| :--------- | :------- | :----- | :-------------------------------------- |
| `file`     | 位置参数 | 必需   | DESIGN.md 的路径（或 `-` 表示标准输入） |
| `--format` | `json`   | `json` | 输出格式                                |

如果发现错误，退出代码为 `1`，否则为 `0`。

### `diff`

比较两个 DESIGN.md 文件并报告令牌级别的更改。

```bash
npx @google/design.md diff DESIGN.md DESIGN-v2.md
```

| 选项       | 类型     | 默认值 | 描述                     |
| :--------- | :------- | :----- | :----------------------- |
| `before`   | 位置参数 | 必需   | "之前" 的 DESIGN.md 路径 |
| `after`    | 位置参数 | 必需   | "之后" 的 DESIGN.md 路径 |
| `--format` | `json`   | `json` | 输出格式                 |

如果检测到回归（"之后" 文件中有更多错误或警告），退出代码为 `1`。

### `export`

将 DESIGN.md 令牌导出为其他格式（tailwind, dtcg）。

```bash
npx @google/design.md export --format tailwind DESIGN.md > tailwind.theme.json
npx @google/design.md export --format dtcg DESIGN.md > tokens.json
```

| 选项       | 类型                 | 默认值 | 描述                                    |
| :--------- | :------------------- | :----- | :-------------------------------------- |
| `file`     | 位置参数             | 必需   | DESIGN.md 的路径（或 `-` 表示标准输入） |
| `--format` | `tailwind` \| `dtcg` | 必需   | 输出格式                                |

### `spec`

输出 DESIGN.md 格式规范（可用于将规范上下文注入智能体提示）。

```bash
npx @google/design.md spec
npx @google/design.md spec --rules
npx @google/design.md spec --rules-only --format json
```

| 选项           | 类型                 | 默认值     | 描述                    |
| :------------- | :------------------- | :--------- | :---------------------- |
| `--rules`      | 布尔值               | `false`    | 附加活动 linting 规则表 |
| `--rules-only` | 布尔值               | `false`    | 仅输出 linting 规则表   |
| `--format`     | `markdown` \| `json` | `markdown` | 输出格式                |

## Linting 规则

Linter 针对解析后的 DESIGN.md 运行七条规则。每条规则产生固定严重级别的发现结果。

| 规则                 | 严重级别 | 检查内容                                                               |
| :------------------- | :------- | :--------------------------------------------------------------------- |
| `broken-ref`         | 错误     | 令牌引用 (`{colors.primary}`) 无法解析为任何已定义令牌                 |
| `missing-primary`    | 警告     | 颜色已定义但不存在 `primary` 颜色 — 智能体将自动生成一个               |
| `contrast-ratio`     | 警告     | 组件 `backgroundColor`/`textColor` 对比度低于 WCAG AA 最低标准 (4.5:1) |
| `orphaned-tokens`    | 警告     | 颜色令牌已定义但未被任何组件引用                                       |
| `token-summary`      | 信息     | 每个章节中定义的令牌数量摘要                                           |
| `missing-sections`   | 信息     | 当其他令牌存在时，可选章节（间距、圆角）缺失                           |
| `missing-typography` | 警告     | 颜色已定义但不存在排版令牌 — 智能体将使用默认字体                      |
| `section-order`      | 警告     | 章节出现在规范定义的规范顺序之外                                       |

### 程序化 API

Linter 也可作为库使用：

```typescript
import { lint } from '@google/design.md/linter';

const report = lint(markdownString);

console.log(report.findings);       // Finding[]
console.log(report.summary);        // { errors, warnings, info }
console.log(report.designSystem);   // Parsed DesignSystemState
```

## 设计令牌互操作性

DESIGN.md 令牌受 [W3C 设计令牌格式](https://www.designtokens.org/) 启发。`export` 命令将令牌转换为其他格式：

- **Tailwind 主题配置** — `npx @google/design.md export --format tailwind DESIGN.md`
- **DTCG tokens.json** ([W3C 设计令牌格式模块](https://tr.designtokens.org/format/)) — `npx @google/design.md export --format dtcg DESIGN.md`

## 状态

DESIGN.md 格式当前版本为 `alpha`。规范、令牌模式和 CLI 正在积极开发中。随着格式成熟，预计会有变化。

## 免责声明

此项目不符合 [Google 开源软件漏洞奖励计划](https://bughunters.google.com/open-source-security) 的条件。

---

> 汉化自 [https://github.com/google-labs-code/design.md](https://github.com/google-labs-code/design.md)
