# The Wayland Protocol（中文译本）

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)

> 原著：**The Wayland Protocol** — Drew DeVault · <https://wayland-book.com/>
> 译者：**bitristan** ｜ 许可：**CC BY-SA 4.0**

《The Wayland Protocol》是一本面向类 Unix 系统的 Wayland 显示服务器协议书籍。本仓库是它的 **简体中文译本**，使用 [mdBook](https://rust-lang.github.io/mdBook/) 编写，可生成 HTML 网站与 PDF。

## 内容简介

译本覆盖：

- 协议设计与线上协议（wire protocol）
- 深入 libwayland（wayland-util、wayland-scanner、代理与资源、接口与监听器）
- Wayland display、全局对象与注册表（registry）
- 缓冲区与表面（surface）：共享内存、dmabuf、表面角色
- XDG shell：xdg_surface、xdg_toplevel、配置与生命周期、弹出窗口与定位器
- Seat 与输入：指针、键盘、XKB、触摸
- 剪贴板与拖放
- 常用协议扩展

> **原著状态**：原书为草稿（draft）。第 1–10 章基本完整；第 11 章与第 12 章的大部分小节在原书中仍是标题占位，译本如实保留。

## 阅读方式

- **在线阅读**：<https://bitristan.github.io/the-wayland-protocol-cn/>
- **PDF**：[book.pdf](book.pdf)（约 130+ 页，内嵌中文字体）
- 或用下面的命令本地构建（mdBook 会生成带侧边栏、全文搜索的网站）

## 构建

需要 [mdBook](https://rust-lang.github.io/mdBook/)；导出 PDF 另需 [WeasyPrint](https://weasyprint.org/)。

```sh
# 安装 mdBook（macOS 示例）
brew install mdbook

# 构建 HTML 网站（输出到 ./book）
mdbook build

# 本地预览（默认 http://localhost:3000）
mdbook serve

# 导出 PDF
weasyprint book/print.html book.pdf
```

## 目录

目录定义见 [src/SUMMARY.md](src/SUMMARY.md)。

- [关于本书](src/前言.md)
- [第 1 章 引言](src/01-introduction/00-简介.md)
  - [1.1 Wayland 总体设计](src/01-introduction/01-Wayland-总体设计.md) · [1.2 目标与目标读者](src/01-introduction/02-目标与目标读者.md) · [1.3 软件包内容](src/01-introduction/03-软件包内容.md)
- [第 2 章 协议设计](src/02-protocol-design/00-协议设计.md)
- [第 3 章 深入 libwayland](src/03-libwayland/00-深入-libwayland.md)
- [第 4 章 Wayland display](src/04-wayland-display/00-Wayland-display.md)
- [第 5 章 全局对象与注册表](src/05-registry/00-全局对象与-registry.md)
- [第 6 章 缓冲区与表面](src/06-surfaces/00-缓冲区与表面.md)
- [第 7 章 XDG shell 基础](src/07-xdg-shell-basics/00-XDG-shell-基础.md)
- [第 8 章 深入探讨表面](src/08-surfaces-in-depth/00-深入探讨表面.md)
- [第 9 章 seat：处理输入](src/09-seat/00-Seat-处理输入.md)
- [第 10 章 深入 XDG shell](src/10-xdg-shell-in-depth/00-深入-XDG-shell.md)
- [第 11 章 剪贴板访问](src/11-clipboard/00-剪贴板访问.md)
- [第 12 章 常用协议扩展](src/12-protocol-extensions/00-常用协议扩展.md)
- [致谢](src/13-致谢.md)

## 目录结构

```
.
├── book.toml          # mdBook 配置（源目录 src/，输出 book/）
├── book.pdf           # 预构建 PDF
├── src/
│   ├── SUMMARY.md     # 全书目录
│   ├── 前言.md
│   ├── 01-introduction/ … 12-protocol-extensions/
│   ├── 13-致谢.md
│   └── assets/        # 图片等静态资源
├── LICENSE            # CC BY-SA 4.0
└── README.md
```

## 翻译约定

- 协议标识符（`wl_*`、`xdg_*`）、XML、代码块**逐字保留**，一律不译。
- 术语遵循中文技术文档通行译法：合成器（compositor）、客户端（client）、服务端（server）、表面（surface）、缓冲区（buffer）、注册表（registry）、全局对象（global）、帧回调（frame callback）等。
- 中文正文使用全角标点，中英文之间以空格分隔。

## 由 AutoClaw 构建

本译本**完全由 [AutoClaw](https://autoglm.ai/) 构建**：从抓取原书、全文中英翻译、术语统一与校对，到 mdBook 站点、GitHub Pages 部署与 PDF 导出，全部由 AutoClaw 自动完成。谨此致以诚挚谢意。🙏

想体验 AutoClaw？通过邀请链接注册：

<https://autoglm.ai/misc/autoclaw-invite?activity_id=autoclaw_fission&channel=fission&target_app=autoclaw&target_app_version=1.18.5&os=mac&IC=W3W76PDH>

## 许可与致谢

本译本是 *The Wayland Protocol*（© 2021 Drew DeVault）的演绎作品，依照原书许可采用 **[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)** 发布，详见 [LICENSE](LICENSE)。

- 原著：Drew DeVault — <https://wayland-book.com/>（源码：<https://git.sr.ht/~sircmpwn/wayland-book>）
- 翻译：bitristan

如果你喜欢这本书，请支持原作者。
