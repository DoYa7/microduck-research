# MicroDuck 机器鸭复刻资料合集

> 整理自 GitHub 开源社区，供豆吖项目参考学习。
> 整理时间：2026-10-28

## 项目概述

MicroDuck 是一个小型双足机器鸭机器人项目，基于强化学习实现行走、平衡和动作。原始项目由 Pollen Robotics 开发，后经社区大量复刻和改进。

### 核心特性
- 双足行走，14-15个舵机驱动
- 强化学习训练行走策略（ONNX模型）
- 树莓派/Radxa Zero 作为主控
- 支持蓝牙手柄遥控
- 3D打印结构件

---

## 参考仓库

| 仓库 | Stars | 说明 |
|---|---|---|
| [pollen-robotics/microduck](https://github.com/pollen-robotics/microduck) | 12.8k | 官方仓库，Rust代码 |
| [apirrone/Open_Duck_Mini](https://github.com/apirrone/Open_Duck_Mini) | 4.5k | 原版Open Duck Mini |
| [pollen-robotics/microduck_rl](https://github.com/pollen-robotics/microduck_rl) | 2.7k | RL训练环境（MuJoCo） |
| [AI-FanGe/Microduck-build-tutorial](https://github.com/AI-FanGe/Microduck-build-tutorial) | 1.5k | 帆哥保姆级教程（抖音同款） |
| [fanhao375/microduck-replica](https://github.com/fanhao375/microduck-replica) | 1.4k | 中文复刻，飞特HD-1910版 |
| [LuwuDynamics/xgoduck_hardware](https://github.com/LuwuDynamics/xgoduck_hardware) | 1.5k | XGO-Duck硬件，Arduino版 |
| [kissqy/Microduck-kissqy](https://github.com/kissqy/Microduck-kissqy) | 366 | 飞特HD1910复刻，无训练版 |

---

## 硬件清单（BOM）

### 方案A：Dynamixel XL330 官方版

| 类别 | 物料 | 数量 | 说明 |
|---|---|---|---|
| 主控 | Raspberry Pi Zero 2 W | 1 | 运行控制程序、Wi-Fi、SSH |
| 舵机控制板 | ROBOTIS OpenRB-150 | 1 | USB连接Pi，驱动XL330舵机总线 |
| 舵机 | Dynamixel XL330-M288-T | 14 | 双腿10个+头颈部4个 |
| 电池 | 6V可充电电池包 | 1 | 成品电池，需足够瞬时电流 |
| IMU | BNO080/BNO085/BNO086 | 1 | I2C连接，姿态读取 |
| 存储 | 16GB+ microSD卡 | 1 | 推荐质量稳定的新卡 |
| 结构件 | 3D打印件 | 1套 | PLA材料，CAD文件在仓库中 |
| 螺丝 | M2/M2.5自攻螺丝 | 若干 | 舵机固定、结构件连接 |

### 方案B：飞特 HD-1910 国产版（更便宜）

| 类别 | 物料 | 数量 | 说明 |
|---|---|---|---|
| 主控 | Radxa Zero 3W | 1 | 替代树莓派，性价比更高 |
| 舵机 | 飞特 HD-1910 | 15 | 含嘴部舵机，国产便宜很多 |
| IMU | 总线IMU板 | 1 | 自制PCB，STM32G031固件 |
| 扩展板 | RPI Robot HAT | 1 | Pollen官方开源HAT |
| 电池 | 6V电池包 | 1 | 同方案A |

---

## 舵机ID映射

| ID | 名称 | 位置 |
|---|---|---|
| 1 | right_ankle | 右踝 |
| 2 | right_knee | 右膝 |
| 3 | right_hip_pitch | 右髋俯仰 |
| 4 | right_hip_roll | 右髋横滚 |
| 5 | right_hip_yaw | 右髋偏航 |
| 6 | left_ankle | 左踝 |
| 7 | left_knee | 左膝 |
| 8 | left_hip_pitch | 左髋俯仰 |
| 9 | left_hip_roll | 左髋横滚 |
| 10 | left_hip_yaw | 左髋偏航 |
| 11 | head_pitch | 头部俯仰 |
| 12 | neck_pitch | 颈部俯仰 |
| 13 | head_yaw | 头部偏航 |
| 14 | head_roll | 头部横滚 |
| 15 | mouth | 嘴部（可选） |

**Dynamixel Wizard 参数：**
- Protocol: 2.0
- Baud Rate: 1Mbps
- Return Delay Time: 0
- PWM Slope: 255

---

## 接线图

### 电源链路
```
6V电池 → 电源开关 → OpenRB-150电源输入 → XL330舵机总线
                                    ↓
5V稳压模块 → Raspberry Pi Zero 2 W（USB供电）
```

### 舵机总线串联
```
OpenRB-150 Dynamixel Port → 分线
├── 右腿链: ID 5 → 4 → 3 → 2 → 1
├── 左腿链: ID 10 → 9 → 8 → 7 → 6
└── 头颈链: ID 12 → 11 → 13 → 14
```

### IMU 接线（I2C）
| IMU引脚 | 树莓派引脚 |
|---|---|
| VIN/3V3 | 3.3V |
| GND | GND |
| SDA | GPIO2 / I2C SDA |
| SCL | GPIO3 / I2C SCL |

---

## 软件部署

### 1. 刷写镜像
下载预构建镜像 `microduck.img.xz`，用 Raspberry Pi Imager 刷入SD卡。

### 2. 配置Wi-Fi
编辑SD卡boot分区的 `network-config` 文件：
```yaml
network:
  version: 2
  wifis:
    wlan0:
      dhcp4: true
      access-points:
        "你的WiFi名":
          password: "你的密码"
```

### 3. 首次启动
- 等待1-3分钟自动扩展分区
- SSH登录：`ssh user@microduck.local`（密码：password）
- 首次登录后改密码：`passwd`

### 4. 运行行走模型
```bash
cd ~/microduck
PYTHONPATH=src .venv/bin/python src/main.py
```

**键盘控制：**
| 按键 | 功能 |
|---|---|
| v | 开启/关闭行走 |
| 方向键上下 | 前进/后退 |
| 方向键左右 | 左右转向 |
| x | 速度归零 |
| q | 停止程序 |

### 5. 蓝牙手柄模式
```bash
bluetoothctl
# 配对手柄后，开机长按START 2秒启动
# A键切换行走，左摇杆控制速度
```

---

## 3D打印与装配

### 打印建议
- 材料：PLA
- 填充：20-30%
- 层厚：0.2mm
- 支撑：需要（腿部零件）

### 装配顺序
1. 安装小腿（舵机+舵盘+螺丝）
2. 安装大腿
3. 组装躯干组件
4. 安装颈部
5. 安装头部
6. 接线（电源+舵机总线+IMU）

### 视频教程
- B站：[机械行者Robo《MicroDuck从零安装教程》](https://space.bilibili.com/312747174/lists/9279084)（共8集）
- 抖音：AI研究室-帆哥（爸爸发的那个视频）

---

## 与豆吖项目的关联参考

### 可借鉴点
1. **舵机控制**：XL330/HD-1910总线舵机驱动方式，可用于豆吖未来的运动机构
2. **IMU姿态融合**：BNO08x的I2C读取和姿态解算
3. **强化学习行走策略**：ONNX模型在边缘设备上的部署
4. **无头运行**：开机自启+蓝牙手柄遥控，不需要显示器
5. **3D打印结构**：轻量化双足设计经验

### 豆吖当前差异
- 豆吖目前是桌面陪伴机器人，不需要行走
- 豆吖主控是STM32H750+ESP32-S3，不是树莓派
- 豆吖核心是语音交互+陪伴，不是运动

---

## 常见问题

### 舵机不动
- 检查电池电压是否正常（6V）
- 检查OpenRB-150是否被USB识别
- 检查舵机ID是否正确
- 检查总线波特率是否1Mbps

### SSH连不上
- 确认Wi-Fi是2.4GHz（Pi Zero 2 W不支持5GHz）
- 等1-3分钟让系统启动完成
- 到路由器查Pi的IP地址，用IP直连

### 机器人站不稳
- 先手扶住再测试
- 检查左右腿舵机ID是否装反
- 检查IMU安装方向是否正确
- 确认walk.onnx模型和硬件匹配

---

## 资料来源

本文件整理自以下开源仓库：
- [AI-FanGe/Microduck-build-tutorial](https://github.com/AI-FanGe/Microduck-build-tutorial)
- [fanhao375/microduck-replica](https://github.com/fanhao375/microduck-replica)
- [pollen-robotics/microduck](https://github.com/pollen-robotics/microduck)

仅供学习参考，版权归原作者所有。
