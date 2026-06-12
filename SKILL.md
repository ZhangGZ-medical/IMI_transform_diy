---
title: "IMI_transform_diy"
summary: "立体定向手术坐标变换工具，计算导针deposit的RAS三维坐标并生成3D Slicer可视化代码"
read_when:
  - 用户需要计算导针deposit的RAS坐标
  - 处理包含traj/deposits sheet的Excel文件进行坐标回填
  - 生成3D Slicer Markups导入代码用于可视化
agent_created: true
---

# IMI_transform_diy - 立体定向手术坐标变换工具

## 技能概述

读取 Excel（含 traj + deposits 两个 sheet），调用 `compute_deposits.py` 计算每个 deposit 的 RAS 三维坐标并回填，然后为每个 traj 生成独立的 3D Slicer Python Console 可运行代码（`.py` + `.md`）。

| 项目 | 内容 |
|------|------|
| **坐标系** | RAS (Right/Anterior/Superior)，单位 mm |
| **计算脚本** | 项目根目录下的 `compute_deposits.py` |
| **每 traj 输出** | 1 个 `.py` + 1 个 `.md`，按 `slicer_import_{case}_traj{N}_deposits` 命名 |

---

## 几何模型约定

### 钟面角定义

| 参数 | 定义 |
|------|------|
| **12点钟参考** | A轴(前方向)投影，平行时fallback到S轴 |
| **钟面角** | 每钟点 = 30°，顺时针，viewer沿entry→tip方向看 |
| **细针-轴线夹角** | 固定 19° |

### 坐标计算公式

沿轨迹向前（远离入点）：

```
position = Tip + depth × u_hat
```

其中：
- `Tip` = traj sheet 的尖端坐标
- `depth` = deposit 深度参数
- `u_hat` = 轨迹单位向量（归一化方向），结合钟面角和19°偏转

---

## 输入格式

输入 Excel 包含两个 sheet：

### traj sheet
6 行 × 3 列，无表头：
```
e#    r#    数值
e#    a#    数值
e#    s#    数值
t#    r#    数值
t#    a#    数值
t#    s#    数值
```

### deposits sheet
```
Row 1: deposit# | 1 | 2 | ... | N   (deposit编号，从1开始)
Row 2: depth    | 数值...
Row 3: angle    | 数值...            (钟面角度)
Row 4: exit     | 数值...
Row 5: R        | (空，待回填)
Row 6: A        | (空，待回填)
Row 7: S        | (空，待回填)
```

---

## 使用流程

### 步骤 1：坐标计算与回填

对每个 traj xlsx 文件运行 `compute_deposits.py`：

```bash
python compute_deposits.py <xlsx文件路径>
```

脚本读取 traj 坐标作为 Entry/Tip，结合每个 deposit 的 depth/angle/exit 参数计算 RAS，**直接回填** deposits sheet 的 R/A/S 三行。

### 步骤 2：生成 Slicer 可视化代码

读取回填后的 xlsx，为每个 traj 生成一对文件：

- `slicer_import_{case}_traj{N}_deposits.py` — 可执行代码
- `slicer_import_{case}_traj{N}_deposits.md` — 含坐标表 + 可复制代码块

**代码规范**：

- **单个 Fiducial Point List**：每个 traj 只创建 1 个 `vtkMRMLMarkupsFiducialNode`，所有 deposit 点通过 `AddFiducial()` + `SetNthFiducialLabel()` 添加
- **Glyph 类型**：`StarBurst`（`slicer.vtkMRMLMarkupsDisplayNode.StarBurst`）
- **Text Scale**：`0.0`（不显示文字标签）
- **颜色**：traj1 红 `[1.0, 0.0, 0.0]`，traj2 绿 `[0.0, 1.0, 0.0]`，traj3 蓝 `[0.0, 0.0, 1.0]`
- **锁定**：`fid.SetLocked(True)`

### 步骤 3：交付

将回填后的 xlsx + 所有 `.py`/`.md` 文件一并交付给用户。

---

## 输出 Slicer 代码模板

```python
import slicer

deposits = [
    ("d1 (dep=6,ang=0)", 19.21, -8.86, 0.12),
    ("d2 (dep=6,ang=0)", 20.24, -8.86, 3.98),
    # ... 更多 deposit 点
]

fid = slicer.mrmlScene.AddNewNodeByClass("vtkMRMLMarkupsFiducialNode")
fid.SetName("{case}_traj{N}_deposits")

for i, (label, r, a, s) in enumerate(deposits):
    try:
        fid.AddFiducial(r, a, s)
        fid.SetNthFiducialLabel(i, label)
    except Exception as ex:
        print(f"FAIL d{i+1}: {ex}")

disp = fid.GetDisplayNode()
disp.SetColor([1.0, 0.0, 0.0])           # 按 traj 编号: 红/绿/蓝
disp.SetSelectedColor([min(x + 0.3, 1.0) for x in color])
disp.SetGlyphType(slicer.vtkMRMLMarkupsDisplayNode.StarBurst)
disp.SetTextScale(0.0)
fid.SetLocked(True)
```

---

## 触发关键词

- 坐标变换
- 立体定向
- 导针坐标
- deposit坐标
- 3D Slicer导入
- RAS坐标
- traj/deposits处理
- IMI_transform

---

## 依赖

- Python 3.x
- openpyxl（Excel 读写）
- 项目脚本：`compute_deposits.py`
