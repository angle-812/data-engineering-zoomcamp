# 开始这里

DataTalks.Club 的 **Data Engineering Zoomcamp** 完整离线副本（2027 期）。

## 打开顺序

1. **`index.html`** —— 主入口。88 个单元、视频直链、进度勾选都在这里
2. **`course/README.html`** —— 官方课程总览
3. **`notes/`** —— 七份笔记模板，边看边填

> 直接双击 `index.html` 就行，不需要装任何东西、不需要联网。
> 用 `file://` 打开时，每个单元右边的「打开」是本地讲义，「GH」是 GitHub 原页。
> 上传到你自己的 GitHub 之后，用网址打开时链接会自动切到 GitHub。

## 目录说明

| 路径 | 内容 |
|---|---|
| `index.html` | 学习看板，进度存在本机浏览器 |
| `course/` | 完整课程仓库（46 MB，952 个文件）：讲义 Markdown + 渲染好的 HTML、图片、Notebook、SQL、docker-compose、Terraform 配置 |
| `course/cohorts/2027/homework/` | 七个模块的作业原题 |
| `notes/` | 我的笔记（`.md` 编辑，`.html` 阅读） |
| `ACCELERATORS.html` | 连不上 GitHub 时怎么办，含实测速度对比 |
| `GITHUB.html` | 把它推到你自己 GitHub 的完整步骤 |
| `push-to-github.bat` | 双击自动推送脚本（会弹一次浏览器授权） |

## 学习顺序

| 周 | 模块 | 关键词 |
|---|---|---|
| 1 | Containerization and Infrastructure as Code | Docker、PostgreSQL、Terraform、GCP |
| 2 | Workflow Orchestration | Kestra、调度、回填、ETL vs ELT |
| 3 | Data Warehousing | BigQuery、分区、聚类、BQML |
| 4 | Analytics Engineering | dbt、模型、测试、文档 |
| 5 | Data Platforms | Bruin、端到端管道、部署 |
| 6 | Batch Processing | Spark、PySpark、GroupBy / Join 原理 |
| 7 | Streaming | Kafka、Redpanda、Flink、窗口 |
| 8–10 | Final Project | 串起全链路 + 互评 |

## 视频在哪

每个单元右侧的 **▶ 视频** 是 YouTube 直链。模块 1 和模块 7 有些单元没单独挂视频，看板顶上「视频列表」进官方播放列表按顺序找。

## 更新

这个副本是 2026-11-11 抓取的。官方更新后，重新下载解压覆盖 `course/` 即可（下载方法见 `ACCELERATORS.html`）。
