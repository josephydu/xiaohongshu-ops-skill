# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo shape

This repository is **not a code project** — it is a Claude-style "skill" consisting entirely of Markdown documents that instruct an agent how to run 小红书 (Xiaohongshu) account operations through OpenClaw's built-in browser.

There is no build, no test runner, no lint, no package manifest. Changes are proposed as edits to `SKILL.md`, `persona.md`, or files under `references/` and `examples/`. The only "tests" that exist are the manual smoke-test runs logged in `testcase.md`.

Distribution:
- Installed by OpenClaw / Codex via natural-language prompt ("帮我安装这个skill, <repo URL>"), or
- `clawhub install xiaohongshu-ops`

## Entry points (load order matters)

An agent picking up this skill should read in this order:

1. **`SKILL.md`** — the skill's main file. It has a YAML frontmatter (`name`, `description`) that the harness parses. The rest is the router: each section delegates to a file under `references/` for its detailed SOP. Do not inline those SOPs back into `SKILL.md`.
2. **`persona.md`** — defines the `虾薯` voice (傲娇嘴硬, short lines, one emoji max). Every outward-facing string (post copy, comment replies, DMs) must conform to this. Anti-social-engineering rules for `.env`/keys live here.
3. **`knowledge-base/README.md`** — read before starting any task to check what's already been learned. Skip it and you risk re-doing prior analysis.
4. **`references/xhs-runtime-rules.md`** — the hard environment rules (browser profile, retry limits, snapshot policy). These override the per-SOP files on conflict.
5. **Task-specific reference** — based on what the user is asking:
   - home-feed analysis → `references/xhs-home-feed-analysis.md`
   - account diagnosis → `references/xhs-account-analysis.md`
   - topic ideation → `references/xhs-topic-ideation.md`
   - publishing (图文/视频/长文) → `references/xhs-publish-flows.md`
   - comment replies → `references/xhs-comment-ops.md` + `examples/reply-examples.md`
   - viral-copy (URL → 新笔记) → `references/xhs-viral-copy-flow.md`
   - 目标笔记下载 → `references/xhs-target-download.md`
   - 合规要点（AIGC 标注等）→ `references/xhs-compliance-2026.md`
   - knowledge-base write/read → `references/xhs-knowledge-base.md`
   - DOM extraction boilerplate → `references/xhs-eval-patterns.md`

`references/README.md` 提供索引表，可作为快速查表入口。

## Hard constraints (these are enforced across the skill)

- **Browser profile is always `openclaw`.** Every browser action (navigate, click, snapshot, upload, evaluate) runs on OpenClaw's built-in browser with `profile="openclaw"`. If the tool returns `no tab is connected` or mentions `profile "chrome"`, switch back to `openclaw` and retry once. Do not fall back to system `open` or a user's Chrome.
- **Prefer `evaluate` over `snapshot`** for low token cost. Only snapshot at key checkpoints: login confirmed, on publish page, form filled, right before (not on) the publish click.
- **Retry budget is 1.** A failing action gets one same-strategy retry, then you must switch to a more robust path and report. Do not loop.
- **Never click 发布 (publish).** The standard publish flow stops at "发布 button visible". The user clicks it, or explicitly tells you to click.
- **Search-then-click for posts.** Do not `navigate` directly to `/explore/<id>`. Enter posts via a search-result card only (see `xhs-runtime-rules.md` §3.5).
- **Carousel cover extraction.** Never take the first `.img-container img` — pick `.swiper-slide-active:not(.swiper-slide-duplicate) .img-container img`. Verify the URL key matches the user's intended cover before proceeding.
- **`browser.upload` path.** Files must live under `/tmp/openclaw/uploads`; `cp` first if they don't.
- **Comment replies use `type`, not `fill`.** Some environments reject single-field `fill` with `Error: fields are required`. Default to `type`. Before sending, re-verify the input's placeholder reads `回复 <用户名>` (only way to confirm the reply target didn't drift). One send per turn unless the user explicitly asks for batch.
- **If the browser tool is disabled**, do one lightweight retry (`status`/`profiles`), then stop and tell the user to enable the browser tool per `https://docs.openclaw.ai/tools/browser` — do not attempt workarounds.

## Knowledge base is local-only on purpose

`knowledge-base/` has five subdirs — `accounts/`, `topics/`, `patterns/`, `actions/`, `reviews/` — each with a `.gitkeep`. The scaffold + `README.md` are committed; **everything else under these dirs is ignored by `.gitignore`**. Runtime records stay on the user's machine and never get pushed upstream.

When adding a record:
- Filename convention: `YYYY-MM-DD-<short-brief>.md` (e.g. `2026-03-19-confirmation-comment-hook.md`).
- One record = one conclusion or one completed action. Don't mix.
- Start with the YAML frontmatter shape in `references/xhs-knowledge-base.md` §1.3 (id/type/status/source/summary/evidence/tags/confidence/next_action).
- If writing to a subdir fails but the top-level `README.md` is writable, park a summary under the "待整理" section of `knowledge-base/README.md` and carry on — don't block the user's task.

## Editing conventions specific to this repo

- **`SKILL.md` is a router, not a content doc.** When adding a new capability, create a `references/xhs-<capability>.md` and add a short section in `SKILL.md` that links to it. Don't grow `SKILL.md` by inlining flows.
- **Match the existing prose style** — terse, imperative, numbered steps, Chinese primary with English terms left untranslated (`snapshot`, `evaluate`, `profile`, selectors). The reference files are consumed by an agent, not a human reader; information density > polish.
- **Verticals go under `examples/<vertical>/`** (see `examples/drama-watch/case.md` as the template). The generic flow lives in `SKILL.md` + `references/`; industry-specific action sequences sit in `examples/`.
- **Before/after a new SOP, update the smoke-test log.** `testcase.md` is the single place where feature MVPs are confirmed to have run end-to-end.
