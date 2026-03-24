---
name: autoresearch
description: >-
  Use when user wants to optimize, improve, benchmark, or evaluate a skill's prompt.
  Triggers on "optimize skill", "improve skill prompt", "benchmark skill", "eval skill",
  "run autoresearch", "tune prompt", "prompt optimization", "skill evaluation",
  "A/B test prompt", "find best prompt", "auto-improve skill".
  Runs automated prompt experiments using the Karpathy autoresearch pattern.
license: MIT
metadata:
  author: zning1994
  version: "1.0.0"
---

# autoresearch

Autonomous prompt optimization for AI agent skills. Runs controlled experiments to find better prompt variants using the [Karpathy autoresearch pattern](https://github.com/karpathy/autoresearch): generate hypothesis, mutate prompt, evaluate, repeat.

## When to use

- 用户说"优化一下这个 skill" / User says "optimize this skill's prompt"
- 用户要对比不同 prompt 版本的效果 / User wants to benchmark prompt variants
- 用户说"run autoresearch on X" / "eval skill X" / "improve skill X"
- 用户对 skill 输出质量不满，想系统性改进 / User is unhappy with skill output quality and wants systematic improvement

**Do not use:**
- 一次性的小改动（直接改 prompt 即可） / One-off prompt tweaks — just edit the prompt directly
- 调试某个特定失败 case / Debugging a specific failure — investigate the root cause instead
- Skill 脚本本身有 bug（代码逻辑问题不是 prompt 问题） / Skill script has a bug — fix the code, not the prompt

## Requirements

- Python 3.10+
- `autoresearch.py` script in the skill directory
- LLM API access (MiniMax, OpenAI, or Anthropic)
- Target skill must have a prompt file (SKILL.md, SYSTEM.md, or similar)

## Procedure

### Step 1: Gather context

Before running, you need:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--target` | Path to the skill directory or prompt file to optimize | `../workspace/skills/brain-search/SKILL.md` |
| `--evals` | Path to eval definition JSON file | `eval.json` |
| `--provider` | LLM provider for running experiments | `minimax` (default), `openai`, `anthropic` |
| `--runs` | Number of runs per experiment (statistical significance) | `5` (default) |
| `--max-experiments` | Maximum experiments before stopping | `30` (default) |
| `--dashboard` | Open live results dashboard in browser | flag, no value |

### Step 2: Create eval.json

Define test inputs and evaluation criteria. Each eval is a binary pass/fail check.

```json
{
  "test_inputs": [
    "search for latest AI agent frameworks",
    "find news about LLM inference optimization",
    "搜一下 transformer 架构的最新进展"
  ],
  "evals": [
    {
      "name": "has_sources",
      "type": "rule",
      "rule": "regex",
      "pattern": "(https?://|Source:|来源:)"
    },
    {
      "name": "no_hallucinated_urls",
      "type": "rule",
      "rule": "banned_phrases",
      "phrases": ["example.com", "placeholder.url"]
    },
    {
      "name": "sufficient_detail",
      "type": "rule",
      "rule": "word_count",
      "min": 50,
      "max": 500
    },
    {
      "name": "contains_summary",
      "type": "rule",
      "rule": "contains",
      "values": ["summary", "key findings", "结论"]
    },
    {
      "name": "no_apology_prefix",
      "type": "rule",
      "rule": "not_contains",
      "values": ["I apologize", "I'm sorry, but"]
    },
    {
      "name": "actionable_output",
      "type": "llm",
      "question": "Does the response provide actionable information the user can immediately use (links, specific facts, concrete next steps)?",
      "pass_description": "The response contains specific actionable items like URLs, concrete facts, or clear next steps",
      "fail_description": "The response is vague, generic, or lacks specific actionable information"
    }
  ]
}
```

**Rule types:**

| Rule | Parameters | Description |
|------|-----------|-------------|
| `regex` | `pattern` | Pass if regex matches output |
| `banned_phrases` | `phrases` (list) | Pass if NONE of the phrases appear |
| `word_count` | `min`, `max` (optional) | Pass if word count is within range |
| `contains` | `value` | Pass if value appears in output (case-insensitive) |
| `not_contains` | `value` | Pass if value does NOT appear in output (case-insensitive) |

**LLM eval type:**

| Field | Description |
|-------|-------------|
| `question` | What to ask the judge LLM about the output |
| `pass` | Description of what a passing output looks like |
| `fail` | Description of what a failing output looks like |

See `eval-guide.md` for detailed guidance on writing effective evals.

### Step 3: Run

```bash
python autoresearch.py \
  --target ../workspace/skills/brain-search/SKILL.md \
  --evals eval.json \
  --provider minimax \
  --runs 5 \
  --max-experiments 30 \
  --dashboard
```

### Step 4: Monitor progress

The script writes results to `results.tsv` in the working directory. Each row is one experiment:

```
experiment_id  parent_id  mutation_description  avg_score  pass_rate  evals_detail  prompt_diff
```

Read the file to check progress:
```bash
cat results.tsv | head -20
```

### Step 5: Present results

When the script finishes (or hits `--max-experiments`), report to the user:
1. **Best variant** — highest avg_score, with the mutation description
2. **Score improvement** — baseline vs best (e.g., "62% -> 88% pass rate")
3. **Key mutations that helped** — which changes moved the needle
4. **Recommended prompt** — the winning prompt text or diff

## Example: optimizing brain-search

```
User: brain-search 的搜索结果经常缺少来源链接，帮我优化一下

1. 确认目标:
   - target: skills/brain-search/SKILL.md
   - 问题: 输出缺少来源链接

2. 创建 eval.json:
   {
     "test_inputs": [
       "search for latest news on OpenAI",
       "搜一下最新的 AI 芯片进展",
       "find recent papers on RAG optimization",
       "what happened with Anthropic this week",
       "查查 GPU 价格趋势"
     ],
     "evals": [
       {
         "name": "has_urls",
         "type": "rule",
         "rule": "regex",
         "pattern": "https?://[^\\s]+"
       },
       {
         "name": "min_2_sources",
         "type": "rule",
         "rule": "regex",
         "pattern": "https?://[^\\s]+.*https?://[^\\s]+"
       },
       {
         "name": "structured_output",
         "type": "llm",
         "question": "Is the output well-structured with clear sections (e.g., bullet points, numbered list, or headers)?",
         "pass": "Output uses clear structure like bullets, numbers, or headers to organize information",
         "fail": "Output is a wall of text without clear structure"
       },
       {
         "name": "concise",
         "type": "rule",
         "rule": "word_count",
         "min": 80,
         "max": 400
       }
     ]
   }

3. 运行:
   python autoresearch.py \
     --target ../workspace/skills/brain-search/SKILL.md \
     --evals eval.json \
     --runs 5 \
     --max-experiments 20

4. 报告结果:
   "经过 18 轮实验，最佳变体将来源链接出现率从 40% 提升到 95%。
    关键改动: 在 Procedure 第 2 步加入 '每条结果必须附原始 URL' 的明确指令。"
```

## Failure handling

| Issue | Action |
|-------|--------|
| LLM API rate limit | Script auto-retries with backoff; if persistent, reduce `--runs` |
| Target file not found | Check path, must be readable prompt/skill file |
| All experiments score 0 | Evals may be too strict — review eval definitions, loosen criteria |
| Script crashes mid-run | Results already written to `results.tsv` are preserved; re-run continues |

## Gotchas

- 每次实验会调用 LLM 多次（runs x test_inputs x llm_evals），注意 API 用量 / Each experiment makes multiple LLM calls — watch API usage
- LLM eval 本身有噪声，`--runs` 设高一点（5+）才有统计意义 / LLM evals are noisy, use 5+ runs for statistical significance
- Rule evals 比 LLM evals 更稳定、更便宜，优先用 rule / Rule evals are more stable and cheaper — prefer them
- Baseline 分数太低（< 20%）说明 eval 定义可能有问题，先修 eval / If baseline score is very low, fix evals first
- 优化 prompt 不能解决架构问题（比如搜索 API 本身返回差结果） / Prompt optimization cannot fix architectural issues
