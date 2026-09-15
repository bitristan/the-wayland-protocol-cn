# 使用 wl_compositor

人们说，命名是计算机科学中最困难的问题之一，而此刻我们正手握证据站在这里。`wl_compositor` 全局对象就是 Wayland 合成器的、呃，合成器。通过这个接口，你可以把你的窗口发送给服务端进行呈现，以便与同时显示在旁边的其他窗口一起被*合成*。合成器有两项职责：创建表面和区域。

套用规范的说法，一个 Wayland *表面* 拥有一个矩形区域，它可以显示在零个或多个输出上、呈现缓冲区、接收用户输入，并定义一个本地坐标系统。这些内容我们稍后都会逐一详细拆解，但让我们先从基础开始：获取一个表面并向其附加一个缓冲区。要获取一个表面，我们首先绑定 `wl_compositor` 全局对象。把第 5.1 节的示例加以扩展，就得到如下代码：

```
struct our_state {
    // ...
    struct wl_compositor *compositor;
    // ...
};

static void
registry_handle_global(void *data, struct wl_registry *wl_registry,
		uint32_t name, const char *interface, uint32_t version)
{
    struct our_state *state = data;
    if (strcmp(interface, wl_compositor_interface.name) == 0) {
        state->compositor = wl_registry_bind(
            wl_registry, name, &wl_compositor_interface, 4);
    }
}

int
main(int argc, char *argv[])
{
    struct our_state state = { 0 };
    // ...
    wl_registry_add_listener(registry, &registry_listener, &state);
    // ...
}
```

注意，我们在调用 `wl_registry_bind` 时指定了版本 4，这是撰写本书时的最新版本。拿到这个引用之后，我们就可以创建一个 `wl_surface` 了：

```
struct wl_surface *surface = wl_compositor_create_surface(state.compositor);
```

在能够呈现它之前，我们必须先给它附加一个像素来源：一个 `wl_buffer`。
