---
name: rednote-photographer-workflow
description: 小红书摄影师作品自动发布工作流。用于搜索知名摄影师、分析其创作风格、获取代表作品并自动生成小红书文案发布。触发条件：用户提到"发布小红书笔记"、"分享摄影师"、"审美提升"、"摄影作品"等关键词时。
---

# 小红书摄影师作品发布工作流

自动完成从摄影师发现到小红书笔记发布的完整流程。

## 工作流程

### Step 1: 搜索摄影师

使用 `tavily_search` 工具搜索摄影师作品，携带以下参数：
- `include_images: true` - 包含图片
- `include_image_descriptions: true` - 包含图片描述
- `count: 20` - 获取更多结果

**示例：**
```json
{
  "query": "{photographer_name} photographylatest",
  "include_images": true,
  "include_image_descriptions": true,
  "count": 20
}
```

**操作步骤：**
1. 使用 Tavily 搜索获取图片和摄影师信息
2. 从返回结果中提取：
   - 图片 URL（`images` 数组）
   - 图片描述（`description` 字段）
   - 摄影师名字、所在地、简介

### Step 2: 获取9张代表作品图片 URL

从 Tavily 搜索结果中提取图片 URL，**不需要下载图片**：

1. 检查 `tavily_search` 返回的 `images` 数组
2. 提取图片 URL 和描述信息
3. 筛选出9张最具代表性的图片 URL

**将 URL 列表保存至草稿文件：**
```
~/.openclaw/workspace-rednote/rednotes/YYYY-MM-DD_{标题}/draft.md
```

**draft.md 格式示例：**
```markdown
# {摄影师名} - 小红书草稿

## 图片 URL
1. https://...
2. https://...
3. https://...

## 摄影师信息
- 名字：
- 所在地：
- 风格：
```

**⚠️ 注意：** 如果某个 URL 无法访问，从搜索结果中替换其他图片 URL，确保所有链接有效。

### Step 3: 分析摄影风格

根据图片 URL 对应的图片描述（`description` 字段）和搜索结果文本分析以下特征：

| 维度 | 分析要点 | 示例 |
|------|---------|------|
| **色调** | 冷/暖、明亮/暗沉、饱和度 | 冷色调、黑灰调、胶片感 |
| **光影** | 对比度、光源方向、阴影处理 | 强烈对比、侧光、剪影 |
| **构图** | 对称、引导线、框架、留白 | 中心构图、对角线、层次感 |
| **情绪** | 孤独、温暖、神秘、活力 | 阴郁、诗意、电影感 |
| **主题** | 街头、人像、风景、建筑 | 都市生活、人文纪实 |

### Step 4: 生成小红书文案

**标题格式：**
```
审美积累｜{风格标签}{摄影师名}
```

**正文模板：**
```
📸 {开场白}——走进{摄影师名}的摄影世界
{摄影师名}是一位来自{城市}的{type}摄影师，{一句话描述其特点}。

他的作品风格偏向 {风格关键词}。在他的照片里，{画面特点}。{色调}是他常用的基调，他擅长通过{技法}来营造照片的{效果}。

📌 作品亮点：
✅ {亮点1：具体描述一个视觉特征}

✅ {亮点2：描述人物与环境的关系}

✅ {亮点3：描述情绪或氛围营造}

#视觉审美积累 #每日审美分享 #审美提升 #审美 #审美积累 #摄影 #风格化摄影 #摄影审美 #一分钟摄影故事
```

**参考示例：**

> 📸 都市光影里的情绪捕手——走进Thomas Kakareko的摄影世界
> Thomas Kakareko是一位来自柏林的街头摄影师和创意顾问，他的镜头总能在灰暗中捕捉到一丝温柔与张力。他既是索尼阿尔法的摄影大使，也是三星德国的合作伙伴，用相机讲述城市里不容易被发现的故事。
> 
> 他的作品风格偏向 冷色调 + 对比光影 + 强烈情绪感。在他的照片里，建筑、街角与行人常常融为一体。黑灰调是他常用的基调，他擅长通过光线与色彩的对比来营造照片的情绪张力。
> 
> 📌 作品亮点：
> ✅ 光影曲线：光线与阴影形成强烈对比，他喜欢用曲线和层次来营造画面的张力。
> 
> ✅ 人物与城市共存：人物不是静止的风景，而在建筑与街头中动态出现，与城市空间形成呼应。
> 
> ✅ 氛围感强烈：他偏爱深色调和"阴郁心情"，调色时在曲线中下功夫，创造出他的个人影像语言。

### Step 5: 发布到小红书

> ⚠️ **重要**：运行期间不要在其他浏览器登录同一个小红书账号，否则会使 MCP server 的 session 失效。

**① 启动 MCP 服务器（必须从 bin/ 目录运行）：**
```bash
# 检查是否已在运行
curl -s http://localhost:18060/api/v1/login/status > /dev/null 2>&1 && echo "已运行" || (
  cd ~/.openclaw/skills/xiaohongshu-mcp/bin && ./xiaohongshu-mcp-darwin-arm64 &
  sleep 2
)
```

**② 检查登录状态：**
```bash
python ~/.openclaw/skills/xiaohongshu-mcp/scripts/xhs_client.py status
```

**③ 如果未登录，执行登录流程（必须从 bin/ 目录运行，cookies.json 在该目录下）：**
```bash
cd ~/.openclaw/skills/xiaohongshu-mcp/bin && ./xiaohongshu-login-darwin-arm64 &
sleep 4
# 截屏发给用户扫码
screencapture -x /tmp/xhs_qrcode.png
```
将 `/tmp/xhs_qrcode.png` 发送给用户，提示：**"请用小红书 App 扫描二维码完成登录"**。
用户扫码后等待5秒，再次运行 `status` 确认登录成功，然后继续。

**④ 将最终文案保存至：**
```
~/.openclaw/workspace-rednote/rednotes/YYYY-MM-DD_{标题}/note.md
```

**⑤ 发布笔记（图片直接传 URL，无需下载）：**
```bash
python ~/.openclaw/skills/xiaohongshu-mcp/scripts/xhs_client.py publish \
    "审美积累｜{风格标签}{摄影师名}" \
    "{正文内容}" \
    "{图片URL1},{图片URL2},{图片URL3},..."
```

**图片数量说明：**
- 理想：9张；最少：1张
- 不足9张时：4-8张可正常发布；1-3张建议补充来源或改发单图

**发布失败处理：**
- 图片上传超时 → 用 `curl -I <URL>` 验证每个 URL 是否有效，替换失效 URL 后重试
- 无法连接服务器 → 确认从 `bin/` 目录启动了 MCP server
- 登录态失效 → 重复步骤③重新扫码登录

## 目录结构规范

每篇笔记的所有素材统一存放在以下目录：
```
~/.openclaw/workspace-rednote/rednotes/YYYY-MM-DD_{标题}/
├── draft.md          # 图片 URL 列表 + 摄影师信息
└── note.md           # 最终文案（标题 + 正文）
```

## 注意事项

1. **图片数量**：建议9张，质量比数量更重要。如果无法获取9张高质量图片，4-5张精品图也可以发布。
2. **图片 URL 有效性**：Tavily 返回的图片 URL 可能有时效性，发布前用 `curl -I` 快速验证可访问性。
3. **版权问题**：仅使用公开账号的内容，尊重摄影师版权
4. **风格一致性**：选择的图片应该有统一的视觉风格
5. **文案原创性**：根据实际图片描述撰写，不要照搬模板

## 常见摄影师推荐

以下是一些适合分享的知名摄影师：

| 摄影师 | 用户名 | 风格特点 |
|--------|--------|----------|
| SamAlive | @samalive | 电影感人像、温暖色调 |
| Thomas Kakareko | @thomaskakareko | 街头摄影、冷色调、情绪感 |
| Brandon Woelfel | @brandonwoelfel | 霓虹夜景、梦幻色彩 |
| Jessica Kobeissi | @jessicakobeissi | 时尚人像、自然光 |

## 故障排除

### Tavily 搜索问题

**搜索结果没有图片？**
- 确保携带 `include_images: true` 参数
- 尝试添加 `include_image_descriptions: true` 获取图片描述
- 更换搜索关键词，如添加 "portfolio"、"photography"

### 图片 URL 问题

**URL 无法访问？**
```bash
# 快速验证 URL 可访问性
curl -I "https://图片URL"
# 返回 200 OK 即为有效
```

**解决方案：**
1. 重新用 Tavily 搜索获取新的图片 URL
2. 调整搜索关键词换一批图片
3. 减少图片数量发布（小红书支持1-9张图）

### 小红书发布问题

**无法连接 MCP 服务器？**
```bash
# 必须从 bin/ 目录启动，否则 cookies.json 路径错误
cd ~/.openclaw/skills/xiaohongshu-mcp/bin && ./xiaohongshu-mcp-darwin-arm64 &
sleep 2
python ~/.openclaw/skills/xiaohongshu-mcp/scripts/xhs_client.py status
```

**登录状态失效？**
```bash
# 必须从 bin/ 目录运行 login 工具
cd ~/.openclaw/skills/xiaohongshu-mcp/bin && ./xiaohongshu-login-darwin-arm64 &
sleep 4
screencapture -x /tmp/xhs_qrcode.png
```
发送截图给用户扫码，扫码后等待5秒，再 `status` 确认。

> ⚠️ 登录期间不要在其他浏览器打开小红书，否则 session 会被踢出。

**图片上传超时？**

错误示例：
```
{"error":"发布失败","code":"PUBLISH_FAILED","details":"小红书上传图片失败: 第5张图片上传超时"}
```
原因：图片 URL 失效或图片格式不支持。
```bash
# 验证 URL 可访问性
curl -I "https://图片URL"   # 返回 200 则有效
```
替换失效 URL 后减少数量重试（先试1-2张）。

### 文案优化建议

**文案不够吸引人？**
- 多观察图片细节
- 使用更具体的形容词
- 加入个人感受和解读
- 参考 Skill 提供的模板结构
