新用户首次使用，执行系统初始化。请按以下步骤执行：

## 第一步：检查项目完整性

确认以下文件存在（它们应该随仓库一起提交）：
- `CLAUDE.md` — Agent 指令
- `data/hot100_problems.json` — 100 题完整数据
- `references/index.json` — 题解索引映射
- `references/LeetCode_Hot100_Python/` — 参考题解
- `references/leetcode-hot-100-master/` — 参考题解
- `.claude/commands/practice.md` — 练习命令
- `.claude/commands/review.md` — Review 命令
- `.claude/commands/hint.md` — 提示命令

如果有缺失，告知用户哪些文件缺失。

## 第二步：创建目录结构

```bash
mkdir -p reports workspace
```

## 第三步：生成 data/hot100_index.json

从 `data/hot100_problems.json` 生成轻量索引，所有题目初始化为等权重：

用 Python 脚本执行：
```python
import json

with open('data/hot100_problems.json') as f:
    data = json.load(f)

index = {
    "meta": {"total": 100, "done": 0, "last_updated": "今天日期"},
    "problems": {}
}

for p in data["problems"]:
    lid = str(p["leetcode_id"])
    index["problems"][lid] = {
        "t": p["title"],
        "d": p["difficulty"][0],  # E/M/H
        "tags": p["tags"][:3],
        "ct": p["codetop_frequency"],
        "pw": 1.0, "bw": 1.0,
        "att": 0, "cor": 0,
        "last": None, "weak": []
    }

with open('data/hot100_index.json', 'w') as f:
    json.dump(index, f, ensure_ascii=False, indent=2)
```

## 第四步：生成 soul.md

创建初始用户画像模板：

```markdown
# Soul — 用户画像

## 基本信息
- 刷题目标: （待填写：秋招/日常练习/竞赛备战）
- 编程语言: Python
- 当前水平: （待评估）
- 开始日期: {今天日期}

## 性格与偏好
- 刷题风格: （待观察）
- 沟通偏好: （待观察）
- 时间习惯: （待观察）
- 特殊偏好: （待观察）

## 算法能力雷达
- 哈希表: ★★☆☆☆ (待评估)
- 双指针: ★★☆☆☆ (待评估)
- 滑动窗口: ★★☆☆☆ (待评估)
- 二叉树: ★★☆☆☆ (待评估)
- 动态规划: ★★☆☆☆ (待评估)
- 回溯: ★★☆☆☆ (待评估)
- 链表: ★★☆☆☆ (待评估)
- 栈/队列: ★★☆☆☆ (待评估)
- 图论: ★★☆☆☆ (待评估)
- 贪心: ★★☆☆☆ (待评估)
- 二分查找: ★★☆☆☆ (待评估)
- 堆: ★★☆☆☆ (待评估)

## 高频薄弱点 TOP 5
（暂无数据，刷题后自动更新）

## 成长轨迹
- {今天日期}: 系统初始化，开始刷题之旅

## 近期状态
- 连续打卡: 0 天
- 本周刷题: 0 题
- 本周正确率: N/A
- 情绪状态: 准备出发
```

## 第五步：询问用户信息

初始化完成后，问用户几个问题来填充 soul.md：

1. "你的刷题目标是什么？（秋招/日常练习/竞赛备战/其他）"
2. "你觉得自己目前算法水平怎么样？（小白/初级/中级/高级）"
3. "你希望我怎么跟你互动？（严格教练/鼓励为主/就事论事）"

根据回答更新 soul.md 的对应字段。

## 第六步：确认完成

```
系统初始化完成！

已生成：
✅ data/hot100_index.json — 100 题全部就绪，等权重
✅ soul.md — 用户画像已创建
✅ reports/ — 报告目录已创建
✅ workspace/ — 工作区已创建

现在可以开始了：
- 说 "练习" 或 /practice 开始刷题
- 说 "/practice 1" 直接刷两数之和
- 说 "/practice 链表" 刷链表专题
```
