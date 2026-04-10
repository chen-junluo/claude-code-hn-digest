# hn-digest

[English README](README.md)

这是一个 Claude Code skill。它会抓取 Hacker News 首页，判断大家最关注什么，用面向商学院博士生的浅白语言解释这些内容，并把结果保存为带日期的本地 markdown 归档。

## 它会做什么

- 抓取当前 Hacker News 首页
- 提取首页可见的信息，例如排名、标题、链接、分数、评论数和来源域名
- 判断哪些话题最吸引 Hacker News 用户注意
- 用英文或中文写一份适合非技术学术读者的摘要
- 把结果保存到 `~/Downloads/hn-digest/`，文件名带时间戳

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
4. 然后它会抓取 Hacker News，生成摘要，并把 markdown 文件保存到本地。

## 输出位置

归档文件会保存到：

`~/Downloads/hn-digest/`

文件名格式是：

`hacker-news-digest-YYYY-MM-DD-HHMM.md`

## 说明

- 这个 skill 用于快速日常摘要和归档。
- 它不是完整的新闻爬虫，也不是历史数据库。
- Hacker News 的分数和评论数只是注意力信号，不代表内容一定正确，也不代表一定重要。
