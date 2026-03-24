<p align="center">
  <h1 align="center">🧠 LeetCode Hot100 自适应刷题系统</h1>
  <p align="center">
    基于 AI Agent 的自适应 LeetCode 刷题训练器<br/>
    为 <a href="https://docs.anthropic.com/en/docs/claude-code">Claude Code</a> 而生 · 兼容任何 AI 编程 CLI 工具
  </p>
  <p align="center">
    <a href="#快速开始">快速开始</a> ·
    <a href="#命令一览">命令一览</a> ·
    <a href="#交互示例">交互示例</a> ·
    <a href="#工作原理">工作原理</a> ·
    <a href="#ai-agent-接入指南">Agent 指南</a>
  </p>
</p>

---

## 这是什么？

一个**人机协同**的 LeetCode 刷题闭环系统，替代"打开力扣 → 随便挑一题 → 独自挣扎"的低效工作流：

```
你说"练习" → Agent 根据你的薄弱点选题
→ 你写代码 → Agent 对照参考答案做详细 Code Review
→ 权重自动调整 → 下次练习自动瞄准你最弱的地方
```

它融合了 **CodeTop 面试高频数据**、**艾宾浩斯遗忘曲线**的间隔重复机制，以及 **AI 驱动的薄弱点分析**，构建出一套随你进步而自动调整的个性化训练计划。

### 核心特性

- **练习模式** — Agent 出 2 题（1 题随机 + 1 题按薄弱加权），你选一题开刷
- **指定刷题** — 可以直接指定题号、题名、或某个分类进行练习
- **智能 Code Review** — 按 P0（正确性）→ P1（复杂度）→ P2（代码风格）优先级审查
- **参考答案对比** — 你的代码 vs 标准题解的逐行 diff 分析
- **四级提示系统** — 卡住了？从方向引导 → 思路框架 → 代码骨架 → 完整答案，逐级递进
- **自适应权重** — 做错了 → 权重升高，下次优先出；掌握了 → 权重下降
- **间隔复习提醒** — 基于遗忘曲线提醒你："两数之和做了 7 天了，该复习了"
- **用户画像 (soul.md)** — Agent 学习你的风格、弱点、性格，持续更新
- **三层记忆隔离** — 严格的上下文管理，防止信息污染

### 适合谁？

- 正在准备面试、想要**有体系地刷题**的同学
- 使用 **Claude Code、Coco、Codex、Aider** 等 AI 编程工具的开发者
- 想要**自动追踪薄弱点和刷题进度**、不想手动记录的人

---

## 快速开始

### 前置条件

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)（推荐）或其他 AI 编程 CLI 工具
- Python 3.10+
- Git

### 安装步骤

```bash
# 1. 克隆本仓库
git clone https://github.com/Brittanywu/Leetcode-Orbit.git
cd Leetcode-Orbit

# 2. 启动 Claude Code
claude
```

进入 Claude Code 后，运行初始化命令：

```
> /init
```

这会自动完成：
- 创建 `data/hot100_index.json`（100 题等权重索引）
- 创建 `soul.md`（你的学习画像）
- 创建 `reports/` 和 `workspace/` 目录
- 询问你的刷题目标和偏好

初始化完成后，直接开始：

```
> 我想练习
```

---

## 命令一览

系统提供三个斜杠命令和自然语言两种交互方式：

### `/practice` — 练习模式

| 用法 | 示例 | 行为 |
|------|------|------|
| 不带参数 | `/practice` 或 `我想练习` | 推荐 2 题（1 随机 + 1 加权），你选一题 |
| 指定题号 | `/practice 206` 或 `来一道反转链表` | 直接生成该题，跳过选题环节 |
| 指定分类 | `/practice 链表` 或 `/practice 动态规划` | 从该分类中推荐 2 题 |

**支持的触发词**：`练习`、`practice`、`刷题`、`我想刷第X题`、`来一道XX`

**练习流程**：
1. 检查是否有需要艾宾浩斯复习的题目
2. 选题（或使用用户指定的题目）
3. 生成 `workspace/current_problem.py`（含题目描述、函数签名、测试用例）
4. 你在文件中编写代码
5. 完成后说 `review`

### `/review` — Code Review

| 触发方式 | 示例 |
|---------|------|
| 斜杠命令 | `/review` |
| 自然语言 | `写完了`、`检查`、`review` |

**审查流程**：
1. 读取你的代码 + 参考答案
2. 运行测试用例，验证正确性
3. 按 P0 → P1 → P2 优先级分析
4. 口头反馈（根据你的 soul.md 调整语气）
5. 同时写入 4 个文件：

| 输出 | 文件 | 说明 |
|------|------|------|
| 详细报告 | `reports/LC_{id}_{title}.md` | 给你看的完整分析（写完不回读） |
| 状态索引 | `data/hot100_index.json` | 更新权重/计数/薄弱点 |
| 用户画像 | `soul.md` | 微调能力雷达、薄弱点排名 |
| 分析日志 | `hot100_user_analysis.md` | 追加本次刷题记录（写完不回读） |

### `/hint` — 四级提示系统

当你在做题过程中卡住了，可以请求提示。系统按级递进，**绝不跳级**：

| 级别 | 触发 | 内容 | 权重影响 |
|------|------|------|---------|
| **Level 1** | `提示`、`hint`、`卡住了` | 方向引导 — 告诉你该用什么**类型**的方法，不剧透 | 无惩罚 |
| **Level 2** | 再说 `hint` | 思路框架 — 分步骤引导，可提及数据结构名称，不给代码 | 无惩罚 |
| **Level 3** | 再说 `hint` 或 `hint 3` | 代码骨架 — 关键逻辑留空 `______` 让你填 | 权重不降低 |
| **Level 4** | `看答案`、`give up`、`我放弃` | 完整答案 + 逐行讲解 | 权重 ×1.2 |

**你也可以直接指定级别**：`给我 level 3 的提示`、`hint 2`

**不消耗提示级别的情况**：
- 具体技术问题（"这里用 defaultdict 还是 dict？"）
- Debug 请求（"为什么我的代码在 [0,0,0] 上出错？"）
- 概念性问题直接回答

### 自然语言交互

你不需要记住斜杠命令，直接用自然语言也行：

```
我想练习                    → 等同于 /practice
我想刷反转链表               → 等同于 /practice 206
来一道链表题                 → 等同于 /practice 链表
写完了 / review / 检查       → 等同于 /review
提示 / hint / 卡住了         → 等同于 /hint
再提示 / hint 2             → 下一级提示
看答案 / 我放弃              → Level 4 完整答案
```

---

## 交互示例

### 示例 1：练习模式

```
你: 我想练习

Agent: 📅 复习提醒：LC 1. 两数之和（距上次做题已过 7 天）

      今天为你准备了两道题：

      🎲 随机题：LC 15. 三数之和 (Medium) [双指针/排序]
      🎯 推荐题：LC 20. 有效的括号 (Easy) [栈]
         ⚠️ 上次薄弱点: 边界条件遗漏

      想刷哪题？输入题号即可。

你: 20

Agent: 题目文件已生成在 workspace/current_problem.py
      打开文件写你的解法，写完后跟我说 "review"。
```

### 示例 2：指定题目

```
你: 来一道反转链表

Agent: 题目文件已生成在 workspace/current_problem.py
      LeetCode 206. 反转链表 (Easy) [递归, 链表]
      打开文件开始写代码吧，写完后告诉我 "review" 即可！
```

### 示例 3：指定分类

```
你: /practice 动态规划

Agent: 今天从动态规划分类为你准备了两道题：

      🎲 随机题：LC 70. 爬楼梯 (Easy) [动态规划]
      🎯 推荐题：LC 322. 零钱兑换 (Medium) [动态规划]
         ⚠️ 上次薄弱点: DP状态转移

      想刷哪题？
```

### 示例 4：Code Review（全部通过）

```
你: review

Agent: 🔍 Code Review — LeetCode 20. 有效的括号

      📋 测试结果: 3/3 通过
      | Case | 输入       | 期望  | 实际  | 结果 |
      |------|-----------|-------|-------|------|
      | 1    | "()"      | true  | true  | ✅   |
      | 2    | "()[]{}"  | true  | true  | ✅   |
      | 3    | "(]"      | false | false | ✅   |

      📊 分析:
      • P0 正确性: ✅ 全部通过
      • P1 复杂度: 时间 O(n) / 空间 O(n)，已达最优
      • P2 代码风格: ✅ 使用字典映射，简洁

      太棒了！上次的边界条件问题已经修复了，
      可以看出你对栈的掌握越来越扎实了！

      📊 已更新:
      - 权重: 1.8 → 0.72 (掌握度提升)
      - 薄弱点 "边界条件遗漏" 已移除
      - 完整报告: reports/LC_020_有效的括号.md
```

### 示例 5：Code Review（有问题）

```
你: review

Agent: 🔍 Code Review — LeetCode 1. 两数之和

      📋 测试结果: 3/3 通过

      📊 分析:
      • P0 正确性: ✅ 全部通过
      • P1 复杂度: 时间 O(n²) ❌ 最优为 O(n)
      • P2 代码风格: ⚠️ 建议用 enumerate() 代替 range(len())

      代码逻辑正确！全部用例通过。不过可以优化：
      用哈希表存储已遍历的数字，可以把时间从 O(n²) 降到 O(n)。

      💡 改进建议:
      1. 用字典记录 {值: 索引}，每次检查 target-num 是否已存在
      2. 用 enumerate() 同时获取索引和值，更 Pythonic

      📊 已更新:
      - 权重: 1.0 → 0.8
      - 薄弱点: [时间复杂度未达最优, 不够Pythonic]
      - 完整报告: reports/LC_001_两数之和.md
```

### 示例 6：提示系统

```
你: 卡住了

Agent: 💡 Level 1 提示 — 方向引导

      这题可以用一种能在 O(1) 时间内查找元素的数据结构来优化。
      想想：遍历到每个数时，怎么快速知道"需要的另一个数"之前有没有出现过？

      想继续思考？还是需要更多提示？说 "hint" 获取下一级帮助。

你: 还是不太懂

Agent: 💡 Level 2 提示 — 思路框架

      分三步思考：
      1. 遍历数组，对于每个元素 num，计算 complement = target - num
      2. 检查 complement 是否在之前遍历过的数中出现过
      3. 如果出现过，返回两个数的索引；如果没有，把当前数记录下来

      关键问题：用什么数据结构来"记录"已遍历的数，使得查找是 O(1) 的？

      需要代码骨架？说 "hint" 获取 Level 3 提示。

你: hint

Agent: 💡 Level 3 提示 — 代码骨架

      ```python
      seen = {}
      for i, num in enumerate(nums):
          complement = ______          # 计算需要配对的数
          if complement in ______:     # 在哪里查找？
              return [______, ______]  # 返回哪两个索引？
          seen[______] = ______        # 把什么存进去？
      ```

      试着把 ______ 填上。如果实在不行，说 "看答案"。
```

---

## 项目结构

```
Leetcode-Orbit/
│
├── CLAUDE.md                    # Agent 指令文件（始终加载）
├── soul.md                      # 用户画像（始终加载）
│
├── data/
│   ├── hot100_problems.json     # 题库完整数据（100 题，按需读单题）
│   └── hot100_index.json        # 轻量状态索引（始终加载）
│
├── references/                  # 参考题解
│   ├── index.json               # 题目 ID → 文件 + 行号范围的映射
│   ├── LeetCode_Hot100_Python/  # 按难度整合的题解
│   │   ├── Hot100_Easy.md
│   │   └── Hot100_Medium.md
│   └── leetcode-hot-100-master/ # 按分类的独立题解文件
│       ├── 哈希/
│       ├── 链表/
│       ├── 二叉树/
│       └── ...
│
├── reports/                     # 每题详细报告（只写不读）
│   └── LC_001_两数之和.md
│
├── workspace/
│   ├── current_problem.py       # 当前题目文件（你在这里写代码）
│   └── .current_meta.json       # 当前题目元数据（Agent 内部使用）
│
├── hot100_user_analysis.md      # 整体分析日志（只写不读）
│
├── .claude/
│   └── commands/                # 斜杠命令
│       ├── practice.md          # /practice — 练习模式
│       ├── review.md            # /review — Code Review
│       └── hint.md              # /hint — 四级提示
│
├── DESIGN_V1.md                 # 系统设计文档：架构、权重、流程
├── DESIGN_V2.md                 # 记忆架构文档：三层分离、soul.md
├── HINT_SYSTEM_PATCH.md         # 提示系统设计文档
│
└── README.md
```

---

## 工作原理

### 三层记忆隔离架构

这是整个系统最核心的设计——严格管理 Agent 上下文中**什么该读、什么不该读**：

| 层级 | 文件 | 加载时机 | 作用 |
|------|------|---------|------|
| **始终加载**（~8KB） | `CLAUDE.md`、`soul.md`、`data/hot100_index.json` | 每轮对话 | Agent 的"工作记忆" |
| **按需加载**（用完即弃） | `hot100_problems.json`（单题）、`references/`（单题题解）、`workspace/*.py` | 出题/审查时 | 临时任务数据 |
| **只写不读** | `reports/*.md`、`hot100_user_analysis.md` | 审查完成后 | 给用户看的详细报告 |

**为什么这样设计？** AI Agent 的上下文窗口是有限的。如果 Agent 每次都回读所有历史报告，老数据会淹没上下文，导致决策质量下降。把"Agent 记忆"和"用户报告"严格分层，两边都能保持高质量。

### 权重系统

每道题有两个权重：

- **练习权重**（`pw`）— 由你的表现驱动：
  - 全部通过且无薄弱点 → `pw × 0.6`（下降，掌握了）
  - 通过但有薄弱点 → `pw × 0.8`（微降）
  - 未通过 → `pw × 1.5 + 惩罚`（升高，优先复习）
  - 使用了 Level 3 提示 → 权重不降低
  - 使用了 Level 4 / 放弃 → `pw × 1.2`

- **实战权重**（`bw`）— `CodeTop 频率 × 薄弱加成 × 遗忘衰减因子`

### 审查优先级

每次 Code Review 严格按 **P0 > P1 > P2** 分优先级：

| 优先级 | 关注点 | 示例 |
|--------|--------|------|
| **P0** | 代码能跑通吗？测试用例过了吗？ | 边界输入导致 RuntimeError |
| **P1** | 时间/空间复杂度是否最优？ | 用了 O(n²) 但 O(n) 的解法存在 |
| **P2** | 代码写法是否 Pythonic、简洁？ | `range(len(x))` → `enumerate(x)` |

### soul.md — 你的学习画像

Agent 维护一份简洁的用户画像（控制在 80 行以内），持续跟踪：

- 算法能力雷达图（哈希表 ★★★★、动态规划 ★☆☆☆☆ 等）
- 高频薄弱点 TOP 5
- 学习风格偏好（"喜欢被鼓励""偏向理解而非背模板"）
- 成长里程碑

Agent 每轮都会读取这个文件，以此**个性化出题和反馈语气**。

### 艾宾浩斯复习提醒

每次开始练习时，系统自动检查是否有需要复习的题目。在距上次做题 **1、2、4、7、15、30** 天时触发提醒：

```
📅 复习提醒:
  - LeetCode 1. 两数之和 (距上次 7天，建议复习)
  - LeetCode 3. 无重复字符的最长子串 (距上次 15天，建议复习)
```

---

## AI Agent 接入指南

> 本章节专门写给 AI 编程 Agent（Claude Code、Coco、Codex、Aider 等），帮助它们理解并操作本项目。

### Agent 速查表

**你的核心指令文件是 `CLAUDE.md`** — 首先读它。它包含：
- 哪些文件始终加载 vs 绝不加载
- Code Review 标准（P0/P1/P2）
- 提示系统的四级递进规则
- 触发词汇表

**记忆规则（必须遵守）：**
1. 始终加载：`CLAUDE.md` + `soul.md` + `data/hot100_index.json`（共约 8KB）
2. 按需读取：从 `data/hot100_problems.json` 读单题数据，从 `references/` 读单题题解
3. **绝不回读**：`reports/*.md` 和 `hot100_user_analysis.md` — 这些是给用户看的，不是给你看的
4. 审查完成后，独立写入 4 个文件：报告文件、index.json、soul.md、analysis.md

**核心数据文件：**
- `data/hot100_index.json` — 紧凑索引，缩写字段：`t`(标题)、`d`(难度 E/M/H)、`pw`(练习权重)、`bw`(实战权重)、`att`(尝试次数)、`cor`(正确次数)、`weak`(薄弱标签)
- `data/hot100_problems.json` — 完整题目数据（描述、测试用例、函数签名）。每次只读一题，绝不整体加载
- `references/index.json` — 题目 ID → 参考答案文件 + 行号范围的映射

**指令映射表：**

| 用户说 | Agent 执行 |
|--------|-----------|
| `练习` / `practice` / `刷题` | 读 index.json → 选 2 题 → 展示选项 → 生成 .py 文件 |
| `练习 206` / `来一道反转链表` | 读 index.json → 直接生成指定题的 .py 文件 |
| `练习 链表` / `practice 动态规划` | 读 index.json → 从该分类中选 2 题 → 展示选项 |
| `review` / `检查` / `写完了` | 读用户代码 + 参考答案 → 跑测试 → P0/P1/P2 分析 → 写 4 个输出 |
| `提示` / `hint` / `卡住了` | 按级递进给出提示（L1 方向 → L2 思路 → L3 骨架 → L4 答案） |
| `看答案` / `give up` | 直接给 Level 4 完整答案 + 逐行讲解 |

---

## 技术栈

这个项目刻意做到**零依赖**，最大化可移植性：

- **存储**：JSON 文件（不需要数据库）
- **语言**：Python 3.10+（用于生成练习文件和运行测试）
- **Agent**：任意 AI 编程 CLI（Claude Code、Coco、Codex、Aider、Cursor Agent 等）
- **数据源**：静态 JSON + Markdown（不调 API、不爬网页）

不需要 `npm install`。不需要 `pip install`。不需要 Docker。克隆即用。

---

## 常见问题

**Q: 必须用 Claude Code 吗？**

不是必须的。Claude Code 是推荐的（因为 CLAUDE.md 和 `.claude/commands/` 是为它优化的），但系统可以配合任何能读文件、跑 Python、理解 Markdown 指令的 AI Agent 使用。

**Q: 会爬 LeetCode 吗？**

不会。所有题目数据静态存储在 `data/hot100_problems.json` 中。不发网络请求、不需要 API Key、不存在限流风险。

**Q: 能用 Python 以外的语言吗？**

目前模板默认生成 Python 文件，但系统在设计上是语言无关的。你可以修改 `CLAUDE.md` 中的模板生成逻辑来支持 Java/C++/Go 等。

**Q: 已有的刷题进度会丢吗？**

不会。系统永远不覆盖已有的用户状态。权重更新是增量的，不会碰你的已有记录。

**Q: 我想分享项目但不想暴露我的薄弱点怎么办？**

`soul.md`、`data/hot100_index.json`、`reports/`、`hot100_user_analysis.md` 这些包含个人数据的文件应加入 `.gitignore`。只有系统骨架是公开的。

---

## 设计文档

| 文档 | 内容 |
|------|------|
| `DESIGN_V1.md` | 系统架构、权重公式、选题算法、Code Review 流程 |
| `DESIGN_V2.md` | 三层记忆架构、soul.md 设计、报告模板、信息流转全图 |
| `HINT_SYSTEM_PATCH.md` | 四级递进提示系统设计 |

---

## 致谢

- [LeetCode](https://leetcode.cn) — 题目来源
- [CodeTop](https://codetop.cc)（[@afatcoder](https://github.com/afatcoder/LeetcodeTop)）— 面试频率数据
- [realnghon/LeetCode_Hot100_Python](https://github.com/realnghon/LeetCode_Hot100_Python) — 参考题解
- `soul.md` 的概念灵感来自 [Cursor](https://cursor.sh) 的用户画像设计
- 间隔复习机制基于艾宾浩斯遗忘曲线研究

---

## 开源协议

MIT

---

<p align="center">
  <sub>由人类 + Claude 协作构建 🧠 专为 AI Agent 扩展而设计</sub>
</p>
