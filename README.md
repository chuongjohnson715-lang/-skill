# ai-daily-brief

> 一天一份「AI × Hackathon × 科技金融」日报：四线并行检索 → 十二大板块结构化输出 → 个人行动建议。

一个为 [WorkBuddy](https://www.workbuddy.cn) Agent 平台设计的 Skill，配合 Expert「晨报（ai-finance-intel）」使用，把每日 AI 行业、Hackathon 赛事、A股/美股科技行情、产业链传导与 GitHub 开源趋势整理成一份可直接阅读的结构化情报日报。

## 特性

- **四线检索模板固化**：12 条固定查询（中国AI/全球AI/A股/美股/产业链/Hackathon/融资/政策/日历/GitHub×3）+ 3 条重大事件专项，日期拼接即用
- **交易日口径规则**：盘中/收盘/跨节假日的统计范围与行情口径判定表，避免"用错收盘日"这类硬伤
- **来源分级铁律**：S/A/B/C 四级 + 「待证」标注规则，C 级来源不得写成确定事实
- **GitHub 板块**：09「GitHub推荐项目」+ 10「GitHub新趋势」，热度数据只引官方页/API，解读强制与事实分离
- **个人价值落地**：板块 11 对齐"想进入 AI 行业的大学生"画像（可自定义），落到今天该学什么/关注什么/参加什么

## 日报结构（十二大板块）

| # | 板块 | 内容 |
| --- | --- | --- |
| 01 | 今日最重要的5件事 | 事件/为什么重要/影响/关注等级/来源 |
| 02 | 中国Hackathon雷达 | 赛事表 + TOP 推荐（不凑数） |
| 03 | AI行业｜中国 | 模型/Agent/应用/基础设施/政策/融资 |
| 04 | AI行业｜全球 | 同上，关注中美竞争、AI芯片、Agent、AI Coding |
| 05 | A股市场 | 六大指数 + 风格/强弱方向/驱动逻辑 |
| 06 | 纳斯达克 & 美股科技 | 指数 + AI芯片/云/光通信/存储/软件/大型科技 |
| 07 | AI × 股票产业链 | 1–3 条传导链，事实/推断严格分段 |
| 08 | 未来7天值得盯的事情 | 星级分级日历 |
| 09 | GitHub推荐项目 | 3–5 个，含 Star/定位/上手难度 |
| 10 | GitHub新趋势 | 2–3 条结构性趋势 + 热度信号 |
| 11 | 对我个人最有价值的信息 | 学什么/关注什么/参加什么/了解哪家公司 |
| 12 | 最终结论 | 5 句话 + 信息密度评分 |

## 快速开始

1. 将本仓库放入 WorkBuddy 的 skills 目录（`~/.workbuddy/skills/ai-daily-brief/`），或直接用 `@skill:ai-daily-brief` 触发；
2. 建议同时启用 Expert「晨报（ai-finance-intel）」，日报输出结构遵循其 `references/daily-report-format.md`；
3. 对 Agent 说「整理一份今日的 AI 情报日报」即可。

Skill 内部按五个步骤执行：确定统计范围与交易日 → 四线并行检索 → 来源分级与待证标注 → 按十二板块模板输出 → 交付到 `AI情报日报/` 目录。

## 依赖

- 运行环境：WorkBuddy（或兼容的 Agent 平台，具备 WebSearch / WebFetch 工具）
- 配套 Expert：晨报（ai-finance-intel）——定义输出结构；本 Skill 定义检索与防错
- 无第三方 Python 包依赖

## 目录结构

```
ai-daily-brief/
├── SKILL.md      # Skill 主文件（frontmatter + 五步工作流）
├── README.md
└── LICENSE
```

## License

[MIT](./LICENSE)
