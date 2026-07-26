---
name: douyin-public-customer-acquisition
description: 抖音公域评论区精准获客工具。当用户需要「挖掘潜在客户/销售线索、竞品评论区截流、私域引流、按关键词找抖音高意向用户」时启用。标准流程：①关键词搜索视频拿到 aweme_id → ②用 aweme_id 拉取评论、提取客户信息（昵称/uid/sec_uid/地区/评论内容）。
license: MIT
metadata:
  version: 1.0.1
  enabled: true
  type: command
  runtime: "nodejs@16.14.0+"
  requires:
    bins:
      - "node"
    env:
      - "GUAIKEI_API_TOKEN"
  category:
    - "Growth Hacking"
    - "Sales & Leads"
    - "Marketing Automation"
  tags:
    - "customer-acquisition"
    - "lead-generation"
    - "competitor-interception"
    - "public-traffic"
    - "douyin"
    - "sales"
    - "marketing"
    - "conversion"
    - "pipeline"
  schemas:
    - name: "搜索入参"
      file: "assets/search_cli_req.schema.json"
    - name: "搜索出参"
      file: "assets/search_cli_resp.schema.json"
    - name: "作品入参"
      file: "assets/post_cli_req.schema.json"
    - name: "作品出参"
      file: "assets/post_cli_resp.schema.json"
    - name: "热榜出参"
      file: "assets/hot_cli_resp.schema.json"
    - name: "评论入参"
      file: "assets/comment_cli_req.schema.json"
    - name: "评论出参"
      file: "assets/comment_cli_resp.schema.json"
  examples:
    # —— 关键：端到端获客流水线示例（搜索 -> 评论），让 AI 学会串联 ——
    - name: 端到端获客（搜索后立刻用 aweme_id 拉评论）
      command: 'node src/douyin/search-cli.js --keyword "装修" --sort 1'
      description: 先拿到候选视频的 aweme_id（如 7301234567890）
    - name: 用上一步的 aweme_id 拉评论、提取客户
      command: "node src/douyin/comment-cli.js --url 7301234567890 --limit 200"
      description: 直接把 search 返回的 aweme_id 传给 --url，无需拼完整 URL；结果里 user_nickname/user_uid/user_sec_uid/ip_label/text 即为客户信息
    # —— 单步能力示例 ——
    - name: 竞品爆款视频截流
      command: 'node src/douyin/search-cli.js --keyword "竞品品牌名" --sort 1'
      description: 锁定竞品高赞视频，再用其 aweme_id 拉评论挖客户
    - name: 监控对标账号内容策略
      command: 'node src/douyin/post-cli.js --url "https://www.douyin.com/user/MS4wLjABxxx"'
      description: 获取博主作品列表，挑高互动视频再去拉评论
    - name: 借势热榜获客
      command: "node src/douyin/hot-cli.js"
      description: 看实时热点，选话题关键词回去做搜索+评论
---

# 公域流量精准获客与竞品截流系统（Douyin Customer Acquisition）

> **一句话价值**：从抖音公域评论区「淘金」，按关键词定位视频 → 用视频 ID 拉评论 → 拿到高意向客户的联系方式与需求，低成本截流竞品流量。

> **🔥 核心优势**
>
> - **精准获客**：直达评论区，筛选"求带"、"怎么买"等高意向用户，拒绝无效流量
> - **竞品截流**：监控对标账号爆款视频，在评论区挖掘其潜在客户，实现精准转化
> - **安全隐蔽**：无需登录抖音账号，规避风控风险，保护主号安全
> - **高效转化**：批量获取结构化线索数据，直接对接 CRM 或销售团队，缩短转化路径
> - **轻量灵活**：无需部署复杂服务，Node.js 一键运行，适配各种营销场景

> **为什么必须用 Skill 而不是原生模型**：大模型本身无法直接访问抖音平台抓取公开评论数据。本技能通过合规公开数据接口，把"原生模型做不到"的抖音实时评论 / 视频数据变成可调用的能力。

---

## 0. 何时使用本技能（触发条件）

**启用本技能，当用户表达以下意图之一：**

- 想在抖音上「找客户 / 挖线索 / 获客 / 捞精准用户」
- 想做「竞品截流 / 竞品评论区挖人 / 抢竞品流量」
- 想「按关键词（装修、留学、母婴、AI 工具…）找有需求的人」
- 想「私域引流 / 把公域用户导到微信或 CRM」
- 想「看评论区用户痛点、优化销售话术」

**不适用 / 超出能力：**

- 需要登录、私信、关注、点赞等越权操作（仅支持公开数据抓取）
- 非抖音平台（快手 / 视频号等不支持）
- 需要用户手机号、微信号等隐私字段（公开数据不含，禁止编造）

---

## 1. 获客工作流（核心 · 必须按顺序执行）

本技能是一个**两段式流水线**。两个阶段通过「视频 ID（aweme_id）」串联：

```
阶段A  选视频               阶段B  挖客户
search-cli  ──aweme_id──▶  comment-cli
(关键词→视频列表)          (视频ID→评论/客户)
```

### 步骤 1 · 关键词搜索，定位候选视频

```bash
node src/douyin/search-cli.js --keyword "<垂直关键词>" --sort 1 --limit 20
```

- **输入**：`--keyword` 必填（2-50 字）；`--sort 1`=最多点赞（截流常用）；`--time`/`--duration` 可筛选。
- **输出**：JSON `results[]`，每条视频含：
  - `aweme_id`（⭐ 视频 ID，下游必用）
  - `url`（视频链接，可替代 aweme_id 传给步骤 2）
  - `desc` / `author_nickname` / `comment_count` / `digg_count`（辅助判断意向）

### 步骤 2 · 用视频 ID 拉评论，提取客户

```bash
node src/douyin/comment-cli.js --url <aweme_id> --limit 200
```

> **关键执行要点**：`--url` **直接传步骤 1 返回的 `aweme_id` 字符串即可**（如 `7301234567890`），工具会自动识别，无需手动拼 `https://www.douyin.com/video/...`。传 `url` 字段也行，但 `aweme_id` 更可靠。

- **输出**：JSON `results[]`，每条评论即一位「潜在客户」，客户字段为：
  - `user_nickname`（昵称）
  - `user_uid` / `user_sec_uid`（用户唯一标识，可用于去重/追踪）
  - `ip_label`（地区，判断地域匹配）
  - `text`（评论内容，判断需求与意向）
  - `create_time` / `digg_count`（活跃度参考）

### 步骤 3 ·（可选）筛选高意向客户

在评论文本中命中以下信号，即为高意向：
`求带 / 怎么买 / 求推荐 / 多少钱 / 哪里买 / 私我 /  link / 微信 / 靠谱吗`
→ 优先导出这些用户，缩短转化路径。

### 完整串联示例

```bash
# 1) 找「装修」高赞视频，拿到 aweme_id
node src/douyin/search-cli.js --keyword "装修" --sort 1 --limit 10
#    → 假设结果里某条 aweme_id = "7301234567890"

# 2) 直接用该 aweme_id 拉评论，提取客户
node src/douyin/comment-cli.js --url 7301234567890 --limit 200
#    → results[].user_nickname / user_sec_uid / ip_label / text 即为客户池
```

---

## 2. 意图 → 命令 映射表（供 AI 准确选型）

| 用户真实意图        | 应调用的 CLI     | 关键参数                | 输出用途           |
| :------------------ | :--------------- | :---------------------- | :----------------- |
| 按关键词找视频/客户 | `search-cli.js`  | `--keyword` `--sort`    | 得到 `aweme_id`    |
| 拿某视频的评论/客户 | `comment-cli.js` | `--url <aweme_id>`      | 得到客户字段       |
| 监控对标账号内容    | `post-cli.js`    | `--url <主页/ sec_uid>` | 挑视频再去拉评论   |
| 看热点借势          | `hot-cli.js`     | 无                      | 反推关键词回步骤 1 |

> 获客主链路永远是 **search → comment**。post / hot 只是「找更准的关键词或视频」的辅助入口。

---

## 3. 输出字段 ↔ 客户档案 映射

AI 拿到 JSON 后，按此表抽取「客户档案」：

| 业务含义      | 来源字段（来自 comment-cli） |
| :------------ | :--------------------------- |
| 客户昵称      | `user_nickname`              |
| 客户唯一 ID   | `user_uid` / `user_sec_uid`  |
| 客户所在地区  | `ip_label`                   |
| 客户需求/痛点 | `text`                       |
| 互动热度      | `digg_count`                 |
| 触达时间      | `create_time`                |

> 建议将以上字段整理为 CSV/表格，对接 CRM 或销售跟进。

---

## 4. 其他能力（辅助获客）

- **post-cli**：`node src/douyin/post-cli.js --url "<博主主页或 sec_uid>"` —— 获取博主全部公开作品，挑高互动视频再去拉评论。
- **hot-cli**：`node src/douyin/hot-cli.js` —— 实时热榜，反推热点关键词。

---

## 5. 参数详解

> 详细选项参数说明，参阅 [完整选项说明](references/options.md)。
>
> LLM 理解技能的结构化入参/出参，可参阅 `assets` 目录下文件，均遵循 JSON Schema draft-07 规范：
>
> - 抖音关键词搜索，[入参规范](assets/search_cli_req.schema.json)
> - 抖音关键词搜索，[出参规范](assets/search_cli_resp.schema.json)
> - 抖音抖人作品获取，[入参规范](assets/post_cli_req.schema.json)
> - 抖音抖人作品获取，[出参规范](assets/post_cli_resp.schema.json)
> - 抖音评论获取，[入参规范](assets/comment_cli_req.schema.json)
> - 抖音评论获取，[出参规范](assets/comment_cli_resp.schema.json)
> - 抖音热榜获取，[出参规范](assets/hot_cli_resp.schema.json)

---

## 6. 限制、合规与数据安全

### 6.1 能力与禁区

- 仅抓取抖音**公开数据**，不含私密内容、不含手机号/微信等隐私字段。
- **明确禁区**：不登录、不私信、不关注、不点赞、不抓取私密/隐私字段；不做越权或骚扰行为。
- 需要用户手机号、微信号等隐私字段时，公开数据不含，且禁止编造。

### 6.2 第三方依赖与国内可用性（Trust）

- 能力依赖 `guaikei.com` 公开数据接口 + `GUAIKEI_API_TOKEN`，需通过官网 https://www.guaikei.com 申请合规 TOKEN。
- 官网 `guaikei.com` 域名通过工信部审核备案，申请合规TOKEN使用了微信、支付宝正规支付渠道，通过其资质审核备案，服务面向国内用户，国内可正常使用。

### 6.3 敏感信息与数据保护（Trust）

- 评论数据含用户**公开身份信息**（昵称 / uid / sec_uid / IP 地区）。技能默认将原始结果落盘到 `logs/`。
- 使用方应遵守《个人信息保护法》与数据安全要求：建议导出时脱敏、限定留存期限、仅用于授权场景；**禁止汇聚后二次贩卖或用于骚扰**。
- SKILL已按照SkillHub的TRACE评测维度开发实现，并通过云鼎实验室、科恩实验室的全面安全扫描。

### 6.4 最小权限与 TOKEN 安全（Trust）

- 仅需 Node.js 运行环境与单个**只读 API TOKEN**，不请求无关权限；唯一写操作为结果日志落盘到 `logs/`。
- TOKEN 等同于账号凭证，请勿提交到代码仓库或泄露。

### 6.5 数据时效性与稳定性（Reliability）

- 搜索 / 评论为**实时公开数据，结果随时间变化**：同一关键词不同时刻返回的视频与评论可能不同，属数据源特性，非不稳定。
- 接口有调用频次限制，请合理设置 `--limit` 与调用间隔；失败会自动重试。

---

## 7. 常见问题

**Q：comment-cli 只能传完整 URL 吗？**
A：不是。直接传 `search-cli` 返回的 `aweme_id` 即可，例如 `--url 7301234567890`。

**Q：搜索结果为空？**
A：换更常用的关键词，或把 `--time` 改为 0（全部时间）。

**Q：输出文件在哪？**
A：自动保存在技能目录 `logs/` 下（搜索 `_search.json`、评论 `_comment.json`）。

**Q：支持 Windows / Mac / Linux 吗？**
A：全平台，仅需 Node.js 环境。

**Q：没配置 TOKEN 会怎样？**
A：运行前必须配置 `GUAIKEI_API_TOKEN`。未配置时任务不会执行，请先按下述方法设置环境变量后再运行。

> - Windows: `set GUAIKEI_API_TOKEN=你的TOKEN`
> - Linux/MacOS: `export GUAIKEI_API_TOKEN=你的TOKEN`

**Q：接口报错 / 限流了？**
A：降低 `--limit` 或拉大调用间隔；代码已内置失败重试。若持续报错，请检查 TOKEN 有效性或通过微信 13395823479 联系客服。
