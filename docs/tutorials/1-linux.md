## 1. OS (Operating System)

### 1-1. Option

目前實驗室機器人開發的作業系統主要會在:

- **Linux**
    - ROS2 (必須在 Linux)
    - 深度學習時的 GPU Acceleration (Linux 比 Windows 好設定環境)
    - Unity (也可以在 Windows)
- **Windows**
    - Unity
    - 文書處理 (MS Word, PPT, excel etc.)
    

<aside>
💡

Linux 是一個開源作業系統, 整體架構比 Windows 透明直觀, 很適合做軟體開發.

</aside>

### 1-2. Linux & Ubuntu

Ubuntu 是基於 Linux 的發行版. (其他發行版有如 Debian, Fedora, Arch etc.)
Linux 是系統內核, 而 Ubuntu 是位於此之上層的操作系統, 

推薦使用的發行版是社群廣大的 Ubuntu.
其中 Ubuntu 有不同的版本號, 例如 20.04, 22.04, 24.04 LTS 等等.

前面的開頭是指年分, 最新版通常不太穩定,
ROS其他配套通常沒有那麼快跟上來

因此我們通常會選用兩年前的LTS版本. 

### 1-3. To use Linux

可能有人會卡在手上只有一台灌 Windows 的電腦, 沒辦法學習 Linux.

想使用 Linux 有以下方式:

- **雙系統（Dual Boot）** (最推薦, 實驗室成員必學)
    - 分割磁碟安裝系統，建議至少留 50GB 分區給 Linux OS 使用.
    - 優點： 原生 OS，效能最佳，無相容性問題.
    - 缺點： 安裝流程較繁瑣，需重啟切換系統.
- **WSL（Windows Subsystem for Linux）** (適合測試用)
    - 介紹、安裝方法請參閱 [WSL 官方網站](https://learn.microsoft.com/zh-tw/windows/wsl/).
    - 優點： 安裝簡單，快速開發.
    - 缺點： 記憶體消耗大, USB 直通、GPU 調用等可能有問題.
- **虛擬機（Virtual Machine）** (不推薦)
    - 使用傳統虛擬機如 VMWare, VirtualBox.
    - 優點： 可在 Windows 下使用近似原生的 Linux 環境.
    - 缺點： 效能低，已經被 Docker 取代.

### 1- 4. Installation

語言務必選擇 English, 未來才能從 error log 去 debug

- Problem 1: No Network, No Wi-Fi icon
    
    Linux does not support MEDIATEK Network Card in default.
    
    [How to install Realtek WLAN driver mt7921 in Ubuntu 22.04?](https://askubuntu.com/questions/1464480/how-to-install-realtek-wlan-driver-mt7921-in-ubuntu-22-04)
    
    note: run `sudo shutdown now` instead of `sudo reboot`.
    
- Probem 2. Change Kernel ( if needed )
    
    Use ubuntu tool `mainline kernels`.
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/850cf3ad-dc94-4a1d-bc3f-7cb537c9851b/a50d65f1-058f-455e-a9f1-0ec2b5ed9407/image.png)
    

- chkpt 1: 灌雙系統，進入 Ubuntu 環境. (如果筆電容量不夠, 至少要知道如何灌系統)
- chkpt2: 安裝 WSL，進入 Ubuntu 環境.

---

## 2. Basic of Linux

### 2-1. Terminal & Navigation

- 參考影片
    
    https://www.youtube.com/watch?v=-fzO7iWCSWY
    
- chkpt 1:  熟悉使用 Gnome terminal
- chkpt 2:  熟悉 file navigation

### 2-2. apt

`apt` 是 Ubuntu 最常用的工具之一，用來管理下載的資源。

- 參考文章
    
     [**Ubuntu APT 指令更新、升級、移除、安裝套件教學](https://www.tokfun.net/os/linux/install-remove-linux-software-using-apt-command/).**
    

- chkpt 1: 熟悉使用 `apt` 安裝、移除、更新package

---

## 3. Useful Tools

run `sudo apt update && sudo apt -y upgrade` first.  

### 3-1. development tools

- **SSH**: `sudo apt install ssh openssh-server`
    - SSH 教學放在後面，先安裝就好。
    
- **Terminator** : `sudo apt install terminator`.
    - chkpt 1: 學常用 hotkeys.
    - optional: customiz theme. ( ref: [terminator-themes](https://github.com/EliverLara/terminator-themes) )
        
        
- **Git** : `sudo apt install git`.
    - Git 教學放在後面，先安裝就好。

### 3-2. GPU/Docker

- **Docker**
    - Follow the `apt` installation steps from [official website](https://docs.docker.com/engine/install/ubuntu/).
    - chkpt 1: 若有 “docker daemon permission denied” 錯誤, 解決並理解原因.
        
        
- **Nvidia driver** (有 nvidia GPU 就安裝)
    - 安裝指令
        
        ```bash
        sudo apt update
        sudo ubuntu-drivers list # check the compatible driver version
        sudo apt install -y nvidia-driver-5xx #change 5xx to 535, 550, 555 …
        sudo reboot
        ```
        
    - chkpt 1: 測試指令 `nvidia-smi` , 檢查安裝成功與否.
        
        
- **Nvidia Docker toolkit** (有 nvidia GPU 就安裝)
    - Follow the `apt` installation on [official website](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).
    - chkpt 1: 運行下方指令, 若遇到問題,  解決並理解原因.
    `sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi`

<aside>
✅

還不懂 Docker 用途沒關係，教學會放在後面。但必須先解決安裝的問題。

</aside>