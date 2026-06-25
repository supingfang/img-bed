# SKILL：X 热点推文 → 公众号文章（一键流程）

> 这是一套固定的标准操作流程（SOP）：把"昨天"X（原 Twitter）上的 AI 热门推文，整理成一篇"舆情观察·热帖解读"风格的微信公众号文章，并完成配图截图、图床上传、排版导入。建议每天在全新会话里跑一遍（历史越短，自动截图与上传越稳定）。

---

## 目标

每天产出一篇可直接发布的公众号文章：选定一条当日热门 AI 推文 → 截一张干净的推文卡片图 → 上传到 GitHub 图床 → 写一篇 1500–2500 字解读文章 → 导入排版工具确认效果。原子单位：一条推文 = 一篇文章。

---

## 总览：五个步骤

1. 抓取昨天的 AI 热门推文（中英文），按点赞排序给出前 10，由我筛选。
2. 针对选定推文，截一张只含推文卡片的干净截图。
3. 把截图上传到 GitHub 图床，拿到 raw 直链。
4. 按"舆情观察·热帖解读"角度写一篇文章，把图嵌进去。
5. 导入 wechat.bmpi.dev 排版工具，确认配图与版式正常。

---

## 第 1 步：抓取并排序热门推文

抓取"昨天"（今天减 1 天）X 上 AI 相关的热门推文，中英文都要（广义 AI：模型 / 产品 / 观点 / 资讯都算），按点赞数排序，给出前 10 名列表（作者 @handle、正文要点、赞 / 转 / 评 / 浏览、发帖时间、链接），交给我筛选。

搜索语法（英文）：
```
https://x.com/search?q=(AI OR "artificial intelligence" OR LLM OR OpenAI OR Anthropic) min_faves:3000 since:昨天 until:今天&f=top
```

搜索语法（中文）：
```
https://x.com/search?q=(AI OR 人工智能 OR 大模型 OR OpenAI OR Claude OR Qwen) lang:zh min_faves:800 since:昨天 until:今天&f=top
```

> 注意：中文圈 min_faves 常不严格生效、绝对互动量远低于英文圈，属正常现象。

---

## 第 2 步：截一张干净的推文卡片

打开选定推文的单帖页，先用 JS 把头像 opacity 强制设为 1，隐藏左侧菜单和右侧栏，再截一张只含推文卡片的干净截图（头像 + @用户名 + 正文 + 时间 / 浏览 + 互动一行）。

头像与页面清理 JS：
```js
document.querySelectorAll('img').forEach(i=>{if(i.src.includes('profile_images')){i.style.opacity='1';i.style.transition='none';}});
document.querySelectorAll('header[role="banner"],[data-testid="sidebarColumn"]').forEach(e=>e.style.display='none');
```

截图硬性要求：
- 必须是真实截图，绝不生成 / 绘制 / 合成推文卡片图；只允许用 JS 改显示样式，不伪造内容。
- 卡片只保留：头像 + @用户名 + 正文 + 时间 / 浏览 + 互动一行。
- 不要包含底部 "Post your reply" 回复框（它会显示我自己的头像）。
- 不要包含推文卡片下方的评论 / 回复区。

常见坑与对策：
- 头像显示灰色占位：仅设 opacity 不够，需对头像容器下所有后代元素强制 opacity=1; visibility=visible，并多等几秒让头像图片加载完。
- 评论区反复出现（X 动态重渲染）：用 MutationObserver 持续隐藏新加入的评论 cell。
- 截图前先滚动到顶部，再截图。

---

## 第 3 步：上传到 GitHub 图床

把截图上传到图床仓库 supingfang/img-bed（分支 main、根目录），文件名用 tweet_<作者>.png。

步骤：
1. 打开上传页 https://github.com/supingfang/img-bed/upload/main 。
2. 上传截图文件，命名为 tweet_<作者>.png（重做时用同名覆盖）。
3. commit 说明填 "Add <作者> tweet screenshot"，直接 Commit 到 main。
4. 验证 raw 直链可正常显示（被缓存时加 ?v= 或 ?cb= 参数强制刷新）。

raw 直链格式：
```
https://raw.githubusercontent.com/supingfang/img-bed/main/<文件名>
```

---

## 第 4 步：写文章（舆情观察·热帖解读）

按"舆情观察·热帖解读"角度写一篇 1500–2500 字的公众号文章（Markdown），把上面的 raw 图片嵌进去。

文章结构：
- 标题
- 导语（带互动数据）
- 帖子说了什么（此处插图）
- 为什么戳中人
- 情绪 / 现象来源（分点）
- 客观的另一面
- 收尾启示
- 文末出处 + 互动引导

图片 Markdown 格式：
```
![截图来源：X（@handle）](https://raw.githubusercontent.com/supingfang/img-bed/main/<文件名>)
```

> 互动数据用"抓取时即时值"，并注明可能随时间变动。

---

## 第 5 步：导入排版工具

打开 https://wechat.bmpi.dev/ ，用 JS 把 Markdown 写进 CodeMirror 编辑器，确认排版和配图正常。

导入 JS：
```js
const cm = document.querySelector('.CodeMirror').CodeMirror;
cm.setValue(/* Markdown 全文 */);
cm.refresh();
```

预览刷新关键点：合成事件无法触发预览刷新。需要一次真实键盘输入——点进编辑器，敲一个空格再删掉（空格 + Backspace），预览即更新。

确认无误后，在页面顶部选一个主题，点复制按钮，粘贴到公众号后台即可发布。

---

## 注意事项汇总

- 每天用新会话跑流程（历史短 = 自动截图 / 上传更可靠）。
- 截图必须是真实截图，绝不生成 / 绘制。
- 干净卡片：不含回复框、不含评论区。
- 文件名固定 tweet_<作者>.png，重做时同名覆盖。
- bmpi.dev 预览刷新必须用真实按键（空格 + Backspace），不能用合成事件。
- 自动截图上传失败时：我手动截图发给你，你用 file_upload 把图传到网页文件框。
- JS 输出里避免返回 URL 形式的字符串（可能被拦截）。
