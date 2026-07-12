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

### USB 网络连接说明

postmarketOS 默认会在设备端创建 USB 网络接口，使用固定 IP 地址 `172.16.42.1/24`，并运行 DHCP 服务器，为电脑分配 `172.16.42.2/24`。

**设备端固定 IP**: `172.16.42.1`
**电脑端获得 IP**: `172.16.42.2`（由设备的 DHCP 服务器分配）

### Windows 系统连接步骤

#### 1. 安装 USB 网络驱动

Windows 11 自带 NCM 驱动，通常可以自动识别。

Windows 10 需要安装 RNDIS 驱动：

1. 设备启动后，用 USB 连接到电脑
2. 打开 **设备管理器**
3. 找到带黄色叹号的设备（可能显示为"未知设备"或"ADB Interface"）
4. 右键选择 **更新驱动程序**
5. 选择 **浏览我的计算机以查找驱动程序软件**
6. 选择 **让我从计算机上的可用驱动程序列表中选取**
7. 选择 **网络适配器**
8. 厂商选择 **Microsoft**
9. 型号选择 **Remote NDIS compatible device**（Windows 10 1709+ 可能显示为 "Remote NDIS based Internet Sharing Device"）
10. 等待驱动安装完成

#### 2. 配置网络适配器

1. 打开 **控制面板 > 网络和共享中心 > 更改适配器设置**
2. 找到新出现的网络适配器（通常命名为"以太网 X"）
3. 右键选择 **属性**
4. 双击 **Internet 协议版本 4 (TCP/IPv4)**
5. 选择 **使用下面的 IP 地址**：
   - IP 地址：`172.16.42.2`
   - 子网掩码：`255.255.255.0`
6. 点击 **确定**

#### 3. 使用 SSH 连接

Windows 10/11 内置 OpenSSH 客户端，直接在 PowerShell 或 CMD 中使用：

```powershell
ssh user@172.16.42.1
```

首次连接会提示确认主机密钥，输入 `yes` 确认。

### Linux/macOS 系统连接步骤

Linux 和 macOS 通常可以自动识别 USB 网络接口并获取 IP。

```bash
ssh user@172.16.42.1
```

### 默认账号

- 用户名：`user`
- 密码：`147147`

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
sudo apk add vim htop wget curl openssh
sudo rc-update add sshd
sudo service sshd start
```

### 配置 SSH 免密登录（推荐）

在电脑上生成 SSH 密钥：

```bash
ssh-keygen -t ed25519
```

将公钥复制到设备：

```bash
ssh-copy-id user@172.16.42.1
```

## 故障排除

### Windows 驱动安装失败

1. 如果设备显示为 ADB Interface，先卸载该驱动
2. 在设备管理器中选择"Composite USB Device"
3. 然后重新安装 RNDIS 驱动

### USB 网络连接失败

**Windows**：
1. 确认驱动已正确安装
2. 确认网络适配器 IP 设置为 `172.16.42.2`
3. 尝试重启网络适配器

**Linux**：
```bash
sudo dhcpcd usb0 --nohook ipv6
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

### SSH 连接被拒绝

1. 确认设备已完全启动
2. 确认 USB 网络已连接
3. 在设备上检查 SSH 服务状态：

```bash
sudo service sshd status
```

## 参考链接

- [postmarketOS Wiki - Xiaomi Mi 8](https://wiki.postmarketos.org/wiki/Xiaomi_Mi_8_(SDM845))
- [postmarketOS Wiki - USB Network](https://wiki.postmarketos.org/wiki/USB_Network)
- [postmarketOS Wiki - Windows FAQ](https://wiki.postmarketos.org/wiki/Windows_FAQ)
- [pmbootstrap Documentation](https://docs.postmarketos.org/pmbootstrap/usage.html)
- [pmaports Repository](https://gitlab.postmarketos.org/postmarketOS/pmaports)