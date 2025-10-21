---
title: "#1双链与yaml-tag的功能区别"
date: 2025-10-20
tags:
  - obsidian
  - quartz
description:
draft: false
aliases:
  - 在obsidian通过双链构建的关系
---

我们要区分两个层面的关系

1. **Obsidian 的语义关系（双链 / [[wikilinks]]）**
    
2. **YAML 的显式关系（tags / related / aliases）**
    

Quartz 会同时解析两种关系，但它们的作用和表现方式完全不同。下面我帮你拆成三层理解。

---

## 🧩 一、Quartz 能识别的“关系源”

Quartz 在构建站点时，会扫描每个 `.md` 文件：

1. **正文中出现的 `[[内部链接]]`**  
    → 生成**图谱（Graph）**与**双向链接（Backlinks）**；
    
2. **Frontmatter 中的 `tags`**  
    → 生成**标签页（Tag Pages）**；
    
3. **Frontmatter 中的 `related`（自定义字段）**  
    → 生成**人工定义的“相关笔记”引用**（取决于 Quartz 版本和配置）；
    
4. **Frontmatter 中的 `aliases`**  
    → 生成**同义词搜索映射**（同一篇文章可以用多个标题被找到）。
    

---

## 🧠 二、两种关系的区别与用途

| 关系类型                 | 定义方式                                   | Quartz 用途                        | 特点            |
| -------------------- | -------------------------------------- | -------------------------------- | ------------- |
| **双链（[[wikilink]]）** | 写在正文中，例如 `[[S3 同步机制]]`                 | 构建全局 **知识图谱** 和 **Backlinks 页面** | 自然语义、自动生成、强连接 |
| **tags**             | YAML 中 `tags: [obsidian, S3]`          | 构建 **标签聚合页**，用于分类浏览              | 弱连接、主题归类      |
| **related**          | YAML 中 `related: [另一篇笔记名]`             | 可定制在页面底部显示“相关文章”                 | 手工维护、语义明确     |
| **aliases**          | YAML 中 `aliases: [S3 备份, Amazon 对象存储]` | 让多个关键词指向同一页面                     | 改善搜索与图谱匹配     |

---

## ⚙️ 三、Quartz 实际生成效果（以你的研究笔记为例）

假设你有三篇笔记：

```text
A.md  ←  Remotely Save 与 S3 同步机制
B.md  ←  Obsidian 插件结构解析
C.md  ←  Quartz 部署与同步架构
```

### 1. 若 A 中出现正文：

```markdown
[[Obsidian 插件结构解析]]
[[Quartz 部署与同步架构]]
```

→ Quartz 自动绘制：

```
A ↔ B
A ↔ C
```

在 Graph 视图中可见，也会在 A 的页尾出现 “Linked References”。

---

### 2. 若 A 的 YAML 写：

```yaml
related:
  - Quartz 部署与同步架构
```

→ Quartz 在渲染 A 页面时，会额外生成一个 “相关文章” 区块，手动指定到 C。  
（不会自动反向生成 B↔A 关系，除非你也在 C 里写 related: A）

---

### 3. 若 A 的 YAML 写：

```yaml
tags: [obsidian, S3, backup]
```

→ Quartz 会创建 `/tags/obsidian/`、`/tags/S3/` 等索引页。  
你在这些页面上能看到所有打过相同 tag 的笔记。

---

## 🧭 四、建议的组合策略（面向“研究型知识库”）

|层级|用途|推荐做法|
|---|---|---|
|**语义连接（正文内）**|自然引用、建立图谱|使用 `[[wikilink]]` 双链连接相关笔记|
|**主题分类（tags）**|聚类研究领域|用 `tags` 标注研究主题、领域、工具名|
|**研究脉络（related）**|精确标注逻辑顺序|在 YAML 用 `related` 构建“论文式引用”结构（可控）|

这样：

- 图谱展示靠 `[[wikilink]]`；
    
- 分类索引靠 `tags`；
    
- 精确上下游研究关系靠 `related`。
    

---

## 💡 举个实战例子

假设你在研究 “Obsidian → S3 → Quartz” 的数据流。

### A.md

```yaml
---
title: Remotely Save 与 S3 同步机制
tags: [obsidian, S3, sync]
related:
  - Quartz 部署与同步架构
---
研究 Obsidian 的 [[Remotely Save 插件结构]]，并分析其与 [[Quartz 部署与同步架构]] 的接口。
```

### B.md

```yaml
---
title: Quartz 部署与同步架构
tags: [quartz, vercel, CI/CD]
related:
  - Remotely Save 与 S3 同步机制
---
Quartz 的构建脚本依赖 S3 上传机制。
```

**结果：**

- 图谱自动连成环；
    
- tag 聚合页显示两篇；
    
- 页面底部出现“相关文章”；
    
- 搜索 “S3” 或 “同步” 都能定位到 A。
    

---

## ✨ 结论

|目标|应用字段|
|---|---|
|构建图谱、知识关系|`[[wikilink]]`|
|分类整理主题|`tags`|
|人工定义上下游、论文式引用|`related`|
|搜索映射、同义词|`aliases`|

**=> Quartz 会生成所有这些结构，但可视化核心来自 `[[wikilink]]`。**

---


下一节：
- [[如何让 tags、wikilink、related 三者形成多层知识网络]]

