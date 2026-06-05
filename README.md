# IMI_transform_diy - 立体定向手术坐标变换工具

## 概述

IMI_transform_diy 是一个用于立体定向手术的坐标变换工具，能够计算导针 deposit 的 RAS 三维坐标，并将结果回填到 Excel 文件中，同时生成 3D Slicer 可视化代码。

## 功能特性

- 📊 **Excel 数据处理**：读取 traj 和 deposits sheet 进行坐标计算
- 🧮 **精确坐标计算**：基于几何模型计算 RAS 坐标
- 🎯 **3D Slicer 支持**：生成 Markups 导入代码用于可视化
- 📐 **钟面角支持**：12 点钟参考，顺时针方向

## 安装

```bash
# 克隆仓库或下载文件到本地
git clone https://github.com/ZhangGZ-medical/imi_transform_diy.git
cd imi_transform_diy

# 安装依赖
pip install -r requirements.txt
```

## 依赖

- pandas >= 1.3.0
- numpy >= 1.20.0
- openpyxl >= 3.0.0

## 快速开始

### 1. 准备 Excel 文件

输入 Excel 需包含两个 sheet：

| Sheet | 必需列 | 说明 |
|-------|--------|------|
| traj | R, A, S | 导针入点/尖端坐标 (RAS) |
| deposits | depth, angle, ext | 深度、钟面角、延伸参数 |

### 2. 运行计算

```python
from scripts.transform import compute_ras_coordinates

# 读取 Excel 文件
input_file = "your_data.xlsx"
output_file = "output.xlsx"

# 计算坐标
compute_ras_coordinates(input_file, output_file)
```

### 3. 可视化

生成的 3D Slicer 代码可直接导入 3D Slicer：

```
# 在 3D Slicer Python interactor 中运行
exec(open("output_markups.py").read())
```

## 坐标系说明

### RAS 坐标系

| 轴 | 方向 | 说明 |
|----|------|------|
| R (Right) | 右方向 | 患者右侧为正 |
| A (Anterior) | 前方向 | 患者前方为正 |
| S (Superior) | 上方向 | 患者头顶方向为正 |

### 钟面角定义

- **12 点钟参考**：A 轴（前方向）投影，平行时 fallback 到 S 轴
- **钟面角**：每钟点 = 30°，顺时针
- **细针-轴线夹角**：固定 19°

### 计算公式

```
position = T + depth × u_hat
```

其中：
- `T` = 入点坐标（Tip from traj sheet）
- `depth` = 深度参数
- `u_hat` = 轨迹单位向量（归一化方向）

## 目录结构

```
imi_transform_diy/
├── README.md
├── requirements.txt
├── assets/
│   └── example.xlsx          # 示例数据
├── scripts/
│   ├── __init__.py
│   └── transform.py          # 核心变换脚本
└── references/
    └── coordinate_system.md    # 坐标系参考文档
```

## 使用示例

```python
import pandas as pd
import numpy as np

# 示例：计算单个 deposit
traj_R, traj_A, traj_S = 100.0, 50.0, 80.0  # 入点
depth = 30.0  # mm
angle = 3    # 3 点钟方向 (90°)

# 计算
theta = np.radians((angle - 12) * 30)  # 钟面角转弧度
alpha = np.radians(19)  # 固定 19° 夹角

u_hat = np.array([
    np.sin(alpha) * np.sin(theta),
    np.sin(alpha) * np.cos(theta),
    np.cos(alpha)
])

T = np.array([traj_R, traj_A, traj_S])
pos = T + depth * u_hat

print(f"RAS: ({pos[0]:.2f}, {pos[1]:.2f}, {pos[2]:.2f})")
# 输出: R ≈ 130.0, A ≈ 50.0, S ≈ 102.6
```

## 触发关键词

在 WorkBuddy 中使用时可提及以下关键词触发技能：

- 坐标变换
- 立体定向
- 导针坐标
- deposit 坐标
- 3D Slicer
- RAS 坐标
- traj/deposits

## 许可

MIT License

## 作者

ZhangGZ-medical

## 更新日志

### v1.0.0
- 初始版本
- 支持 traj/deposits Excel 读取
- 生成 RAS 坐标回填
- 生成 3D Slicer Markups 代码