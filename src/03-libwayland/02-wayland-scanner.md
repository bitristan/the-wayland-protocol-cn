# wayland-scanner

Wayland 软件包附带一个二进制文件：`wayland-scanner`。这个工具用于从第 2.3 节讨论过的 Wayland 协议 XML 文件生成 C 头文件和胶水代码。在 “wayland” 软件包的构建过程中，这个工具被用来为核心协议 `wayland.xml` 预先生成头文件和胶水代码。这些头文件会成为 `wayland-client-protocol.h` 和 `wayland-server-protocol.h`——不过你通常会包含 `wayland-client.h` 和 `wayland-server.h`，而不是直接使用它们。

这个工具的用法相当简单（`wayland-scanner -h` 有概要说明），可以归纳如下。生成客户端头文件：

```
wayland-scanner client-header < protocol.xml > protocol.h
```

生成服务端头文件：

```
wayland-scanner server-header < protocol.xml > protocol.h
```

生成胶水代码：

```
wayland-scanner private-code < protocol.xml > protocol.c
```

不同的构建系统配置自定义命令的方式各不相同——请查阅你的构建系统的文档。一般来说，你会在构建时运行 `wayland-scanner`，然后把你的应用程序与生成的胶水代码一起编译和链接。

现在就随便拿一个 Wayland 协议来试试吧，如果你手头正好有的话（例如，`wayland.xml` 很可能就在 `/usr/share/wayland` 里）。打开生成的胶水代码和头文件，在阅读后续章节时随手对照，以便理解 libwayland 提供的原语在生成代码中是如何被实际运用的。
