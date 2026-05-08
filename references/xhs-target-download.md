# XHS 目标笔记下载

本文件定义「目标笔记下载」流程：输入一条小红书笔记 URL，输出正文、图片、关键元信息，供后续分析或复刻使用。

## 0. 适用场景

- 下载爆款笔记做复刻前的结构化拆解
- 下载自己/竞品历史笔记做归档
- 下载参考笔记的封面与图片做二次设计输入

## 1. 输入

- `source_url`：小红书笔记 URL（`https://www.xiaohongshu.com/explore/...` 或分享短链）
- `output_dir`：下载产物落地目录（默认 `/tmp/openclaw/downloads/<note_id>/`）

## 2. 下载流程

1. 使用内置浏览器 `profile="openclaw"` 打开笔记 URL
2. 先校验页面不是 404、可见标题和作者
3. 抽取元信息（evaluate 优先）：
   - `title`、`author`、`publish_time`
   - `likes`、`comments`、`collects`
   - `tags`、`body_text`
4. 抽取图片 URL（轮播页强制规则）：
   - 优先 `.swiper-slide-active:not(.swiper-slide-duplicate) .img-container img`
   - 兜底 `.swiper-slide-active .img-container img`
   - 遍历所有 slide 时，记录 `swiper-slide-active` 切换前后的 `currentSrc || src`
5. 图片去重：按 URL 末段 key（如 `1040g3k...`）去重，避免存重复图
6. 下载图片到 `output_dir/images/`，正文与元信息写 `output_dir/meta.json`

## 3. 输出结构

```
<output_dir>/
  meta.json        # title / author / stats / tags / body / image_keys
  images/
    01-<key>.jpg
    02-<key>.jpg
    ...
  body.md          # 正文纯文本（便于检索）
```

## 4. 失败降级

- 页面 404 或内容被删：记录失败原因，不重试，返回用户
- 图片抓取只拿到 duplicate：snapshot 刷新后再试一次，仍失败则只保存已获图
- 长文章正文截断：优先保留开头 + 结尾，标注"内容截断"
- 视频笔记：默认只抓封面和元信息，不下视频本体（除非用户明确要求）

## 5. 使用原则

- 不重复下载：先检查 `output_dir` 是否已有同 `note_id` 记录
- 不做批量爬取：单次任务单条 URL，避免触发风控
- 下载仅用于个人分析/复刻前拆解，不做二次分发
