# 第三章    TurtleBot 3 導航模擬

> 本章目標：讓 TurtleBot 3 在 Gazebo 11 裡完成一次完整導航任務。
> 
> 
> 你將學會兩種做法──
> 
> - **方案 A：官方一鍵腳本**（最快上手）
> - **方案 B：手動四步曲**（最能看懂系統細節）

完成後，你就能熟練用 RViz2 或 Python 指令把turtlebot送往任意座標。

---

### 3-1　選擇模擬場景turtlebot

| world 檔 | 特點 | 指令範例 |
| --- | --- | --- |
| **empty_world** | 空地形，適合測試導航基礎 | `ros2 launch turtlebot3_gazebo empty_world.launch.py` |
| **turtlebot3_world** | 具桌椅障礙，官方預設教學場 | `ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py` |
| **turtlebot3_house** | 迷宮式住宅，路徑複雜 | `ros2 launch turtlebot3_gazebo turtlebot3_house.launch.py` |

> 如果只想先看車子會動，用 empty_world 即可。
> 

> 如果是希望測試導航，建議使用t**urtlebot3_world**
> 

---

## 方案 A　— 一鍵腳本 (`tb3_simulation_launch.py`)

1. **啟動**
    
    ```bash
    bash
    複製編輯
    export TURTLEBOT3_MODEL=burger               # waffle 或 waffle_pi 亦可
    ros2 launch nav2_bringup tb3_simulation_launch.py \
         headless:=False
    
    ```
    
    - 預設自動載入 `turtlebot3_world`、spawn 機器人、啟動 Nav2、RViz2，以及 `/teleop` 節點。
    - 重要可選參數
        
        
        | 引數 | 用途 | 範例 |
        | --- | --- | --- |
        | `slam:=True/False` | `True` 走 **SLAM + 導航**；`False` 用現成地圖 | 預設 `False` |
        | `map:=<file>` | 指定離線地圖 YAML | `map:=$HOME/maps/office.yaml` |
        | `world:=<world>` | 換成其他 Gazebo 場景 | `world:=turtlebot3_house` |
        | `headless:=True` | 不開 Gazebo GUI（遠端 CI 用） |  |
    
    [github.com](https://github.com/ros-planning/navigation2/blob/main/nav2_bringup/launch/tb3_simulation_launch.py?utm_source=chatgpt.com)
    
2. **RViz2 操作**
    - 右上 *Nav2 Goal* → 點擊目標並拖曳方向
        
        *觀察 `/plan`（紫）、`/local_plan`（綠）與 `/cmd_vel` 箭頭正常輸出。
        

---

## 方案 B　— 手動設定

### Step 1　啟動 Gazebo + Robot

```bash
bash
複製編輯
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py

```

### Step 2　建立或載入地圖

| 你已經有 YAML 地圖？ | 行動 |
| --- | --- |
| **沒有** | 啟動 SLAM Toolbox：`ros2 launch turtlebot3_cartographer cartographer.launch.py use_sim_time:=True`→ 用 `teleop_twist_keyboard` 探索環境 |
| **有** | 跳到 Step 3 |

完成探索後存圖：

```bash
bash
複製編輯
ros2 run nav2_map_server map_saver_cli -f $HOME/maps/world

```

地圖會生成 `world.yaml / world.pgm`。

### Step 3　啟動 Nav2

```bash
bash
複製編輯
ros2 launch turtlebot3_navigation2 navigation2.launch.py \
     use_sim_time:=True \
     autostart:=True \
     map:=$HOME/maps/world.yaml

```

> Lifecycle 節點全部變成 active 才算成功。若卡在 configuring，多半是 /tf 或 /odom 缺漏。docs.nav2.org
> 

### Step 4　RViz2 初始化

1. **Initial Pose**：按 *2D Pose Estimate*，在機器人真實位置點一下並拖方向。
2. **設定目標**：按 *Nav2 Goal* 點任意障礙物外區域，烏龜車應即刻規劃並移動。

---

### 3-4　用 Python 自動送目標（Nav2 Simple Commander）

```python
python
複製編輯
#!/usr/bin/env python3
from nav2_simple_commander.robot_navigator import BasicNavigator, NavigationResult
import rclpy

rclpy.init()
nav = BasicNavigator()
nav.waitUntilNav2Active()

goal = nav.setInitialPose([0.0, 0.0, 0.0])             # 先設定初始位姿
goal = nav.makePoseStamped(x=2.5, y=-1.0, yaw=1.57)     # 目標
nav.goToPose(goal)

while not nav.isTaskComplete():
    feedback = nav.getFeedback()
    if feedback:
        print(f"Distance remaining: {feedback.distance_remaining:.2f} m")

result = nav.getResult()
print(f"Navigation result: {result == NavigationResult.SUCCEEDED}")

```

> Script 需 ros-humble-nav2-simple-commander，可同步掛在 CI 作自動路徑驗證。docs.nav2.org
> 

---

### 3-5　常見坑位速查

| 症狀 | 快速檢查 |
| --- | --- |
| **RViz 報 `map` frame 不存在** | `ros2 topic echo /tf_static |
| **機器人不動** | `ros2 topic echo /cmd_vel -n 1` 若有速度而 `/odom` 不變，Gazebo 控制器未載入；若 `/cmd_vel` 一直 0，看看 BT Navigator 是否處於 *rejected* |
| **`Timed out waiting for transform base_link → odom`** | world.launch.py 沒有 `gazebo_ros_diff_drive` plugin；或者機器人 namespace 與 tf 不一致。 |
| **地圖漂移** | 模擬必須 `use_sim_time:=True`，且 `/clock` 正常跳動；否則 time 同步錯誤、AMCL covariance 無限放大。 |

---

### 3-6　小結

- **一鍵腳本** (`tb3_simulation_launch.py`) 最快，但黑盒子多。
- **手動四步曲** 清楚看到 Gazebo → SLAM → Map Server → Nav2 各環節。
- 之後要做參數優化或行為樹客製，建議走手動流程，再導回自定義 launch。