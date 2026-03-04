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



## 定义

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

