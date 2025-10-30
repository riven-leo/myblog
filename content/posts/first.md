---
title: "小小博客轻松拿下！"
date: 2025-10-30T20:47:53+08:00
tags: ["hugo", "git", "github", "vscode"]
categories: ["里程碑"]
weight: 50
show_comments: true
katex: false
draft: true
description: "并非轻松"
---

<!--more-->
下面是用ai，根据整个构建博客的过程，生成的关于运用工具、实际流程、部分困难和解决方式的总结
---

# 一、用到的工具与概念（技术栈）

* Hugo（静态站点生成器，extended 版本用于 SCSS 支持）
* VS Code（主要开发环境，尽量用 GUI 操作 Git）
* Git / GitHub（代码与 Pages 托管、Actions CI）
* GitHub Actions（自动构建 + 部署到 Pages：actions/configure-pages、upload-pages-artifact、deploy-pages）
* Clash（用于解决本地 git HTTPS 连接的代理问题，使用 TUN 模式解决网络访问）
* 主题：`hugo-theme-flat`（主题 + exampleSite）
* 本地预览：`hugo server -D`（热重载）
* Front Matter / archetypes（文章模板与元数据）
* layouts/partials（自定义主题局部模板覆盖，例如 footer）

---

# 二、主要步骤（我们实际执行的流程）

1. 在 GitHub 建仓 `myblog`，在本地用 VS Code 打开工作目录。
2. 克隆或在本地初始化 Hugo 项目（`hugo new site .`；在遇到非空目录时使用 `--force` 或在子目录生成）。
3. 把选好的主题 `hugo-theme-flat` 放到 `themes/` 下（注意主题目录名要和 `config.toml` 里的 `theme` 一致）。
4. 复制主题内的 `exampleSite` 内容到站点根（**推荐**，快速得到 demo 配置、示例文章、static 资源和 config）。
5. 调整 `config.toml`：

   * `theme = "hugo-theme-flat"`
   * 将 `themesDir` 修正为 `themes`（不要用 `../..` 导致路径向上跳转问题）
   * 修改 `title`、`description`、`menus`、`footer_rows` 等字段为自己的信息
6. 本地运行预览 `hugo server -D`，观察输出并调试：

   * 处理短代码缺少参数导致的错误（例如 `twitter_simple` 报错）——删除或修正示例文章。
   * 处理 Raw HTML、远程资源取回失败等 WARN（可忽略或通过配置关闭日志）。
7. 主题定制：

   * 不直接改主题源文件。若要覆盖，创建 `layouts/partials/footer.html` 放在项目根以覆盖默认 footer；针对 RSS 链接，删除对应代码段以移除底部的 Feed 链接。
   * 不用复制 `layouts/` 或 `archetypes/` 除非需要定制模板或文章模板。
8. 配置 `archetypes`：

   * 编写 `archetypes/default.md` / `archetypes/posts.md` / `archetypes/essays.md`，使用 `{{ .Date }}`、`{{ replace .Name "-" " " | title }}` 等占位使 `hugo new` 自动注入元数据。
   * 增加主题自定义字段（`weight`、`show_comments`、`katex` 等）以便控制排序、评论和公式显示。
9. 本地写作流程：

   * 在 `content/posts/`（项目总结）和 `content/essays/`（思考随笔）下创建 `.md` 文件并填 front matter（`title`、`date`、`tags`、`categories`、`draft`）。
   * 本地预览时用 `hugo server -D` 查看草稿。
10. 版本控制与部署：

    * 把 `.github/workflows/deploy.yml` 加入仓库，使用 GitHub Actions 自动构建与部署（我们采用 `actions/configure-pages` + `actions/upload-pages-artifact` + `actions/deploy-pages` 的官方流程）。
    * workflow 中要注意：`--source ./`（指向站点根），`path: ./public`，指定 `HUGO_VERSION` 与是否需要安装 Dart Sass（如果主题不使用 SCSS，可以删掉 Dart Sass 步骤）。
    * 修正 workflow 中的 shell 写法（避免 `\` 续行缩进导致 `--minify: command not found`），最稳妥把 `hugo ...` 写成一行。
11. 推送与触发：

    * 在 VS Code 中 commit & push 到 `main` 分支触发 Actions。Actions 成功后，Pages 从 `gh-pages` 分支或 GitHub Pages 的 settings 自动生效，站点上线。
12. 网络与 git 问题处理：

    * 遇到 `Failed to connect to github.com port 443`，开启 Clash 的 TUN 模式或给 Git 配置 http/https 代理，亦或使用 SSH remote URL（需配置 SSH key）来解决访问 GitHub 的不稳定问题。
    * 当本地与远程历史不同步且 push/pull 失败时，采用**重新 clone 到新文件夹**然后把本地修改（内容文件）复制到新 clone 的仓库再提交的安全流程，避免直接覆盖 `.git` 导致版本库损坏。

---

# 三、我们遇到的具体困难与解决办法（详列）

1. **`hugo new site .` 报错：目录非空**

   * 问题：Hugo 要求空目录或使用 `--force`。
   * 解决：用 `hugo new site . --force` 或在子目录运行 `hugo new site site/`。

2. **主题没被识别：module not found / 名字与目录不匹配**

   * 问题：下载主题时文件夹名带 `-main` 或 config 里的 `theme` 名称不一致，或 `themesDir` 配置错误。
   * 解决：把主题文件夹重命名为 `hugo-theme-flat` 或把 `config.toml` 中 `theme` 改为实际文件夹名；把 `themesDir` 改回 `themes`。

3. **短代码与 exampleSite 的错误导致构建失败**

   * 问题：示例文章用旧短代码或缺少参数（例如 `twitter_simple`），还有尝试远程抓取 Vimeo 报错。
   * 解决：删除 example 的示例文章或修正短代码；在构建时忽略相应 warn 或提供参数；删除或替换 example content。

4. **PowerShell 显示 `//localhost:1313/` 且无法直接点击**

   * 问题：PowerShell 输出省略 `http:`，无法直接点击。
   * 解决：在浏览器地址栏输入 `http://localhost:1313/` 或在 `hugo server` 加 `-O` 自动打开浏览器。

5. **Hugo 多行命令在 workflow 中报 `--minify: command not found`**

   * 问题：YAML 多行续行加 `\` 且续行前有缩进会让 shell 把参数当成新命令。
   * 解决：把命令写成单行：`hugo --source ./ --minify --baseURL "${{ steps.pages.outputs.base_url }}/"`。

6. **push/pull 失败报 `Failed to connect to github.com port 443`**

   * 问题：网络或代理问题导致 git HTTPS 无法连通。
   * 解决：启用 Clash 的 TUN 模式或设置 Git 的 http/https 代理，或改用 SSH（配置 SSH key）。

7. **本地与远程历史冲突、push 被拒绝**

   * 问题：远程仓库可能被 Actions 修改过或本地历史不一致。
   * 解决：最稳妥流程是 **重新 clone 到新文件夹**，把本地内容（`content/`、`static/`、自定义 `layouts/`）复制过来再提交；避免直接覆盖原仓库文件夹里的 `.git`。

8. **RSS 链接在 footer 自动生成，但想移除**

   * 问题：主题自动通过 `.Site.Sections` 生成 feed 链接。
   * 解决：在项目 `layouts/partials/footer.html` 覆盖默认实现，删除产生 feed 的那段循环，保留 `footer_rows` 部分作为版权信息。

9. **想自动为新文章注入时间与标题**

   * 解决：使用 `archetypes` 模板（`archetypes/default.md`），放 `date: {{ .Date }}` 和 `title: "{{ replace .Name "-" " " | title }}"`，然后通过 `hugo new` 自动生成 front matter。

---
这篇post发布意味着，博客基本框架已经搭好。在接下来的时间，还需要大量的时间来打磨和个性化，加油吧，呱~