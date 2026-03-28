# WeWrite

公众号文章全流程 AI Skill —— 从热点抓取到草稿箱推送，一句话搞定。

兼容 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 和 [OpenClaw](https://github.com/anthropics/openclaw) 的 skill 格式。安装后说「写一篇公众号文章」即可触发完整流程。

## 它能做什么

```
"写一篇公众号文章"
  → 抓热点 → 选题评分 → 框架选择 → 写作（7层去AI痕迹）
  → SEO优化 → AI配图 → 微信排版 → 推送草稿箱
```

首次使用时会通过对话引导你设置公众号风格，之后每次只需一句话。

## 核心能力

| 能力 | 说明 | 实现 |
|------|------|------|
| 热点抓取 | 微博 + 头条 + 百度实时热搜 | `scripts/fetch_hotspots.py` |
| SEO 评分 | 百度 + 360 搜索量化评分 | `scripts/seo_keywords.py` |
| 选题生成 | 10 选题 × 3 维度评分 + 历史去重 | `references/topic-selection.md` |
| 框架生成 | 5 套写作骨架（痛点/故事/清单/对比/热点） | `references/frameworks.md` |
| 文章写作 | 7 层去 AI 痕迹 + 维度随机化 + 风格适配 | `references/writing-guide.md` |
| SEO 优化 | 标题策略 / 摘要 / 关键词 / 标签 | `references/seo-rules.md` |
| 视觉 AI | 封面 3 创意 + 内文 3-6 配图 | `toolkit/image_gen.py` |
| 排版发布 | Markdown → 微信内联样式 HTML → 草稿箱 | `toolkit/cli.py` |
| 效果复盘 | 微信数据分析 API 回填阅读/分享数据 | `scripts/fetch_stats.py` |
| 风格学习 | 从你的人工修改中提取写作偏好 | `scripts/learn_edits.py` |

## 7 层去 AI 痕迹

WeWrite 的写作不是"写完再改"，而是从第一句话开始就像人在写。针对朱雀等 AI 检测工具，在 7 个统计维度同时制造"人味"：

| 层 | 对抗目标 | 手段 |
|---|---------|------|
| 1. 词汇层 | 词汇分布过"中位" | 冷/温/热/野四温度梯度混搭 |
| 2. 句法层 | 句子复杂度均匀 | 破句、自我纠正、长短句剧烈交替 |
| 3. 信息密度层 | 每段信息量均匀 | 高密度段后必跟低密度段 |
| 4. 连贯性打破层 | 过渡太丝滑 | 硬切、跑题再回来、非线性展开 |
| 5. 具体性注入层 | 抽象泛化 | 具体时间/地点/人物/非整数数字 |
| 6. 情绪真实感层 | 情绪平铺 | 情绪弧线（克制→爆发→犹豫） |
| 7. 维度随机化层 | 跨文章模式识别 | 6 维度池随机抽 2-3 个贯穿全文 |

每篇文章写完后执行 8 项自检，不通过则定向重写。

## 安装

### 1. 获取代码

```bash
git clone https://github.com/oaker-io/wewrite.git
```

### 2. 挂载为 Skill

**Claude Code**：

```bash
# 方式 A：添加 skill 路径到设置
# 方式 B：复制到 skills 目录
cp -r wewrite ~/.claude/skills/wewrite
```

**OpenClaw**：

```bash
# 复制到 openclaw skills 目录
cp -r wewrite /path/to/openclaw/skills/wewrite
```

### 3. 安装 Python 依赖

```bash
pip install -r requirements.txt
```

### 4. 配置（可选）

```bash
cp config.example.yaml config.yaml
```

编辑 `config.yaml`，填入：

- **微信公众号** `appid` / `secret` — 推送草稿箱和数据统计需要
- **图片生成 API key** — doubao-seedream 或 OpenAI DALL-E

不配也能用——环境检查会自动识别缺失的 API，走降级路径（生成本地 HTML + 输出图片提示词）。

## 快速开始

```
你：写一篇公众号文章

（首次使用会引导你设置公众号风格，也可以说"用默认的"跳过）

你：写一篇关于 AI Agent 的公众号文章
你：交互模式，写一篇关于效率工具的推文
你：帮我润色一下刚才那篇
你：看看最近文章数据怎么样
```

## 目录结构

```
wewrite/
├── SKILL.md                # Skill 主文件（Agent 读取并执行）
├── config.example.yaml     # API 配置模板
├── style.example.yaml      # 风格配置模板（首次使用时自动生成 style.yaml）
├── requirements.txt
│
├── scripts/                # 数据采集与学习
│   ├── fetch_hotspots.py     # 多平台热点抓取
│   ├── seo_keywords.py       # SEO 关键词分析
│   ├── fetch_stats.py        # 微信文章数据回填
│   ├── build_playbook.py     # 从历史文章生成写作 Playbook
│   └── learn_edits.py        # 学习人工修改
│
├── toolkit/                # Markdown → 微信工具链
│   ├── cli.py                # CLI（preview / publish / themes）
│   ├── converter.py          # Markdown → 内联样式 HTML
│   ├── theme.py              # YAML 主题引擎
│   ├── publisher.py          # 微信草稿箱 API
│   ├── wechat_api.py         # access_token / 图片上传
│   ├── image_gen.py          # AI 图片生成（doubao / OpenAI）
│   └── themes/               # 4 套预置排版主题
│
├── references/             # Agent 按需读取的参考文档
│   ├── writing-guide.md      # 写作规范 + 7 层去 AI 痕迹
│   ├── frameworks.md         # 5 种写作框架
│   ├── topic-selection.md    # 选题评估规则
│   ├── seo-rules.md          # 微信 SEO 规则
│   ├── visual-prompts.md     # 视觉 AI 提示词规范
│   ├── wechat-constraints.md # 微信平台技术限制
│   └── style-template.md    # 风格配置字段说明
│
├── output/                 # 生成的文章（运行时产生）
├── corpus/                 # 历史文章语料（可选，用于 Playbook 学习）
└── lessons/                # 修改记录（自动生成）
```

运行时自动生成的文件（不入 git）：`style.yaml`、`history.yaml`、`playbook.md`、`corpus/`、`lessons/`

## 风格配置

首次使用时 Agent 会通过对话引导你生成 `style.yaml`。你也可以手动创建：

```bash
cp style.example.yaml style.yaml
```

核心字段：

```yaml
name: "公众号名称"
industry: "行业"
topics:
  - "方向1"
  - "方向2"
tone: "写作风格描述"
theme: "professional-clean"  # 排版主题
```

详见 `references/style-template.md`。

## 排版主题

| 主题 | 风格 |
|------|------|
| `professional-clean` | 专业简洁（默认） |
| `tech-modern` | 科技风（蓝紫渐变） |
| `warm-editorial` | 暖色编辑风 |
| `minimal` | 极简黑白 |

预览：`python3 toolkit/cli.py themes`

## 图片生成

通过 `config.yaml` 切换 provider：

| Provider | 适用场景 | 获取 Key |
|----------|---------|----------|
| `doubao` | 中文提示词效果好，国内快 | [火山引擎 Ark](https://console.volcengine.com/ark) |
| `openai` | DALL-E 3，国际通用 | [OpenAI](https://platform.openai.com) |

不配置时自动跳过生图，输出提示词供手动生成。

## 风格学习

WeWrite 能从你的修改中学习写作偏好：

1. **导入历史文章**：把 `.md` 文件放入 `corpus/`，Agent 会提取风格特征生成 `playbook.md`
2. **学习修改**：对 Agent 说"学习我的修改"，它会 diff 你的改稿，积累 5 次后自动更新 Playbook
3. **Playbook 优先**：一旦生成，Playbook 中的规则优先于通用写作规范

## Toolkit 独立使用

即使不用 Agent，工具链也可以独立使用：

```bash
# 预览 Markdown → 微信 HTML
python3 toolkit/cli.py preview article.md --theme professional-clean

# 发布到微信草稿箱
python3 toolkit/cli.py publish article.md --cover cover.png --title "文章标题"

# 抓热点
python3 scripts/fetch_hotspots.py --limit 20

# SEO 分析
python3 scripts/seo_keywords.py --json "AI大模型" "科技股"

# AI 生图
python3 toolkit/image_gen.py --prompt "描述" --output cover.png --size cover
```

## 工作流程

```
Step 0  环境检查（静默通过或引导修复，设置降级标记）
  ↓
Step 1  加载风格配置（不存在则进入 Onboard）
  ↓
Step 2  热点抓取（降级：WebSearch）
  ↓
Step 2.5 历史去重 + SEO 数据
  ↓
Step 3  选题生成（10 选题 × 评分）
  ↓
Step 3.5 框架选择（5 套骨架）
  ↓
Step 4  维度随机化 + 文章写作（7 层去 AI 痕迹）
  ↓
Step 5  SEO 优化 + 去 AI 逐层验证（8 项自检）
  ↓
Step 6  视觉 AI（封面 + 内文配图）
  ↓
Step 7  排版推送（降级：本地 HTML）
  ↓
Step 7.5 写入发布历史
  ↓
Step 8  回复用户
```

默认全自动。说"交互模式"可在选题/框架/配图处暂停确认。

## License

MIT
