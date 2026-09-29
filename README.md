# LOFTER Helper Plus

> 基于 [Lumiarna](https://greasyfork.org/zh-CN/scripts/568674) 的 **LOFTER Helper** 修改而来。
> 增强批量导出、标签搜索、暂停/取消、并发下载等能力。
> 使用需要登录。
---

## 📖 简介

**LOFTER Helper Plus** 是一个 Tampermonkey 用户脚本，用于一键导出 LOFTER 上的合集或单篇文章。

主要功能：

- 作者主页批量导出全部文章（基于 API，无需滚动加载）
- 合集自动识别，支持合并/单篇/跳过策略
- 关键词搜索（匹配**标题 + 标签 + 合集名**，不区分大小写）
- 支持暂停、继续、取消导出任务
- 并发抓取正文和图片，速度更快
- 自动将 `nos.netease.com` 图床地址重写为 `lf127.net`，解决 403 问题
- 图片文件名附带原文地址，便于溯源
- 按 `LOFTER_作者_合集名_贴文名` 格式命名单篇文件

---
## 📦 安装

1. 安装 [Tampermonkey](https://www.tampermonkey.net/) 浏览器扩展。
2. 在 Tampermonkey 中添加本脚本：
   - 方式一：[![Greasy Fork](https://img.shields.io/badge/安装-Greasy%20Fork-blue?logo=greasyfork)](https://greasyfork.org/zh-CN/scripts/597927-lofter-helper-plus)
   - 方式二：[![安装](https://img.shields.io/badge/安装-LOFTER%20Helper%20Plus-brightgreen?logo=tampermonkey)](https://cdn.jsdelivr.net/gh/HloYemi/LOFTER-Helper-Plus@main/LOFTER-Helper-Plus.user.js)
3. 打开 LOFTER 任意页面，网页右上角会出现按钮，确认脚本已启用。
---

## 🔧 与原版 LOFTER Helper 的主要区别

本脚本在原版基础上做了以下修改：

### 1. 改用 API 获取作者全部文章

原版依赖页面滚动加载，容易漏文章、翻页逻辑也不稳定。  
现在改为通过 LOFTER API：

- 先请求合集列表 `getCollectionList`
- 再逐个合集请求 `getCollectionDetail`
- 一次性拿到作者所有合集内的文章

参考了 [SrakhiuMeow/lofter-getter](https://github.com/SrakhiuMeow/lofter-getter) 的 API 调用思路。

### 2. 修复关键词搜索

原版只能按标题匹配，而 LOFTER 上大量图片贴标题为空，导致搜不到。

现在搜索同时匹配：

- `post.title`
- `post.tagList`（标签数组，直接 join 成字符串）
- `collectionName`（合集名）

### 3. 新增暂停 / 继续 / 取消

导出过程中，页面右侧会出现：

- **⏸ 暂停**：暂停所有抓取任务，可恢复
- **✕ 取消**：立即终止当前导出，已下载内容不会写出 ZIP

### 4. 并发抓取，速度优化

- 文章抓取并发：4
- 图片下载并发：4
- 请求间隔从 800ms / 300ms 降至 200ms / 80ms

### 5. 修复图片 403 问题

LOFTER 部分图片使用 `nos.netease.com` 域名，脚本请求会返回 403。  
现在自动将：

```
https://nos.netease.com/imglf6/xxx
```

重写为：

```
https://imglf6.lf127.net/xxx
```

同时下载图片时携带 `Referer: https://www.lofter.com/`，绕过防盗链。

### 6. 图片文件名加入原文地址

单篇和合集导出的图片文件名中会包含文章地址（去掉 `https://`），例如：

```
LOFTER_祢尔MERE_原神堆堆_发发头像_miermere.lofter.com_post_4d02bec5_2b76b7b09_01.jpg
```

### 7. 主页懒加载，减少页面负担

打开作者主页时不会自动请求 API，只有点击“导出全部”按钮后才开始拉取文章列表。  
设置面板中的合集策略也是按需加载。

### 8. 其他细节

- 单篇文件名格式：`LOFTER_作者_合集名_贴文名`
- 合集导出按合集分文件夹，图片按文章序号命名
- 支持 Markdown / TXT 两种导出格式
- 兼容 `photoLinks` 为字符串或数组两种情况
---
## 🚀 使用

### 导出作者全部作品

1. 打开作者主页，例如 `https://miermere.lofter.com/`
2. 点击右侧“**导出全部**”按钮
3. 输入关键词（多个关键词用 `|` 分隔，留空则导出全部）
4. 等待抓取完成，浏览器会自动下载 ZIP 文件

### 导出单篇文章

1. 打开任意文章页
2. 点击右侧“**导出本篇**”按钮
3. 文本和图片会分别下载

### 设置

点击右侧 **⚙** 图标可以：

- 切换导出格式（Markdown / TXT）
- 跳过图片
- 设置散章策略（合并 / 单篇 / 跳过）
- 设置每个合集单独的策略

---

## ⚠️ 注意事项

- 本脚本依赖 LOFTER 的非公开 API，**官方更新后可能随时失效**。
- 请合理设置并发和延迟，不要高频请求，避免对 LOFTER 服务器造成压力。
- 图片下载可能因图床策略变化而失败，脚本会打印警告并跳过。
- 导出的所有内容版权归原作者所有，**请勿用于商业用途或二次传播**。
- Cookie 信息由浏览器自动携带，脚本不会上传任何数据。

---

## 📜 免责声明

本脚本仅供个人学习、研究及技术交流使用，不得用于任何商业用途或非法目的。

使用者应自觉遵守相关法律法规及 LOFTER 平台的服务条款。因使用本脚本产生的一切后果，由使用者自行承担，脚本作者不承担任何责任。

本脚本的编写与发布，不代表对 LOFTER 平台任何内容的授权或认可。所有下载内容的版权归原作者所有，请勿进行二次传播或商业利用。

---

## 🙏 致谢

- 原脚本作者：[Lumiarna](https://greasyfork.org/zh-CN/scripts/568674)（LOFTER Helper）
- API 调用思路参考：[SrakhiuMeow](https://github.com/SrakhiuMeow/lofter-getter)（lofter-getter）
- 修改与维护：[HloYemi/LOFTER-Helper-Plus](https://github.com/HloYemi/LOFTER-Helper-Plus)

---

## 📄 许可证

本项目采用 **MIT License**，详见 [LICENSE](./LICENSE) 文件。
