---
name: design-doc-coach
description: '从 1 句话定位出发,通过 5 维度采访产出 <project>_design.md 0.3 版草稿(章节骨架 + 锁定决策 + AC 拆分 + 风险登记 + 模块布局)。当用户说"开新项目 X"、"写设计文档"、"先出个 design doc"、"我要开新坑"、"立项"时调用。不在已有完整设计文档(§1 + §2 + §6 三块齐)的项目上调用 — 那种情况跑 spec-driven-bootstrap。本 SKILL 不写实现代码,也不替用户做技术栈决策。'
allowedPrompts: [{tool: "Write", prompt: "生成项目设计文档 <project>_design.md(含 12 节骨架 + TODO 标记)"}]
---

# design-doc-coach

## 何时调用

**触发关键词**(任一):
- 用户说"开新项目 X / 开新坑 / 立项 / 起个项目"
- 用户说"写设计文档 / 出个 design doc / 做个架构设计"
- 用户说"我想搞 XXX,先出个方案"
- 用户粘贴了 1 句话定位,问"怎么往下走 / 怎么落地"

**何时 NOT 调用**:
- 目标项目已有完整 `<project_name>_design.md` 含 §1 + §2 + §6 → 跑 `spec-driven-bootstrap`
- 用户只是问"如何改一段代码" / "实施某个 Story" → 跑项目专属 `<project>-spec-driven-dev/SKILL.md`
- 用户的提问明显是另一个项目的元操作(比如 lingshu 仓管理 / 装环境)→ 不要误触发

## 核心原则(3 条不妥协)

1. **单一职责**:只做采访 + 草稿,**不写代码不做架构决策**
2. **不替用户决策**:技术栈 / 模块结构 / 风险阈值等锁定项必须由用户在采访中显式确认;SKILL 只提供选项 + 整理
3. **草稿必须留 TODO**:所有用户未填项必须标 `<!-- TODO: <你确认> -->`,审稿时一眼能看到未决项

## 5 维度参数(采访提纲)

| # | 维度 | 必问项 | 选填项 |
|---|---|---|---|
| 1 | **项目元数据** | 名称(显示/文件 slug)/ 一句话定位 / 目标用户 | 当前阶段 / 是否已有仓库 |
| 2 | **技术栈** | 语言版本 / 构建工具 / 核心框架 | 测试框架 / 关键依赖锁定 |
| 3 | **模块结构** | monorepo / multi-repo / single-module / flat | 主要子模块清单(若适用)|
| 4 | **拆分原则** | 迭代单位(Story/Iteration/Spec/Sprint)/ id 模式 / 单迭代粒度上限(文件数/提交数)| 一句话定位示例 |
| 5 | **验证手段 + 风险阈值** | AC 黑盒测试方式 / 单测+集成+E2E 金字塔 / 性能基线 / R-XX 入宪章的分值阈值 / 风险维度类别 | lint/format/CI 卡点 / 已识别的高风险项 |

> 注:小型工具类项目可裁剪到 1+2+4 三维度,SKILL 在 Step 1 询问"是否走精简版"。

## 工作流(5 步)

### Step 0:开场 - 收 1 句话定位

如果用户没给,主动问(用纯文本或 AskUserQuestion 单题二选一):

> "请用 1-2 句话告诉我项目是做什么的、给谁用、解决什么痛点。"

这一步不阻塞,允许用户只给关键词(动词 + 宾语),**但**至少需要 1 个核心定位词。

### Step 1:5 维度采访(用 `AskUserQuestion` 强制结构化)

按 5 维度表分批问。每个维度 2-4 个选项 + Other 让用户自定义。**不要把采访问题塞进 prompt**,必须走交互。

**示例(维度 1:项目类型)**:
- Q: 项目类型?
  - 选项:后端服务 / CLI 工具 / 库 / Web 前端 / Agent / 其他

**示例(维度 2:主语言)**:
- Q: 主语言?
  - 选项:Java / Python / TypeScript / Go / Rust / 其他

**示例(维度 4:迭代单位)**:
- Q: 迭代单位叫什么?
  - 选项:Story / Iteration / Task / Spec / Sprint / 其他

⚠️ **不要问"为什么"**:5 维度都是事实收集,不解释动机。"为什么用 Spring"留给用户自己写进 `_design.md §1`。

### Step 2:章节骨架选择

根据项目复杂度选模板:

| 复杂度 | 模板 | 章节数 | 适用场景 |
|---|---|---|---|
| 中大型(多人协作 / 3 个月+)| **标准 12 节** | 12 | **默认** |
| 小型(个人工具 / 概念验证)| **精简 6 节** | 6 | 用户明确说"小项目"才用 |

**12 节标准模板**(默认):
```
§0   项目一句话定位(放在最前,任何读者 30s 内 get)
§1   项目原则(锁定决策清单,带"为什么")
§2   技术栈(语言 / 构建工具 / 依赖表 + 版本锁)
§3   模块结构(目录树 + 依赖方向)
§4   核心接口 / 数据结构(关键类型契约)
§5   扩展点(可插拔抽象清单 — 各语言实现机制不同:Java SPI/Spring SPI / Python entry_points / Go interface / Rust trait / Node package exports 等)
§6   AC 总账(黑盒验收清单,每条对应一个未来 Story / 迭代)
§7   NFR(性能 / 可靠性 / 可观测性基线)
§8   错误码约定(命名 + 类别 + 上限)
§9   兼容矩阵(JDK / Python / Node 版本,SemVer 策略)
§10  依赖表(锁定 + 风险标注,版本号到 patch)
§11  风险登记(每条 R-XX + 分值 + 缓解方案)
§12  Glossary(术语表,精简 ≤ 20 项)
```

**6 节精简模板**(用户明确要"小"才用):
```
§0  一句话定位
§1  技术栈 + 原则(合并)
§2  模块结构 + 核心接口 + 扩展点(合并)
§3  AC 总账 + NFR(合并)
§4  错误码 + 兼容矩阵(合并)
§5  风险登记 + Glossary(合并)
```

### Step 3:输出 `<project_name>_design.md` 0.3 版

渲染规则:
- 输出到用户指定目录,文件名 `<project_slug>_design.md`(kebab-case)
- 顶部加 frontmatter(项目元数据):
  ```yaml
  ---
  name: <project_slug>
  version: 0.3-draft
  status: awaiting-review
  generated-by: design-doc-coach v1.0
  last-updated: <YYYY-MM-DD>
  ---
  ```
- 每个章节底部留 `<!-- TODO: <你确认> -->` 标记未填项
- **不引入新决策**:每个锁定项必须来自 Step 1 采访
- 每个锁定项旁标 `[from-interview: Q<n>-opt<m>]` 让用户追溯来源

⚠️ **不要在这一步补未在采访中出现的细节**(例如"我们一定要用 Kafka")。采访没问 → 标 TODO,**不要补默认**。

### Step 4:留 TODO 让你审稿

输出末尾加:

```markdown
---

## 审稿 TODO(你做完这些再跑 spec-driven-bootstrap)

- [ ] 把 `<!-- TODO -->` 标记全替换为确认内容(优先 §1 §2 §6)
- [ ] §6 AC 总账补到至少 5 条黑盒验证项
- [ ] §11 风险登记补到至少 3 条 R-XX,每条带分值 + 缓解方案
- [ ] 确认 §2 / §10 的版本锁定是当前实际值(grep 一下当前 lock 文件)
- [ ] 跑 `git diff <prev_design>.md` 看本轮改动(若有旧版)
- [ ] 改 frontmatter `status: awaiting-review → ready-for-bootstrap`
```

## 反模式(看到立刻停)

| ❌ 反模式 | 为什么不许 |
|---|---|
| 替用户做技术选型(例如默认选 Spring Boot)| 锁定决策必须用户显式确认,SKILL 只提供选项 |
| 跳过 Step 1 采访,直接问用户"写完整描述"| 5 维度的结构化采访才能产出可蒸馏的 `_design.md` |
| 把采访问题全部塞进 prompt(不交互)| `AskUserQuestion` 强制结构化输出 + 进度可见 |
| 章节补默认内容(如默认"用 Kafka 做消息队列")| 未在采访出现 → 标 TODO,**不要补默认** |
| 输出后不标 TODO| 用户审稿时找不到未决项 → 误以为完成 |
| 跳过 Step 4 审稿提示直接跑 bootstrap| `_design.md` 草稿阶段就喂给 SOP 会污染宪章蒸馏 |
| 把采访硬塞成"我替你写完"| 用户没说的话不要替他补 |

## 输出检查清单(自检)

- [ ] `_design.md` 存在 + 顶部有 frontmatter(`status: awaiting-review`)
- [ ] §0 一句话定位 + 章节骨架按 Step 2 选定(默认 12 节)
- [ ] 至少 5 条 `<!-- TODO -->` 标记可见
- [ ] 每个锁定项标 `[from-interview: Q<n>-opt<m>]` 来源追溯
- [ ] Step 4 审稿 TODO 列表落在文末
- [ ] `_design.md` 总行数 200-500(精简 6 节版 ≥ 80 行)

## 下一步(接力到 spec-driven-bootstrap)

审稿完成后,告诉 Claude:

> 设计文档 `<path>/<project>_design.md` 已审完,vX.Y(`status: ready-for-bootstrap`)。请跑 `spec-driven-bootstrap` 生成 3 件套 SOP。

预期:`spec-driven-bootstrap` SKILL 自动加载 → 读设计文档 → 蒸馏 3 件套 → emit 项目专属 SKILL。

---

**Last updated**: 2026-09-21
**Version**: 1.3(v1.2 → v1.3: +**6 节精简模板 §2 把"扩展点"也并进来**("模块结构 + 核心接口 + 扩展点")— 解决"小项目用 entry_points / interface 注册多 provider 时,精简设计文档缺扩展点设计依据"gap;v1.2 的 §5 改名 + YAML escape 修复不变;其他不变)
