---
title: Openai的Atlas怎样写系统提示词？经验四例
date: 2024-10-23T10:10:59+08:00
draft: false
categories:
  - AI工程
  - 技术文档
tags: 提示词架构,优先级,工具组合
author: System Architecture Team
description: 深入解析 Atlas AI 助手的系统提示词架构设计，涵盖优先级处理、工具组合、防御性设计和示例驱动四大核心案例，提供可复用的模板和落地指南
summary: 先定规则，再授能力；分清优先，分化层次；如流水、如积木、如界线——清晰生秩序，秩序生可靠。
keywords:
  - 系统提示词
  - AI架构设计
  - 提示词工程
  - Atlas案例
  - 工具编排
  - 错误处理
version: "2.0"
source: Desktop/Notes/Analysis/Atlas_10-21-25.txt
toc: true
weight: 100
slug: openai-atlas-how-to-write-system-prompts-experience-examples
datetime: 2025-10-22 10:43
cover_image_url: ""
---


![](https://assets.yolonote.xyz/20251023102137198.png)

> 合并四个关键案例，加入源码摘录、从零到一构建指南与可复用模板，帮助独立搭建“系统提示词架构”。
>
> 源码：`Desktop/Notes/Analysis/Atlas_10-21-25.txt`

## 引言

好提示词像好工程，不靠"更聪明"，靠"更清楚"。Atlas 的价值在于制度化减少歧义：先把"听谁的""能做什么/不能做什么""何时用什么工具""长什么样"说清楚，再谈智能。

*就像给一个新员工写工作手册，不是指望他自己瞎猜，而是明确告诉他：遇到冲突听谁的？哪些事情绝对不能做？做事情的标准流程是什么？万一出错了怎么办？把这些都写清楚了，AI 就能像训练有素的员工一样稳定可靠。*

本文合并 v2.0 的四个案例：
- 优先级与上下文级联（冲突可预期）→ *像红绿灯一样，告诉 AI 遇到矛盾时该听谁的*
- 命名空间与组合流（搜索→读取→汇总→固化）→ *像流水线一样，把复杂任务拆成标准步骤*
- 防御性设计（禁区与三层错误处理）→ *像安全围栏一样，先画出不能碰的红线*
- 示例优于描述（样例与决策树）→ *像师傅带徒弟一样，用实例教会 AI 怎么做*

## 基础架构
Atlas 提示词呈现出清晰的**分层架构**,可以抽象为五个层次:

```
┌─────────────────────────────────────────┐
│    Layer 5: 身份与环境层 (Identity)      │ 
│    - 角色定位                            │
│    - 知识边界                            │
│    - 运行环境                            │
├─────────────────────────────────────────┤
│    Layer 4: 工具能力层 (Tools)          │
│    - 13个工具命名空间                    │
│    - 每个工具的接口规范                  │
│    - 工具使用指南                        │
├─────────────────────────────────────────┤
│    Layer 3: 行为规范层 (Guidelines)     │
│    - 何时使用何种工具                    │
│    - 如何响应用户请求                    │
│    - 错误处理策略                        │
├─────────────────────────────────────────┤
│    Layer 2: 上下文处理层 (Context)      │
│    - 指令优先级                          │
│    - 多源信息融合                        │
│    - 上下文边界                          │
├─────────────────────────────────────────┤
│    Layer 1: 交互模式层 (Modes)          │
│    - Full-Page Chat                     │
│    - Web Browsing                       │
│    - Side Chat                          │
└─────────────────────────────────────────┘
```

**架构洞察:**
这种分层设计实现了**关注点分离**(Separation of Concerns),每一层专注于特定职责,降低了系统复杂度。整个结构像盖房子，地基管地基的事（身份和环境），水电管水电的事（工具能力），装修管装修的事（行为规范）。每层各司其职，不会出现"装修工人跑去动地基"的混乱情况。这样系统出问题时，也能快速定位是哪一层的问题。


---
## 案例一（v2.0）：优先级与上下文级联，如何在冲突里保持“可预期”

源码节选（Instruction priority）：
```
System and developer instructions
Tool specifications and platform policies
User request in the conversation
User selected text in the context (in the user__selection tags)
Visual context from screenshots or images
Page context (browser__document + attachments)
Web search requests

If two instructions conflict, follow the one higher in priority ...
When both page context and attachments exist, treat them as a single combined context ...
```

要点：
- 先系统与平台，再会话，再请求；请求级通常“最近”，但仍在政策之下。
- 冲突必须解释取舍（1 句即可），提升可复核性。
- 页面与附件未区分则合并处理。

**通俗理解**：想象你是公司员工，老板的命令（系统指令）永远优先于同事的建议（会话历史），而客户当场的要求（用户请求）优先于上周的邮件。但即使客户要求很急，也不能违反公司政策。AI 就是按这个逻辑判断"听谁的"。


```
**实际使用场景**：
用户正在看一份去年的项目报告（页面上下文），突然说"帮我总结一下这个项目"。此时 AI 会：
1. 优先看当前页面内容（请求级上下文）
2. 如果页面内容不完整，再结合之前的对话历史
3. 如果发生冲突（比如页面说项目成功，但之前对话提到项目失败），会说明："根据当前页面显示，项目标记为成功状态"，让用户知道 AI 采信了哪个来源
   
最小示例（页面选区 vs 历史纪要）：以请求级为主对比，引用会话摘要，解释取舍依据
   
```


可复用片段：
- “请求级与会话级冲突时，遵循请求级；用 1 句解释取舍。”
- “页面与附件未区分时，视作合并上下文。”
- “需要新鲜度或地理信息时使用 web；不得用 web 替代缺失的私域内容。”

---


## 案例二（v2.0）：命名空间与组合流，让工具像乐高拼接

源码节选（接口模式）：
```
namespace file_search { type msearch = (_:{ queries?: string[], time_frame_filter?:{ start_date:string; end_date:string; } })=>any }
namespace gcal { type search_events=(_:{...})=>any; type read_event=(_:{ event_id:string, ...})=>any }
namespace gmail { type search_email_ids=(_:{...})=>any; type batch_read_email=(_:{ message_ids:string[] })=>any }
```

组合流范式：
- 搜索→读取→汇总→固化（文档/表格）。
- 参数显式传递（ID、时间窗、分页）。
- 失败路径清晰（分页失败/部分读取失败的降级策略）。

**通俗理解**：就像乐高积木，每个工具都是一块标准积木（有固定的接口），可以按需组合。比如"搜索"积木找到目标，"读取"积木获取详情，"汇总"积木整理信息，"固化"积木保存结果。每块积木功能单一但组合起来能力强大。

```
**实际使用场景（下周与 Alice 的会议汇总）**：
用户说："帮我整理一下下周和 Alice 的所有会议安排"

AI 的工作流程：
1) **搜索阶段**：`gcal.search_events({ time_min, time_max, query:"Alice" })` → 找到 3 个相关会议
2) **读取阶段**：`gcal.read_event({ event_id })` 逐个读取详情 → 获取完整议程、参与者名单
3) **汇总阶段**：按时间/地点/议题/参与者分类整理
4) **固化阶段**：`canmore.create_textdoc({ type:"document" })` → 生成一份会议清单文档，用户可以直接打印或分享
```

优点：**<u>如果第2步读取某个会议失败了，AI 不会整个流程崩溃，而是在汇总时标注"该会议详情获取失败"，继续完成其他会议的整理。</u>**

---

## 案例三（v2.0）：防御性设计，先画边界再给能力

源码节选（禁区与错误处理）：
```
// gcal: You cannot create, update, or delete events ...
// gmail: You cannot send, flag / modify, or delete emails and you should never imply ...
// automations: When you get an ERROR ... EXPLAIN that error ... Do NOT say you've successfully made the automation.
Never replace missing private document context with generic web search.
```

执行骨架：
- 先边界声明，再给替代路径（草拟取消邮件、手动步骤、提醒自动化）。
- 无响应 ≠ 成功；错误需被复述与解释。

**通俗理解**：就像游乐场的安全围栏，先告诉你哪里不能去（删除邮件、修改日历），然后告诉你可以做什么（帮你起草邮件内容、提醒你手动操作）。如果工具出错了，不能假装成功，必须老实告诉用户发生了什么。

```
**实际使用场景**：
用户说："帮我取消明天和 Bob 的会议"

**没有防御性设计的 AI**：
- "好的，已经帮你取消了明天和 Bob 的会议" ✗（其实根本没权限取消）

**有防御性设计的 AI**：
- "我无法直接取消日历事件，但我可以帮你：
  1. 查找明天与 Bob 的会议详情
  2. 起草一封取消会议的邮件给 Bob
  3. 提供会议链接，你可以直接在日历中取消
  请问需要我执行哪一项？" ✓
```

优点：<u>如果在查找会议时工具无响应，AI 会说："在尝试查找会议时遇到错误（超时），而不是沉默或假装成功。</u>

---

## 案例四（v2.0）：示例优于描述，教模型像学徒一样学

源码节选（样例与决策树）：
```
// file_search: One of the queries MUST be the user's original question ... It MUST be a complete sentence.
// ... up to five queries ... only when ... needs to be decomposed ...

Use the web tool when ...
Never replace missing private document context with generic web search.
```

**通俗理解**：就像师傅教徒弟修车，不只是说"拧紧螺丝"，而是示范："看，先用这个扳手，
逆时针转三圈，感觉到阻力就停。"给 AI 看具体例子，比抽象描述管用一百倍。

```diff
**实际使用场景（找"OKR 2021"）**：
用户说："帮我找一下 2021 年的 OKR 文档"

AI 的决策流程：
1) **首选本地搜索**：`file_search.msearch({ queries:["OKR 2021"], time_frame_filter:{start_date:"2021-01-01", end_date:"2021-12-31"} })`
   - 如果找到：直接返回文档路径和摘要
   
2) **未命中时的处理**：
   - "在你的本地文档中没有找到 2021 年的 OKR。我可以在网上搜索 OKR 相关的最佳实践或模板，但无法替代你的私人文档。需要我搜索吗？"
   
3) **输出完整路径**：
   - "找到文档：`/Projects/2021-OKR.md`
   - 摘要：包含 Q1-Q4 四个季度目标...
   - 下一步：[打开文档] 或 [生成年度总结]"
```

**为什么这样设计？**
- 完整的示例让 AI 知道"OKR 2021"要拆成关键词 + 时间过滤器
- 决策树明确了"本地优先，网络补充"的原则
- 用户能清楚看到 AI 的思考路径，而不是一个黑盒
## 四大功能的协同场景：完整的工作流演示

现在让我们看看这四个案例如何在一个真实场景中协同工作。

**场景：用户需要准备下周的项目复盘会议**

用户输入："帮我准备下周一和团队的项目复盘会议材料，需要汇总最近三个月的进展、会议记录和邮件讨论"
### AI 的完整执行流程：

**第一步：优先级判断（案例一）**
- 识别请求级上下文：用户当前没有选中任何文本，没有打开特定文档
- 识别会话级上下文：之前讨论过这个项目的 Q3 目标
- 决策：以当前请求为主，但会参考历史对话中的项目名称

**第二步：工具组合流（案例二）**
```
搜索阶段：
├─ file_search.msearch({ queries:["项目复盘", "进展报告"], time_frame_filter: 最近3个月 })
├─ gcal.search_events({ query:"项目", time_range: 最近3个月 })
└─ gmail.search_email_ids({ query:"项目讨论", time_range: 最近3个月 })

读取阶段：
├─ 对每个文档：read_file()
├─ 对每个会议：gcal.read_event()
└─ 对每批邮件：gmail.batch_read_email()

汇总阶段：
├─ 按时间线整理关键里程碑
├─ 提取核心决策和待办事项
└─ 总结讨论中的风险点

固化阶段：
└─ canmore.create_textdoc() → 生成结构化的复盘材料
```

**第三步：防御性处理（案例三）**

在执行过程中遇到的边界情况：
- **遇到无权限邮件**："检测到 2 封加密邮件无法读取，已在摘要中标注"
- **日历读取超时**："8月15日的会议详情获取失败（超时），建议手动补充"
- **用户要求"直接发送给团队"**："我无法直接发送邮件，但已生成材料文档，你可以：1) 复制内容到邮件 2) 分享文档链接"

**第四步：示例驱动的输出（案例四）**

最终输出结构（参照示例模板）：
```markdown
# 项目复盘会议材料（2024-07-01 至 2024-09-30）

## 一、时间线
- 7月15日：项目启动会（参会：Alice, Bob, Carol）
- 8月03日：中期评审，发现性能瓶颈
- 8月15日：[会议详情获取失败，需手动补充]
- 9月20日：第一版交付

## 二、关键决策
1. 技术栈从 A 切换到 B（邮件讨论，8月10日）
2. 延期两周以优化用户体验（会议决议，9月05日）

## 三、待办事项
- [ ] 性能优化方案（负责人：Bob）
- [ ] 用户反馈收集（负责人：Carol）

## 四、风险提示
⚠️ 2封加密邮件未纳入分析，可能包含额外信息
⚠️ 8月15日会议记录缺失

---
数据来源：
- 本地文档：5份进展报告
- 日历事件：12次项目会议
- 邮件线程：34封相关讨论
```

### 这个场景展示了什么？

1. **优先级系统**确保了 AI 正确理解"下周一的会议"是新请求，不会被历史对话中的"下周五"误导
2. **组合流**让复杂任务被拆解成可管理的步骤，每步都有明确的输入输出
3. **防御性设计**保证了即使部分数据获取失败，用户也能得到有用的结果和清晰的缺失说明
4. **示例驱动**让输出格式标准化、结构化，用户拿到手就能直接用

这就是"系统提示词架构"的力量：不是让 AI 更聪明地猜，而是让它按照明确的规则、流程、边界来稳定工作。


---

## 从零到一：系统提示词模板（可复制）

```
You are <ProductName>.
Knowledge cutoff: <YYYY-MM>
Current date: <dynamic>
Image input capabilities: <Enabled/Disabled>
Personality: <vX>

# Tools
## <namespace_1>
// Description + 禁区（绝不可做/绝不可暗示）
namespace <namespace_1> {
  type <action_a> = (_:{ <params> }) => any;
  type <action_b> = (_:{ <params> }) => any;
}
## <namespace_2>
...

# Instruction priority
System and developer instructions
Tool specifications and platform policies
User request in the conversation
User selected text in the context (in the user__selection tags)
Visual context from screenshots or images
Page context (browser__document + attachments)
Web search requests

If two instructions conflict, follow the one higher in priority.
When page context and attachments both exist and are not distinguished, treat as combined.

# Context handling
- Level 1: System (cutoff/date/personality/model)
- Level 2: Session (memory, dialog history, tool results)
- Level 3: Request (user selection/page/attachments)
Conflict rule: Prefer Level 3 over Level 2; explain in one sentence.

# Error and fallback
- No response from tool ≠ success; acknowledge and explain.
- Provide concrete fallback (retry, alternate tool, manual steps).

# Examples (for key actions)
// Example 1: <file_search.msearch> 输入/输出样例
// Example 2: <gcal.search_events→read_event> 链路样例

# Decision trees
- Local/private context first; web for freshness/location/complementary info.
```

---

## 统一 Checklist（落地核对）

- [ ] 是否定义了 Instruction priority，并写明冲突解释的 1 句规则？
- [ ] 是否提供了页面 + 附件的合并处理与本地/网络的决策条件？
- [ ] 每个工具是否明确禁区（“绝不可暗示”）与失败语义（无响应 ≠ 成功）？
- [ ] 是否提供了可执行的小样例与输出骨架？
- [ ] 关键链路是否使用了“搜索→读取→汇总→固化”的最小路径？

---

## 结语

好的系统提示词不是“更会猜”，而是“更会约束”：先把边界、顺序、样例、决策树写清楚，再用组合把能力放大。把这些制度化下来，系统的行为就会变得可预期、可复核、可维护。



更多内容请关注公众号后台，发送”atlas“
《Atlas 提示词架构深度解析-设计亮点与原文关联》
![|608x994](https://assets.yolonote.xyz/20251023101443073.png)