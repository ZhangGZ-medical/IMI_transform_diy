---
name: IMI_transform_diy
description: >-
  立体定向手术导针→deposit RAS坐标变换计算技能。
  用于读取包含traj（导针入点/尖端坐标）和deposits（depth/angle/exit参数）sheet的Excel文件，
  计算每个deposit的RAS坐标并回填，以及生成3D Slicer Markups导入代码。
  涉及RAS坐标系、A轴12点钟参考方向、19°细针夹角、钟面角（每钟点30°）等几何模型。
---

# IMI Transform DIY — 导针→deposit坐标变换

## 用途

对立体定向手术规划中的导针轨迹数据计算deposit的RAS三维坐标，并输出到Excel和3D Slicer可视化环境。

## 触发条件

当用户需要：
- 计算导针deposit的RAS坐标（"计算deposit坐标"、"导针coordinate变换"）
- 处理包含traj/deposits sheet的Excel文件进行坐标回填
- 生成3D Slicer Markups导入代码用于可视化
- 批量处理多个deposit（不限数量）

## 使用流程

### 步骤1：坐标计算

执行 `scripts/compute_deposits.py`：

```bash
python scripts/compute_deposits.py <xlsx文件路径>
```

**输入要求**（Excel文件结构）：
- `traj` sheet: 每行(label, axis, value)，如 `('e1','r1',33.9)`, `('t1','r2',28.21)` 等
- `deposits` sheet: 第1行deposit编号，depth/angle/exit参数行，R/A/S输出行

脚本自动解析、计算并回填R/A/S坐标到原文件。

### 步骤2：生成Slicer导入文档

生成带时间戳的Slicer Python Console导入代码.md文档：

```python
from scripts.slicer_import import save_slicer_md
save_slicer_md(xlsx_path)  # 自动生成 slicer_import_2026-06-02_09-30.md 等唯一文件名
```

或仅获取代码字符串（不写文件）：

```python
from scripts.slicer_import import generate_slicer_code
code = generate_slicer_code(xlsx_path)
```

将.md中的代码粘贴到 Slicer 的 View → Python Interactor 执行（首次需 `slicer.util.pip_install('openpyxl')`）。

**文件命名**：每次调用 `save_slicer_md()` 生成唯一文件名 `slicer_import_{日期}_{时间}.md`，历史结果不会被覆盖。

## 几何模型约定

详见 `references/coordinate_model.md`，核心参数：

| 参数 | 值/定义 |
|------|---------|
| 坐标系 | RAS (Right/Anterior/Superior), 单位mm |
| 12点钟参考 | A轴(前方向)投影，平行时fallback到S轴 |
| 钟面角 | 每钟点=30°，顺时针(viewer沿entry→tip) |
| 细针-轴线夹角 | 固定19° |
| 深度方向 | 沿轨迹向前(远离入点): T + depth×u_hat |

## 输入输出格式

**traj sheet**:
```
e1, r1, 33.9
e1, a1, -25.79
e1, s1, 68.24
t1, r2, 28.21
t1, a2, -30.63
t1, s2, 21.14
```

**deposits sheet** (输入→输出):
```
deposit#, 1,      2,      3
depth,    8,      8,      8
angle,    0,      0,      90
exit,     8,      4,      8
R,        <计算>, <计算>, <计算>
A,        <计算>, <计算>, <计算>
S,        <计算>, <计算>, <计算>
```
