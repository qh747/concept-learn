# 存档系统（references/save-system.md）

> 本文件为 SKILL.md 的存档系统细节下沉，由 SKILL.md 通过 context pointer 引用。agent 在涉及存档写入/读取时加载本文件。

## 存档目录结构

在当前工作区下创建 `.concept-learn` 目录，每个学习主题一个子目录：

```
当前工作区/
  .concept-learn/
    react/
      save.md
    vue/
      save.md
    cpp-stl/
      save.md
```

子目录名 = 主题名 slug 化（转小写、空格转连字符、去除特殊字符）。

## 存档文件格式

每个主题子目录下有一个 `save.md` 文件，格式为 **Markdown + YAML frontmatter**：

```markdown
---
topic: React
goal: 应用
detail_level: 详尽
self_check: true
output_mode: step
template: quick-start
outline:
  - stage: 1
    title: JSX 语法
    summary: JSX 的基本语法和规则
  - stage: 2
    title: 组件与 props
    summary: 组件定义、props 传递
current_stage: 3
stage_status:
  - stage: 1
    status: completed
  - stage: 2
    status: completed
  - stage: 3
    status: in_progress
  - stage: 4
    status: not_started
created_at: 2026-07-26
updated_at: 2026-07-26
---

# React 学习存档

## 大纲
1. JSX 语法 —— JSX 的基本语法和规则
2. 组件与 props —— 组件定义、props 传递
3. state 与生命周期 —— 状态管理与生命周期钩子
4. hooks —— useState、useEffect 等核心 hooks

## 第 1-2 阶段：<已讲内容>
## 第 3 阶段：state 与生命周期（进行中）<部分内容>

## 自检记录
- 第 1 阶段：通过 / 第 2 阶段：跳过
```

## 存档字段说明

| 字段 | 含义 |
|------|------|
| `topic` | 学习主题（原始名称，如 "React"） |
| `goal` | 学习目标导向（应用/理解/应试） |
| `detail_level` | 解释详略（精炼/详尽） |
| `self_check` | 是否自检（true/false），持久化，支持中途切换 |
| `output_mode` | 阶段内输出模式（step 分步输出 / all 全部输出），默认 step，支持中途切换 |
| `template` | 本次学习采用的模板名（从模板加载流程获得），可选字段 |
| `outline` | 完整大纲（阶段列表，每阶段含编号、标题、简述） |
| `current_stage` | 当前所在阶段编号（从 1 开始） |
| `stage_status` | 每个阶段的状态（not_started/in_progress/completed） |
| `created_at` | 存档创建时间（YYYY-MM-DD） |
| `updated_at` | 最近更新时间（YYYY-MM-DD） |

## 阶段状态机

- `not_started`：该阶段未开始讲解。
- `in_progress`：该阶段讲解中（用于中途存档）。
- `completed`：该阶段已讲完。**讲完即 completed，阶段内容保存在正文中。**

## 恢复时 `current_stage` 和 `stage_status` 的优先级规则

1. **以 `stage_status` 为主，`current_stage` 仅作参考。**
2. 恢复时按以下顺序查找：
   - 优先找状态为 `in_progress` 的阶段中最小编号者。
   - 如果没有 `in_progress`，找状态为 `not_started` 的阶段中最小编号者。
   - 如果全是 `completed`，则学习已完成，输出完整内容总结后删除存档。
3. 找到继续阶段后，**同步更新 `current_stage` 为该阶段编号**。
4. 如果出现多个 `in_progress`（异常状态），取最小编号，并提示用户"检测到存档异常，已从第 N 阶段恢复"。

## 主动存档

用户可随时主动触发存档，触发词包括："保存进度"、"存档"等。

触发后：
1. 将当前进度写入 `.concept-learn/<主题slug>/save.md`（覆盖旧文件）。
2. 更新 `updated_at` 字段。
3. 回复确认："已存档至 .concept-learn/<主题slug>/save.md，当前进度：第 N 阶段（<阶段标题>）。"

## 存档覆盖策略

每次存档覆盖旧文件，**不保留历史版本**。

## 退出时提示

当用户显式表达退出意图（触发词："退出"、"结束学习"、"不学了"、"今天就到这"、"下次再学"、"先保存"等）时：

1. 检查当前是否有未保存的进度。

**"未保存进度"的判定标准：**
- **已有存档**：自上次存档后，有任意阶段的 `stage_status` 发生了改变（如从 `not_started` 变成 `completed`，或从 `not_started` 变成 `in_progress`）。
- **无存档（首次学习）**：只要进入了任何阶段的讲解（`current_stage > 0`），视为有未保存进度。
- **还在大纲确认阶段就退出**：视为无未保存进度，直接退出，不询问。

2. 若有未保存进度，提示："你当前的学习进度尚未保存，是否需要保存？"
3. 提供三个选项：
   - **是** → 执行存档，然后退出。
   - **否** → 不存档，直接退出。
   - **取消退出** → 不存档，继续学习。

**只响应显式退出信号**，不因用户临时岔开话题而触发退出提示。