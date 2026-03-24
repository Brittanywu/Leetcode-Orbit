用户想要练习。请严格按照以下步骤执行：

## 第零步：判断用户是否指定了题目

解析用户输入，判断是否指定了具体题目。支持以下格式：
- `/practice 1` 或 `/practice #1` → LeetCode 题号 1
- `/practice 两数之和` → 按中文标题匹配
- `/practice two sum` → 按英文标题匹配（不区分大小写）
- `/practice 哈希` 或 `/practice 链表` → 按分类/标签筛选（见第三步变体）
- `我想刷第 206 题` / `来一道反转链表` → 自然语言中提取题号或标题

**如果用户指定了具体题目** → 跳过第二、三、四步，直接到第五步生成文件。
**如果用户指定了分类/标签** → 第三步改为从该分类中选题（见下方变体）。
**如果用户没有指定** → 正常执行完整流程。

## 第一步：加载状态

读取以下两个文件（它们应该已在上下文中，但若不在则读取）：
- `data/hot100_index.json` — 获取所有题目的权重、做题次数、薄弱点
- `soul.md` — 获取用户画像（能力雷达、偏好）

## 第二步：艾宾浩斯复习提醒

检查 `hot100_index.json` 中所有 `last` 不为 null 且 `cor > 0` 的题目。
计算距今天数 `days_since`，如果 days_since 恰好等于 1, 2, 4, 7, 15, 30 中的任何一个，生成复习提醒。

如果有复习提醒，先展示：
```
📅 复习提醒:
  - LeetCode {id}. {title} (距上次 {days}天，建议复习)
```

## 第三步：选题（1随机 + 1加权）

> **变体 — 按分类/标签筛选**: 如果用户指定了分类（如"链表"、"动态规划"），
> 则将下面的"全部题目"替换为该分类下的题目子集，其余逻辑不变。

### 题目A — 随机题（偏向未做过的题）
1. 从 index 中筛选 `att == 0` 的题目列表 `undone`
2. 如果 `undone` 不为空，以 70% 概率从中随机选一题；否则从全部题目中随机选
3. 如果 `undone` 为空，从全部题目中随机选

### 题目B — 加权推荐题
1. 从全部题目中排除题目A
2. 按 `pw`（practice_weight）做加权随机抽样选一题
3. 如果该题有 `weak` 标签，在展示时特别标注

确保 A ≠ B。

## 第四步：展示两题供选择

格式如下（使用 emoji 区分）：
```
今天为你准备了两道题：

🎲 随机题: LeetCode {id}. {title} ({difficulty}) [{tags}]
   {description 前50字}...

🎯 推荐题: LeetCode {id}. {title} ({difficulty}) [{tags}]
   {description 前50字}...
   {如果有weak标签: "⚠️ 上次薄弱点: {weak}"}

想刷哪题？输入题号即可。
```

## 第五步：生成题目文件

用户选择后：
1. 从 `data/hot100_problems.json` 中读取该题的完整数据（只读这一题，不读整个文件）
2. 生成 `workspace/current_problem.py`，格式如下：

```python
"""
LeetCode {leetcode_id}. {title}
难度: {difficulty}
标签: {tags}
链接: {url}

【题目描述】
{description}

【示例】
{examples formatted}

【提示/约束】
{constraints}
"""

from typing import List, Optional

# ===== 请在下方编写你的解法 =====

class Solution:
    {function_signature}
        # 在这里写你的代码
        pass


# ===== 测试用例 (可自行添加) =====
if __name__ == "__main__":
    sol = Solution()
    # 测试用例1
    print(sol.{function_name}({test_input_1}))  # 期望: {expected_1}
    # 测试用例2
    print(sol.{function_name}({test_input_2}))  # 期望: {expected_2}
    # 测试用例3
    print(sol.{function_name}({test_input_3}))  # 期望: {expected_3}
```

3. 同时在 `workspace/` 下创建一个隐藏的元数据文件 `workspace/.current_meta.json`：
```json
{
  "leetcode_id": {id},
  "title": "{title}",
  "mode": "practice",
  "started_at": "{ISO timestamp}",
  "test_cases": [... from problems.json ...]
}
```

4. 告诉用户：
```
题目文件已生成在 workspace/current_problem.py
打开文件开始写代码吧，写完后告诉我 "review" 即可！
```

## 注意事项
- 选题时用 Python 的 random 模块实现随机性（通过 bash 运行 python 一行命令）
- 读取 hot100_problems.json 时只提取当前题的数据，不要一次性读入全部
- 生成文件后，problems.json 的内容不保留在上下文中
