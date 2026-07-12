# 安装 postmarketOS 到 Xiaomi Mi 8 (equuleus)

## 前置条件

1. **解锁 Bootloader**
   - 小米官网申请解锁：https://www.miui.com/unlock/
   - 按照官方指引完成解锁

2. **安装 Fastboot**
   - Windows: 安装 Android SDK Platform Tools
   - Linux: `sudo apt install android-sdk-platform-tools`
   - macOS: `brew install android-platform-tools`

3. **下载构建产物**
   - 在 GitHub Actions 页面下载 `pmos-equuleus-images` artifact
   - 解压得到 `boot.img`、`rootfs.img` 等文件

## 刷入步骤

### 1. 进入 Fastboot 模式

```bash
adb reboot bootloader
```

或手动操作：关机状态下，同时按住 **音量下键 + 电源键**

### 2. 刷入内核

```bash
fastboot flash boot boot.img
```

### 3. 刷入根文件系统

```bash
fastboot flash userdata rootfs.img
```

### 4. 重启设备

```bash
fastboot reboot
```

## 首次启动

### 通过 USB 网络连接

1. 设备启动后，通过 USB 连接到电脑
2. 检查网络接口：

```bash
ip addr show
```

3. 默认 IP 地址为 `172.16.42.15`，通过 SSH 连接：

```bash
ssh user@172.16.42.15
```

### 默认账号

- 用户名：`user`
- 密码：`123456`

> 建议首次登录后立即修改密码：`passwd`

## 配置网络

### 连接 WiFi

```bash
nmtui
```

或使用命令行：

```bash
nmcli dev wifi connect "SSID" password "PASSWORD"
```

## 后续配置

### 更新系统

```bash
sudo apk update
sudo apk upgrade
```

### 安装额外软件包

```bash
# 安装常用工具
sudo apk add vim htop wget curl

# 安装 SSH 服务器
sudo apk add openssh
sudo rc-update add sshd
sudo service sshd start
```

## 故障排除

### Fastboot 找不到设备

```bash
# Linux
sudo usermod -aG plugdev $USER
sudo systemctl restart udev

# Windows
安装小米 USB 驱动
```

### 无法启动

1. 确认 Bootloader 已解锁
2. 检查 boot.img 和 rootfs.img 是否完整
3. 尝试重新刷入：

```bash
fastboot erase boot
fastboot erase userdata
fastboot flash boot boot.img
fastboot flash userdata rootfs.img
```

### USB 网络连接失败

```bash
# 在电脑上配置 USB 网络
# Linux
sudo ip addr add 172.16.42.1/24 dev usb0
sudo ip link set usb0 up

# Windows
在设备管理器中配置 USB 网卡
```

## 参考链接

- [postmarketOS Wiki - Xiaomi Mi 8](https://wiki.postmarketos.org/wiki/Xiaomi_Mi_8_(SDM845))
- [pmbootstrap Documentation](https://docs.postmarketos.org/pmbootstrap/usage.html)
- [pmaports Repository](https://gitlab.postmarketos.org/postmarketOS/pmaports)