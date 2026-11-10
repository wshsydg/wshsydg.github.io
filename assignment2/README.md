# 徐永舜 · 个人博客

一个纯静态的单文件个人博客站点，零构建、零依赖后端，可直接推到 GitHub Pages 与 Cloudflare Pages。

## 文件

- `index.html` —— 整站（HTML + CSS + JS + 矢量插画全在一个文件里，图片零外链）
- `posts.json` —— 文章的主数据源（标题、栏目、日期、摘要、Markdown 正文）
- `admin.html` —— 站内发布后台（写文章、插图、提交到仓库）
- `assets/bg-sky.jpg` —— 页面背景图，取自参考图的蓝白绘面（从原图左右两段纯净天空取样、随机拼接后柔化而成，2400×1350）
- `assets/posts/` —— 文章插图，后台自动传到这里
- `README.md` —— 本文件

## 本地预览

双击 `index.html` 即可，或用任意静态服务器：

```
python -m http.server 8000
```

然后打开 http://localhost:8000

## 部署

### GitHub Pages

1. 新建一个仓库（建议名字 `blog` 或 `xuyongshun.github.io`；后者可直接用自定义域名 `xuyongshun.github.io`）。
2. 把 `index.html`、`README.md` 放到仓库根目录，提交并推送。
3. 仓库 → Settings → Pages → Source 选 `Deploy from a branch`，Branch 选 `main`，目录 `/ (root)`，保存。
4. 等 1 分钟左右，访问 `https://<用户名>.github.io/<仓库名>/`。

注意：站点是单文件 hash 路由（`#/about`、`#/works` 等），不需要任何重写规则，GitHub Pages 与 Cloudflare Pages 都不用额外配置。

### Cloudflare Pages

1. 登录 Cloudflare Dashboard → Workers & Pages → Create → Pages → Connect to Git，选中同一个仓库。
2. 构建设置：Framework preset 选 `None`，Build command 留空，Build output directory 填 `/`（若用子目录则填子目录名）。
3. 保存部署，之后每次 push 自动重新发布。

也可以直接用 Upload assets 上传 `index.html` 单文件。

## 内容维护

所有内容都在 `index.html` 底部的 `<script>` 里，三个数据数组（文章已移出，见下节）：

| 数组 | 对应栏目 | 说明 |
| --- | --- | --- |
| `SECTIONS` | 首页六入口 | 每张栏目卡：`id`（路由）、`t`（标题）、`k`（小标）、`d`（一句话）、`n`（数量/状态） |
| `WORKS` | 作品 | 每个对象一个项目，含 `id`、`name`、`event`、`form`、`year`、`lede`、`meta`、`body`、`state` |
| `AWARDS` | 荣誉 | 奖项列表 |

新增一个作品：复制一个对象、改 `id`（会变成链接 `#/works/<id>`）与各字段，列表与详情页自动生成，不用改代码结构。

## 发布文章

文章不在 `index.html` 里，而是存在仓库的 `posts.json`。发布 = 往这个文件里加一条数据并提交；两个平台都是"提交即自动重新部署"，所以下面的两种发法，提交成功后都会在 30–60 秒内自动上线。

### 发法一：站内发布后台（推荐）

1. 在 GitHub 建一个 **Fine-grained token**：头像 → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new。Repository access 只勾这一个仓库，Permissions 里只勾 `Contents: Read and write`。复制生成的 `github_pat_…`。
2. 打开站底部的「点此上传文章」按钮（即 `admin.html`），在「连接设置」里填仓库 `用户名/仓库名`、分支（一般 `main`）、令牌，保存。
3. 写标题、栏目、日期、摘要、正文（Markdown），点「发布到仓库」。右下角会给出这次提交的链接。
4. 插图：点「插入图片」选文件，图片会被传到 `assets/posts/` 并在正文光标处插入 `![说明](路径)`。传之前先压缩，单张控制在 300 KB 以内最稳。

令牌只存在当前浏览器的 localStorage，不写进代码、不上传；它等于这个仓库的写权限，别在公用电脑上留着，换电脑要重新填，用完可点「清除令牌」。

### 发法二：直接在 GitHub 网页改（兜底）

打开仓库里的 `posts.json`，点铅笔编辑，在 `posts` 数组最前面粘一段：

```json
{
  "slug": "fengxiang-field-note-1",
  "status": "live",
  "kind": "田野日志",
  "date": "2026.11.10",
  "title": "凤翔田野笔记 · 第一天",
  "summary": "一句话摘要，列表里显示。",
  "body": "## 小标题\n\n第一段正文……\n\n- 列表项\n\n> 受访人原话"
}
```

提交即可。注意 JSON 里字符串不能真的换行，段落之间用 `\n\n`。

### 字段说明

| 字段 | 说明 |
| --- | --- |
| `slug` | 文章地址 `#/writing/<slug>`，只用英文数字与 `-`，**定了别改**（改了旧链接失效） |
| `status` | `live` 可读；`draft` 只在列表里显示「正文整理中」 |
| `kind` / `date` / `title` / `summary` | 栏目、日期、标题、一句话摘要 |
| `body` | Markdown 正文；留空等于草稿 |

### 正文支持的 Markdown

`## 小标题`、`### 次级标题`、空行分段、`- 无序列表`、`1. 有序列表`、`> 引用`、`**粗体**`、`*斜体*`、反引号行内代码、`---` 分隔线、`![说明](路径)` 插图（图片下一行不超过 60 字会自动当图注）、`[文字](链接)` 超链接、三个反引号围起来的代码块。

### 阅读页

点开文章进入窄栏阅读页（`#/writing/<slug>`）：大标题、栏目与日期一行元信息、正文限宽约 34 个全角字、行高 2.05、图片满栏带阴影、引用用左侧蓝线。想调版式改 `index.html` 里 `/* ============ 文章：阅读页 ============ */` 那一段。

### 本地预览的注意点

`posts.json` 是运行时 `fetch` 读的，双击 `index.html`（`file://`）会被浏览器拦住，此时列表用的是 `index.html` 里的 `POSTS_FALLBACK` 兜底数据（只有那几条草稿和说明文）。要完整预览就用 `python -m http.server 8000` 起个本地服务器。线上（GitHub / Cloudflare Pages）不受影响。

### 「知识」栏目

现在是空状态页，入口可点击，页面中心是动效文字 “Happy Everyday”（逐字浮现 + 轻呼吸，背景有缓慢漂移的浅蓝圆斑）。

将来有内容时，把 `#view-notes` 里的 `.empty-stage` 整块替换成卡片列表（复用 `.cards` + `.card` 结构），不用改路由。

### 封面小人

三只砖块小人是内联 SVG 手绘的，颜色写在各自的 `fill` 里。点击任意一只会轮换问候语（`GREETS` 数组），气泡会重新弹出。想换成插画图片时，把三处 `<svg class="fig">` 换成 `<img class="fig" src="assets/fig-1.png">` 即可。

问候语自动循环：`GREETS` 里是六种语言的两句话（英 `Hello / Welcome`、中 `你好 / 欢迎`、日 `こんにちは / ようこそ`、韩 `안녕하세요 / 환영합니다`、泰 `สวัสดี / ยินดีต้อนรับ`、法 `Bonjour / Bienvenue`），`setInterval` 每 4.2 秒换一种，气泡重新弹出；点一下小人立即切到下一种语言并重置计时，页面切到后台或封面滚出屏幕时暂停。增删语言只需改 `GREETS` 与 `GREET_LANGS` 两个数组（顺序一一对应）。

正文与小人之间有一条分界线：窄屏是文字下方的一条横向渐隐细线（`.figs::before`），900px 以上变成两栏之间的竖向渐隐细线。

### 页面背景

背景是 `body` 之后的第一块 `<div class="bg">`，由三层组成：

1. `img.bg-photo` —— `assets/bg-sky.jpg`，`object-fit: cover` 铺满，提供原图的蓝白绘面底纹
2. `.bg-veil` —— 中央白色径向渐变，中间近乎纯白保证正文可读，越靠边越透，让两侧的绘面显出来
3. 内联 SVG（压在面纱之上，所以云能飘在页面中间）—— 四枚蕾丝花片（`<g id="doily">`）、底部蕾丝花边（`<pattern id="laceBand">`）、两侧白色笔触补笔、四团循环飘过的白云
4. 整层 `position: fixed`，不随页面滚动，像一层静止的画框

白云动画：`<g id="cloud">` 是云的母版（白色云团 + 浅蓝云底），页面上用四个 `<use class="drift c1..c4">` 复用，各自 `animation-duration` / `animation-delay` 不同（76s–150s），从画布左侧 -620px 匀速飘到右侧 2260px，无缝循环。想加减速改 `.bg .drift.cN` 的 `animation-duration`；`prefers-reduced-motion` 时整组云会隐藏。

换背景图：把新图命名为 `assets/bg-sky.jpg` 覆盖即可（建议 2400 px 宽以上、无人物与文字）。
想更强烈：把 `.bg-veil` 的各档透明度整体降 0.1–0.2（如 `.90` 改 `.7`、`.34` 改 `.2`）；想更淡则反之。

### 浮动资料卡

首页的「六个入口」（我的项目 / 学生工作 / 作品 / 文章 / 荣誉 / 知识）、以及作品、文章、荣誉三个栏目都用同一套卡片：`.cards` 容器 + `.card` 卡片。

卡片是白底半透明、圆角 18 px、带蓝调投影，hover 上浮 9 px 并加深投影，底部一条蓝色细线从左展开。点击整张卡片即进入对应详情（作品卡进 `#/works/<id>`）。

新增栏目入口改 `SECTIONS` 数组；新增作品卡改 `WORKS` 数组，卡片与作品列表会同步生成。

### 「我的项目」页与「学生工作」页

这两页（`#/about`、`#/student`）的正文都是静态 HTML，不在数据数组里：改文字直接编辑 `index.html` 里对应 `<section>` 下的 `.prose-body`。`#/about` 的五段分别是项目问题与主线、概念厘清与绣种谱系、口述史方法、传播与平台规划、进展与下一步；`#/student` 写的是两学年的任职与职责，页面下方各有一条时间线。

作品想挂外部成品页时，在 `WORKS` 里加两个字段：`link`（完整网址）与 `linkLabel`（按钮文字），资料卡下方会多出一枚外链，作品详情页的「素材状态」下方也会出现同一个链接。目前 `nihu-digital`（凤翔泥虎结构拆解）挂的是 `https://fengxiangnihu.pages.dev`。

### 顶栏

深蓝底（`#0E3A87`），左侧署名 `Xu Yongshun`，右侧五个栏目文字为浅蓝，hover 变白并出现下划线；当前所在栏目的文字与下划线保持白色常亮。

## 媒体规范（自托管）

- 视频：H.264 MP4，压到单个 25 MB 以内，放 `assets/`（Cloudflare Pages 单文件上限 25 MiB）
- 音频：mp3 128 kbps 或 m4a AAC，采访录音用单声道，原始 wav 留本地不推仓库
- 图片：webp，质量 80，证书类保留可读清晰度
- 原始素材不要进仓库，仓库只放压缩后的可上站版本

## 不入站的内容（隐私）

- `入党` 文件夹（含考试真题、含身份信息的表格、结业证书）
- `综测活动证明`、综合素质测评表（含成绩排名）
- 含学号的文件（如 `徐永舜 2025106104.pdf`）
- 他人实名访谈录音/逐字稿：发布前需取得受访者同意，否则化名处理

## 待补

- 联系方式（邮箱 / GitHub 主页 / 其他站点）尚未加入页脚，需要你提供
- 部分作品年份未知，列表里显示 `—`
- 获奖等级与年份按文件夹中的材料整理，需按证书原件核对；证书图整理后可加入荣誉页做证书墙
- 作品的「我的工作」字段按材料推断填写，需你确认实际分工
