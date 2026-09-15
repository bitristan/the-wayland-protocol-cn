# XKB 简介

我们清单上的下一个输入设备是键盘，但在讨论它们之前，我们需要先停下来给你补充一些背景知识。键盘映射是键盘输入中一个至关重要的细节，而 XKB 是在 Wayland 上处理它们的推荐方式。

当你在键盘上按下一个键时，它会向计算机发送一个 *扫描码*（scancode），这不过是分配给该物理按键的一个数字。在我的键盘上，扫描码 1 是 Escape 键，'1' 键是扫描码 2,'a' 是 30,Shift 是 42，依此类推。我用的是美式 ANSI 键盘布局，但还有许多其他布局，它们的扫描码各不相同。在我朋友的德式键盘上，扫描码 12 产生 'ß'，而我的产生 '-'。

为了解决这个问题，我们使用一个叫做 “xkbcommon” 的库，它的名字源于它所扮演的角色：从 XKB（X KeyBoard）中提取出来、放入独立库的公共代码。XKB 定义了大量的键 *符号*（symbol），例如 XKB_KEY_A、XKB_KEY_ssharp（ß，来自德语）以及 XKB_KEY_kana_WO（を，来自日语）。

不过，识别这些键并把它们与这样的键符号对应起来，只是问题的一部分。按住 Shift 键时 'a' 可以产生 'A'，在片假名模式下 'を' 写作 'ヲ'，而严格说来 'ß' 虽然确实有大写形式，却几乎从不使用，更肯定从来不会有人去输入。像 Shift 这样的键叫做 *修饰键*（modifier），而像平假名、片假名这样的组叫做 *组*（group）。有些修饰键可以 *暂锁*（latch），比如 Caps Lock。XKB 具备处理所有这些情况的原语，并维护一个状态机，用来跟踪你的键盘正在做什么，并准确判断出用户想要输入的是哪些 *Unicode 码点*。

## 使用 XKB

那么 xkbcommon 究竟是怎么用的呢？第一步是链接到它并引入头文件 `xkbcommon/xkbcommon.h`。[^1] 大多数使用 xkbcommon 的程序都必须管理三个对象：

- xkb_context：用于配置其他 XKB 资源的句柄
- xkb_keymap：从扫描码到键符号的映射
- xkb_state：把键符号转换为 UTF-8 字符串的状态机

配置它的过程通常如下：

1. 使用 `xkb_context_new` 创建一个新的 xkb_context，除非你在做一些奇怪的事情，否则给它传 `XKB_CONTEXT_NO_FLAGS`。
2. 以字符串形式获取一个键盘映射。*
3. 使用 `xkb_keymap_new_from_string` 为该键盘映射创建一个 `xkb_keymap`。键盘映射只有一种格式，即 `XKB_KEYMAP_FORMAT_TEXT_V1`，你将把它传给 format 参数。同样，除非你在做一些奇怪的事情，否则 flags 就用 `XKB_KEYMAP_COMPILE_NO_FLAGS`。
4. 使用 `xkb_state_new` 配合你的键盘映射创建一个 xkb_state。该状态会增加键盘映射的引用计数，所以如果你自己不再需要它了，就用 `xkb_keymap_unref`。
5. 从键盘获取扫描码。*
6. 把这些扫描码喂给 `xkb_state_key_get_one_sym` 以得到键符号，喂给 `xkb_state_key_get_utf8` 以得到 UTF-8 字符串。嗒哒！

** 这些步骤将在下一节讨论。*

从代码上说，这个过程如下所示：

```c
#include <xkbcommon/xkbcommon.h> // -lxkbcommon
/* ... */

const char *keymap_str = /* ... */;

/* Create an XKB context */
struct xkb_context *context = xkb_context_new(XKB_CONTEXT_NO_FLAGS);

/* Use it to parse a keymap string */
struct xkb_keymap *keymap = xkb_keymap_new_from_string(
    xkb_context, keymap_str, XKB_KEYMAP_FORMAT_TEXT_V1,
    XKB_KEYMAP_COMPILE_NO_FLAGS);

/* Create an XKB state machine */
struct xkb_state *state = xkb_state_new(keymap);
```

然后，要处理扫描码：

```c
int scancode = /* ... */;

xkb_keysym_t sym = xkb_state_key_get_one_sym(xkb_state, scancode);
if (sym == XKB_KEY_F1) {
    /* Do the thing you do when the user presses F1 */
}

char buf[128];
xkb_state_key_get_utf8(xkb_state, scancode, buf, sizeof(buf));
printf("UTF-8 input: %s\n", buf);
```

有了这些细节，我们就准备好着手处理键盘输入了。

[^1]: xkbcommon 附带一个 pc 文件：使用 `pkgconf --cflags xkbcommon` 和 `pkgconf --libs xkbcommon`，或者你的构建系统首选的消费 pc 文件的方式。
