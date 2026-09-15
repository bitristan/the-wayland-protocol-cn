# XDG 表面

在 xdg-shell 领域里，表面被称为 `xdg_surfaces`，这个接口带来了两类 XDG 表面——顶层窗口和弹出窗口——共有的一小部分功能。每一类 XDG 表面的语义仍然差异很大，因此它们必须通过一个额外的角色来显式指定。

`xdg_surface` 接口提供了额外的请求，用于分配更具体的弹出窗口和顶层窗口角色。一旦我们把一个对象绑定到 `xdg_wm_base` 全局对象上，就可以用 `get_xdg_surface` 请求来为一个 `wl_surface` 获取一个 `xdg_surface`。

```xml
<request name="get_xdg_surface">
  <arg name="id" type="new_id" interface="xdg_surface"/>
  <arg name="surface" type="object" interface="wl_surface"/>
</request>
```

`xdg_surface` 接口除了提供为你的表面分配更具体的顶层窗口或弹出窗口角色的请求之外，还包含了两类角色共有的一些重要功能。在我们继续介绍顶层窗口和弹出窗口专属的语义之前，先来回顾一下这些功能。

```xml
<event name="configure">
  <arg name="serial" type="uint" summary="serial of the configure event"/>
</event>

<request name="ack_configure">
  <arg name="serial" type="uint" summary="the serial from the configure event"/>
</request>
```

xdg-surface 最重要的 API 就是这一对：`configure` 和 `ack_configure`。你可能还记得，Wayland 的一个目标就是让每一帧都完美无缺。这意味着不会有一帧带着只应用了一半的状态变更被显示出来，而为了实现这一点，我们必须让这些变更在客户端和服务端之间保持同步。对于 XDG 表面而言，这对消息就是支撑这一点的机制。

我们现在只覆盖基础内容，所以就这样概括这两个事件的重要性：当来自服务端的这些事件告知你某个表面的配置（或重新配置）时，把它们应用到一份待定状态上。当一个 `configure` 事件到达时，应用待定的变更，用 `ack_configure` 确认你已完成，然后渲染并提交一个新帧。我们会在下一章中实际演示这一点，并在第 8.1 节详细解释。

```xml
<request name="set_window_geometry">
  <arg name="x" type="int"/>
  <arg name="y" type="int"/>
  <arg name="width" type="int"/>
  <arg name="height" type="int"/>
</request>
```

`set_window_geometry` 请求主要用于使用客户端侧装饰（decoration）的应用，用来区分它们的表面中哪些部分被视为窗口的一部分，哪些部分不是。最常见的情况是，它被用来把渲染在窗口背后的客户端侧投影从窗口的一部分中排除出去。合成器可能会运用这一信息来管理它自己安排窗口、与窗口交互的行为。
