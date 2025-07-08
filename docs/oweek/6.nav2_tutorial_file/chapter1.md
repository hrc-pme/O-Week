# 第一章　安裝與環境準備

> 本章教你在 Ubuntu 22.04 LTS 上，從零安裝 ROS 2 Humble、Nav2、TurtleBot3 及 Gazebo Classic 11，並跑通第一個導航範例。後續章節都以此環境為基礎，請一步一步完成。
> 

---

### 1-1　為何選 Gazebo Classic

- **官方支援最完整**：TurtleBot3 的模擬範例及多數社群教學仍以 Gazebo Classic（Gazebo 11）為主。
- **直接整合 gazebo_ros_pkgs**：Classic 版以 `gazebo_ros_pkgs` 提供 ROS2 Plugin，不需另行橋接層。[classic.gazebosim.org](https://classic.gazebosim.org/tutorials?tut=ros2_installing&utm_source=chatgpt.com)
- **安裝維護簡單**：Ubuntu 22.04 內建 `gazebo11` 套件，無相依衝突問題。[classic.gazebosim.org](https://classic.gazebosim.org/tutorials?tut=install_ubuntu&utm_source=chatgpt.com)

---

### 1-2　系統需求與建議硬體

| 項目 | 建議規格 | 說明 |
| --- | --- | --- |
| **OS** | Ubuntu 22.04 LTS Desktop 64-bit | Server 版需自行裝桌面套件才能跑 RViz2 / Gazebo。 |
| **CPU** | ≥4 Cores（建議 8 Threads↑） | Gazebo + Nav2 多執行緒，核心越多模擬越順暢。 |
| **GPU** | NVIDIA GTX 1050 Ti↑ | Gazebo 11 使用 OGRE；獨顯能減少渲染延遲。 |
| **RAM** | ≥8 GB | 4 GB 可跑，但同時開 RViz2 可能吃掉 Swap。 |
| **磁碟** | 20 GB 可用空間 | ROS2 + Gazebo + 工作空間 ≈8 GB。 |

---

### 1-3　安裝 ROS 2 **Humble** Desktop-Full

```bash
bash
複製編輯
# 1) 加入 ROS2 APT 倉庫
sudo apt update && sudo apt install -y curl gnupg lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) \
signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" \
| sudo tee /etc/apt/sources.list.d/ros2.list

# 2) 安裝 Desktop-Full（含 RViz2、rqt、rosbag、gazebo_ros_pkgs）
sudo apt update
sudo apt install -y ros-humble-desktop-full       # 約 4 GB

# 3) 啟用環境變數
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc

```

Desktop-Full 會自動拉入 `gazebo11`、`gazebo_ros_pkgs` 及其 ROS Plugin，相容性最佳。[docs.ros.org](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html?utm_source=chatgpt.com)

---

### 1-4　補裝 Nav2（若你用最小安裝）

```bash
bash
複製編輯
sudo apt install -y ros-humble-navigation2 \
                    ros-humble-nav2-bringup \
                    ros-humble-nav2-simple-commander

```

該元件群為 Nav2 1.x，含行為樹、Recoveries、Costmap2D、Map Server 等完整功能。[docs.nav2.org](https://docs.nav2.org/getting_started/index.html?utm_source=chatgpt.com)

---

### 1-5　安裝 **Gazebo Classic 11**（如 Desktop-Full 未自帶）

```bash
bash
複製編輯
sudo apt install -y gazebo11 libgazebo11-dev       # Gazebo 本體
sudo apt install -y ros-humble-gazebo-ros-pkgs     # ROS2 Plugin
gazebo                                              # 測試，應開啟空白視窗

```

[classic.gazebosim.org](https://classic.gazebosim.org/tutorials?tut=install_ubuntu&utm_source=chatgpt.com)[classic.gazebosim.org](https://classic.gazebosim.org/tutorials?tut=ros2_installing&utm_source=chatgpt.com)

---

### 1-6　建立工作空間並安裝 **TurtleBot3**（建議源碼）

```bash
bash
複製編輯
# 1) 建立 overlay
mkdir -p ~/turtlebot3_ws/src
cd ~/turtlebot3_ws/src

# 2) Clone 官方 humble 分支
git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3.git
git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git

# 3) 解析相依與編譯
cd ~/turtlebot3_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install

# 4) 載入 overlay
echo "source ~/turtlebot3_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc

```

自編譯可讓你後續自由修改 URDF、控制器或加入自訂插件。[emanual.robotis.com](https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/?utm_source=chatgpt.com)[github.com](https://github.com/ros-simulation/gazebo_ros_pkgs?utm_source=chatgpt.com)

---

### 1-7　設定常用環境變數

將下列行寫入 `~/.bashrc`（示例使用 **burger** 模型，Domain ID = 0）：

```bash
bash
複製編輯
export TURTLEBOT3_MODEL=burger
export ROS_DOMAIN_ID=0
export GAZEBO_MODEL_PATH=$HOME/turtlebot3_ws/src/turtlebot3_simulations/turtlebot3_gazebo/models:$GAZEBO_MODEL_PATH

```

---

### 1-8　健康檢查

| 檢查 | 指令 | 應見結果 |
| --- | --- | --- |
| Gazebo | `gazebo` | 空場景視窗，無錯誤。 |
| gazebo_ros_plugin | `ros2 run gazebo_ros gazebo_ros_api_plugin` | 如果 Gazebo 已開啟，終端可顯示 plugin 連接成功。 |
| ROS 節點圖 | `rqt_graph` | 顯示 `/parameter_events`、`/rosout` 等系統節點。 |

---

### 1-9　Smoke Test：空地圖導航

1. **啟動 Gazebo 空世界**
    
    ```bash
    bash
    複製編輯
    export TURTLEBOT3_MODEL=burger
    ros2 launch turtlebot3_gazebo empty_world.launch.py
    
    ```
    
2. **啟動 Nav2**
    
    ```bash
    bash
    複製編輯
    # 新終端機
    ros2 launch nav2_bringup tb3_simulation_launch.py \
         headless:=False use_sim_time:=True autostart:=True
    
    ```
    
3. **RViz2 操作**
    - **2D Pose Estimate** → 點選烏龜車附近設定初始位姿
    - **Nav2 Goal** → 目標點 + 拖曳方向
    
    烏龜車應能規劃直線並行駛到目標。
    

---

### 1-10　常見坑與排除

| 症狀 | 可能原因 | 快速修正 |
| --- | --- | --- |
| `package 'turtlebot3_gazebo' not found` | 忘了 `colcon build` 或未 source overlay | 重新 build 並 `source ~/turtlebot3_ws/install/setup.bash`。 |
| `Failed to find a free participant index` | 多機 Domain 衝突 | 每台機器設不同 `ROS_DOMAIN_ID`。 |
| RViz2 地圖不動 / 時間倒退 | 未設定 `use_sim_time` | Nav2 launch 加 `use_sim_time:=True`。 |
| `ignoring laser scan! frame does not exist` | TF 缺 `base_link` 或 URDF 未載入 | 檢查 `robot_state_publisher`、URDF 座標系一致。 |
| Gazebo FPS 過低 | 缺專用 GPU 或驅動 | 安裝 NVIDIA 驅動並切換 X.org。 |

---

### 1-11　進階選讀

- **VS Code Dev Container**：`ms-ros/vscode-dev-containers` 有 `ros-humble-desktop` 模板，可用 Docker 隔離環境。
- **源碼編譯 Nav2**：若需自訂 BT Plugin，可在 `~/nav2_ws/src` clone `navigation2`（`humble-devel` 分支）再 `colcon build`。
- **WSL2 測試**：Win11 + GPU Passthrough 可執行 Gazebo，但網路與多節點延遲較大，不建議做最終驗證。

---

### 1-12　小結

- 已在 **Ubuntu 22.04** 安裝 **ROS 2 Humble Desktop-Full**
- 裝好 **Gazebo Classic 11**、**gazebo_ros_pkgs** 並完成基礎測試
- 從源碼編譯 **TurtleBot3** 與 Gazebo 模型
- 成功執行空地圖導航 Smoke Test

## 補充：

### Humble Hawksbill 與「一般版本」(非 LTS／Rolling) 的核心差異一覽

| 面向 | **Humble Hawksbill (LTS)** | **一般版本**<br>（Iron、Jazzy 等非 LTS + Rolling） | 說明 |
| --- | --- | --- | --- |
| **支援週期** | **5 年**：2022-05 → 2027-05 [discourse.ros.org](https://discourse.ros.org/t/ros-2-humble-hawksbill-released/25729?utm_source=chatgpt.com) | **18 個月**（非 LTS）或 **無固定 EOL**（Rolling 持續更新）[ros.org](https://www.ros.org/reps/rep-2000.html?utm_source=chatgpt.com) | LTS 保證長期安全更新與重大 Bug 修補；非 LTS 僅短期維護，Rolling 為每日快照。 |
| **API 穩定性** | 版本全生命週期內保證 **API/ABI 不破壞** | 非 LTS 每半年～一年可做破壞式變更；Rolling 隨時可能破壞 | 有長-尾產品或教學課程建議用 LTS。 |
| **平台階層** | Tier 1：Ubuntu 22.04 amd64/arm64、Windows 10 amd64；Tier 2：RHEL 8 等 [docs.ros.org](https://docs.ros.org/en/foxy/Releases/Release-Humble-Hawksbill.html) | 非 LTS 可能同時支援舊／新 Ubuntu；Rolling 追蹤最新 Ubuntu | LTS 只鎖定單一 Ubuntu LTS，方便驗證相依性。 |
| **主要新增功能** | *Content-Filtered Topics*、`ros2 topic` 新關鍵字、`launch` 變數作用域、`ament_cmake_gen_version_h`、SROS2 CRL 支援、RViz2 改善等 [docs.ros.org](https://docs.ros.org/en/foxy/Releases/Release-Humble-Hawksbill.html) | 每一版各有重點（Iron：stateless typesupport；Jazzy：Python 3.12 & C++23…） | LTS 會回執安全修補，但**不**回執破壞式新功能。 |
| **變體 (variant) 套件** | 可裝 `ros-humble-base` / `desktop` / **`desktop-full`** | 同樣有三種變體；Rolling 亦提供 nightly meta-pkgs | Desktop-Full 附 RViz2、Gazebo、rosbag2 等桌面工具。 |
| **Gazebo 整合** | 官方教稿仍以 **Gazebo Classic 11 + gazebo_ros_pkgs** 為主 | 非 LTS/Rolling 亦可切換 Ignition/Harmonic；API 變化較快 | Humble 使用 Classic 不需 ROS-Ign bridge，簡潔穩定。 |
| **預設 DDS** | Fast DDS（預載 Cyclone、Connext、Gurum 可切換） | 相同，但 Rolling 可能改變預設或實驗新 RMW [docs.ros.org](https://docs.ros.org/en/foxy/Concepts/About-Different-Middleware-Vendors.html?utm_source=chatgpt.com) | DDS 可於執行時以 `RMW_IMPLEMENTATION` 切換。 |
| **適用族群** | 產品化、長期維護專案、教科書、學年度專案 | 嘗鮮最新 API 功能、核心開發者、短期研究原型 | 評估風險 vs. 穩定度後擇一。 |

---

### 什麼情況選 Humble (Humble = Humble Hawksbill)？

1. **需要 5 年以上可重現的環境**：企業部署、教育課程、比賽。
2. **硬體/軟體驗證成本高**：一次驗證可沿用整個產品生命週期。
3. **團隊大、跨國協作**：API 穩定可減少 refactor 成本。

### 什麼時候用「一般版本」？

- **想用最新 C++23/Python 3.12** 等新語言特性（Jazzy 起）。
- **參與 ROS 2 核心或 Nav2 等社群開發**，需隨 upstream 同步。
- **短期研究**：半年內完成的論文/原型，樂於承擔相依破壞風險。


