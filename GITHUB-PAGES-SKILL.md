---
name: evan-lei-github-pages
description: Manage evan-lei's personal GitHub Pages site (evan-lei.github.io). Use when the user wants to add a new project, update courseware, sync changes to GitHub, or ask about the site structure, directory layout, or deployment workflow.
---

# evan-lei.github.io 管理指南

## 站点信息

| 项目 | 值 |
|------|-----|
| 公开地址（自定义域名，主推 ✅） | `https://evan-minho.com/` |
| 公开地址（Cloudflare Pages，备用） | `https://evan-lei-github-io.pages.dev/` |
| 公开地址（Vercel，备用） | `https://evan-lei-github-io.vercel.app/` |
| 公开地址（GitHub Pages，备用） | `https://evan-lei.github.io/` |
| GitHub 仓库 | `https://github.com/evan-lei/evan-lei.github.io` |
| 本地工作目录 | `/Users/user/Documents/evan-lei.github.io` |
| GitHub 用户名 | `evan-lei` |
| Git 邮箱 | `ding.lei@hellogroup.com` |

**部署方式**：`git push` 到 GitHub 后，Cloudflare Pages 自动检测并在 30 秒内重新部署，无需任何手动操作。

---

## 项目命名规则

课件/攻略类项目统一用 **"跟着爸爸去 XXX"** 命名，例如：
- 跟着爸爸去中国钓鱼（`fishing/`）
- 跟着爸爸去三亚（`sanya/`）
- 跟着爸爸去沙漠（`desert/`）
- 跟着爸爸去露营（`camping/`）

---

## 目录结构

```
/Users/user/Documents/evan-lei.github.io/
├── index.html                       ← 根目录主页（项目入口列表）
├── 455fdbdc9ef2517c3fa35ebb6aaf2955.txt  ← 微信域名验证文件（勿删）
├── .gitignore
├── fishing/                         ← 跟着爸爸去中国钓鱼
│   ├── index.html
│   └── source/                      ← 本地备份，已上传至 minho-1256297898 COS（source/ 已加入 .gitignore）
├── sanya/                           ← 跟着爸爸去三亚
│   └── index.html                   ← 素材在 sanya-1256297898 COS
├── desert/                          ← 跟着爸爸去沙漠
│   └── index.html                   ← 素材在 desert-1256297898 COS
├── camping/                         ← 跟着爸爸去露营
│   ├── index.html                   ← 素材在 camping-1256297898 COS
│   └── source/                      ← 本地备份，已上传至 COS（source/ 已加入 .gitignore）
└── <新项目名>/
    └── index.html
```

**独立仓库（勿塞进本目录）**：与 `jiufu-fishing` 同级，例如工程笔记 **`/Users/user/Documents/harness-engineering-notes`** → GitHub `evan-lei/harness-engineering-notes`。

---

## 推送到 GitHub

推送使用 Personal Access Token（HTTPS 方式），token 临时嵌入 remote URL：

```bash
cd /Users/user/Documents/evan-lei.github.io

git remote set-url origin https://evan-lei:<YOUR_GITHUB_PAT>@github.com/evan-lei/evan-lei.github.io.git
git push origin main
git remote set-url origin https://github.com/evan-lei/evan-lei.github.io.git
```

如 token 失效，在此重新生成：`https://github.com/settings/tokens/new`（勾选 `repo` 权限）。

---

## 腾讯云 COS（仅用于存储图片/视频素材）

> ⚠️ 大陆 COS 节点未备案时会强制下载 HTML，**不要用 COS 托管 HTML 页面**，HTML 部署统一走 Cloudflare Pages。

| 字段 | 值 |
|------|----|
| SecretId | `<YOUR_TENCENT_SECRET_ID>` |
| SecretKey | `<YOUR_TENCENT_SECRET_KEY>` |
| AppID | `1256297898` |
| 地域 | `ap-beijing` |

**Bucket 命名规则**：`<项目名>-1256297898`，每个项目独立一个。

已有 bucket：
- `minho-1256297898` — fishing 课件素材
- `sanya-1256297898` — 跟着爸爸去三亚图片/视频
- `desert-1256297898` — 跟着爸爸去沙漠图片/视频
- `camping-1256297898` — 跟着爸爸去露营图片/视频

### 上传命令

```bash
export PATH="$PATH:$HOME/Library/Python/3.9/bin"
coscmd config -a <YOUR_TENCENT_SECRET_ID> -s <YOUR_TENCENT_SECRET_KEY> -b <bucket名> -r ap-beijing

# 批量上传目录
coscmd -b <bucket名> -r ap-beijing upload -rs <本地目录>/ /

# 公开访问 URL 格式
# https://<bucket名>.cos.ap-beijing.myqcloud.com/<路径>
```

### 上传前必须压缩（节省流量费）

取原始 vs 压缩版中**更小的**上传。

**图片（macOS 内置 `sips`）**
```bash
# JPG/JPEG：最大1920px，质量80
sips -Z 1920 -s formatOptions 80 input.jpg --out output.jpg
# PNG（地图/截图）：最大1920px
sips -Z 1920 input.png --out output.png
```

**视频（`ffmpeg`）**
```bash
# 主视频（保留音频，720p）
ffmpeg -i input.mp4 -c:v libx264 -crf 26 -preset fast \
  -vf scale=-2:720 -c:a aac -b:a 96k -movflags +faststart output.mp4

# Live Photo 短片（无音频，480p）
ffmpeg -i input.MOV -c:v libx264 -crf 26 -preset fast \
  -vf scale=-2:480 -an -movflags +faststart output.mp4
```

### 上传前检查尺寸，规划 UI 布局

上传素材之前，先用以下命令查看每张图片/视频的**宽高和方向**，再决定页面布局：

```bash
# 批量查看图片尺寸和方向
for f in *.jpg *.jpeg *.png *.JPG *.JPEG *.PNG; do
  [ -f "$f" ] || continue
  w=$(sips -g pixelWidth "$f" | awk '/pixelWidth/{print $2}')
  h=$(sips -g pixelHeight "$f" | awk '/pixelHeight/{print $2}')
  orient=$([ "$w" -gt "$h" ] && echo "横图" || echo "竖图")
  echo "$orient  ${w}x${h}  $f"
done

# 查看视频尺寸
ffprobe -v error -select_streams v:0 \
  -show_entries stream=width,height -of csv=p=0 input.mp4
```

**布局规划原则：**
- **全横图**：2×2 或 3 列网格平铺
- **全竖图**：单列或并排 2 列
- **竖图 + 横图混排**：竖图单独占一列（`grid-row: 1 / span N`），横图堆叠在另一列
- **地图/路线图（通常竖向）**：放左列大图，右侧放环境横图；若有视频，右侧最后一格放视频缩略图（见下方预览视频规范）

---

### 预览视频规范（点击播放，禁止自动播放）

**原则：预览网格中的视频禁止 `autoplay`，改为点击播放。**

原因：
- 自动播放会在页面加载时立即拉取视频流，拖慢页面速度
- 腾讯云 COS 按下行流量计费，autoplay 会造成不必要的费用

**正确写法：**
```html
<div class="pthumb">
  <video src="https://xxx.cos.ap-beijing.myqcloud.com/xxx.mp4"
         muted playsinline preload="metadata"></video>
  <div class="pthumb-play"></div>
</div>
```
- `preload="metadata"` 仅加载元数据（时长、首帧），不拉取完整视频流
- 禁止使用 `autoplay` 和 `loop`
- 必须有 `.pthumb-play` 叠加层作为播放按钮

**点击播放 JS（在页面初始化时绑定一次）：**
```js
document.querySelectorAll('.pthumb-play').forEach(btn => {
  btn.addEventListener('click', e => {
    e.stopPropagation(); // 阻止冒泡，避免触发父级 openGal
    const video = btn.previousElementSibling;
    if (!video || video.tagName !== 'VIDEO') return;
    if (video.paused) {
      video.play();
      btn.style.opacity = '0';
      video.addEventListener('pause', () => { btn.style.opacity = ''; }, { once: true });
      video.addEventListener('ended', () => { btn.style.opacity = ''; video.load(); }, { once: true });
    } else {
      video.pause();
    }
  });
});
```

**CSS（`.pthumb-play` 必须可点击）：**
```css
.pthumb-play {
  position: absolute; inset: 0;
  display: flex; align-items: center; justify-content: center;
  cursor: pointer; transition: opacity .2s;
}
.pthumb-play::after {
  content: '';
  width: 52px; height: 52px; border-radius: 50%;
  background: rgba(255,255,255,.88);
  box-shadow: 0 2px 12px rgba(0,0,0,.35);
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='%23222'%3E%3Cpath d='M8 5v14l11-7z'/%3E%3C/svg%3E");
  background-size: 26px; background-repeat: no-repeat; background-position: 58% center;
}
```
注意：**不要**加 `pointer-events: none`，否则点击事件无法触发。
**按钮必须用白底深色图标**（不要用黑底白图标），视频首帧通常偏暗，黑色按钮会隐身。

---

### 图片/视频查看器交互规范

所有项目的 lightbox / gallery 必须遵守以下三条规则：

#### 1. 触摸滑动导航
```js
let _touchX = 0, _touchY = 0;
lb.addEventListener('touchstart', e => {
  _touchX = e.touches[0].clientX;
  _touchY = e.touches[0].clientY;
}, { passive: true });
lb.addEventListener('touchend', e => {
  if (!lb.classList.contains('open')) return;
  const dx = e.changedTouches[0].clientX - _touchX;
  const dy = e.changedTouches[0].clientY - _touchY;
  if (Math.abs(dx) > 50 && Math.abs(dx) > Math.abs(dy) * 1.5) navigate(dx < 0 ? 1 : -1);
}, { passive: true });
```
- 左滑 → 下一张，右滑 → 上一张
- 触发条件：水平位移 > 50px 且横向幅度 > 纵向 × 1.5（避免上下滚动误触发）

#### 2. 键盘方向键导航
```js
document.addEventListener('keydown', e => {
  if (!lb.classList.contains('open')) return;
  if (e.key === 'Escape') closeLb();
  if (e.key === 'ArrowLeft')  navigate(-1);
  if (e.key === 'ArrowRight') navigate(1);
});
```

#### 3. 跨 Tab 连续翻页（仅适用于有图片/视频分 Tab 的 gallery）
- 图片翻到最后一张再点"下一张" → 自动切换到"视频"Tab 第一个
- 视频往前翻到第一个再点"上一张" → 自动切换回"图片"Tab 最后一张
- 实现：`navigate` 函数中判断边界时调用 `switchTab`，而非取模循环

参考实现见 `camping/index.html` 的 `galNav` 函数。

---

## 工作流：添加新项目

1. 新建项目目录：`mkdir -p <项目名>`
2. 创建 `<项目名>/index.html`
3. 在腾讯云新建 bucket `<项目名>-1256297898`，**压缩后**上传素材，HTML 内用 COS 绝对 URL
4. 在根目录 `index.html` 的 `.grid` 里新增 `.card`，指向 `./<项目名>/`
5. 更新根目录 `README.md`：项目列表表格 + 目录结构
6. 提交推送（Cloudflare Pages 自动部署）：
   ```bash
   git add <项目名>/ index.html README.md
   git commit -m "add <项目名>"
   git push origin main   # 记得用 token，见上方"推送到 GitHub"
   ```

---

## 换电脑时恢复 SKILL

1. Clone 仓库后，把本文件复制到 Cursor 的 skills 目录：
   ```bash
   mkdir -p ~/.cursor/skills/evan-lei-github-pages
   cp GITHUB-PAGES-SKILL.md ~/.cursor/skills/evan-lei-github-pages/SKILL.md
   ```
2. 用真实的密钥替换 SKILL.md 里的占位符（`<YOUR_GITHUB_PAT>`、`<YOUR_TENCENT_SECRET_ID>`、`<YOUR_TENCENT_SECRET_KEY>`）：
   - GitHub PAT：`https://github.com/settings/tokens` 生成
   - 腾讯云密钥：`https://console.cloud.tencent.com/cam/capi` 查看
3. 重启 Cursor，SKILL 即可生效。
4. 每次更新 SKILL 后，记得把内容（脱敏后）同步回 `GITHUB-PAGES-SKILL.md` 并推送 GitHub。

---

## 注意事项

- **Cloudflare Pages 单文件限制 25MB**，超过会部署失败。所有素材必须放 COS，不要把图片/视频提交进 git
- `fishing/source/`、`camping/source/`、根目录 `source/` 均已加入 `.gitignore`，不会再被误提交
- 单文件严禁超过 100MB，否则 git push 失败（GitHub 限制）
- 微信验证文件 `455fdbdc9ef2517c3fa35ebb6aaf2955.txt` 必须保留在根目录，勿删
- `.gitignore` 已排除 `.DS_Store`、`*.zip`、`幼儿课需求.md`、`跟着爸爸去中国钓鱼.html`
