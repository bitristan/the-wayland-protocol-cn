# seat（座位）：处理输入

将应用程序显示给用户只是 I/O 等式的一半——大多数应用程序还需要处理输入。为此，seat 提供了一种对 Wayland 上输入事件的抽象。用哲学的话说，Wayland seat 指的是用户坐下来操作计算机的一个“座位”，它最多关联一个键盘和至多一个“指针”设备（即鼠标或触摸板）。触摸屏、绘图板设备等也定义了类似的关系。

务必记住，这只是一层*抽象*:Wayland 显示上呈现的 seat 未必与现实一一对应。在实践中，一个 Wayland 会话上很少会出现多于一个 seat 可用的情况。如果你给计算机插上第二个键盘，它通常会被分配到与第一个相同的 seat，键盘布局等等会随着你开始在其中一个上打字而动态切换。这些实现细节留给 Wayland 合成器去斟酌。

从客户端的角度来看，这件事相当直接。如果你绑定到 `wl_seat` 全局对象，就能访问以下接口：

```xml
<interface name="wl_seat" version="7">
  <enum name="capability" bitfield="true">
    <entry name="pointer" value="1" />
    <entry name="keyboard" value="2" />
    <entry name="touch" value="4" />
  </enum>

  <event name="capabilities">
    <arg name="capabilities" type="uint" enum="capability" />
  </event>

  <event name="name" since="2">
    <arg name="name" type="string" />
  </event>

  <request name="get_pointer">
    <arg name="id" type="new_id" interface="wl_pointer" />
  </request>

  <request name="get_keyboard">
    <arg name="id" type="new_id" interface="wl_keyboard" />
  </request>

  <request name="get_touch">
    <arg name="id" type="new_id" interface="wl_touch" />
  </request>

  <request name="release" type="destructor" since="5" />
</interface>
```

**注意**：这个接口已经更新过很多次——绑定全局对象时要注意版本。本书假定你绑定的是最新版本，在撰写本书时即版本 7。

这个接口相对简单直接。服务端会向客户端发送一个 `capabilities` 事件，以表明该 seat 支持哪些类型的输入设备——用 `capability` 值的位域表示——客户端据此可以绑定它想要使用的输入设备。例如，如果服务端发送的 `capabilities` 满足 `(caps & WL_SEAT_CAPABILITY_KEYBOARD) > 0`，客户端便可使用 `get_keyboard` 请求为该 seat 获取一个 `wl_keyboard` 对象。每种具体输入设备的语义将在后续章节中介绍。

在进入那些内容之前，我们先介绍一些通用的语义。

## 事件序列号

Wayland 客户端可能执行的某些操作，需要借助一种轻量形式的认证，即以输入事件序列号的形式。例如，打开弹出窗口的客户端（右键唤出的上下文菜单就是一种弹出窗口）可能希望在服务端“抓取”来自受影响 seat 的所有输入事件，直到该弹出窗口被关闭。为防止这一特性被滥用，服务端可以给它发送的每个输入事件分配一个序列号，并要求客户端在请求中包含其中一个序列号。

当服务端收到这样的请求时，它会查找与该序列号关联的输入事件并做出判断。如果该事件发生得太久远、或属于错误的表面、或不是正确的类型——例如，它可以拒绝在晃动鼠标时发起的抓取，却允许在点击时发起的抓取——它就可以拒绝该请求。

从服务端的角度看，它只需在发送每个输入事件时附带一个递增的整数，并记录下对特定用例而言被视为有效的序列号，以供日后校验。客户端从自己的输入事件处理器中接收到这些序列号，只需立即把它们回传，便能执行所需的操作。

我们将在后面的章节中更详细地讨论这些内容，那时我们会开始介绍需要输入事件序列号来校验的具体请求。

## 输入帧

出于实际原因，来自输入设备的单个输入事件可能被拆分成若干个 Wayland 事件。例如，当你使用滚轮时，`wl_pointer` 会发出一个 `axis` 事件，但它会另外发出一个事件，告诉你这是*哪种*轴：滚轮、触摸板上的手指、把滚轮向侧面拨动，等等。若用户在足够短的时间内同时做了这些动作，来自输入源的同一个输入事件也可能包含一些鼠标移动或按键操作。

这些相关事件在语义上的分组方式因输入类型而略有差异，但 `frame` 事件通常是它们共有的。简而言之，如果你把从某个设备收到的所有输入事件缓冲起来，然后等待 `frame` 事件表明你已收到单个输入“帧”的全部事件，你就能把这批缓冲的 *Wayland* 事件解读为一个单独的*输入*事件，然后重置缓冲区，开始收集下一帧的事件。

如果这听起来太复杂，不必担心。许多应用程序无需操心输入帧。只有当你开始进行更复杂的输入事件处理时，才需要关注这一点。

## 释放设备

当你用完某个设备后，每个接口都有一个 `release` 请求可供你清理它。它大致如下所示：

```xml
<request name="release" type="destructor" />
```

足够简单。
