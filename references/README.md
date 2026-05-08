# References 索引

本目录收纳各项 SOP。agent 根据任务类型加载对应文件；`SKILL.md` 里也有路由指针。

## 快速查表

| 任务 | SOP 文件 |
|---|---|
| 运行环境与浏览器约束 | `xhs-runtime-rules.md` |
| 首页推荐流分析 | `xhs-home-feed-analysis.md` |
| 账号诊断 | `xhs-account-analysis.md` |
| 选题灵感 | `xhs-topic-ideation.md` |
| 发布（图文/视频/长文） | `xhs-publish-flows.md` |
| 评论检查与回复 | `xhs-comment-ops.md` |
| 爆款笔记复刻（URL → 新笔记） | `xhs-viral-copy-flow.md` |
| 目标笔记下载（URL → 素材归档） | `xhs-target-download.md` |
| 知识库沉淀与检索 | `xhs-knowledge-base.md` |
| 合规要点（AIGC 标注等） | `xhs-compliance-2026.md` |
| DOM 通用提取脚本 | `xhs-eval-patterns.md` |

## 加载优先级

1. 任何任务前：先过 `xhs-runtime-rules.md`（硬约束）和 `xhs-compliance-2026.md`（合规）
2. 再按任务类型加载对应 SOP
3. 涉及落库时追加 `xhs-knowledge-base.md`
