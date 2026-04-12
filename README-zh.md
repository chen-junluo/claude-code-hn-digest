# hn-digest

[English README](README.md)

这是一个 Claude Code skill。它会先抓取 Hacker News 首页并返回完整编号列表，再让用户按条目选择深入模式，最后把带日期的 markdown digest 保存到本地。

## 这次更新了什么

当前版本加入了两阶段工作流：

- 第一步先抓取 Hacker News 首页，并返回所有可见条目的完整编号列表
- 每个条目都包含 HN 信号、一句话摘要、一句为什么值得关注，以及 2 到 3 个建议深入方向
- 第二步只在用户选定条目编号并指定模式后执行，模式可以是 `academic`、`industry` 或 `both`
- 同一次运行里，不同条目可以用不同模式
- 针对性提取现在围绕六类焦点展开，而且“观点整合”始终排在第一位
- 最终 markdown 归档采用固定章节顺序，并标明每个选定条目是否真的抓取了评论线程

## 它会做什么

- 抓取当前 Hacker News 首页
- 提取首页可见信息，例如排名、标题、链接、分数、评论数、提交者和来源域名
- 在任何深入分析前，先在聊天中返回完整编号话题列表
- 根据每条内容，给出来自六类焦点的建议深入方向
- 只对用户选中的讨论线程做一次批量抓取
- 用英文或中文写一份适合非技术学术读者的 digest
- 把结果保存到 `~/Downloads/hn-digest/`，文件名带时间戳

## 模式与焦点类型

第二步支持三种模式：

- `academic`：观点整合、事实与原话、研究切口、商业含义
- `industry`：观点整合、技术选型与实现、可以自己做什么
- `both`：六类焦点全部包含

六类焦点分别是：

1. 事实与原话
2. 观点整合
3. 研究切口
4. 商业含义
5. 技术选型与实现
6. 可以自己做什么

## 适合谁用

这个 skill 适合想快速了解 Hacker News 首页，但不想自己逐条阅读的人。它尤其适合商学院博士生、研究者和教师，因为它会尽量把技术新闻和市场、组织、劳动、监管、战略这些问题联系起来。

## 文件说明

- `skill.md`：Claude Code skill 定义
- `README.md`：英文说明
- `README-zh.md`：中文说明
- `LICENSE`：MIT 许可证
- `.gitignore`：忽略本地编辑器和系统杂项文件

## 如何使用

1. 把这个 skill 放到 Claude Code 的 skills 目录中。
2. 用类似下面的方式调用：
   - `/hn-digest`
   - `Summarize Hacker News and save an archive.`
   - `Give me a business-school-friendly summary of today's Hacker News.`
3. 如果你没有指定语言，这个 skill 会先问你要中文还是英文。
4. 第一步会先返回完整编号话题列表。
5. 然后你用“条目编号 + 模式”回复，例如 `4 industry; 18 both`。
6. 第二步会抓取选中的线程，写出最终 digest，并把 markdown 归档保存到本地。

使用前，如果你当前网络无法访问 Hacker News，请先打开 VPN。

## 输出位置

归档文件会保存到：

`~/Downloads/hn-digest/`

文件名格式是：

`hacker-news-digest-YYYY-MM-DD-HHMM.md`

保存后的 digest 固定使用以下章节顺序：

1. Title
2. Source
3. Fetched at
4. Part 1: Summary
5. Part 2: Targeted extraction
6. Caveats

## 说明

- 这个 skill 用于快速日常摘要和归档。
- 它不是完整的新闻爬虫，也不是历史数据库。
- Hacker News 的分数和评论数只是注意力信号，不代表内容一定正确，也不代表一定重要。
