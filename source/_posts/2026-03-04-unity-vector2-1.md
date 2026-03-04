---
title: Unity Vector2 基础 & 朝向应用
date: 2026-3-4 22:23:43 +0800
tags: [note,unity]
typora-root-url: ..
---

*笔记部分内容来自AI生成，尚未完全校验，仅供参考。*



精通Vector2相关知识可以让移动、朝向相关的编程变得手到擒来。

除此以外，Vector2有一些关于性能的细节。



在 [Github](https://github.com/LazyKnightX/unity-samples/tree/master/Samples/Vector2/Assets/Scripts) 查看与本文相关的演示（Unity 2022.3）。



**Vector2的基础应用：设置朝向**

* [LookToPlayer.cs](https://github.com/LazyKnightX/unity-samples/blob/master/Samples/Vector2/Assets/Scripts/LookToPlayer.cs) → 使用 `transform.up` 调整对象朝向指定目标（Vector2）或鼠标位置。
* [LookToPlayerSmooth.cs](https://github.com/LazyKnightX/unity-samples/blob/master/Samples/Vector2/Assets/Scripts/LookToPlayerSmooth.cs) → 拓展了 `Mathf.LerpAngle` 的使用方式 & 鼠标范围有效性检测，要注意 eulerAngle 情况下的朝向修正细节。



## 概念

* Vector2.Angle

* Vector2.normalized
* Vector2.Dot
* Vector2.magnitude / Vector2.sqrMagnitude
* transform.up = value
* transform.right = value
* transform.forward = value
* 零向量防御
* Mathf.Approximate
* 子弹接近目标时突然加速现象（Update执行） → overshoot / 抖动 / 反转
* clamp
* normalized + speed
* Translate(value) vs .position = value
* Vector2.Lerp
* 预测子弹：`predicted = targetPos + targetVel * (dist/speed)`
* Vector2.Angle vs Vector2.SignedAngle
* Quaternion 原理
* 取反方向 & 反方向判定
* 大规模方向判断 & 距离筛选 → 空间分区 + Job System / Burst 。需注意四叉树动态分区对Burst不友好。



## 基础 | V2.one/zero 等的常用情景

Vector2具有两个轴，分别为X/Y。

**部分简写：**

* Vector2.zero -> new Vector2(0f, 0f)
* Vector2.one -> new Vector2(1f, 1f)
* Vector2.up -> new Vector2(0f, 1f)
* Vector2.down -> new Vector2(0f, -1f)
* Vector2.right -> new Vector2(1f, 0f)
* Vector2.left -> new Vector2(-1f, 0f)

**Vector2.zero 常用的方式：**

* 重置速度：`rigidbody2D.velocity = Vector2.zero`
* 清空加速度/力：`rb.AddForce(Vector2.zero)` （其实没必要，但写出来很清晰）
* 初始化位置偏移：`spawnPosition = player.transform.position + Vector2.zero`
* 作为“无方向”的默认值：`moveDirection = Vector2.zero`
* 在 lerp/slerp 等插值结束时的目标值

**Vector2.one 的使用情况：**

Vector2.one 使用频率比 zero 低，但也很典型。

* 缩放重置：`transform.localScale = Vector2.one`
* 均匀缩放：`transform.localScale = Vector2.one * 1.5f`
* 创建一个对角线方向的单位向量：`Vector2 diagonal = Vector2.one.normalized` → ≈ (0.707, 0.707)
* 某些 UI 布局或网格系统中表示“每个方向都走一步”

**Vector2.up 的使用情况：**

* 2D 角色初始朝向：`transform.up = Vector2.up`（很多 2D 游戏默认向上是正面）
* 向上跳跃：`rb.velocity = Vector2.up * jumpForce`
* 重力方向反向模拟：`Physics2D.gravity = -Vector2.up * 9.8f`
* 激光/枪口方向：`bulletDirection = transform.up`
* 与 transform.right 配合做本地坐标系计算：

```csharp
// 向右前方45度发射
Vector2 shootDir = (transform.right + transform.up).normalized;
```

在 2D 项目中，强烈建议养成用 `Vector2.up/right` 代替硬编码 (0,1)/(1,0) 的习惯：

* 可读性更好
* 如果以后改成 3D 或改坐标系，改动量最小
* 别人接手代码时一眼就能明白意图

## 应用 | 让一个2D物体朝向鼠标光标位置

**需要搞定以下问题**

1. 设置2D物体朝向
2. 获取鼠标光标位置
3. 计算两个Vector2之间的方向向量（并归一化）

**核心思路**

在2D游戏中：

* `transform.up` 代表物体本地Y轴正方向（通常是“朝向”）
* `transform.right` 是X轴正方向

将朝向设置为鼠标方向，就是让 transform.up 指向从物体位置到鼠标的世界坐标差的方向向量（归一化后）。

前提：假设在 `Update()` 或 `FixedUpdate()` 中执行，物体有 `Transform` 组件，使用 **正交相机（Orthographic）** 。

### 方法1：直接设置 `transform.up` （简单直接）

```csharp
void LookAtMouse() {
    // 1. 获取鼠标在世界坐标的位置（返回Vector3，设置类型为Vector2时，会自动处理Z=0的2D转换）
    Vector2 mouseWorldPos = Camera.main.ScreenToWorldPoint(Input.mousePosition);
    
    // 2. 计算从物体到鼠标的方向向量，并归一化
    Vector2 direction = (mouseWorldPos - (Vector2)transform.position).normalized;
    
    // 3. 直接设置朝向（up 就是本地Y轴）
    transform.up = direction;
}
```

* 优点：一行核心代码，直观，Unity自动处理旋转。
* 调用：`LookAtMouse()`， 放在 `Update()` 里，每帧更新。
* 场景：塔防炮台、2D角色自动瞄准。

### 方法2：用角度计算 + `Quaternion.Euler` （更灵活，可控制旋转速度）

```csharp
void LookAtMouse() {
    Vector2 mouseWorldPos = Camera.main.ScreenToWorldPoint(Input.mousePosition);
    Vector2 direction = mouseWorldPos - (Vector2)transform.position;
    
    // 计算角度（Atan2 返回 -180~180°，完美用于2D）
    float angle = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg;
    
    // 设置Z轴旋转（2D只用Z）
    transform.rotation = Quaternion.Euler(0f, 0f, angle);
}
```

变体（平滑旋转，加插值）：

```csharp
float targetAngle = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg;
float currentAngle = transform.eulerAngles.z;
float newAngle = Mathf.LerpAngle(currentAngle, targetAngle, rotationSpeed * Time.deltaTime);
transform.rotation = Quaternion.Euler(0f, 0f, newAngle);
```

* 优点：可轻松添加 `LerpAngle` 实现平滑转动；角度易读，便于限制旋转范围（如只转180°）。
* 场景：需要“缓慢瞄准”的Boss、限制转角的枪管。

## 性能

可研究下方内容原理，进一步深入理解Vector2的性能。

* Vector2.magnitude → 会调用 Math.Sqrt 。
* Vector2.sqrMagnitude → 不会调用 Math.Sqrt ，因此相比 Vector2.magnitude 性能快。
* Math.Sqrt → 理解数学运算在CPU上的代价，即可理解Vector2的性能卡点。
* struct Vector2 → V2是值类型，即便每帧运行1000次 new Vector2 ，也不会产生GC。但注意装箱情况。

## 移动 & 物理系统

纯Transform驱动移动情况下一般不抖动。

如果和物理系统混用（如Rigidbody2D），可能出现抖动，常见于 `Update` 方法内更新position + 卡顿时。（ `transform.position += PositionInfo` ）

原理：物理系统每次FixedUpdate同步Transform。

## 判定两个 Vector2 是否相同

不建议直接使用 `==` or `.Equals` 。

**原理：**Unity重载 `==` 和 `!=` 运算符，采用**逐分量精确比较**。

但 float 浮点数多次运算时可能精度不准，易诱发幽灵BUG。

```csharp

[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static bool operator ==(Vector2 lhs, Vector2 rhs)
{
    float num = lhs.x - rhs.x;
    float num2 = lhs.y - rhs.y;
    return num * num + num2 * num2 < 9.9999994E-11f;
}

[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static bool operator !=(Vector2 lhs, Vector2 rhs)
{
    return !(lhs == rhs);
}
```



### 推荐方式

**首选：** `Vector2.Distance(a, b) < epsilon` 或 `Vector2.sqrMagnitude < epsilonSqr`



例子：

```csharp
bool ApproximatelySame(Vector2 a, Vector2 b, float tolerance = 0.0001f)
{
    return Vector2.Distance(a, b) < tolerance;
    // 更高效写法（避免开方）：
    // return (a - b).sqrMagnitude < tolerance * tolerance;
}
```



**次选：判断方向是否大致相同，忽略长度**

`Vector2.Dot(a.normalized, b.normalized) > cosineThreshold`



例子：

```csharp
bool DirectionsApproximatelySame(Vector2 a, Vector2 b, float minDot = 0.999f)
{
    // 先检查是否接近零向量，避免除零
    if (a.sqrMagnitude < 0.0001f || b.sqrMagnitude < 0.0001f)
        return a.sqrMagnitude < 0.0001f && b.sqrMagnitude < 0.0001f;

    return Vector2.Dot(a.normalized, b.normalized) >= minDot;
}
```

minDot 对应角度：

- 0.9999 ≈ ±2.56°
- 0.999 ≈ ±8.1°
- 0.99 ≈ ±25.8°
- 0.9848 ≈ ±10°



**第三选择：Unity 内置工具类 Mathf.Approximately**



例子：

```csharp
bool IsApproximatelyEqual(Vector2 a, Vector2 b)
{
    return Mathf.Approximately(a.x, b.x) && Mathf.Approximately(a.y, b.y);
}
```



`Mathf.Approximately` 内部实现：

```csharp
public static bool Approximately(float a, float b)
{
    return Abs(b - a) < Max(1E-06f * Max(Abs(a), Abs(b)), Epsilon * 8f);
    // 相对误差 + 绝对误差结合，比较智能
}
```



**推荐实践写法：**

```csharp
public static class Vector2Extensions
{
    private const float DefaultEpsilon = 0.00001f;          // 位置比较
    private const float DirectionEpsilon = 0.001f;          // 方向比较
    private const float DirectionDotThreshold = 0.9998f;    // ≈ ±2.56°

    /// <summary> 两个向量是否“足够接近”（位置/点比较） </summary>
    public static bool Approximately(this Vector2 a, Vector2 b, float epsilon = DefaultEpsilon)
    {
        return (a - b).sqrMagnitude < epsilon * epsilon;
    }

    /// <summary> 两个方向是否“基本相同”（忽略长度） </summary>
    public static bool SameDirection(this Vector2 a, Vector2 b, float minDot = DirectionDotThreshold)
    {
        float magA = a.sqrMagnitude;
        float magB = b.sqrMagnitude;

        if (magA < 1e-6f || magB < 1e-6f)
            return magA < 1e-6f && magB < 1e-6f;

        return Vector2.Dot(a, b) >= minDot * Mathf.Sqrt(magA * magB);
        // 更高效写法，避免 normalized
    }
}
```



### 小结

**最常用、最安全**：`(a - b).sqrMagnitude < tolerance * tolerance`

**方向专用**：`Vector2.Dot(a.normalized, b.normalized) > threshold`

**零向量要特殊处理**：避免 normalized 时的 NaN

**永远不要**：直接 `a == b` 或 `a.Equals(b)`（除非你明确知道没有浮点运算）



## 角度计算

`Vector2.Angle(direction, transform.up)`





## 拓展

### 衍生概念

* Quaternion.LookRotation
* Quaternion.Slerp
* Rect + Rect.Contains(mousePosition)

### 以安全的方式获取鼠标转世界坐标点

传给 `Camera.main.ScreenToWorldPoint` 的向量参数如果出现**极值（+∞、-∞）**，会报错。

在测试时，如果鼠标移动到窗口外，再获取时（`Input.mousePosition`），有可能会出现这种情况：

```
Screen position out of view frustum (screen pos inf, -inf, 0.000000) (Camera rect 0 0 1084 463)
UnityEngine.Camera:ScreenToWorldPoint (UnityEngine.Vector3)
LookToPlayerSmooth:Update () (at Assets/Scripts/LookToPlayerSmooth.cs:26)
```

这种情况在编辑器开发测试中尤其常见。



**可以给镜头写一个拓展方法来避开这个现象。**

```csharp
public static class CameraExtensions
{
    public static bool TryScreenToWorldPoint(this Camera cam, Vector3 screenPos, out Vector3 worldPos, float depth = 10f)
    {
        worldPos = Vector3.zero;

        if (float.IsNaN(screenPos.x) || float.IsInfinity(screenPos.x) ||
            float.IsNaN(screenPos.y) || float.IsInfinity(screenPos.y))
        {
            return false;
        }

        if (screenPos.x < 0 || screenPos.x > Screen.width ||
            screenPos.y < 0 || screenPos.y > Screen.height)
        {
            return false;
        }

        screenPos.z = depth;
        worldPos = cam.ScreenToWorldPoint(screenPos);
        return true;
    }
}

// Sample:
if (Camera.main.TryScreenToWorldPoint(Input.mousePosition, out Vector3 target, 10f))
{
    // 正常使用 target
}
else
{
    // 保持上一帧的方向 或 什么都不做
}
```



**其它可能出现这种现象的情况：**

* **鼠标完全移出游戏窗口（尤其是 Editor 中）** → 在 Unity Editor 里把鼠标快速移出 Game 视图，或者点到其他窗口，再快速移回来，Input.mousePosition 有概率变成垃圾值（inf/-inf/nan）。

* **构建后的独立窗口 + Alt+Tab / 最小化再还原** → Windows 系统下，窗口失去焦点 → 重新获得焦点时，鼠标位置有时会短暂变成非法值。

* **多显示器 + 鼠标在第二屏幕上** → 当鼠标跑到主游戏窗口之外的显示器区域，Input.mousePosition 会超出正常范围，甚至变成负无穷。

* **Input.mousePosition 在 Update() 里被其他系统/插件篡改** → 比如某些 UI 框架、输入管理器、第三方插件在某些条件下会临时修改 mousePosition。

* **极少数情况下：分辨率刚改变、分屏模式切换、VR/多相机等** → 相机 rect 或 screen size 还没来得及更新。

## 提示

* **方向向量为零：**鼠标正好在物体位置时， `direction.normalized` 是 NaN。

```csharp
// 修复方式：
Vector2 dir = (mouseWorldPos - (Vector2)transform.position);
if (dir.sqrMagnitude > 0.001f) {  // 用sqrMagnitude避免sqrt，可提升性能
    transform.up = dir.normalized;
}
```

* **多相机：**用 `Camera.main` 获取主相机比较保险，其它使用 `GetComponent<Camera>` 。

* **Z轴问题**：2D物体position.z通常0，但ScreenToWorldPoint会设z=-远裁，必须强制 (Vector2) 转换。

* **性能优化**：可缓存 Camera.main 到变量。
* **系统冲突**：用 LateUpdate() 避免与动画冲突。
* <del>**3D兼容**：可用 `transform.forward` 代替 `transform.up` 。</del>（待验）
* 可用 `Debug.DrawRay(transform.position, transform.up * 2f, Color.red)` 可视化朝向。
