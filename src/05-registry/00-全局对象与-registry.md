# 全局对象与注册表

如果你还记得第 2.1 节的内容，每个请求和事件都与一个对象 ID 相关联，但到目前为止我们还没有讨论过对象是如何创建的。当我们收到一条 Wayland 消息时，我们必须知道该对象 ID 代表的是哪个接口，才能对其进行解码。我们还必须以某种方式协商可用对象、新对象的创建，以及为它们分配 ID。在 Wayland 中，我们一次性解决了这两个问题——当我们*绑定*一个对象 ID 时，我们就此约定在所有未来的消息中用于该对象的接口，并在本地状态中保存一份对象 ID 到接口的映射。

为了给这一切做引导，服务端提供了一份*全局对象*的列表。这些全局对象本身往往就以其自身的价值提供信息和功能，但最常见的情况是，它们被用来代理其他对象，以满足各种用途，例如创建应用程序窗口。这些全局对象本身也有自己的对象 ID 和接口，我们必须以某种方式分配并约定它们。

既然现在你脑海中无疑已经浮现出先有鸡还是先有蛋的问题，那我就来揭示这个秘密诀窍：当你建立连接时，对象 ID 1 已经被隐式地分配给了 `wl_display` 接口。回顾一下该接口，请注意其中的 `wl_display::get_registry` 请求：

```xml
<interface name="wl_display" version="1">
  <request name="sync">
    <arg name="callback" type="new_id" interface="wl_callback" />
  </request>

  <request name="get_registry">
    <arg name="registry" type="new_id" interface="wl_registry" />
  </request>

  <!-- ... -->
</interface>
```

`wl_display::get_registry` 请求可用于将一个对象 ID 绑定到 `wl_registry` 接口，它是 `wayland.xml` 中接着出现的下一个接口。鉴于 `wl_display` 始终拥有对象 ID 1，下面这条线上消息应该就说得通了（采用大端序）：

```
C->S    00000001 000C0001 00000002            .... .... ....
```

当我们把它拆解开来时，第一个数字是对象 ID。第二个数字的最高 16 位是消息的总长度（以字节为单位），最低位则是请求的操作码。其余的字（这里只有一个）是参数。简而言之，这在对象 ID 1（`wl_display`）上调用请求 1（从 0 开始计数），该请求接受一个参数：一个新生成的、供新对象使用的 ID。请注意 XML 文档中，这个新 ID 已被预先定义为受 `wl_registry` 接口支配：

```xml
<interface name="wl_registry" version="1">
  <request name="bind">
    <arg name="name" type="uint" />
    <arg name="id" type="new_id" />
  </request>

  <event name="global">
    <arg name="name" type="uint" />
    <arg name="interface" type="string" />
    <arg name="version" type="uint" />
  </event>

  <event name="global_remove">
    <arg name="name" type="uint" />
  </event>
</interface>
```

我们将在接下来的章节中讨论的正是这个接口。
