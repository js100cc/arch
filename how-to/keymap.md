# keymap

键盘映射，也就是改键，是为了方便我们的键盘操作。

## 确定改键需求

每个人的键盘使用习惯不一样，有的人习惯将 `Caps_Lock` 改为 `Esc` 或 `Control`，
有的人甚至将整个键盘布局完全改掉。因此，不同需求的人采取的策略可能不一样。

我的基本需求是，不论是在 `virtual console` 还是在 `GUI terminal` 中都要实现：

1. `Caps_Lock` 改为 `Control`
2. `virtual console` 中，右 `Alt` 跟 `GUI terminal` 效果一致
3. `GUI terminal` 中，在键盘硬件没有右 `Win` 键的情况下模拟右 `Win` 键

这些需求的目的在于：

1. 我使用 `Vim` 也使用 `Readline`，`Caps_Lock` 改为 `Control` 最经济
2. `virtual console` 中，右 `Alt` 键跟 `GUI terminal` 的表现不同，也要改
3. 我使用 `i3wm`，`Win` 键是 `Mod` 键，有时需要有右 `Win` 键，尽管用的不多

## Virtual Console

参考 `man`，`keymaps`，`showkey`，`loadkeys`，`vconsole.conf` 等命令。

```shell
sudo mkdir -p /usr/local/share/kbd/keymaps
sudo vim /usr/local/share/kbd/keymaps/keys.map
# 下面是其内容
keymaps 0-255
keycode 58 = Control
keycode 100 = Alt
# keycode 通过 showkey 交互得到，以交互得到的数值为准

# 重启后，永久生效，vim /etc/vconsole.conf
KEYMAP=/usr/local/share/kbd/keymaps/keys.map

# 不用重启，临时生效
sudo loadkeys /usr/local/share/kbd/keymaps/keys.map

# 注意，keys.map 存放的路径和命名可以是任意的，这里只是一种惯例和个人习惯
```

## GUI terminal

`virtual console` 和 `GUI terminal` 其实都是 `virtual terminal`，只不过
`vconsole` 由 kernel 提供，而我们通常用到的是图形界面的 terminal 窗口程序。

Terminal 的改键方式更多样化，使用不同的 DE (Desktop Environment)可能采取不同的
方式，比如有些 DE 直接提供了 GUI 工具改键。我使用的 `i3wm`，它基于 `xorg`，
提供了 `xev`，`setxkbmap`，`xmodmap` 等工具用来改键。

这些工具的选项和规则很多，具体可以参考 `man`，这里我列出我的改键方式。

```shell
# ~/.xinitrc 添加
[[ -f ~/.xprofile ]] && . ~/.xprofile
[[ -f ~/.Xmodmap ]] && . ~/.Xmodmap

# ~/.xprofile 添加
setxkbmap -option ctrl:nocaps

# ~/.Xmodmap 添加
keycode 135 = Super_R Super_R
```

我觉得这种方式更通用，有些 DE 直接支持，有些 DE 或 distro 可以参考文档设置调用
上述文件。

`setxkbmap` 的规则用来改 `Caps_Lock` 更方便，查看 `man setxkbmap` 和
`ls /usr/share/X11/xkb/rules` 中的更多规则。但它似乎没有提供改 `Menu` 键为
`Super_R` 的规则，这时候我选择用 `xmodmap` 来改键，它更通用，但比较繁琐，好在
经过尝试，我的这个需求只需要一行配置。

## 输入延时和频率

对于我来讲，经常使用到 vim 和 CLI，xorg 默认 input delay 和 rate 太慢（其它
OS 也一样）。

我的做法是使用 `xorg` 提供的 `xset`

```shell
# vim ~/.xprofile
xset r rate 200 30

# 具体数值可以根据自己习惯调整，这里 200 表示 200ms delay， 30 是 30HZ rate
```

对于 `virtual console`，至少在 Arch Linux 的表现是不错的，因而不需要设置。

## 其它 OS 和 DE

Windows 有第三方的工具，比如 `sharpkeys` 可以改键，有内置的工具改 delay 和
rate。

MacOS 有内置的工具改 `Caps_Lock` 和 `input delay/rate`。

GNOME 有 `tweak-tools` 之类的工具。

KDE 我用的少，但根据其无所不用其极的赶超 Windows 的图形可配置的理念，想来应该
直接原生的支持这些设置功能。

## 改键的优缺点

Linux 的一大特点是高度可配置，改键就是其中一项。

优点如前所述，方便键盘操作，缺点也如前所述，每个人的习惯是不一样的，一台电脑
经过一个人的改动之后其他人可能完全用不了。因此不要在公用电脑上进行比如改键这样
的改动。

但对于个人电脑或个人专用电脑，我的建议是怎么高效怎么来，或者我们认为是那样的就行。

## Reference

[Linux console/Keyboard configuration](https://wiki.archlinux.org/index.php/Linux_console/Keyboard_configuration)
