# libwayland 实现

我们在第 1.3 节中简要介绍过 libwayland——它是最流行的 Wayland 实现。本书的大部分内容适用于任何实现，但接下来的两章里，我们将带你熟悉这一个实现。

Wayland 软件包为 wayland-client 和 wayland-server 提供了 pkg-config 规范——如何与它们链接，请查阅你所用的构建系统的文档。自然，大多数应用程序只会链接其中之一。该库包含一些简单的原语（比如链表），以及一份预编译的 `wayland.xml`——即 Wayland 核心协议。

我们将从介绍这些原语开始。
