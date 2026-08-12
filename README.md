# ShittimChests · 什亭之箱

> 连接所有奇迹的起点 ✨

ShittimChests 组织主页的源码 —— 一个零构建的静态单页站点，采用 Blue Archive 主题。

## 技术构成

无框架、无打包器、无依赖安装步骤。仓库根目录即站点根目录。

| 文件 | 职责 |
| --- | --- |
| `index.html` | 全部页面结构：navbar / hero / projects / about / footer |
| `css/style.css` | 全站唯一样式表，设计令牌定义在 `:root` 的 `--ba-*` 变量中 |
| `images/` | logo、hero 背景、成员头像 |

滚动淡入效果由 `index.html` 底部的内联 `IntersectionObserver` 脚本驱动，给 `.fade-in` 元素添加 `.visible` 类。

## 本地预览

需要通过 HTTP server 访问，直接用 `file://` 打开会让相对路径资源表现异常。

```bash
python3 -m http.server 8000
# 然后打开 http://localhost:8000
```

## 自动化检查

每次 push 都会触发 [Proof HTML](.github/workflows/proof-html.yml)，用 W3C Nu 验证器校验 HTML 与 CSS，并通过 HTMLProofer 检查图片路径、内部锚点、favicon 以及所有外部链接的可达性（失败自动重试 3 次）。

本地复现同一套检查（需要 Docker）：

```bash
docker run --rm -v "$PWD:/src" -w /src -e INPUT_DIRECTORY=./ anishathalye/proof-html:2.2.5
```

新开的 issue 和 PR 会由 [Auto Assign](.github/workflows/auto-assign.yml) 自动指派。

## 相关项目

- [cyterx-api](https://github.com/ShittimChests/cyterx-api) —— 统一的 AI 模型聚合与分发网关
- [bluearchive.site](https://bluearchive.site) —— 蔚蓝站点，面向碧蓝档案玩家的信息展示平台

## License

MIT
