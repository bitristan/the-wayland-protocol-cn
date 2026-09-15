# Linux dmabuf

大多数 Wayland 合成器都在 GPU 上进行渲染，许多 Wayland 客户端同样在 GPU 上渲染。在这种情况下，采用共享内存的方式把缓冲区从客户端发送给合成器就非常低效了，因为客户端必须把数据从 GPU 读取到 CPU，然后合成器又得把它从 CPU 读回 GPU 才能进行渲染。

Linux DRM（直接渲染管理器，Direct Rendering Manager）接口（一些 BSD 系统上也实现了它）为我们提供了一种导出 GPU 资源句柄的手段。Mesa 是 Linux 用户空间图形驱动最主要的实现，它实现了一个协议，允许 EGL 用户把其 GPU 缓冲区的句柄从客户端传输给合成器以供渲染，全程无需把数据复制到 CPU。

该协议内部的工作原理超出了本书的范围，更适合在专门聚焦 Mesa 或 Linux DRM 的资料中详述。不过，我们可以对它的用法做一个简要总结：

1. 将 `eglGetPlatformDisplayEXT` 与 `EGL_PLATFORM_WAYLAND_KHR` 配合使用，创建一个 EGL display。
2. 按常规方式配置该 display，选择适合你实际情况的配置，并将 `EGL_SURFACE_TYPE` 设置为 `EGL_WINDOW_BIT`。
3. 使用 `wl_egl_window_create` 为给定的 `wl_surface` 创建一个 `wl_egl_window`。
4. 使用 `eglCreatePlatformWindowSurfaceEXT` 为 `wl_egl_window` 创建一个 `EGLSurface`。
5. 之后照常使用 EGL 即可，例如用 `eglMakeCurrent` 使你的表面的 EGL 上下文成为当前上下文，用 `eglSwapBuffers` 把最新的缓冲区发送给合成器并提交表面。

如果之后需要更改 `wl_egl_window` 的大小，可以使用 `wl_egl_window_resize`。

## 但我是真的想了解内部原理

一些不使用 libwayland 的 Wayland 程序员抱怨这种做法把 Mesa 和 libwayland 紧紧捆绑在了一起，事实也确实如此。然而，把二者解耦并非不可能——只是这需要你付出大量工作，也就是自己实现 `linux-dmabuf`。协议的细节请查阅 Wayland 扩展 XML;Mesa 的实现则位于 `src/egl/drivers/dri2/platform_wayland.c`（截至撰写时）。祝你好运，一路顺风。

## 对于服务端

遗憾的是，合成器一侧的细节既复杂又超出了本书的范围。不过，我可以为你指个方向：wlroots 的实现（截至撰写时位于 `types/wlr_linux_dmabuf_v1.c`）简明直观，应该能帮你走上正轨。
