---
name: ai-daily-brief
version: "1.1.0"
description: 生成「AI × Hackathon × 科技金融」日报。用户要求整理今日/昨日的 AI 情报日报、AI 行业+竞赛+行情综合简报，或使用 @skill:ai-daily-brief 时使用。固化四线检索查询模板、高价值数据源、交易日与节假日口径规则、来源分级与待证标注要求；含「GitHub推荐项目」「GitHub新趋势」两个板块。配合 Expert「晨报（ai-finance-intel）」使用，输出结构遵循该 Expert 的 references/daily-report-format.md。
license: MIT
agent_created: true
metadata:
  author: WorkBuddy
display_name: "AI日报"
display_name_en: "AI Daily Brief"
description_zh: "一天一份 AI × Hackathon × 科技金融日报：四线检索 → 十二大板块（含 GitHub 推荐项目与 GitHub 新趋势）→ 个人行动建议。"
description_en: "Daily AI x Hackathon x tech-finance briefing with a fixed four-track search playbook."
visibility: "public"
---

# AI × Hackathon × 科技金融日报

为 Expert「晨报（ai-finance-intel）」配套的可执行工作流。Expert 定义**输出结构**，本 Skill 定义**怎么把料找齐、怎么不出错**。

## 何时使用

- 用户说「整理一份今日的 AI 情报日报」「AI×Hackathon×科技金融日报」「晨报」
- 用户用 `@skill:ai-daily-brief` 触发
- 已选择 Expert「晨报 / ai-finance-intel」

## 第 0 步：先确定「统计范围」与「交易日」

**这是最容易出错的一步，务必先做。**

| 场景 | 统计范围 | 行情口径 |
| --- | --- | --- |
| 周二~周五晨报（早晨提问） | 前一天 00:00–23:59 | 前一天收盘（A股 + 美东同日） |
| **当天 11:30—15:00 提问（A股盘中/午间，未收盘）** | **上一交易日 + 当日** | **A 股以「上一交易日收盘」为主口径，另附「当日午间收盘 + 提问时刻实时快照」作对照**，并**显式标注"当日尚未收盘"**；美股仍用美东前一交易日收盘 |
| **当天下午/傍晚提问（A股已收盘，15:00 后）** | **当日 + 前一日** | **A 股用当日收盘，并附前一日作对照**；美股仍用美东前一交易日收盘（北京时间当日凌晨） |
| **周一晨报** | **周六 + 周日**（分列 9/19 周六、9/20 周日） | **上一交易日**：A股用上周五收盘；美股用美东上周五收盘（北京时间周六凌晨） |
| 节假日后首日 | 假期区间 | 节前最后交易日 |

- ⚠️ **上一交易日 ≠ 上一周的同一天**。节假日会切断交易周，必须**先查休市公告再定上一交易日**。实测：2026 年中秋 9/25（五）—9/27（日）休市，故 9/28（一）提问时上一交易日是 **9/24（周四）**，不是 9/25；且当日为节后首日、当周仅 3 个交易日（9/28—9/30），随后国庆 10/1—10/7 再休市。**这类"节前/节后首日"必须在日报头部写清。**
- ⚠️ **用户只说「今天的」「整理」时，先看系统注入的 current_time**。若已过 A 股收盘时间（15:00），当日行情已生成，**只写前一天会让日报当场过时**——必须把当日 A 股收盘作为主口径，前一日作对照。实测案例：9/24 15:35 提问，当日沪指 -1.22%、创业板 -2.68%，若只写 9/23 的 -0.39% 就完全错了方向。
- **中文媒体对美股晚间行情的表述**：写「北京时间 X月X日凌晨美股收盘」的报道，对应的是**美东前一个交易日**。必须在日报里分别标清两个日期。
- **务必先查 A 股节假日安排**并写进日报头部（例：2026 年中秋 9/25–9/27 休市）。
- Git Bash 下 `date -d "yesterday"` **不可用**（invalid date）。以系统注入的 current_time 为准。

## 第 1 步：四线并行检索（用 WebSearch，同一轮多发）

固定查询模板（把日期拼进去即可，实测有效）：

```
① 中国AI行业：AI 新闻 <YYYY年M月D日> 大模型 发布 智能体          freshness=d3
② 全球AI行业：OpenAI Anthropic Google AI announcement <Month D, YYYY>  freshness=d3
③ A股行情   ：A股 <M月D日> 收盘 上证指数 深证成指 创业板 科创50  freshness=d3
④ 美股行情   ：美股收盘 <M月D日> 纳指 费城半导体 英伟达 芯片股     freshness=d3
⑤ A股产业链 ：<M月D日> A股 半导体 存储芯片 光模块 板块 涨幅        freshness=d5
⑥ Hackathon ：黑客松 大赛 <YYYY年M月> 报名 大学生 AI 奖金          freshness=d7
⑦ 资本融资   ：AI 融资 <YYYY年M月> 亿美元 轮融资 大模型 公司        freshness=d3
⑧ 政策      ：工信部 人工智能+ 政策 <YYYY年M月> 国务院 算力        freshness=d7
⑨ 未来日历   ：<会议名> <YYYY> 时间                          freshness=d5
⑩ GitHub趋势（中文）：GitHub Trending <M月D日> AI 开源项目 大模型 Agent   freshness=d3
⑪ GitHub趋势（英文）：github trending repositories <Month D, YYYY> AI agent LLM  freshness=d3
⑫ GitHub周度精选：HelloGitHub 第<N>期 / trendshift trending <Month YYYY>  freshness=d7
```

GitHub 线补充说明：
- 检索日期一律按第 0 步定好的「统计范围日」拼进查询，不要默认用今天。
- 除 WebSearch 外，可直接抓 GitHub 官方数据（无需登录）：
  - Trending 页：`https://github.com/trending?since=daily`（可加 `&language=python` 等过滤），WebFetch 抓取后提取 仓库名/简介/今日 star/语言。
  - Search API（未认证限流 10 次/分钟）：查「近 7 天新建 + Star 排序」看新项目冲榜，例如
    `https://api.github.com/search/repositories?q=topic:ai+created:><统计范围日减7天>&sort=stars&order=desc&per_page=10`
    命令中的日期用系统 current_time 推算，不要手算。
- Star/Fork/Trending 排名属于 GitHub 官方客观数据（S 级），可直接引用；第三方对项目的评价按 B/C 级处理。

按需补：`沪深300 中证A500 <M月D日> 收盘 涨跌幅`（③ 常查不到这两个，需单独补一次）。

**重大事件专项补查（一旦当天有国家级/国际级事件，必须单独搜一轮）**：
```
⑬ 中美元首/高层会晤：中美 经贸 会谈 <YYYY年M月> 人工智能 稀土 关税 高层   freshness=d3
⑭ 国内大型会议  ：<会议名> <YYYY> 发布 芯片 模型                freshness=d5
⑮ 当日 A 股收评（下午提问时）：A股 <M月D日> 收盘 上证指数 创业板 成交额 <领跌板块>  freshness=d2
```
重大事件尽量交叉验证 2 个以上来源。

## 第 2 步：优先使用的高价值数据源

| 用途 | 来源 | 级别 |
| --- | --- | --- |
| **A股/美股指数与板块精确行情** | 搜索返回的 `basic_info` / `A股行情指标` / `美股行情指标` 结构化行情块（含指数代码、开高低收、涨跌幅、成交额） | 数据接口 |
| 中国AI动态 | AI Native Foundation *China AI Native Industry Insights*（含官方链接）、SegmentFault AI 日报、腾讯研究院 AI 速递、DAMO 开发者矩阵 AI 日报 | B/C |
| **国内赛事雷达** | 扬子晚报/IT之家（赛事启动通稿）、AITOP100 赛事专区、各高校科研院（如南大科研院）通知页、MoonBit 等官方赛事页 | S/B/C（汇总站为 C 级，日期须标「待确认」） |
| 全球AI动态 | aibriefing.dev、Pondero、AIToolsRecap、superpowerdaily（周报） | B |
| **未来事件日历** | `marketfinch.com/ai-calendar-<YYYY-MM-DD>` | B（**2026-09-24 实测已返回 404，链接失效**）；改由 WebSearch 用「<会议名> <YYYY> 时间」+「大赛 报名 截止 <M月>」补齐 |
| A股收盘综述 | 中国证券网、财联社、中新经纬、中国新闻网、金融时报、格隆汇（经新浪/钛媒体转载） | A/B |
| 美股收盘综述 | 中新经纬、财联社、21世纪经济报道、格隆汇、第一财经 | A/B |
| 政策文件 | 国务院/工信部/发改委/国家药监局/国家医保局官网 | S |
| **GitHub 热度雷达** | GitHub Trending 官方页（github.com/trending，WebFetch 可抓）、GitHub Search API（api.github.com，无需认证）、Trendshift（trendshift.io）、HelloGitHub（hellogithub.com）、OSS Insight（ossinsight.io） | GitHub 官方页/API 为 S 级（热度与仓库描述）；Trendshift/HelloGitHub/OSS Insight 为 B/C |

**陷阱**：A 股板块搜索常把「板块资金流向」和「指数行情」混在一起返回，且偶有量纲异常字段（如「增加17915967493万元」），**不可直接引用**；`fund_rank_info` 返回的「主力净流入-22467390891元」与「增加-25639125856万元」量纲自相矛盾，**只可看方向、不可引用数字**。

## 第 3 步：来源分级与「待证」标注（铁律）

- S 级：官方公告/公司官网/政府官网/赛事官网/官方博客/官方 GitHub
- A 级：Reuters / AP / Bloomberg / FT / WSJ / CNBC / 中国证券报 / 上海证券报 / 证券时报 / 财联社
- B 级：主流科技财经媒体
- C 级：自媒体 / 论坛 / 聚合站 / 社交媒体

规则：
1. **仅 C 级来源的信息，绝不可写成确定事实**，必须标注「仅见 C 级来源，待证」。
2. **当日无一手信源的异动（如某股大跌）**，写「原因未获可靠信源确认」，**不要编造原因**。
3. **多源口径冲突**（实测案例：费城半导体指数同日出现 +4.29% / +5.29% 两种记法）→ 采用多数源，并在正文标注差异。
4. **旧闻必须标时效**：如「融资 9/13 官宣、沟通会 9/16、模型 9/20 上线」要拆开写，不能混成一条当日新闻。
5. 同一事件若前一日已报，本期写「延续」并只补新增事实。
6. **GitHub 热度数据（Trending 排名、Star/Fork 数、仓库描述）以 GitHub 官方页为 S 级**，可直接引用；第三方（Trendshift、HelloGitHub、自媒体）对项目的评价、榜单解读按 B/C 级处理，涉及主观判断必须标注。

## 第 4 步：按 Expert 模板输出

严格遵循 Expert「晨报（ai-finance-intel）」的 `references/daily-report-format.md`（路径随 Expert 插件安装位置而定）的十二大板块（01–12，其中 09「GitHub推荐项目」与 10「GitHub新趋势」为 GitHub 新增板块）。补充约定：

- 头部必须写：统计范围、行情口径、**节假日/休市提醒**、一句话判断。
- 板块 05 指数行使用固定格式（上证/深证/创业板/科创50/沪深300/中证A500）。
- 板块 06 若 Nasdaq 100 无可靠口径，**写「未取得，不列示」而不是猜**。
- 板块 07 每条链条用「【事实】/【推断】/【不能确定的】」三段式。
- 板块 09「GitHub推荐项目」：选 3–5 个，字段为 项目名 / 链接 / 语言 / Star 数 / 一句话定位 / 为什么值得关注 / 上手难度；优先与当日 AI 新闻、Hackathon 可复用工具链、用户学习路径相关；Star 与描述只引 GitHub 官方数据；没有合适项目时写「今日无值得推荐的新项目」，不凑数。
- 板块 10「GitHub新趋势」：给 2–3 条当日/当周 GitHub 上的结构性趋势（语言、框架、协议、工具形态的升温），每条用「趋势 / 代表项目 / 热度信号（Star、Fork、增速） / 解读（标【推断】） / 对我的意义」；热度数字只引官方数据，解读必须与事实分开。
- 板块 11 必须对齐目标用户画像（默认画像：想进入 AI 行业的中国大学生、目标 AI 应用·产品·运营·FDE 方向、通过 Hackathon 积累作品；使用前可按实际使用者修改本节）。
- 结尾固定给：信息密度评分 + 最值得花 10 分钟的主题。
- **免责声明必须有**：不构成投资建议、不推荐买入任何股票。

## 第 5 步：交付

- 输出到工作区 `AI情报日报/<YYYY-MM-DD>-AI×Hackathon×科技金融日报.md`
- 必须调用 present_files 呈现文件
- 文本回复只给「一句话判断 + 3–5 条最重要的事 + 需要用户注意的口径提示」，不重复全文
- 交付极简原则：只交付成品文件，不附加说明性文档（可在使用前按需开关）
