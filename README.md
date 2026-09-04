# 小派机器人啦啦操训练流水线

> 让 Mini Pi Plus 学会啦啦操舞蹈，参加世界人形机器人运动会！

基于 [高擎机器人](https://github.com/HighTorque-Robotics) 开源框架，完整流水线：

```
视频 → 3D人体姿态提取 → 运动重定向 → RL训练 → Sim2Sim验证 → 真机部署
```

## 快速开始

### 1. 环境搭建

```bash
cd ~/tree_PI
bash setup_env.sh
```

### 2. 放置视频

```bash
# 将啦啦操视频放到 videos/ 目录
cp ~/Downloads/cheerleading.mp4 ~/tree_PI/videos/
```

### 3. 运行完整流水线

```bash
bash pipeline/run_all.sh videos/cheerleading.mp4 cheerleading
```

或者分步执行：

```bash
# 步骤1: 视频 → 3D姿态
python pipeline/01_extract_motion.py videos/cheerleading.mp4

# 步骤2: 3D姿态 → 机器人动作
python pipeline/02_retarget.py output/gvhmr_cheerleading/hmr4d_results.pt

# 步骤3: CSV → NPZ
python pipeline/03_preprocess.py --fps 30 --end 1800 --visualize

# 步骤4: RL训练
bash pipeline/04_train.sh

# 步骤5: 导出和部署
python pipeline/05_deploy.py
```

## 目录结构

```
~/tree_PI/
├── HT_GVHMR-and-GMR/         # 视频→3D姿态 + 运动重定向
│   ├── GVHMR/                 #   GVHMR 视频推理
│   ├── GMR/                   #   GMR 运动重定向
│   └── args.txt               #   推理参数模板
├── Mini-Pi-Plus_BeyondMimic/ # RL训练 + Sim2Real框架
│   ├── scripts/               #   训练/部署脚本
│   ├── source/motion/         #   运动数据 (NPZ格式)
│   └── source/whole_body_tracking/  # 机器人模型
├── videos/                    # 原始啦啦操视频
├── body_models/               # SMPL人体模型权重
├── output/                    # GVHMR输出
└── pipeline/                  # 流水线脚本
    ├── run_all.sh             # 一键执行
    ├── 01_extract_motion.py   # 步骤1: GVHMR
    ├── 02_retarget.py         # 步骤2: GMR
    ├── 03_preprocess.py       # 步骤3: CSV→NPZ
    ├── 04_train.sh            # 步骤4: 训练
    └── 05_deploy.py           # 步骤5: 部署
```

## 环境要求

| 组件 | 版本 | 说明 |
|------|------|------|
| GPU | RTX 3090/4090+ | 24GB显存推荐 |
| Isaac Sim | v5.0.0 | NVIDIA官方 |
| Isaac Lab | v2.2.0 | 基于Isaac Sim |
| Python | 3.10/3.11 | 跟随Isaac Sim |
| Ubuntu | 20.04+ | Linux系统 |
| MuJoCo | >=2.3.0 | Sim2Sim验证 |

## 核心技术

### GVHMR (Global View Human Mesh Recovery)
- 从视频中提取全身3D人体姿态和形状
- 输出 SMPL 参数序列
- 支持任意视角视频输入

### GMR (General Motion Retargeting)
- 将人体运动映射到机器人关节空间
- 支持多种机器人类型 (`pi_plus_head`)
- 考虑关节限制和运动风格

### RSL-RL (PPO训练)
- 基于 RSL-RL 的 PPO 算法
- 4096个并行环境，2-6小时收敛
- DeepMimic 风格奖励函数

### Sim2Real 流水线
- Isaac Sim 训练 → ONNX 导出 → MuJoCo 验证 → RKNN 部署
- 充分的域随机化保证 sim2real 迁移

## 常见问题

### GVHMR 提取效果不好？
- 确保视频中人物全身可见
- 使用正面或侧面视角
- 尝试调整 GVHMR 参数

### 训练不收敛？
- 增加域随机化
- 调整奖励权重
- 减少动作长度先做短动作测试

### Sim2Real 差距大？
- 确保训练中 domain randomization 充分
- 先在 MuJoCo 中验证动作合理性
- 调整 PD 控制器增益

## 参考项目

- [Mini-Pi-Plus_BeyondMimic](https://github.com/HighTorque-Robotics/Mini-Pi-Plus_BeyondMimic)
- [HT_GVHMR-and-GMR](https://github.com/HighTorque-Robotics/HT_GVHMR-and-GMR)
- [BeyondMimic](https://github.com/HybridRobotics/whole_body_tracking)
- [RSL-RL](https://github.com/leggedrobotics/rsl_rl)
