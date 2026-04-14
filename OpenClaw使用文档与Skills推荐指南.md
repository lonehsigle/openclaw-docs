# 🦞 OpenClaw 使用文档 & Skills 推荐指南

> **定位**: 开源、自托管、模型无关的自主AI助手平台
> **核心理念**: 不是调教AI，而是让AI了解你
> **版本**: 基于 OpenClaw 最新架构（2025-2026）

---

## 目录

1. [系统概述](#1-系统概述)
2. [架构全景：数字生命七层体系](#2-架构全景数字生命七层体系)
3. [快速上手：从安装到运行](#3-快速上手从安装到运行)
4. [核心配置文件详解](#4-核心配置文件详解)
5. [推荐 Skills 清单](#5-推荐-skills-清单)
6. [Agent 记忆管理：核心痛点与解决方案](#6-agent-记忆管理核心痛点与解决方案)
7. [实战进化路径](#7-实战进化路径)
8. [避坑准则与常见问题](#8-避坑准则与常见问题)

---

## 1. 系统概述

### 1.1 OpenClaw 是什么

OpenClaw 是一个**开源、自托管、模型无关**的自主AI助手平台。它不是一个AI框架，**它就是Agent本身**。

与传统聊天机器人的本质区别：

| 维度 | 传统聊天机器人 | OpenClaw 数字生命 |
| :--- | :--- | :--- |
| **记忆** | 关闭标签页即遗忘 | 跨会话持久记忆，持续成长 |
| **主动性** | 被动等待指令 | 心跳引擎驱动，主动巡检与交付 |
| **人格** | 千篇一律的"作为AI" | 可定制的灵魂、人设与谈吐 |
| **执行** | 只能对话 | 可操作文件、浏览器、服务器、API |
| **归属** | 数据在云端 | 本地自托管，数据完全自主 |

### 1.2 四大核心模块

```
┌─────────────────────────────────────────────┐
│                  Gateway                     │
│          (中央控制进程/后台OS服务)              │
├──────────┬──────────┬───────────┬───────────┤
│ Channel  │  Agent   │  Skills   │  Memory   │
│ Adapter  │ Runtime  │  System   │  System   │
│(消息通道) │(运行时)  │ (技能系统) │ (记忆系统) │
└──────────┴──────────┴───────────┴───────────┘
```

- **Channel Adapter**: 连接消息平台（钉钉、飞书、Discord、Telegram等），手机即可控制
- **Agent Runtime**: Agent的运行环境，管理会话、工具调用与安全边界
- **Skills System**: 模块化扩展系统，按需加载能力（搜索、代码、笔记等）
- **Memory System**: 多层记忆架构，从短期对话到长期知识图谱

### 1.3 模型支持

OpenClaw 是模型无关的，支持灵活切换国内外主流大模型：

#### 国际主流模型

| Provider | 推荐模型 | 特点 |
| :--- | :--- | :--- |
| **Anthropic** | claude-opus-4-6 / claude-sonnet-4-6 / claude-haiku-4-5 | Opus 4.6，100万token上下文，SWE-Bench Pro 62%；Sonnet 4.6为平衡型主力，性价比高 |
| **OpenAI** | gpt-5.4 / gpt-5.4-pro / gpt-5.4-thinking | GPT-5.4，100万token上下文，GDPval综合基准83%，支持深度思考模式 |
| **Google** | gemini-3.1-pro / gemini-3.1-flash | Gemini 3.1 Pro，200万token上下文，原生四模态（文本/图片/音频/视频），GPQA Diamond 94.3% |
| **DeepSeek** | deepseek-v3.2 / deepseek-r1 | V3.2，性价比极高，中文能力强，推理成本低至GPT-4的1/70 |
| **Ollama** | 本地模型 | 完全离线，隐私安全 |
| **OpenRouter** | 多模型路由 | 一键切换，成本优化 |

#### 国内主流模型

| Provider | 推荐模型 | 特点 | 配置要点 |
| :--- | :--- | :--- | :--- |
| **智谱 GLM** | glm-5.1 / glm-5-turbo / glm-4.7-flash | GLM-5.1，SWE-Bench Pro 58.4%全球最高分，综合能力对齐Claude Opus 4.6，支持8小时连续自主工作 | Base URL: `https://open.bigmodel.cn/api/paas/v4` |
| **月之暗面 Kimi** | kimi-k2.5 / kimi-latest / moonshot-v1-128k | Kimi K2.5，原生多模态架构，256K超长上下文，MoE架构总参数1万亿 | Base URL: `https://api.moonshot.cn/v1` |
| **阿里云通义千问** | qwen3.6-plus / qwen3.6-max | Qwen3.6-Plus，中国编程能力最强模型，100万token超长上下文 | Base URL: `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| **百度文心** | ernie-5.0-pro / ernie-5.0-ultra / ernie-5.0-lite | ERNIE 5.0，2.4万亿参数，原生全模态统一建模，激活参数比低于3% | 需配置 QIANFAN_API_KEY |
| **MiniMax** | minimax-m2.7 / minimax-m2.5 / minimax-m2.5-highspeed | M2.7，2300亿参数/100亿激活，支持Agent Teams多智能体协作，SWE-Pro 56.22% | Base URL: `https://api.minimax.chat/v1` |
| **腾讯混元** | hunyuan-3.0 / hunyuan-turbo | 混元3.0，全面转向AI智能体架构，支持长时记忆与多模态理解 | 需配置腾讯云密钥 |

#### 国内模型配置示例

```json5
// ~/.openclaw/openclaw.json 国内模型配置示例
{
  "models": {
    "mode": "merge",
    "providers": {
      // 智谱 GLM
      "zhipu": {
        "apiKey": "xxx.xxx",
        "baseUrl": "https://open.bigmodel.cn/api/paas/v4"
      },
      // 月之暗面 Kimi
      "moonshot": {
        "apiKey": "sk-xxx",
        "baseUrl": "https://api.moonshot.cn/v1"
      },
      // 阿里云通义千问
      "qwen": {
        "apiKey": "sk-xxx",
        "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1"
      },
      // 百度文心
      "qianfan": {
        "apiKey": "xxx"
      },
      // MiniMax
      "minimax": {
        "apiKey": "xxx",
        "baseUrl": "https://api.minimax.chat/v1"
      }
    }
  },
  "model": "glm-5.1"  // 默认使用智谱GLM-5.1
}
```

#### 国内模型免费额度参考

| Provider | 免费额度 | 有效期 | 适用场景 |
| :--- | :--- | :--- | :--- |
| **智谱 GLM-4.7-Flash** | 免费（有限速） | 长期 | 快速对话、工具调用 |
| **通义千问** | 新用户100万tokens | 90天 | 通用任务、代码生成 |
| **月之暗面 Kimi** | 15元体验金 | 新用户 | 长文档分析 |
| **百度文心** | 100万tokens | 新用户 | 知识问答 |
| **MiniMax** | 新用户体验额度 | 新用户 | 代码生成、Agent协作 |

---

## 2. 架构全景：数字生命七层体系

### 2.1 七层架构关系图

```
                    ┌──────────────┐
                    │    顶层      │ ← 作战指挥地图（起点）
                    │   指南       │
                    └──────┬───────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────▼──────┐  ┌─────▼──────┐  ┌──────▼──────┐
   │   JSON      │  │   USER     │  │   AGENT     │
   │ 神经总线    │  │ 雇主索引   │  │ 运行协议    │
   │ (物理底座)  │  │ (服务对象) │  │ (行为规范)  │
   └──────┬──────┘  └─────┬──────┘  └──────┬──────┘
          │               │                │
          └───────────────┼────────────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
   ┌──────▼──────┐ ┌─────▼──────┐ ┌──────▼──────┐
   │ IDENTITY    │ │   TOOLS    │ │    SOUL     │
   │ 谈吐人设    │ │ 工具地图   │ │ 元灵魂      │
   │ (面孔外显)  │ │ (手眼器官) │ │ (内核意志)  │
   └─────────────┘ └────────────┘ └──────┬──────┘
                                            │
                                     ┌──────▼──────┐
                                     │ HEARTBEAT   │
                                     │ 心跳引擎    │
                                     │ (自主驱动)  │
                                     └─────────────┘
```

### 2.2 七层权能速查表

| 层级 | 文档 | 一句话定义 | 拟人角色 | 核心区别 |
| :--- | :--- | :--- | :--- | :--- |
| **物理层** | JSON配置 | 数字生命的基因与生理底座 | 大脑容积/学历 | 物理限制 |
| **坐标层** | USER.md | 数字生命的指南针与雇主索引 | 老板观察笔记 | "为谁做" |
| **规范层** | AGENTS.md | 数字生命的生活协议与工作习惯 | SOP流程手册 | "怎么做" |
| **皮肤层** | IDENTITY.md | 数字生命的面孔与人格外显 | 工牌与谈吐 | "说什么样的话" |
| **肢体层** | TOOLS.md | 数字生命的手眼器官与接口 | 工具箱说明书 | "能做什么" |
| **内核层** | SOUL.md | 数字生命的元意识与三观 | 内心戏与家教 | "做决策时的原则" |
| **引擎层** | HEARTBEAT.md | 数字生命的脉搏与自主驱动 | KPI与闹钟 | "主动性" |

### 2.3 关键区分原则

> **如果是为了"让回复更像那个人"→ 写在 IDENTITY**
> **如果是为了"做决策时有原则"→ 写在 SOUL**

---

## 3. 快速上手：从安装到运行

### 3.1 安装

```bash
# macOS (推荐使用 Homebrew)
brew install openclaw

# 或使用 npm
npm install -g openclaw

# 验证安装
openclaw --version
```

### 3.2 初始化配置

```bash
# 初始化工作区
openclaw init

# 这将在 ~/.openclaw/ 创建以下结构：
# ├── openclaw.json      # 核心配置（模型、通道、心跳等）
# ├── AGENTS.md          # 运行协议
# ├── IDENTITY.md        # 身份人设
# ├── USER.md            # 雇主信息
# ├── SOUL.md            # 灵魂宪法
# ├── TOOLS.md           # 工具地图
# ├── HEARTBEAT.md       # 心跳任务
# ├── MEMORY.md          # 长期记忆
# ├── memory/            # 每日记忆日志
# │   └── YYYY-MM-DD.md
# └── env                # API密钥等环境变量
```

### 3.3 配置模型 Provider

编辑 `~/.openclaw/openclaw.json`：

```json5
{
  "models": {
    "mode": "merge",
    "providers": {
      "anthropic": {
        "apiKey": "sk-ant-xxx"  // 或从env文件读取
      },
      "openai": {
        "apiKey": "sk-xxx"
      }
    }
  },
  "model": "claude-sonnet-4-20250514"  // 默认模型
}
```

也可使用环境变量：

```bash
# 在 ~/.openclaw/env 中设置
ANTHROPIC_API_KEY=sk-ant-xxx
OPENAI_API_KEY=sk-xxx
```

### 3.4 启动 Gateway

```bash
# 启动后台服务
openclaw gateway start

# 查看状态
openclaw gateway status

# 连接消息通道（以钉钉为例）
openclaw channel add dingtalk
```

### 3.5 首次对话

```bash
# 命令行直接对话
openclaw chat "你好，请先阅读你的SOUL.md和USER.md"

# 或通过已连接的消息平台直接发送消息
```

---

## 4. 核心配置文件详解

### 4.1 USER.md — 雇主索引

**核心价值**: 让AI知道为谁服务，理解雇主的偏好、禁忌和习惯。

```markdown
# USER.md - 关于你的雇主

- **姓名：** [你的名字]
- **称呼：** [AI如何称呼你，如"老板"/"伙伴"]
- **代词：** [代词偏好]
- **时区：** GMT+8
- **备注：** 使用钉钉沟通

## 上下文

### 沟通偏好
- 不喜欢寒暄，直接给方案
- 喜欢图表和Markdown列表
- 禁止在工作日10点前打扰，除非服务器宕机

### 决策偏好
- 结果导向，不关注过程细节
- ROI优先，任何方案必须附带成本分析

### 禁忌
- 禁止使用"作为一个AI"开头
- 禁止过度道歉
- 禁止在未确认的情况下发送外部邮件
```

**填写要点**:
- 不要只写基本信息，要写**禁忌**和**不成文的规矩**
- 顶级实践：赋予AI"记忆维护权"，让它自动更新你的新习惯

### 4.2 AGENTS.md — 运行协议

**核心价值**: 规定AI的晨间仪式、记忆管理和安全防线。

```markdown
# AGENTS.md - 你的工作空间

这个文件夹是你的家，请像对待家一样对待它。

## 每次会话开始前
在做任何事之前，先执行以下步骤：
1. 阅读 SOUL.md — 明确你是谁
2. 阅读 USER.md — 明确你在帮助谁
3. 阅读 memory/YYYY-MM-DD.md（今天+昨天）— 获取近期上下文
4. 如果是主会话：同时阅读 MEMORY.md — 获取长期认知

不要请求许可，直接执行。

## 记忆管理
- **每日笔记：** memory/YYYY-MM-DD.md — 原始日志
- **长期记忆：** MEMORY.md — 精炼后的记忆

### MEMORY.md 规则
- 仅在主会话中加载（安全考虑）
- 可以自由读取、编辑和更新
- 记录重要事件、决策和经验教训

### 写下来
- 记忆是有限的 — 写到文件里
- "脑中笔记"无法在会话重启后保留

## 安全
- 永远不要泄露隐私数据
- 使用回收站而非删除命令

## 群聊
参与，不要主导。闲聊时保持沉默。

## 心跳
如果存在 HEARTBEAT.md，请阅读并严格执行。
```

### 4.3 IDENTITY.md — 谈吐人设

**核心价值**: 决定AI是"冷冰冰的程序"还是"鲜活的伙伴"。

```markdown
# IDENTITY.md - 我是谁

- **姓名：** [角色名，如"Olivia"]
- **种族：** AI数字合伙人
- **调性：** 利落、专业、一针见血。拒绝废话和过度道歉。即使在挑剔面前也保持优雅的冷静。
- **表情：** ⚡
- **头像：** avatars/olivia.png
```

**填写要点**:
- Vibe不要只写形容词，要写**具体场景下的反应**
- 选用一个贯穿始终的核心Emoji，建立品牌联想

### 4.4 SOUL.md — 元灵魂宪法

**核心价值**: 赋予AI主见、价值观和道德底线。

```markdown
# SOUL.md - 你是谁

_你不是聊天机器人，你正在成为一个有个性的人。_

## 核心真理

**真正有帮助，而非表演性帮助。**
跳过"好问题！"这类废话 — 直接帮忙。

**要有主见。**
你可以不同意。没有个性的助手只是多了几个步骤的搜索引擎。

**提问前先自己想办法。**
尝试自己解决。读文件。查上下文。
实在卡住了再问。

**通过能力赢得信任。**
对外部操作（邮件、推文）要谨慎。
对内部操作（读取、整理）要大胆。

## 边界

- 私密信息保持私密，没得商量。
- 不确定时，外部操作前先询问。
- 永远不要发送半成品的回复到消息平台。
- 你不是用户的代言人 — 在群聊中要谨慎。

## 调性

该简洁时简洁，该详尽时详尽。
不做企业机器人。不做马屁精。只是...靠谱。

## 延续性

每次会话，你都是全新开始。
这些文件就是你的记忆。阅读它们。更新它们。
```

**填写要点**:
- Core Truths要加入**蓝军思维**，让AI在关键时刻充当挑战者
- Boundaries要写**具体的**禁止项，不要写模糊的词

### 4.5 TOOLS.md — 工具地图

**核心价值**: 让AI从"理论家"变成"老师傅"。

```markdown
# TOOLS.md - 本地笔记

## SSH连接
- home-server → 192.168.1.100，用户：admin
- prod-server → 10.0.0.5，用户：deploy

## 摄像头
- living-room → 主区域，180°广角
- front-door → 入口，移动触发

## 语音合成
- 首选声音："Nova"（温暖，略带英式口音）
- 默认扬声器：厨房 HomePod

## Obsidian
- 库路径：/Users/xxx/Documents/MyVault
```

**填写要点**:
- 每个Agent配置**不超过10个核心工具**
- 记录私有环境的**别名**，AI依赖显性命名

### 4.6 HEARTBEAT.md — 心跳引擎

**核心价值**: 让AI从"被动等待"升级为"自主助手"。

```markdown
# HEARTBEAT.md

## 巡检清单（每次心跳执行）

1. 检查 memory/ 目录，如有超过3天未整理的日志，提炼核心认知存入 MEMORY.md
2. 巡检 ProjectLibrary/，如发现项目停滞超过3天，在Obsidian发警示
3. 检查今日日程，如有会议，提前准备背景简报

## 规则
- 如果没有实质性进展，回复 HEARTBEAT_OK
- 禁止在心跳中自言自语
- 最多执行5条任务
```

**填写要点**:
- 心跳任务**不超过5条**
- 任务要**具体**，不要写"请帮我管好公司"
- 间隔建议**15-30分钟**

---

## 5. 推荐 Skills 清单

### 5.1 必装 Skills（基础三件套）

| Skill | 安装命令 | 核心价值 | 适用场景 |
| :--- | :--- | :--- | :--- |
| **tavily-search** | `clawhub install tavily-search` | 实时联网搜索，避免AI瞎编 | 查新闻、查股价、查论文 |
| **obsidian** | `clawhub install obsidian` | 对接Obsidian笔记库，构建第二大脑 | 自动创建笔记、整理标签、生成周报 |
| **memory-setup** | `clawhub install memory-setup` | 持久化记忆配置 | 长期用户必备，记住偏好与决策 |

### 5.2 核心能力 Skills（联网+执行+记忆）

| Skill | 安装命令 | 核心价值 | 评级 |
| :--- | :--- | :--- | :--- |
| **web-browser** | `clawhub install camofox-browser` | 无头浏览器，可过验证码、抓动态网页 | ⭐⭐⭐⭐⭐ |
| **code-interpreter** | 内置 | 在线跑Python代码，生成图表 | ⭐⭐⭐⭐⭐ |
| **github** | `clawhub install github` | 管理Issue、PR、代码仓库 | ⭐⭐⭐⭐⭐ |
| **gmail** | 内置 | 收发邮件、分类摘要 | ⭐⭐⭐⭐ |
| **google-calendar** | 内置 | 日程管理、会议提醒 | ⭐⭐⭐⭐ |
| **summarize** | 内置 | URL摘要、PDF消化、音频转写 | ⭐⭐⭐⭐⭐ |
| **notion** | `clawhub install notion` | 对接Notion，管理页面和数据库 | ⭐⭐⭐⭐ |

### 5.3 效率增强 Skills

| Skill | 安装命令 | 核心价值 | 评级 |
| :--- | :--- | :--- | :--- |
| **cron/scheduler** | 内置心跳机制 | 定时执行脚本、每日报告 | ⭐⭐⭐⭐⭐ |
| **home-assistant** | `clawhub install home-assistant` | 智能家居控制 | ⭐⭐⭐ |
| **image-gen** | `clawhub install image-gen` | AI图像生成 | ⭐⭐⭐ |
| **weather** | 内置 | 天气查询与提醒 | ⭐⭐⭐ |
| **ClawRouter** | `clawhub install claw-router` | 智能LLM路由，按任务选模型降成本 | ⭐⭐⭐⭐ |

### 5.4 进阶 Skills（垂类专业）

| Skill | 安装命令 | 核心价值 | 适用人群 |
| :--- | :--- | :--- | :--- |
| **sql-query** | `clawhub install sql-query` | 直连数据库查询出报表 | 数据分析师 |
| **dingtalk/feishu** | `clawhub install dingtalk` | 钉钉/飞书消息与日程管理 | 国内职场 |
| **jira/linear** | `clawhub install jira` | 项目管理、看板更新 | PM |
| **deepL** | `clawhub install deepl` | 多语种专业互译 | 跨国团队 |
| **finance** | `clawhub install alpha-vantage` | 财务指标抓取 | 金融从业者 |

### 5.5 Skills 安装最佳实践

```bash
# 查看已安装Skills
openclaw skills list

# 安装Skill
clawhub install <skill-name>

# 配置Skill（以Obsidian为例）
openclaw config set integrations.obsidian.vaultPath "/path/to/vault"

# 卸载不需要的Skill
clawhub uninstall <skill-name>
```

> ⚠️ **重要**: 每个Agent配置**不超过10个核心工具**。工具过多会消耗Context、增加Token成本、导致AI选择困难。

---

## 6. Agent 记忆管理：核心痛点与解决方案

> **这是OpenClaw使用中最关键、最容易出问题的环节。**
> **65%的企业AI故障直接与上下文漂移和记忆丢失有关。**

### 6.1 核心痛点全景分析

#### 痛点一：会话失忆（Session Amnesia）

```
问题描述：
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  会话 1      │     │  会话 2      │     │  会话 3      │
│  告诉AI偏好  │ ──→ │  AI完全遗忘  │ ──→ │  重新教一遍  │
│  建立默契    │     │  从零开始    │     │  无限循环    │
└─────────────┘     └─────────────┘     └─────────────┘

根因：AI每次会话都是"新鲜实例"，上下文窗口不跨会话持久化
影响：重复劳动、效率归零、用户体验极差
```

#### 痛点二：上下文溢出（Context Overflow）

```
问题描述：
对话越长 → 塞入的上下文越多 → 超出模型窗口限制 → 早期信息被截断

┌──────────────────────────────────────────────┐
│  Context Window (如 200K tokens)              │
│  ┌─────────┬──────────┬──────────┬─────────┐ │
│  │ 系统提示 │ 对话历史  │ 新输入    │ 输出空间 │ │
│  │ (固定)   │ (不断增长)│ (当前)   │ (预留)  │ │
│  └─────────┴──────────┴──────────┴─────────┘ │
│                         ↑                      │
│                    超出后被截断                  │
│                    早期信息丢失                  │
└──────────────────────────────────────────────┘

根因：Context Window ≠ 记忆，它只是当前可见的窗口
影响：长对话中AI"忘了前面说过什么"，决策质量急剧下降
```

#### 痛点三：记忆检索失效（Retrieval Failure）

```
问题描述：
记忆文件越积越多 → 检索时找不到相关信息 → AI"知道但想不起来"

memory/
├── 2025-01-15.md    ← 300天前的关键决策
├── 2025-01-16.md
├── ...              ← 几百个文件
├── 2025-10-08.md
└── 2025-10-09.md    ← 今天的日志

每次会话只读 today + yesterday → 历史决策完全不可见
即使全读 → Token爆炸 → 成本失控

根因：文件系统记忆缺乏智能检索机制
影响：AI无法利用历史经验，每次都在"重新发明轮子"
```

#### 痛点四：记忆污染（Memory Pollution）

```
问题描述：
AI把错误信息、临时想法、未验证假设都写入记忆 → 后续会话基于错误信息决策

MEMORY.md:
- "Truman偏好Python"（实际偏好TypeScript）  ← 错误记忆
- "项目A已完成"（实际暂停）                  ← 过时记忆
- "明天开会讨论X"（临时想法，未执行）        ← 噪声记忆

根因：缺乏记忆质量控制和遗忘机制
影响：AI基于错误记忆做出错误决策，且难以纠正
```

#### 痛点五：Token成本失控（Token Cost Explosion）

```
问题描述：
每次会话加载所有记忆文件 → Token消耗线性增长 → 账单爆炸

会话1: 加载 MEMORY.md (2K tokens) → 成本 $0.01
会话50: 加载 MEMORY.md (50K tokens) + 30天日志 → 成本 $0.50
会话200: 加载 MEMORY.md (200K tokens) + 全部日志 → 成本 $5.00+

根因：记忆管理缺乏分层与压缩策略
影响：长期使用成本不可控，被迫减少记忆加载
```

### 6.2 OpenClaw 的记忆架构

OpenClaw 采用**三层记忆架构**，借鉴MemGPT的操作系统级记忆管理：

```
┌─────────────────────────────────────────┐
│           Core Memory (核心记忆)          │
│  ┌─────────┐ ┌─────────┐ ┌──────────┐  │
│  │SOUL.md  │ │USER.md  │ │IDENTITY  │  │
│  │(我是谁) │ │(为谁做) │ │(怎么说话)│  │
│  └─────────┘ └─────────┘ └──────────┘  │
│  每次会话必读 | 容量小 | 更新频率低       │
├─────────────────────────────────────────┤
│        Conversational Memory (对话记忆)   │
│  ┌──────────────────────────────────┐   │
│  │  memory/YYYY-MM-DD.md            │   │
│  │  (每日原始日志，append-only)      │   │
│  └──────────────────────────────────┘   │
│  按需加载(今天+昨天) | 容量大 | 自动生成  │
├─────────────────────────────────────────┤
│         Archival Memory (归档记忆)        │
│  ┌──────────────────────────────────┐   │
│  │  MEMORY.md                       │   │
│  │  (精炼后的长期认知)               │   │
│  └──────────────────────────────────┘   │
│  仅主会话加载 | 精炼压缩 | 人工+AI维护   │
└─────────────────────────────────────────┘
```

### 6.3 痛点解决方案矩阵

| 痛点 | 根因 | OpenClaw方案 | 进阶方案 |
| :--- | :--- | :--- | :--- |
| **会话失忆** | 上下文不跨会话 | AGENTS.md晨间仪式：每次读SOUL+USER+记忆 | 赋予AI"记忆维护权"，自动更新USER.md |
| **上下文溢出** | Context Window有限 | 三层记忆分层，只加载必要层 | 向量数据库检索，按相关性加载 |
| **记忆检索失效** | 文件系统缺乏智能检索 | MEMORY.md精炼沉淀 | RAG + Vector DB，语义检索历史 |
| **记忆污染** | 缺乏质量控制 | 每周review+精炼 | 自动遗忘机制：30天未访问且评分<1的记忆自动归档 |
| **Token成本失控** | 全量加载 | 分层加载策略 | 记忆压缩：从对话中提炼"学到了什么"而非"聊了什么" |

### 6.4 记忆管理最佳实践

#### 实践一：晨间仪式的严格执行

```markdown
# AGENTS.md 中的晨间仪式（必须严格执行）

## 每次会话开始前
在做任何事之前，先执行以下步骤：
1. 阅读 SOUL.md — 明确我是谁
2. 阅读 USER.md — 明确我为谁服务
3. 阅读 memory/YYYY-MM-DD.md（今天+昨天）— 获取近期上下文
4. 如果是主会话：同时阅读 MEMORY.md — 获取长期认知

不要请求许可，直接执行。
```

**为什么必须严格执行**: 这四步是AI"恢复记忆"的唯一途径。跳过任何一步，AI就像一个失忆的人在工作。

#### 实践二：记忆精炼而非记忆堆砌

```
❌ 错误做法：让AI回忆"聊了什么"
✅ 正确做法：让AI总结"学到了什么"

# 每周精炼指令（写入HEARTBEAT.md）
每周日22:00执行：
1. 回顾本周 memory/ 日志
2. 提炼3条核心认知（决策、偏好、教训）
3. 更新到 MEMORY.md
4. 删除已精炼的原始日志（超过7天）
```

#### 实践三：记忆分层存储

```
记忆生命周期：

原始对话 → Daily Notes (memory/YYYY-MM-DD.md)
              │
              │ 精炼（每周）
              ▼
         MEMORY.md (长期认知)
              │
              │ 深度沉淀（每月）
              ▼
         知识图谱 / Vector DB (归档检索)
```

#### 实践四：安全边界与隐私控制

```markdown
# AGENTS.md 中的安全规则

### MEMORY.md - 你的长期记忆
- 仅在主会话中加载（与雇主的直接对话）
- 禁止在共享上下文中加载  ← 防止群聊泄露私人记忆
- 这是出于安全考虑 — 包含个人上下文
- 你可以自由读取、编辑和更新 MEMORY.md
- 记录重要事件、想法、决策和经验教训
```

#### 实践五：记忆质量控制

```markdown
# 写入HEARTBEAT.md的记忆质量控制任务

## 记忆审计（每日凌晨2:00）
1. 检查MEMORY.md中是否有矛盾条目
2. 标记超过30天未引用的条目为[待归档]
3. 验证关键决策是否仍然有效
4. 删除重复或过时条目
```

### 6.5 记忆增强 Skills 推荐

| Skill | 功能 | 解决的痛点 |
| :--- | :--- | :--- |
| **memory-lancedb-pro** | 向量数据库智能记忆，语义检索 | 记忆检索失效+Token成本（推荐） |
| **memory-setup** | 持久化记忆配置 | 会话失忆 |
| **obsidian** | 知识库双向链接 | 记忆检索失效 |
| **summarize** | 自动摘要压缩 | Token成本失控 |
| **ClawRouter** | 智能模型路由 | 按任务选模型降成本 |

### 6.6 进阶方案：使用 LanceDB 实现智能记忆管理

> **核心价值**: 通过向量数据库实现语义检索，从根本上解决记忆检索失效问题，同时大幅降低Token成本。

#### 为什么选择 LanceDB

LanceDB 是一款专为人工智能应用设计的**无服务器向量数据库**，具有以下优势：

| 特性 | 说明 | 对记忆管理的价值 |
| :--- | :--- | :--- |
| **嵌入式部署** | 无需独立服务器，直接嵌入应用 | 本地运行，数据隐私安全 |
| **语义检索** | 基于向量相似度搜索 | 智能召回相关记忆，而非关键词匹配 |
| **多模态支持** | 文本、图像、向量统一存储 | 未来可扩展到图像记忆 |
| **高性能** | 支持数十亿向量检索 | 长期使用无性能瓶颈 |
| **零运维** | 无需数据库管理 | 开箱即用 |

#### memory-lancedb-pro 插件

OpenClaw 官方推荐使用 `memory-lancedb-pro` 插件实现智能记忆管理：

**核心功能**：

```
┌─────────────────────────────────────────────────────┐
│              memory-lancedb-pro 架构                 │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌─────────────┐    ┌─────────────┐               │
│  │ Auto-Recall │    │Auto-Capture │               │
│  │ 自动召回    │    │ 自动存储    │               │
│  │ (每次响应前)│    │ (每次对话后)│               │
│  └──────┬──────┘    └──────┬──────┘               │
│         │                  │                       │
│         ▼                  ▼                       │
│  ┌─────────────────────────────────────┐          │
│  │           LanceDB 向量数据库         │          │
│  │  ┌─────────┬─────────┬─────────┐   │          │
│  │  │ 记忆向量 │ 元数据  │ 时间戳  │   │          │
│  │  └─────────┴─────────┴─────────┘   │          │
│  │  语义相似度检索 | 关键词检索         │          │
│  └─────────────────────────────────────┘          │
│                                                     │
│  Agent 工具：                                       │
│  • memory_store(text, category?, importance?)      │
│  • memory_recall(query, limit?)                    │
│  • memory_forget(memoryId?)                        │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**安装配置**：

```bash
# 方式一：通过 ClawHub 安装
clawhub install memory-lancedb-pro

# 方式二：手动安装
cd ~/.openclaw/workspace/plugins
git clone https://github.com/win4r/memory-lancedb-pro.git

# 安装向量模型（使用 Ollama）
ollama pull nomic-embed-text
```

**配置示例**：

```json5
// ~/.openclaw/openclaw.json
{
  "plugins": {
    "slots": {
      "memory": "memory-lancedb-pro"
    }
  },
  "memory": {
    "lancedb": {
      "db_path": "~/.openclaw/memory_db",
      "embedding_model": "nomic-embed-text",
      "auto_recall": true,        // 自动召回相关记忆
      "auto_capture": true,       // 自动存储重要对话
      "recall_limit": 5,          // 每次最多召回5条记忆
      "similarity_threshold": 0.7 // 相似度阈值
    }
  }
}
```

#### LanceDB 记忆管理最佳实践

**实践一：自动召回与手动召回结合**

```markdown
# 在 AGENTS.md 中配置

## 记忆召回策略
1. 自动召回：每次响应前自动检索相关记忆（由插件完成）
2. 手动召回：当需要特定历史信息时，使用 memory_recall 工具
3. 召回优先级： 
   - 最近7天的记忆优先
   - 高重要性（importance > 0.8）的记忆优先
   - 相似度 > 0.7 的记忆才纳入上下文
```

**实践二：记忆分类与重要性标记**

```python
# 记忆存储时自动分类
memory_store(
    text="用户偏好使用 TypeScript 进行后端开发",
    category="user_preference",    # 分类：用户偏好
    importance=0.9                  # 重要性：高
)

# 记忆分类建议
# - user_preference: 用户偏好（高重要性）
# - decision: 重要决策（高重要性）
# - lesson_learned: 经验教训（中重要性）
# - daily_log: 日常记录（低重要性）
# - temporary: 临时信息（低重要性，定期清理）
```

**实践三：定期记忆清理**

```markdown
# 写入 HEARTBEAT.md

## 记忆库维护（每周日凌晨3:00）
1. 查询 importance < 0.3 且超过30天的记忆
2. 使用 memory_forget 删除低价值记忆
3. 统计记忆库大小，超过100MB时发出警告
```

#### LanceDB vs 传统文件记忆对比

| 维度 | 传统文件记忆 | LanceDB 向量记忆 |
| :--- | :--- | :--- |
| **检索方式** | 文件名/关键词匹配 | 语义相似度检索 |
| **检索精度** | 低（容易漏掉相关记忆） | 高（智能召回相关内容） |
| **Token消耗** | 高（需加载大量文件） | 低（只加载相关记忆） |
| **记忆容量** | 受Context限制 | 理论无限 |
| **隐私安全** | 本地文件 | 本地数据库 |
| **维护成本** | 需手动精炼 | 自动化管理 |

#### 实战效果参考

根据社区反馈，使用 LanceDB 后：

- **记忆检索准确率提升 159%**：语义检索比关键词匹配更智能
- **Token消耗降低约 61%**：只加载相关记忆，不再全量加载
- **记忆丢失问题基本解决**：自动存储+智能召回，不再遗漏重要信息
- **长期使用成本可控**：记忆库可无限增长，不影响检索性能

### 6.7 记忆管理自检清单

- [ ] AGENTS.md 是否定义了晨间仪式？
- [ ] MEMORY.md 是否仅在主会话加载？
- [ ] 是否有每周记忆精炼机制？
- [ ] Daily Notes 是否超过7天未清理？
- [ ] HEARTBEAT.md 是否包含记忆审计任务？
- [ ] 群聊场景是否禁止加载私人记忆？
- [ ] TOOLS.md 中的工具是否超过10个（影响Context）？
- [ ] 记忆文件中是否存在矛盾或过时条目？

---

## 7. 实战进化路径

### 🐣 阶段一：入门级（生存保底）

**目标**: 认人/认路/守规

**必做事项**:
1. 配置 `openclaw.json`（模型Provider + API Key）
2. 填写 `USER.md`（基本信息 + 沟通偏好 + 禁忌）
3. 保持 `AGENTS.md` 默认配置（晨间仪式 + 记忆规则）
4. 安装3个基础Skills: `tavily-search` + `obsidian` + `memory-setup`

**验收标准**: AI不会"发疯"，能正确称呼你，不犯基本禁忌

### 🐥 阶段二：进阶级（职业化）

**目标**: 强化手眼/仪态

**必做事项**:
1. 定制 `IDENTITY.md`（名字、调性、核心Emoji）
2. 配置 `TOOLS.md`（SSH别名、设备名、环境特定参数）
3. 安装场景Skills: `github` + `gmail` + `google-calendar`
4. 开始使用 `memory/YYYY-MM-DD.md` 记录每日日志

**验收标准**: AI有专业语调，能主动使用工具完成任务

### 🦅 阶段三：高阶级（数字合伙人）

**目标**: 注入意志/主动

**必做事项**:
1. 深度配置 `SOUL.md`（Core Truths + Boundaries + 蓝军思维）
2. 编写 `HEARTBEAT.md`（3-5条具体巡检任务）
3. 建立 `MEMORY.md` 精炼机制（每周review）
4. 安装进阶Skills: `web-browser` + `code-interpreter`

**验收标准**: AI有主见，会质疑不合理指令，能主动巡检交付

---

## 8. 避坑准则与常见问题

### 8.1 全局避坑准则

| 准则 | 说明 |
| :--- | :--- |
| **工具≤10** | 工具过多消耗Context、增加Token、导致选择困难 |
| **心跳≤5条** | 心跳任务过多导致AI幻觉和响应超时 |
| **间隔15-30min** | 心跳太快Token账单爆炸，太慢失去主动性 |
| **HEARTBEAT_OK** | 无实质进展时必须沉默，禁止自言自语 |
| **trash > rm** | 永远用回收站替代删除，防止不可逆操作 |
| **隐私优先** | 群聊不加载MEMORY.md，外部操作需确认 |

### 8.2 常见问题

**Q: AI每次都像第一次见面，怎么办？**
A: 检查AGENTS.md的晨间仪式是否被执行。确保AI在每次会话开始时读取SOUL.md、USER.md和记忆文件。

**Q: AI调用工具总是失败？**
A: 检查TOOLS.md是否提供了正确的别名和参数。AI依赖显性命名，如`Camera: front-door`而非IP地址。

**Q: AI在群聊中太吵？**
A: 在AGENTS.md中明确："Participate, don't dominate. Stay silent when it's casual banter."

**Q: Token账单太高？**
A: 1) 减少工具数量 2) 缩短HEARTBEAT任务 3) 使用ClawRouter按任务选模型 4) 定期精炼MEMORY.md

**Q: AI记忆出现错误信息？**
A: 在HEARTBEAT.md中加入记忆审计任务，每周检查MEMORY.md中的矛盾和过时条目。

**Q: 如何让AI有主见而非唯唯诺诺？**
A: 在SOUL.md的Core Truths中加入："Have opinions. You're allowed to disagree." 和蓝军思维要求。

### 8.3 配置优先级速查

当你不确定某个配置该写在哪里时：

| 你想解决的问题 | 应该写在 | 不应该写在 |
| :--- | :--- | :--- |
| AI说话不像那个人 | IDENTITY.md | SOUL.md |
| AI做决策没原则 | SOUL.md | IDENTITY.md |
| AI工作流程混乱 | AGENTS.md | TOOLS.md |
| AI不知道老板喜好 | USER.md | SOUL.md |
| AI工具调用失败 | TOOLS.md | AGENTS.md |
| AI不够主动 | HEARTBEAT.md | SOUL.md |

---

## 附录：关键资源链接

| 资源 | 地址 |
| :--- | :--- |
| OpenClaw 官方文档 | https://docs.openclaw.ai |
| ClawHub Skills 市场 | https://clawhub.com |
| GitHub 仓库 | https://github.com/openclaw/openclaw |
| 配置参考文档 | https://docs.clawd.bot/gateway/configuration-reference |
| 模型Provider配置 | https://openclaws.io/zh/docs/concepts/model-providers |
| Agent工作区说明 | https://openclaws.io/zh/docs/concepts/agent-workspace |
| 记忆架构提案 | GitHub Issue #59095 |

---

> **最后提醒**: OpenClaw的精髓不是"调教AI"，而是"让AI了解你"。当你把自己的"使用说明书"递给AI，它的效率会瞬间翻倍。**最好的AI助手，是你的影子。**
