# Jetson AGX Thor USB 開機安裝與 VNC 開啟教學

這份文件整理 NVIDIA Jetson AGX Thor Developer Kit 的兩個常用設定：

1. 使用 USB 隨身碟開機並安裝 Jetson BSP / Jetson Linux
2. 開啟 VNC，從另一台電腦遠端操作 Thor 桌面

參考來源：

- NVIDIA Jetson AGX Thor Quick Start Guide  
  https://docs.nvidia.com/jetson/agx-thor-devkit/user-guide/latest/quick_start.html
- NVIDIA Jetson VNC Setup  
  https://developer.nvidia.com/embedded/learn/tutorials/vnc-setup

## 1. 使用 USB 隨身碟開機安裝 Jetson Linux

Jetson AGX Thor 從 JetPack 7 開始支援使用 bootable installation USB stick 安裝 Jetson BSP / Jetson Linux。這個流程類似在一般電腦上用 Ubuntu USB 安裝系統，但要注意：這不是 Live USB，不能用來直接試跑完整 Jetson Linux。

USB 安裝器的用途是：

- 從 USB 隨身碟開機
- 啟動 Jetson BSP 安裝程式
- 將 Jetson Linux 安裝到 Thor 的 NVMe
- 安裝完成後，從 NVMe 開機進入系統

## 2. 需要準備的東西

- 一台 Windows / macOS / Linux 電腦
- 至少 25 GB 可用空間
- USB 隨身碟，建議 16 GB 以上
- Jetson AGX Thor Developer Kit
- Thor 原廠 USB-C 電源供應器
- 螢幕、鍵盤、滑鼠，或使用 headless serial console

## 3. 下載 Jetson ISO

到 NVIDIA JetPack 下載頁面，選擇 Jetson AGX Thor Developer Kit 對應版本，下載 Jetson ISO。

NVIDIA 官方 Quick Start 也提供 direct download link，但實際使用時建議以 JetPack Download Page 上最新版本為準。

## 4. 製作可開機 USB 隨身碟

不能直接把 ISO 檔案複製到 USB 隨身碟。必須用映像檔燒錄工具建立 bootable USB。

官方教學使用 Balena Etcher：

https://etcher.balena.io/

操作流程：

1. 開啟 Balena Etcher
2. 選擇下載好的 Jetson ISO
3. 選擇 USB 隨身碟
4. Flash
5. 等待寫入與驗證完成

## 5. 接線與開機

### 有螢幕模式

1. 將螢幕接到 Thor 的 HDMI 或 DisplayPort
2. 接上 USB 鍵盤與滑鼠
3. 插入剛製作好的 bootable USB 隨身碟
4. 接上原廠 USB-C 電源
5. 按下 Thor 電源鍵開機

官方建議使用隨附的 USB-C 電源供應器，以避免供電不穩。

### Headless 模式

如果沒有螢幕，可以透過 Debug-USB serial console 安裝。

做法是用 USB 線將電腦連到 Thor 的 Debug-USB port，然後在電腦上用 terminal program 開啟 serial console。

注意：某些 UEFI firmware 版本在 headless terminal 顯示上可能有已知問題，若畫面異常，請參考官方文件中的 headless workaround。

## 6. 從 USB 開機

Thor 預設通常會在插入 USB 隨身碟時優先從 USB 開機，所以大多數情況不需要手動改 boot order。

如果沒有從 USB 開機：

1. 開機時在 NVIDIA logo / pre-boot options 畫面連按 `Esc`
2. 進入 UEFI setup menu
3. 選擇 `Boot Manager`
4. 選擇 USB 隨身碟
5. Save & Exit
6. 讓 Thor 從 USB 開機

## 7. 安裝 BSP 到 NVMe

從 USB 開機後，會進入 Jetson BSP installation GRUB menu。

建議選擇：

```text
Install on NVMe
```

安裝開始後，畫面會出現安裝訊息。官方文件提到安裝大約需要 10 分鐘左右。

如果安裝過程中出現 QSPI capsule update 提示，請按：

```text
Y
```

這個更新是為了讓裝置 firmware 與目前 Jetson ISO 相容，不建議跳過。

## 8. 安裝完成後拔掉 USB

安裝完成後 Thor 會重新開機。

重要：從 Jetson ISO r38.4.0 開始，UEFI boot order 可能仍然會把 USB 放在最前面。因此安裝完成後務必拔掉 USB 隨身碟，否則可能又重新進入 USB 安裝器。

拔掉 USB 後，讓 Thor 從 NVMe 開機。

## 9. 第一次開機設定

第一次從 NVMe 進入 Jetson Linux 時，會進入 `oem-config` 初始設定。

需要設定：

- 語言
- 鍵盤
- NVIDIA license
- 網路
- 時區
- 使用者名稱
- 密碼
- hostname

完成後即可登入 Jetson Linux。

## 10. 開啟 VNC

VNC 可以讓同一個網路內的另一台電腦遠端操作 Thor 桌面。

注意：

- Thor 和用來連線的電腦必須在同一個網路
- 網路速度會影響遠端桌面流暢度
- NVIDIA 官方 VNC 教學使用 GNOME 的 Vino server
- VNC server 通常需要使用者已在 Thor 本機桌面登入後才會啟動

## 11. 啟用 VNC Server

在 Thor 上執行：

```bash
cd /usr/lib/systemd/user/graphical-session.target.wants
sudo ln -s ../vino-server.service ./.
```

設定 VNC 不跳出本機確認、不要求加密：

```bash
gsettings set org.gnome.Vino prompt-enabled false
gsettings set org.gnome.Vino require-encryption false
```

設定 VNC 密碼。將 `your_password` 換成你要使用的 VNC 密碼：

```bash
gsettings set org.gnome.Vino authentication-methods "['vnc']"
gsettings set org.gnome.Vino vnc-password $(echo -n 'your_password' | base64)
```

重新開機：

```bash
sudo reboot
```

## 12. 查詢 Thor IP

重新開機並登入桌面後，查詢 Thor 的 IP：

```bash
ip addr
```

常見介面：

- Ethernet：`eth0` 或類似名稱
- Wi-Fi：`wlan0` 或類似名稱
- USB device mode Ethernet：`l4tbr0`

也可以用：

```bash
hostname -I
```

假設查到 IP 是：

```text
10.0.1.61
```

## 13. 從 Windows 連線 VNC

在 Windows 安裝 VNC Viewer，例如 RealVNC Viewer：

https://www.realvnc.com/en/connect/download/viewer/

開啟 VNC Viewer 後輸入：

```text
10.0.1.61
```

如果有設定密碼，輸入前面設定的 VNC password。

## 14. 從 Linux 連線 VNC

可以使用 `gvncviewer`：

```bash
sudo apt update
sudo apt install -y gvncviewer
gvncviewer
```

開啟後輸入 Thor 的 IP。

也可以使用 Remmina。

## 15. 從 macOS 連線 VNC

macOS 內建 Screen Sharing。

開啟 Finder，選擇：

```text
Go -> Go to Folder
```

輸入：

```text
/System/Library/CoreServices/Applications
```

開啟 `Screen Sharing`，輸入 Thor IP 連線。

## 16. 常見問題

### VNC 連不上

先確認 Thor 和電腦是否在同一個網路：

```bash
ping 10.0.1.61
```

確認 Thor 的 VNC port 是否有開：

```bash
ss -tulpn | grep 5900
```

### VNC server 沒有自動啟動

官方教學提醒：VNC server 通常只會在使用者登入 Jetson 本機桌面後可用。

如果希望開機後自動可用，需要在 Thor 桌面系統設定中啟用 automatic login。

### 找不到 `vino-server.service`

某些新版桌面環境可能沒有預裝 Vino。可以先查：

```bash
ls /usr/lib/systemd/user/vino-server.service
```

如果不存在，可考慮安裝或改用其他 VNC server，例如 `x11vnc` 或 TigerVNC。若只是要分享實體桌面，常見替代方案是：

```bash
sudo apt update
sudo apt install -y x11vnc
x11vnc -storepasswd
x11vnc -display :0 -auth guess -forever -loop -noxdamage -shared -rfbport 5900
```

然後從 VNC Viewer 連：

```text
<Thor_IP>:5900
```

## 17. 建議流程總結

第一次安裝 Thor：

1. 下載 Jetson ISO
2. 用 Etcher 燒錄到 USB 隨身碟
3. Thor 插入 USB 後開機
4. 選擇 `Install on NVMe`
5. 如有 QSPI capsule update，按 `Y`
6. 安裝完成後拔掉 USB
7. 從 NVMe 開機並完成 `oem-config`
8. 登入桌面
9. 設定 VNC
10. 從 Windows / macOS / Linux 用 VNC Viewer 連線
