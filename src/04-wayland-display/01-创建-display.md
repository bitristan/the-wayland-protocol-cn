# 创建 display

打开你的文本编辑器——是时候写下我们的第一行代码了。

## 面向 Wayland 客户端

连接到一个 Wayland 服务端，并创建一个 `wl_display` 来管理该连接的状态，是相当容易的：

```c
#include <stdio.h>
#include <wayland-client.h>

int
main(int argc, char *argv[])
{
    struct wl_display *display = wl_display_connect(NULL);
    if (!display) {
        fprintf(stderr, "Failed to connect to Wayland display.\n");
        return 1;
    }
    fprintf(stderr, "Connection established!\n");

    wl_display_disconnect(display);
    return 0;
}
```

让我们编译并运行这个程序。假设你阅读本书时正使用某个 Wayland 合成器，结果应当如下所示：

```sh
$ cc -o client client.c -lwayland-client
$ ./client
Connection established!
```

`wl_display_connect` 是客户端建立 Wayland 连接最常见的方式。其签名为：

```c
struct wl_display *wl_display_connect(const char *name);
```

“name” 参数是 Wayland display 的名称，通常是 `"wayland-0"`。你可以在我们的测试客户端中把那个 `NULL` 换成它，亲自试一试——它很可能会奏效。它对应的是 `$XDG_RUNTIME_DIR` 中某个 Unix 套接字的名称。不过更推荐使用 `NULL`，此时 libwayland 会：

1. 如果设置了 `$WAYLAND_DISPLAY`，则尝试连接 `$XDG_RUNTIME_DIR/$WAYLAND_DISPLAY`
2. 否则，尝试连接 `$XDG_RUNTIME_DIR/wayland-0`
3. 否则，失败 :(

这样一来，用户就可以通过把 `$WAYLAND_DISPLAY` 设置为想要的 display，来指定他们想让客户端运行在哪个 Wayland display 上。如果你有更复杂的需求，也可以自行建立连接，并从某个文件描述符创建 Wayland display：

```c
struct wl_display *wl_display_connect_to_fd(int fd);
```

无论你以何种方式创建 display，也都可以通过 `wl_display_get_fd` 获取该 `wl_display` 正在使用的文件描述符。

```c
int wl_display_get_fd(struct wl_display *display);
```

## 面向 Wayland 服务端

对服务端而言，过程也相当简单。display 的创建与套接字的绑定是分开进行的，以便你有时间在任何客户端能够连接到它之前对 display 进行配置。下面是另一个最小示例程序：

```c
#include <stdio.h>
#include <wayland-server.h>

int
main(int argc, char *argv[])
{
    struct wl_display *display = wl_display_create();
    if (!display) {
        fprintf(stderr, "Unable to create Wayland display.\n");
        return 1;
    }

    const char *socket = wl_display_add_socket_auto(display);
    if (!socket) {
        fprintf(stderr, "Unable to add socket to Wayland display.\n");
        return 1;
    }

    fprintf(stderr, "Running Wayland display on %s\n", socket);
    wl_display_run(display);

    wl_display_destroy(display);
    return 0;
}
```

让我们也把它编译并运行一下：

```sh
$ cc -o server server.c -lwayland-server
$ ./server &
Running Wayland display on wayland-1
$ WAYLAND_DISPLAY=wayland-1 ./client
Connection established!
```

使用 `wl_display_add_socket_auto` 会让 libwayland 自动决定 display 的名称，其默认为 `wayland-0`，或者 `wayland-$n`，具体取决于 `$XDG_RUNTIME_DIR` 中是否已有其他 Wayland 合成器的套接字。不过，和客户端一样，你在配置 display 时还有一些其他选择：

```c
int wl_display_add_socket(struct wl_display *display, const char *name);

int wl_display_add_socket_fd(struct wl_display *display, int sock_fd);
```

添加套接字之后，调用 `wl_display_run` 会运行 libwayland 内部的事件循环，并阻塞直到 `wl_display_terminate` 被调用。这个事件循环是什么？翻到下一页便知！
