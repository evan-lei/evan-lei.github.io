---
name: evan-lei-github-pages
description: Manage evan-lei's personal GitHub Pages site (evan-lei.github.io). Use when the user wants to add a new project, update courseware, sync changes to GitHub, or ask about the site structure, directory layout, or deployment workflow.
---

# evan-lei.github.io 管理指南

## 站点信息

| 项目 | 值 |
|------|-----|
| 公开地址 | `https://evan-lei.github.io/` |
| GitHub 仓库 | `https://github.com/evan-lei/evan-lei.github.io` |
| 本地工作目录 | `/Users/user/Documents/evan-lei.github.io` |
| GitHub 用户名 | `evan-lei` |
| Git 邮箱 | `ding.lei@hellogroup.com` |

## 目录结构

```
/Users/user/Documents/evan-lei.github.io/          ← 本地工作目录（对应 evan-lei.github.io 仓库）
├── index.html                       ← 根目录主页（项目入口列表）
├── 455fdbdc9ef2517c3fa35ebb6aaf2955.txt  ← 微信域名验证文件（勿删）
├── .gitignore
├── fishing/                         ← 项目：跟着爸爸去中国钓鱼
│   ├── index.html                   ← 课件主文件
│   └── source/                      ← 课件图片/视频素材
│       ├── 0.JPEG ~ 26.mp4
│       └── ...
└── <新项目名>/                      ← 未来新项目放这里
    ├── index.html
    └── source/（如有素材）
```

**访问规则**：
- 主页：`https://evan-lei.github.io/`
- 钓鱼课件：`https://evan-lei.github.io/fishing/`
- 新项目：`https://evan-lei.github.io/<项目名>/`

## Git Remotes

本地目录同时关联两个远端（历史原因）：

```bash
git remote -v
# origin  https://github.com/evan-lei/minho.git         ← 归档用，可忽略
# pages   https://github.com/evan-lei/evan-lei.github.io.git  ← 线上正式版
```

**推送永远用 `pages`，不用 `origin`。**

## 认证

推送使用 Personal Access Token（HTTPS 方式）。
Token 需在推送时临时嵌入 remote URL，推送后移除：

```bash
# 推送前设置（含 token）
git remote set-url pages https://evan-lei:<TOKEN>@github.com/evan-lei/evan-lei.github.io.git

# 推送
git push pages main

# 推送后清除 token
git remote set-url pages https://github.com/evan-lei/evan-lei.github.io.git
```

如 token 失效，让用户在此处重新生成：`https://github.com/settings/tokens/new`（勾选 `repo` 权限）。

## 工作流：修改已有课件后同步

编辑 `fishing/index.html` 或替换 `fishing/source/` 内的素材后：

```bash
cd /Users/user/Documents/evan-lei.github.io
git add fishing/
git commit -m "update fishing courseware"
git push pages main
```

## 工作流：添加新项目

1. 在 `/Users/user/Documents/evan-lei.github.io/` 下新建项目目录：
   ```bash
   mkdir -p <项目名>/source
   ```
2. 将 HTML 主文件命名为 `index.html` 放入目录
3. 素材放 `<项目名>/source/`，HTML 内用相对路径 `./source/` 引用
4. **在根目录 `index.html` 的 `.grid` 里新增一张 `.card`**，指向 `./<项目名>/`
5. 提交推送：
   ```bash
   git add <项目名>/ index.html
   git commit -m "add <项目名>"
   git push pages main
   ```

## 注意事项

- `source/` 内的视频文件（.MOV/.mp4）超过 50MB 时 GitHub 会警告但不报错，可正常使用
- 单文件严禁超过 100MB，否则 push 失败
- 微信验证文件 `455fdbdc9ef2517c3fa35ebb6aaf2955.txt` 必须保留在根目录
- `.gitignore` 已排除 `.DS_Store`、`*.zip`、`幼儿课需求.md`、`跟着爸爸去中国钓鱼.html`
- 本地同名文件 `跟着爸爸去中国钓鱼.html` 是钓鱼课件的开发工作区，等同于 `fishing/index.html`，修改后需手动同步：`cp 跟着爸爸去中国钓鱼.html fishing/index.html`
