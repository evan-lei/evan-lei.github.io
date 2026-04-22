# 工程笔记（静态页）

本目录为单页 **`index.html`**，排版针对长文与表格优化，并内嵌 **Mermaid** 渲染流程图。

## 作为独立 GitHub 仓库发布

1. 在 GitHub 上 **New repository**（例如 `harness-engineering-notes`），不设模板、可公开。
2. 本地仅推送本目录内容时，可临时拷贝：

   ```bash
   mkdir -p ~/work/harness-notes && cd ~/work/harness-notes
   cp /path/to/evan-lei.github.io/academic/index.html .
   cp /path/to/evan-lei.github.io/academic/README.md .
   # 可选：存档 Markdown
   cp "/path/to/客户端Harness Engineering 使用探索.md" ./source.md

   git init
   git add .
   git commit -m "Initial: Harness Engineering 使用探索"
   git branch -M main
   git remote add origin https://github.com/<你的用户名>/<仓库名>.git
   git push -u origin main
   ```

3. 仓库 **Settings → Pages**：Source 选 **Deploy from a branch**，Branch 选 **main** / **/(root)**，保存后约 1 分钟可访问 `https://<用户名>.github.io/<仓库名>/`（若使用 `username.github.io` 主站仓库则根目录即站点）。

## 源稿

Markdown 存档：`客户端Harness Engineering 使用探索.md`（与网页同步维护时可更新后重新用 Pandoc 生成正文再套模板，或手工改 `index.html`）。

## 技术说明

- 正文由 **Pandoc (GFM)** 转换，再经表格横向滚动包裹、Mermaid 块替换，避免通用 MD→HTML 工具破坏版式。
- 图表依赖 CDN：`mermaid@10`（需联网加载）。
