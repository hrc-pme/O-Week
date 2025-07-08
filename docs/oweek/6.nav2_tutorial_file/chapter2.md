# 第二章　Nav2 × TurtleBot3 常用 Topic 全解

> 本章不是把所有 Topic 全部貼給你，而是把 80 % 最常在模擬／真機診斷時用到的 20 % Topic 梳理成一張腦內速查表。閱讀建議：
> 
> 1. 先看 **2-1~2-2** 打基礎 →
> 2. 用 **2-3 大表** 對照 RViz2 / `rqt_graph` →
> 3. 跑 Smoke Test 時邊 `ros2 topic echo`，加深印象。

---

### 2-1　ROS 2 傳輸回顧：Topic vs. Service vs. Action

| 類型 | 特點 | 例子（Nav2/TB3） |
| --- | --- | --- |
| **Topic** | Pub/Sub，持續串流，無回覆 | `/scan` 連續推雷射資料；`/cmd_vel` 每 20–50 Hz 驅動底盤 [emanual.robotis.com](https://emanual.robotis.com/docs/en/platform/turtlebot3/basic_operation/?utm_source=chatgpt.com) |
| **Service** | 請求-回覆，短任務 | `/global_costmap/clear_entirely_global_costmap` 清圖 |
| **Action** | 具 *Goal / Feedback / Result*，可取消 | `/navigate_to_pose` 長任務導航 [docs.clearpathrobotics.com](https://docs.clearpathrobotics.com/docs/ros/tutorials/navigation_demos/actions/?utm_source=chatgpt.com)[docs.ros.org](https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html?utm_source=chatgpt.com) |

> 後兩者本質仍用 Topic 互傳，所以 ros2 topic list 能看到 Action 的隱藏 Topic（/navigate_to_pose/_action/*）。
> 

---

### 2-2　宏觀架構：把 Topic 分五層記

```
bash
複製編輯
   ┌─────────────┐
   │  5. 指令層  │ /navigate_to_pose (Action)
   ├─────────────┤
   │  4. 控制層  │ /cmd_vel
   ├─────────────┤
   │  3. 路徑層  │ /plan  /path
   ├─────────────┤
   │  2. 定位層  │ /amcl_pose  /odom  /tf
   ├─────────────┤
   │  1. 感測層  │ /scan  /camera/...  /clock
   └─────────────┘

```

*感測 → 定位 → 路徑 → 控制 → 指令* 五階，任何導航問題先鎖定在哪層訊號斷了再追根究柢，會快很多。

---

### 2-3　TurtleBot3 + Nav2 最常用 Topic 一覽（Humble + Gazebo Classic）

| 層 | Topic 名稱 | Msg 型別 | 主要 **Publisher** | 主要 **Subscriber** | 用途 / 關鍵細節 |
| --- | --- | --- | --- | --- | --- |
| 感測 | `/scan` | `sensor_msgs/LaserScan` | `gazebo_ros_laser` Plugin | `nav2_costmap_2d`, SLAM, RViz2 | 2D 雷射；頻率 10 Hz，雷射 FoV 決定 costmap 靈敏度。 |
|  | `/clock` | `rosgraph_msgs/Clock` | Gazebo | 全系統（`use_sim_time`） | 模擬時用；若時間不同步，AMCL 會飄。 |
|  | `/camera/*`（選） | `sensor_msgs/Image` 等 | 相機 Plugin | SLAM / VSLAM | TB3 Gazebo 預設無，需手動載入相機。 |
| 定位 | `/odom` | `nav_msgs/Odometry` | Diff-drive Controller | `ekf_node`、Nav2 | 輪速里程計；Gazebo 模擬誤差低，真機有漂移。 |
|  | `/amcl_pose` | `geometry_msgs/PoseWithCovarianceStamped` | AMCL | RViz2、/tf | 地圖座標下之機器人估計。 |
|  | `/tf` / `/tf_static` | `tf2_msgs/TFMessage` | `robot_state_publisher`, 各演算法 | 全系統 | 框架骨幹；`ros2 run tf2_tools view_frames` 可視化。 |
| 路徑 | `/plan`（全域） | `nav_msgs/Path` | Global Planner | Nav2 BT Navigator | A* / SmacPlanner 輸出，地圖座標系。 [docs.ros.org](https://docs.ros.org/en/noetic/api/nav_msgs/html/msg/Path.html?utm_source=chatgpt.com) |
|  | `/local_plan` | `nav_msgs/Path` | Controller Server | Nav2 BT Navigator | Dynamic motion primitive；資料點稀疏更平滑。 |
|  | `/path`（RViz 可選） | `nav_msgs/Path` | Planner / Controller | RViz2 | RViz2 overlay。 |
| 控制 | `/cmd_vel` `/cmd_vel_stamped`* | `geometry_msgs/Twist` 或 `TwistStamped` | Nav2 Controller | Diff-drive Controller / Base | Humble 預設 `Twist`，Jazzy 之後改用 `TwistStamped` [emanual.robotis.com](https://emanual.robotis.com/docs/en/platform/turtlebot3/navigation/?utm_source=chatgpt.com) |
|  | `/velocity_smoother/raw_cmd_vel` | `Twist` | Controller | Velocity Smoother | 可選配 node，過濾急加速度。 |
| 指令 | `/navigate_to_pose/goal` 等 Action 子 Topic | `nav2_msgs/action/NavigateToPose`* | RViz2 “Nav2 Goal” | BT Navigator | 長任務；`ros2 action send_goal` CLI 範例見附錄。 |
|  | `/initialpose` | `geometry_msgs/PoseWithCovarianceStamped` | RViz2 “2D Pose Estimate” | AMCL | 在 RViz 點一下即可發送。 |
- Action 其實拆分成 `goal` / `feedback` / `result` 三個隱藏 Topic，由 `rclcpp_action` 幫你打包處理。

* `cmd_vel_stamped` 只在 **TwistStamped mode** 開 `enable_stamped_cmd_vel:=True` 時出現。

---

### 2-4　CLI 快速檢查腳本

```bash
bash
複製編輯
# 檢查 Topic 是否存在
ros2 topic list | grep -E 'scan|odom|amcl_pose|cmd_vel$|plan'

# 查看 Topic 頻率
ros2 topic hz /scan

# Echo 前 5 則雷射資料
ros2 topic echo --qos-profile sensor_data -n 5 /scan

# 檢查 TF 樹
ros2 run tf2_tools view_frames  &&  evince frames.pdf

```

> 排雷訣竅：
> 
> - `/scan` 沒資料？→ 先看 Gazebo Console 是否報 **update rate 0**。
> - `/cmd_vel` 發了卻不動？→ `ros2 topic echo /odom` 看速度是否 0；多半是安全控制或底盤電流保護。

---

### 2-5　與 `rqt_graph` 對照練習

1. `rqt_graph &` → **“Nodes Only”** 顯示
2. 用 RViz2 發 `/initialpose`、`Nav2 Goal` 一次
3. 觀察 `/plan → /cmd_vel → /odom → /amcl_pose` 流向是否閉合

做完一次，你就能在 10 秒內判斷「是感測沒進來？還是控制沒出去？」

---

### 2-6　小結

- **五層心智模型** 幫你快速定位問題。
- 熟練 `ros2 topic` / `rqt_graph` 是 Nav2 故障診斷的第一步。
- 記住最常動手的 Topic：`/scan`、`/odom`、`/tf`、`/amcl_pose`、`/plan`、`/cmd_vel`、`/navigate_to_pose`。