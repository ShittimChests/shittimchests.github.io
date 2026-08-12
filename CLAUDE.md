# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目性质

ShittimChests 组织的单页展示站点，由 GitHub demo-repository 模板改造而来。**零构建**：手写 HTML + 单个 CSS 文件，仓库根目录即站点根目录，没有打包器、框架或依赖安装步骤。

## 常用命令

没有 build / lint / test 脚本（`package.json` 里没有 `scripts` 字段）。

```bash
# 本地预览（必须用 HTTP server，直接 file:// 打开会让相对路径资源表现异常）
python3 -m http.server 8000

# 本地跑与 CI 完全相同的 HTML/链接检查（需要 Docker）
docker run --rm -v "$PWD:/src" -w /src -e INPUT_DIRECTORY=./ anishathalye/proof-html:2.2.5
```

## 架构与约定

### 文件职责

- `index.html` — 全部页面结构，四个区块：navbar / hero / projects / about，加 footer。内容语言为 `zh-CN`。
- `css/style.css` — 全站唯一样式表，按区块顺序组织，末尾集中放动画与响应式断点（768px / 480px）。
- `images/` — logo、hero 背景、成员头像。

### 设计令牌

所有颜色、圆角、阴影、过渡曲线都定义在 `css/style.css` 的 `:root` 中，前缀 `--ba-*`（Blue Archive 主题蓝 `#128AFA` 为主色）。新增样式一律引用令牌，不要硬编码色值或 `0.3s ease`。

### 滚动淡入的隐式契约

`index.html` 底部的内联 `<script>` 在 `DOMContentLoaded` 时用 `IntersectionObserver` 给 `.fade-in` 元素加 `.visible`。CSS 中 `.fade-in` 默认 `opacity: 0`。因此：

- 新增卡片若加了 `fade-in` 类，就依赖这段脚本才可见；
- 脚本只在 DOMContentLoaded 时查询一次 DOM，之后动态插入的 `.fade-in` 元素不会被观察，会永久保持不可见。

### 样式位置的两处例外

- hero 背景图写在 `index.html` 的行内 `style` 上，不在 CSS 里——只改 `.hero` 规则不会生效。
- `.brand-icon` 和 `.member-avatar` 是模板遗留的未使用样式，实际 HTML 用的是 `.brand-logo` 和 `.member-avatar-img`。

### HTML 实体约定

符号与表情统一用实体写法（`&#x2728;`、`&mdash;`、`&rarr;`、`&#x221E;`），不直接嵌字面字符。新增内容请沿用。

## CI 行为

- **`.github/workflows/proof-html.yml`** — 每次 push 先 `actions/checkout`，再对仓库根目录跑 `anishathalye/proof-html@v2.2.5`。两级检查：W3C Nu 验证器校验 HTML 与 CSS（`--errors-only`），HTMLProofer 检查图片路径、内部锚点、favicon、Open Graph 以及所有**外部**链接的可达性（默认 `enforce_https`，链接检查失败自动重试 3 次）。这是本仓库唯一的自动化质量关卡，退出码可信。

  历史坑（已在 `fix/ci-and-readme` 中修复，改动 workflow 时别改回去）：原配置缺少 checkout 步骤，工作区为空，日志显示 `Ran on 0 files!`；且 pin 的 v1.1.0 会 `rescue` 掉 HTMLProofer 的异常后以 0 退出。两者叠加导致这个 job 长期**恒绿且零覆盖**。

  ⚠️ 由于 CSS 校验已开启，新增 vendor 前缀属性（现有 `-webkit-background-clip`、`-webkit-text-fill-color` 等）时留意 Nu 验证器是否报错。favicon 检查默认开启，`index.html` 的 `<link rel="icon">` 不要删。

  `ignore_url_re` 里跳过了 `space.bilibili.com` 和 `linux.do`——这两个站点对 CI runner 的 IP 做反爬拦截（分别返回 412 和 429），链接有效但重试三次也过不了。GitHub 自身偶发 503，靠 v2 的重试自愈，不需要加白名单。

- **`.github/workflows/auto-assign.yml`** — 新开的 issue 和 PR 一律自动指派给 `ItzPlana`。

## 已知陷阱

`package.json` 声明了 `@primer/css@17.0.1`，但整个仓库没有任何地方引用它，也没有 `node_modules`。这是 GitHub 模板的遗留物，不要假设 Primer 的 class 可用。

## Git 约定

提交信息使用 Conventional Commits（`feat:` / `fix:`），功能分支形如 `feat/blue-archive-redesign`，经 PR 合入 `main`。
