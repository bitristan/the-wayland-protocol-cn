# Wayland display

到目前为止，在我们讲解 Wayland 协议如何在客户端与服务端之间对对象进行共同所有权管理时，一直遗漏了一个关键细节：这些对象最初究竟是如何被创建的。Wayland display，即 `wl_display`，隐式地存在于每一条 Wayland 连接之上。它拥有如下接口：

```xml
<interface name="wl_display" version="1">
  <request name="sync">
    <arg name="callback" type="new_id" interface="wl_callback"
       summary="callback object for the sync request"/>
  </request>

  <request name="get_registry">
    <arg name="registry" type="new_id" interface="wl_registry"
      summary="global registry object"/>
  </request>

  <event name="error">
    <arg name="object_id" type="object" summary="object where the error occurred"/>
    <arg name="code" type="uint" summary="error code"/>
    <arg name="message" type="string" summary="error description"/>
  </event>

  <enum name="error">
    <entry name="invalid_object" value="0" />
    <entry name="invalid_method" value="1" />
    <entry name="no_memory" value="2" />
    <entry name="implementation" value="3" />
  </enum>

  <event name="delete_id">
    <arg name="id" type="uint" summary="deleted object ID"/>
  </event>
</interface>
```

对一般 Wayland 用户而言，其中最有趣的是 `get_registry`，我们将在下一章详细讨论它。简而言之，注册表用于分配其他对象。接口的其余部分则用于对连接做日常维护，除非你打算编写自己的 libwayland 替代品，否则通常并不重要。

相反，本章将聚焦于 libwayland 与 `wl_display` 对象相关联的一组函数，它们用于建立并维护你的 Wayland 连接。这些函数用于操作 libwayland 的内部状态，而与线上协议的请求和事件没有直接关系。

我们将从这些函数中最重要的一项开始：建立 display。对客户端而言，这会涵盖连接服务端的实际过程；对服务端而言，则是配置一个供客户端连接的 display 的过程。
