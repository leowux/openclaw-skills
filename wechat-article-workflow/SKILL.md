---
name: wechat-article-workflow
description: Complete workflow for WeChat official account articles, including material gathering, content creation, quality check, and publishing. Use this skill when requested to research, write, review, format, or publish an article/post for a WeChat public account.
---

# WeChat Article Workflow

微信公众号文章完整工作流。

## 1. 搜集素材 (Research)
- 收集权威信息、数据和案例。
- 使用 `tavily-search`, `web_fetch`, `baoyu-url-to-markdown` 获取优质来源。
- **输出物**: 保存信息至 `articles/YYYY-MM-DD_标题/research_keyword.md`（包含关键数据来源和引用链接）。

## 2. 创作内容 (Writing)
- 基于收集的素材撰写文章。
- **工具**: 
  - `baoyu-format-markdown`: 格式化文章结构
  - `baoyu-cover-image`: 生成封面图
  - `baoyu-article-illustrator`: 生成文章配图
  - `baoyu-image-gen` / `baoyu-infographic`: 自定义插图或信息图
- **结构建议**: 吸引点击的标题 -> 导语(点明价值) -> 背景/引入 -> 核心分析 -> 案例/数据 -> 总结/建议 -> 作者/引导语
- **输出物**: 文章正文保存为 `articles/YYYY-MM-DD_标题/article.md`，配图保存在 `assets/` 子目录。

## 3. 检查修改 (Review)
发布前务必执行以下检查：
- **事实与合规**: 核实数据来源、无未经核实的爆料、涉及第三方措辞中性客观、敏感话题加风险提示。
- **可读性**: 段落适中（手机阅读友好），清晰合理的小标题，重点加粗，无错别字。
- **格式**: 封面图尺寸规范（900×383px 或 2.35:1），图片包含 alt 描述。
- 使用 `memory_search` 核对是否与历史内容冲突。重大内容或疑问需人工（Leo）确认。

## 4. 发布文章 (Publish)
- 使用 `baoyu-markdown-to-html` 将 MD 转换为微信兼容 HTML。
- 使用 `baoyu-post-to-wechat` 发布到公众号（发布前确认公众号已登录且填好标题、作者、摘要、封面图等元数据）。
- **默认模式**: 保存为草稿。仅在明确要求时直接发布。

## 目录结构规范
```
workspace/
└── articles/YYYY-MM-DD_标题/
    ├── research_keyword.md
    ├── article.md
    ├── article.html
    ├── publish_info.json
    └── assets/
        └── cover.jpg
```
