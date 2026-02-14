---
name: ai-daily-digest
description: "从 Karpathy 推荐的 90 个顶级技术博客抓取最新文章，AI 多维评分筛选，生成每日精选日报。支持 Kimi/GLM/GPT 等 OpenAI 兼容 API。触发命令: /digest"
---

# AI Daily Digest

从 Karpathy 推荐的 90 个热门技术博客中抓取最新文章，通过 AI 评分筛选，生成每日精选摘要。

## 命令

### `/digest`

运行每日摘要生成器。

---

## 环境变量

| 变量 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `LLM_API_KEY` | ✅ | - | API Key（Kimi/GLM/GPT 等） |
| `LLM_API_URL` | ❌ | `https://api.moonshot.cn/v1/chat/completions` | API 端点 |
| `LLM_MODEL` | ❌ | `moonshot-v1-8k` | 模型名称 |

### 预设配置

**Kimi（默认）**:
```bash
export LLM_API_URL="https://api.moonshot.cn/v1/chat/completions"
export LLM_API_KEY="your-kimi-api-key"
export LLM_MODEL="moonshot-v1-8k"
```

**GLM-4**:
```bash
export LLM_API_URL="https://open.bigmodel.cn/api/paas/v4/chat/completions"
export LLM_API_KEY="your-glm-api-key"
export LLM_MODEL="glm-4-flash"
```

**GPT**:
```bash
export LLM_API_URL="https://api.openai.com/v1/chat/completions"
export LLM_API_KEY="your-openai-api-key"
export LLM_MODEL="gpt-4o-mini"
```

---

## 脚本目录

| 脚本 | 用途 |
|------|------|
| `scripts/digest.ts` | 主脚本 - RSS 抓取、AI 评分、生成摘要 |

---

## 交互流程

### Step 1: 收集参数

```
question({
  questions: [
    {
      header: "时间范围",
      question: "抓取多长时间内的文章？",
      options: [
        { label: "24 小时", description: "仅最近一天" },
        { label: "48 小时 (Recommended)", description: "最近两天，覆盖更全" },
        { label: "72 小时", description: "最近三天" },
        { label: "7 天", description: "一周内的文章" }
      ]
    },
    {
      header: "精选数量",
      question: "AI 筛选后保留多少篇？",
      options: [
        { label: "10 篇", description: "精简版" },
        { label: "15 篇 (Recommended)", description: "标准推荐" },
        { label: "20 篇", description: "扩展版" }
      ]
    },
    {
      header: "输出语言",
      question: "摘要使用什么语言？",
      options: [
        { label: "中文 (Recommended)", description: "摘要翻译为中文" },
        { label: "English", description: "保持英文原文" }
      ]
    }
  ]
})
```

### Step 2: 执行脚本

```bash
cd ${SKILL_DIR}

export LLM_API_KEY="<key>"
export LLM_API_URL="https://api.moonshot.cn/v1/chat/completions"
export LLM_MODEL="moonshot-v1-8k"

npx tsx scripts/digest.ts \
  --hours <timeRange> \
  --top-n <topN> \
  --lang <zh|en> \
  --output ./output/digest-$(date +%Y%m%d).md
```

### Step 3: 结果展示

**成功时**：
- 📁 报告文件路径
- 📊 简要摘要：扫描源数、抓取文章数、精选文章数
- 🏆 **今日精选 Top 3 预览**

**报告结构**（Telegram 友好格式）：
1. **📝 今日看点** — AI 归纳的 3-5 句宏观趋势总结
2. **🏆 今日必读 Top 3** — 标题、链接、摘要、推荐理由、关键词
3. **📊 数据概览** — 统计 + 高频话题
4. **分类文章列表** — 按 6 大分类分组展示

---

## 参数映射

| 交互选项 | 脚本参数 |
|----------|----------|
| 24 小时 | `--hours 24` |
| 48 小时 | `--hours 48` |
| 72 小时 | `--hours 72` |
| 7 天 | `--hours 168` |
| 10 篇 | `--top-n 10` |
| 15 篇 | `--top-n 15` |
| 20 篇 | `--top-n 20` |
| 中文 | `--lang zh` |
| English | `--lang en` |

---

## 环境要求

- `tsx` 运行时（通过 `npx tsx` 自动安装）
- LLM_API_KEY 环境变量
- 网络访问（需要能访问 RSS 源和 LLM API）

---

## Cron 定时任务

可配置 OpenClaw Cron 每日自动生成：

```json
{
  "name": "daily-digest",
  "schedule": { "kind": "cron", "expr": "0 8 * * *", "tz": "Asia/Shanghai" },
  "payload": {
    "kind": "agentTurn",
    "message": "运行 /digest 命令生成今日技术日报，参数：48小时、15篇、中文"
  },
  "sessionTarget": "isolated"
}
```

---

## 信息源

90 个 RSS 源来自 [Hacker News Popularity Contest 2025](https://refactoringenglish.com/tools/hn-popularity/)，由 [Andrej Karpathy 推荐](https://x.com/karpathy)。

包括：simonwillison.net, paulgraham.com, overreacted.io, gwern.net, krebsonsecurity.com, antirez.com, daringfireball.net 等顶级技术博客。
