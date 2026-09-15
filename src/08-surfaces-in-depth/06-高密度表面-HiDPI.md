# 高密度表面（HiDPI）

在过去几年里，高端显示器的像素密度出现了巨大的飞跃，新式显示器在相同的物理面积内塞进了往年两倍之多的像素。我们把这类显示器称为“HiDPI”，即“high dots per inch”（每英寸高点数）的缩写。然而，这类显示器远远领先于它们的“LoDPI”同类，以至于必须做出应用程序层面的改变，才能恰当地利用它们。如果在相同的空间内把屏幕分辨率翻倍，而我们又不给予它们任何特殊考量，就会把所有用户界面的尺寸减半。对于大多数显示器而言，这会使文字无法阅读，并使交互元素小得令人不适。

不过作为交换，我们的矢量图形获得了大幅提升的图形保真度，尤其是在文字渲染方面。Wayland 通过为每个输出添加一个“缩放系数”（scale factor）来应对这一问题，并期望客户端将这个缩放系数应用到它们的界面上。此外，对 HiDPI 没有感知的客户端会以不作为的方式来表明这一局限，从而让合成器得以通过放大它们的缓冲区来弥补。合成器通过相应的事件通告每个输出的缩放系数：

```
<interface name="wl_output" version="3">
  <!-- ... -->
  <event name="scale" since="2">
    <arg name="factor" type="int" />
  </event>
</interface>
```

请注意，这是在第 2 版中加入的，因此在绑定 `wl_output` 全局对象时，你必须将版本设置为至少 2 才能接收到这些事件。然而，这还*不足以*决定在你的客户端中使用 HiDPI。要做出这个判断，合成器还必须为你的 `wl_surface` 发送 `enter` 事件，以表明它已经“进入”（即正显示在）某一个或某几个特定的输出上：

```
<interface name="wl_surface" version="4">
  <!-- ... -->
  <event name="enter">
    <arg name="output" type="object" interface="wl_output" />
  </event>
</interface>
```

一旦你知道了客户端所显示于其上的输出的集合，就应该取缩放系数中的最大值，把缓冲区的尺寸（以像素为单位）乘以这个值，然后以 2x 或 3x（或 Nx）的缩放来渲染用户界面。随后，像这样表明缓冲区是以何种缩放来准备的：

```
<interface name="wl_surface" version="4">
  <!-- ... -->
  <request name="set_buffer_scale" since="3">
    <arg name="scale" type="int" />
  </request>
</interface>
```

**注意**：这要求 `wl_surface` 的版本为 3 或更新。这也是你在绑定 `wl_compositor` 时应当传给 `wl_registry` 的版本号。

在下一次 `wl_surface.commit` 时，你的表面将采用这个缩放系数。如果它大于该表面所显示于其上的某个输出的缩放系数，合成器会将它缩小。如果它小于某个输出的缩放系数，合成器则会将它放大。
