# 导针→deposit RAS坐标变换几何模型

## 坐标系

- **RAS坐标系**: R=Right(右), A=Anterior(前), S=Superior(上)
- 所有坐标单位为毫米(mm)

## 几何定义

### 1. 轨迹方向 (trajectory)

- **导针入点** E = (r1, a1, s1)
- **导针尖端** T = (r2, a2, s2)
- **轨迹方向**: u = (T - E) / |T - E|, viewer沿E→T方向观察

### 2. 深度延伸

- P_depth = T + depth × u_hat
- depth沿轨迹向前(远离入点)延伸

### 3. 钟面角 (clock angle)

- **12点钟参考**: A轴(前方向, 0,1,0)在垂直平面上的投影
- **轨迹平行A轴时**: fallback到S轴(0,0,1)投影
- **每钟点 = 30°**: 0点=0°=12点, 1点=30°, 2点=60°, 3点=90°, 6点=180°, 9点=270°
- **顺时针测量**: viewer沿entry→tip方向看
- 计算公式: dir(θ) = cos(θ)×ref_hat - sin(θ)×(u_hat×ref_hat)

### 4. 细针方向

- **与轴线夹角**: 固定19°
- needle_dir = cos(19°)×u_hat + sin(19°)×clock_dir

### 5. Deposit位置

- deposit = P_depth + exit × needle_dir

## 参考方向投影算法

```
ref_axis = (0, 1, 0)  # A轴
ref = ref_axis - (ref_axis·u_hat)×u_hat  # 投影到垂直平面
if |ref| < 1e-10:
    ref_axis = (0, 0, 1)  # fallback S轴
    ref = ref_axis - (ref_axis·u_hat)×u_hat
ref_hat = ref/|ref|
perp = u_hat × ref_hat  # 右手定则
```
