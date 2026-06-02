---
name: AgentLang
version: 0.1.0
author: jcoral
description: 面向 agent 的轻量指令语言，直接嵌入 markdown 书写。
license: MIT
tags: [agent, language, markdown, skill]
language: zh
dependencies: none
---

# AgentLang

利用 markdown 标题层级作为结构骨架，编号列表作为执行步骤。对非程序员友好，对 agent 无歧义。

---

## 语言结构

```markdown
### [Entrypoint: agentlang]   ← 应用入口标识

#### def: ActionName       ← 定义一个动作（无参数，agent 推断）
1. step 1
2. step 2

#### def: ActionName(arg0, arg1)  ← 定义一个动作（显式参数）
1. do something with arg0
2. if condition: return arg1

#### Main                  ← 执行入口，程序从这里开始
1. call ActionName: 自然语言描述意图
2. call ActionName("value"): 带参数调用
```

---

## 关键字

共 8 个关键字，覆盖所有控制流：

| 关键字 | 作用 | 示例 |
|--------|------|------|
| `def:` | 定义动作 | `#### def: FetchData(url)` |
| `call` | 调用动作（本地或其他 agent） | `call FetchData: 获取用户列表` |
| `if` | 条件判断 | `if fails: return` |
| `else` | 备选分支 | `else: retry` |
| `repeat` | 循环 | `repeat until done` |
| `parallel` | 并行执行 | `parallel:` + 子步骤列表 |
| `confirm` | 暂停等待用户确认或输入 | `confirm: 是否继续？` |
| `return` | 终止当前动作并输出结果 | `return data` |

---

## Markdown 书写规则

1. `### [Entrypoint: agentlang]` — H3 + 方括号 = 应用入口，标识这是一个 AgentLang 程序
2. `#### def: Name` — H4 + `def:` = 动作定义
3. `#### Main` — H4 + `Main` = 执行入口
4. 编号列表 = 顺序步骤
5. `parallel:` 后跟 `-` 无序列表 = 并行步骤
6. 冒号后的中文描述 = 给 agent 的意图补充（可选）

---

## 完整示例

```markdown
### [Entrypoint: agentlang]
#### def: FetchData(url)
1. get data from url
2. if fails: repeat until success or 3 attempts
3. return data

#### def: CleanData
1. remove duplicates
2. fill missing values
3. if data is empty: return error

#### def: GenerateReport
1. parallel:
   - call Analyst: 分析趋势
   - call Visualizer: 生成图表
2. merge results into report
3. return report

#### Main
1. call FetchData("https://api.example.com/users"): 获取原始数据
2. call CleanData: 清洗上一步的结果
3. confirm: 数据清洗完成，是否继续生成报告？
4. call GenerateReport: 生成最终报告
5. if report has errors: return "pipeline failed"
```

---

## Agent 解析指南

解析规则（按顺序执行）：

1. 扫描所有 `#### def:` 建立动作索引
2. 从 `#### Main` 开始逐步执行
3. 遇到 `call X` → 解析为"执行 X"，不区分本地 def 还是外部 agent（统一模型）
4. 遇到 `call X: 描述` 或 `call X("arg"): 描述` → 描述作为执行时的上下文补充
5. 遇到 `parallel:` → 下属 `-` 列表项并发执行
6. 遇到 `confirm` → 暂停执行，等待用户响应；用户拒绝则终止当前流程
7. 步骤中的自然语言 → agent 自行理解并执行
8. 参数：默认由 agent 从上下文推断；显式传参用 `Name(arg0, arg1)` 形式

### 结构识别

| Markdown 元素 | 语义 |
|---------------|------|
| `### [Entrypoint: agentlang]` | 应用入口，标识一个 AgentLang 程序 |
| `#### def: Name` | 动作定义开始 |
| `#### def: Name(args)` | 带参数的动作定义 |
| `#### Main` | 执行入口 |
| `1. 2. 3.` 编号列表 | 顺序步骤 |
| `- ` 无序列表（在 `parallel:` 下） | 并行步骤 |

### 执行模型

- 步骤按编号顺序执行，除非遇到 `parallel:`
- `call` 是唯一的调用机制，统一处理本地动作和外部 agent
- `confirm` 暂停执行流，等待用户响应；用户拒绝则终止当前流程
- `if` / `else` 提供分支，agent 根据运行时状态判断
- `repeat` 创建循环，需配合终止条件（`until`、次数限制）
- `return` 终止当前动作，将结果输出给调用方
- 每个步骤的返回值隐式传递给下一步骤（除非显式指定）
