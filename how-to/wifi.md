# wifi

对于笔记本尤其是没有网线接口的笔记本，在使用 U 盘安装系统时，最好就顺带把
wifi 工具装上，免得后面发现没工具可用，还得通过 chroot 的方式回锅。

我通常只使用 `iw` 和 `wpa_supplicant` 两个工具。

```shell
sudo pacman -S iw wpa_supplicant

ip a                           # 查看网卡 interface，这里假设是 wlan0
ip link set wlan0 up           # 开启网卡
iw dev wlan0 scan | grep SSID  # 得到附近所有能检测到的 wifi
                               # 这个命令很有用，免得我们忘记 wifi 名
                               # 这里假设是 $SSID

wpa_passphrase $SSID > /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
# 通常，我会做 symbolic link，先 wpa_supplicant-$SSID.conf
# 再 ln -sf wpa_supplicant-$SSID.conf wpa_supplicant-wlan0.conf

# 开机自启
systemctl enable wpa_supplicant@wlan0.service

# 立即生效
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
```

以上，在 `vconsole` 或 `GUI terminal` 中都有效， 这是我推荐的方式。

`vconsole` 中还有 `wifi-menu` 命令可用，GUI 中有 `NetworkManager` 等工具可用。
但我对它们用的少，在此不提。
