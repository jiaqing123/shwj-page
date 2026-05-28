# Gmeek 博客搭建与维护指南

> 博客地址：https://jiaqing123.github.io/shwj-page  
> 仓库地址：https://github.com/jiaqing123/shwj-page  
> Gmeek 官方仓库：https://github.com/Meekdai/Gmeek

---

## 一、项目文件结构

```
shwj-page/
├── .github/workflows/
│   └── Gmeek.yml          # GitHub Actions 自动构建部署
├── config.json             # 博客核心配置文件
├── Gmeek.py                # 静态页面生成脚本（从 Gmeek 官方仓库拉取）
├── requirements.txt        # Python 依赖
├── templates/              # Jinja2 HTML 模板
├── plugins/                # Gmeek 插件
├── img/                    # 静态图片资源
├── backup/                 # Issue 数据备份（自动生成）
├── docs/                   # 生成的静态网站文件（自动生成）
│   ├── index.html          # 博客首页
│   ├── post/               # 文章详情页
│   ├── postjson/           # 文章 JSON 数据
│   └── rss.xml             # RSS 订阅
└── Gmeek使用指南.md         # 本文件
```

---

## 二、搭建步骤（已完成的回顾）

### 2.1 前期准备

1. GitHub 仓库：`jiaqing123/shwj-page`
2. 仓库 Settings → Pages → Source 选择 **GitHub Actions**

### 2.2 核心文件配置

**config.json** — 博客配置（必须）：

```json
{
    "title": "jiaqing123",                              // 博客标题
    "subTitle": "生活不只有代码，还有诗和远方。",         // 副标题
    "avatarUrl": "",                                    // 头像图片链接
    "GMEEK_VERSION": "last",                            // Gmeek 版本，last=最新
    "displayTitle": "jiaqing123",                       // 页面显示标题
    "homeUrl": "https://jiaqing123.github.io/shwj-page",// 博客主页 URL
    "onePageListNum": 15,                               // 每页显示文章数
    "singlePage": ["about"],                            // 独立页面（关于页等）
    "i18n": "CN",                                       // 语言：CN=中文
    "themeMode": "manual",                              // 主题模式
    "dayTheme": "light",                                // 日间主题
    "nightTheme": "dark_colorblind",                    // 夜间主题
    "urlMode": "issue",                                 // URL 模式：issue编号
    "showPostSource": 1,                                // 显示文章来源链接
    "rss": 1,                                           // 启用 RSS
    "bottomText": "❤️ 由 Gmeek 强力驱动",                // 页脚文字
    "indexStyle": "postList",                           // 首页样式
    "postListNew": false                                // 最新文章列表
}
```

**.github/workflows/Gmeek.yml** — 关键配置要点：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `permissions.issues` | `read` | ⚠️ **必须**，否则无法读取 Issue |
| `permissions.contents` | `write` | ⚠️ **必须**，用于写入生成文件 |
| `permissions.pages` | `write` | ⚠️ **必须**，用于部署 |
| `permissions.id-token` | `write` | ⚠️ **必须**，用于认证 |
| `upload-pages-artifact path` | `docs` | ⚠️ **必须**，Gmeek 输出在 docs/ 目录 |

### 2.3 踩过的坑

| # | 问题 | 原因 | 解决方案 |
|---|------|------|----------|
| 1 | `Node.js 20 deprecated` 警告 | GitHub 2026年弃用 Node 20 | 升级所有 actions 到最新版 + 设置 `FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true` |
| 2 | `Gmeek.py: error: arguments required` | Gmeek.py 需要命令行参数 | `python Gmeek.py ${{ secrets.GITHUB_TOKEN }} ${{ github.repository }}` |
| 3 | `post.html` 模板找不到 | 只下载了 Gmeek.py，缺少 templates/ | 用 sparse checkout 拉取完整源码（templates + plugins + img） |
| 4 | `git sparse-checkout` 报错 | cone 模式下不能混合文件和目录 | 加 `--no-cone` 参数 |
| 5 | 构建成功但页面空白 | 缺少 `issues: read` 权限 | 在 permissions 中添加 `issues: read` |
| 6 | 首页一直显示旧内容 | Gmeek 输出在 docs/ 而非根目录 | `upload-pages-artifact path` 从 `'.'` 改为 `'docs'` |

---

## 三、添加文章（写博客）

### 3.1 写一篇文章

1. 打开 https://github.com/jiaqing123/shwj-page/issues/new
2. **标题** = 文章标题
3. **正文** = 文章内容（支持 Markdown 格式）
4. ⚠️ **右侧 Labels** → 必须至少选一个标签（如 `blog`）
5. 点击 **Submit new issue**

提交后 GitHub Actions 自动触发构建，约 1 分钟后博客更新。

### 3.2 创建标签（Labels）

在 https://github.com/jiaqing123/shwj-page/labels 创建常用标签：

| 标签名 | 颜色 | 用途 |
|--------|------|------|
| `blog` | `#006b75` | 普通博文 |
| `about` | `#1f883d` | 关于页面（固定页） |
| `tech` | `#0969da` | 技术文章 |
| `life` | `#bc4c00` | 生活随笔 |

### 3.3 创建独立页面（如"关于"页）

在 `config.json` 的 `singlePage` 数组中已配置了 `["about"]`。创建关于页：

1. 新建 Issue，标题为 `关于我`
2. 添加 `about` 标签（⚠️ 只用这一个标签）
3. 正文写自我介绍
4. 提交 → 自动生成独立页面

### 3.4 文章格式示例

````markdown
## 我的第一篇文章

这是一段文字。支持 **加粗**、*斜体*、~~删除线~~。

### 代码块

```python
print("Hello, Gmeek!")
```

### 图片

直接拖放图片到 Issue 编辑器中即可。

### 引用

> 这是一段引用文字。
````

---

## 四、日常维护

### 4.1 修改博客配置

编辑仓库中的 `config.json` 文件，修改后需要**手动触发全量重建**：

1. 打开 https://github.com/jiaqing123/shwj-page/actions/workflows/Gmeek.yml
2. 点击 **Run workflow**
3. 勾选 `clean_cache`（清除缓存）
4. 点击 **Run workflow** 执行

### 4.2 修改文章

1. 打开对应的 Issue
2. 编辑标题或正文
3. 保存 → Actions 自动触发更新

### 4.3 删除文章

直接 **Close** 对应的 Issue 即可。文章会从博客中移除。如需彻底删除，Close 后再 Delete Issue。

### 4.4 更新 Gmeek 版本

当 Gmeek 官方发布新版本时：

1. Actions 中手动 Run workflow（⚠️ 勾选 `clean_cache`）
2. `config.json` 中 `GMEEK_VERSION` 设为 `"last"` 会自动使用最新版
3. 如需固定版本，改为具体版本号如 `"v2.25"`

### 4.5 自定义主题/样式

在 `config.json` 中通过以下字段注入自定义代码：

```json
{
    "style": "<style>/* 自定义 CSS */</style>",
    "script": "<script>/* 自定义 JS */</script>",
    "allHead": "<!-- 添加到 <head> 的内容 -->",
    "scriptBodyEnd": "<!-- 添加到 </body> 前的内容 -->"
}
```

### 4.6 本地调试（可选）

如需在本地预览博客效果：

```bash
cd "D:\My Documents\GitHub\shwj-page"
python Gmeek.py <你的GitHub_Token> jiaqing123/shwj-page
# 生成的文件在 docs/ 目录
# 用浏览器打开 docs/index.html 预览
```

> GitHub Token 获取：Settings → Developer settings → Personal access tokens → Generate new token（勾选 `repo` 和 `user` 权限）

---

## 五、故障排查

| 症状 | 可能原因 | 检查方法 |
|------|----------|----------|
| Actions 构建失败 | 查看 Actions 日志 | https://github.com/jiaqing123/shwj-page/actions |
| 博客页面空白 | 没有带 Label 的 Issue | 确保 Issue 至少有一个标签 |
| 修改不生效 | 需要全局重建 | Run workflow 勾选 `clean_cache` |
| 图片不显示 | 图片链接失效 | 使用 GitHub Issue 拖放上传的图片 |
| 404 错误 | Pages 未启用 | Settings → Pages → Source = GitHub Actions |

---

## 六、快速命令参考

```bash
# 拉取远程更新
cd "D:\My Documents\GitHub\shwj-page"
git pull origin main

# 提交本地修改
git add -A
git commit -m "描述修改内容"
git push origin main
```

---

> 📅 搭建日期：2026-05-28  
> 🔗 博客地址：https://jiaqing123.github.io/shwj-page
