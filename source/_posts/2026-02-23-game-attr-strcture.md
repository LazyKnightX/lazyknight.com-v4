---
title: 小谈游戏属性结构
date: 2026-2-23 16:05:58 +0800
tags: [game-programming]
---

谈谈一些常见的游戏属性逻辑设计结构。

*本文结合了Grok AI部分讨论内容编写，感谢老马的贡献！*



**TOC**

* 1. 普通加减法
* 2. Modifier List + 排序求值



## 普通加减法（最基础）

在这种做法里，属性直接储存到战斗对象类，通过简单的加减法进行操作。

通常适用于MVP阶段，游戏早期阶段。

```
class CombatObject:
    float hp
    float atk
    float atkBonus
    TakeDamage(target):
        dmg = target.atk * (1 + target.atkBonus)
        this.hp -= dmg
```

## Modifier List + 排序求值

有序Modifier列表 + 最终求值 (Attribute / Stat Modifier System) 。

### 核心计算逻辑

1. 先按属性类型分组Modifiers，减少遍历次数。
2. 按优先级排序Modifiers
3. 遍历Modifiers
4. 根据Modifier操作类型（FlatAdd, PercentAdd, PercentMult），执行相关数值累计
5. 计算最终属性值
6. 保存到缓存 (可选)
7. 清理“脏”标记
8. 通报属性变更消息

### 要点

1. 枚举定义StatType
2. 使用struct定义修改器，避免GC
3. 优先级作用要明确定义
4. 实现Stats容器类，并将其作为模块挂载到需要属性的实体

### 具体实现参考

**属性类型（轻量 enum）**

```
enum StatType:
    生命
    回血
    伤害
    攻速            // 每秒攻击次数
    护甲
    幸运
    弹道数量
    冷却缩减        // 通常 0～100+%
    暴击率
    暴击伤害
    // ...
```

**单条修改器（使用struct避免GC）**

```
struct StatModifier:
    StatType Stat                  // 属性类型
    float Value                    // 属性值
    ModifierOperation Operation    // 操作类型 - 平加 / 百分比加 / 百分比乘（最后一种慎用）
    int Priority                   // 优先级 - 越小越先算（建议：0=基础，10=平加，20=百分比加，30=乘法）

    string SourceName              // 调试/tooltip 用，例如 "松树", "幸运+1", "T1升级"
    // 可选: Guid/unique id        // 如果需要移除特定Modifier，可据此移除
```

**常见优先级顺序（非常重要，决定数值感觉）**

| 优先级 | 操作类型       | 典型例子                            | 为什么这个顺序             |
| ------ | -------------- | ----------------------------------- | -------------------------- |
| 0      | Base           | 角色初始值 + 每级成长               | 最基础                     |
| 5      | FlatAdd (正)   | +4 伤害、+20 最大生命               | 先加再乘最直观             |
| 10     | FlatAdd (负)   | -2 攻速（debuff）                   | 负值也先处理               |
| 15     | PercentAdd     | +30% 伤害、+15% 攻速                | 常见加成                   |
| 20     | PercentMult    | ×1.5 最终伤害（极稀有，如某些神器） | 极品加成，最后乘区，防爆炸 |
| 25     | Override / Set | 强制设为 100% 暴击（特殊状态）      | 几乎覆盖一切               |

**属性容器（挂在玩家 / 武器 / 角色上）**

```
class StatsContainer:
    SerializableDictionary<StatType, float> _baseStats = new()  // 角色属性基础值 // 读取角色配置 SO 获取
    StatModifier modifiers = new()                              // 属性修改器列表

    Dictionary<StatType, float> _finalCache = new()             // 缓存最终值
    bool _isDirty = true                                        // 脏标记（只在变化时重算）

    void AddModifier(StatModifier mod):
        modifiers.Add(mod)
        _isDirty = true

    void RemoveModifier(StatModifier mod):                      // 或按来源批量移除
        modifiers.Remove(mod)
        _isDirty = true

    float GetStatValue(StatType type):
        if (_isDirty) RecalculateAll()
        return _finalCache.TryGetValue(type, out var v) ? v : GetBase(type)

    float GetBase(StatType type):
        return _baseStats.TryGetValue(type, out var v) ? v : 0f

    void RecalculateAll()
        _finalCache.Clear()

        // 按属性分组，减少遍历次数
        groups = modifiers.GroupBy(m => m.Stat)

        foreach (g in groups):
            StatType stat = g.Stat
            float final = GetBase(stat)

            float flatAdd = 0
            float percentAdd = 0
            float mult = 1f

            # 按优先级排序（List.Sort 或 OrderBy）
            List<StatModifier> sorted = g.OrderBy(m => m.Priority).ToList()

            foreach m in sorted:
                switch (m.Operation)
                    case ModifierOperation.FlatAdd:
                        flatAdd += m.Value
                        break
                    case ModifierOperation.PercentAdd:
                        percentAdd += m.Value
                        break
                    case ModifierOperation.PercentMult:
                        mult *= (1f + m.Value)
                        break

            final = final + flatAdd
            final = final * (1f + percentAdd)
            final = final * mult

            // optional: clamp / round
            // final = max(0.1f, round(final * 100f) / 100f)

            _finalCache[stat] = final

        _isDirty = false

        // Fire event / reactive subject
        // OnStatsChanged.OnNext(Unit.Default)
```

**SO示例：升级**

```
[CreateAssetMenu]
public class UpgradeSO : ScriptableObject
{
    public string upgradeName;
    public List<StatModifierTemplate> modifiers;   // template → real StatModifier on pickup
}
```

### 常见公式

最终值 = (基础 + 所有平加) × (1 + 所有百分比加总和) × (乘区)

例如：
* 基础伤害 10
* +4 平加、+6 平加 → 平加 = 10
* +30%、+45% → 百分比加 = 0.75
* 最终 = (10 + 10) × (1 + 0.75) = 20 × 1.75 = 35

### 拓展：加成区和优先级管理

建议使用轻量enum定义属性所属加成区类型，避免hard-code混乱，降低心智负担。

同时强制策划填写数据必须填写Bucket字段，确保属性加成正确。

```csharp
// 在 StatModifier 里加一个 Bucket 字段（enum 或 int）
public enum StatBucket
{
    BaseFlat = 0,           // 0-9
    Additive = 1,           // 10-19
    SpecialAdditive = 2,    // 20-24
    Multiplicative = 3,     // 25-29
    Override = 4            // 30+
}

public readonly struct StatModifier
{
    public readonly StatType Stat;
    public readonly float Value;
    public readonly ModifierOperation Operation;
    public readonly StatBucket Bucket;          // 新增：必须选桶
    public readonly int SubPriority;            // 桶内小优先级（可选，0默认）
    public readonly string Source;              // "策划需求#123: 伤害×1.5 More"
    // ...
}
```

对应演算逻辑可变化：

```csharp
float final = GetBase(stat);  // Bucket 0 已经包含在 base 里了

// Bucket 1 & 2: Additive Percent
float totalAddPct = 0f;
foreach (var mod in modifiers.Where(m => m.Bucket <= StatBucket.SpecialAdditive))
{
    if (mod.Operation == PercentAdd) totalAddPct += mod.Value;
}
final *= (1f + totalAddPct);

// Bucket 3: Multiplicative
float totalMult = 1f;
foreach (var mod in modifiers.Where(m => m.Bucket == StatBucket.Multiplicative))
{
    totalMult *= (1f + mod.Value);   // 或 *= mod.Value，如果是 More 风格
}
final *= totalMult;

// Bucket 4: Override（最后处理，极少）
foreach (var mod in modifiers.Where(m => m.Bucket == StatBucket.Override).OrderBy(m => m.SubPriority))
{
    if (mod.Operation == Set) final = mod.Value;
    // 或其他 override 逻辑
}
```

### 拓展：R3 + ObservableCollections 推荐写法（UI友好）

```c#
using R3;
using ObservableCollections;

// StatsContainer 部分关键改动
public class StatsContainer : MonoBehaviour
{
    // ObservableList → 自动通知变化
    public ObservableList<StatModifier> Modifiers { get; } = new();

    // 每个属性的最终值用 ReactiveProperty
    private readonly Dictionary<StatType, ReactiveProperty<float>> _statProperties = new();

    private readonly Subject<Unit> _onDirty = new();  // 手动脏通知

    public StatsContainer()
    {
        // 预创建所有属性的 RP（避免运行时分配）
        foreach (StatType type in Enum.GetValues(typeof(StatType)))
        {
            _statProperties[type] = new ReactiveProperty<float>(GetBase(type));
        }

        // 监听修改器列表变化 → 标记脏
        Modifiers.ObserveAdd().Subscribe(_ => MarkDirty());
        Modifiers.ObserveRemove().Subscribe(_ => MarkDirty());
        Modifiers.ObserveReplace().Subscribe(_ => MarkDirty());
        Modifiers.ObserveReset().Subscribe(_ => MarkDirty());

        // 每当脏了，就重算（可以用帧延迟或 UniTask.NextFrame 优化）
        _onDirty.ThrottleFirst(TimeSpan.FromMilliseconds(50))  // 防抖，50ms 内多次变化只算一次
                .Subscribe(_ => RecalculateAll());
    }

    public ReadOnlyReactiveProperty<float> GetStatAsRP(StatType type)
    {
        return _statProperties[type].ToReadOnlyReactiveProperty();
    }

    private void MarkDirty() => _onDirty.OnNext(Unit.Default);

    private void RecalculateAll()
    {
        var groups = Modifiers.GroupBy(m => m.Stat);

        foreach (var group in groups)
        {
            var type = group.Key;
            float baseVal = GetBase(type);

            float flat = 0f;
            float pctAdd = 0f;
            float mult = 1f;

            foreach (var mod in group.OrderBy(m => m.Priority))
            {
                switch (mod.Operation)
                {
                    case ModifierOperation.FlatAdd: flat += mod.Value; break;
                    case ModifierOperation.PercentAdd: pctAdd += mod.Value; break;
                    case ModifierOperation.PercentMult: mult *= (1f + mod.Value); break;
                }
            }

            float final = (baseVal + flat) * (1f + pctAdd) * mult;

            // 写入 RP，不会产生多余 GC（RP 内部优化好）
            _statProperties[type].Value = final;
        }
    }
}
```

UI绑定示例：

```c#
// 在 HUD 或 Tooltip 上
damageText.BindTo(GetStatAsRP(StatType.Damage))
          .Subscribe(v => damageText.text = $"伤害: {v:F1}");
```

### 常见坑 & 优化点（用 Profiler 能看到）

- **不要** 在 RecalculateAll 里频繁 ToList() 或 OrderBy().ToList() → GC 杀手
  - 解法：如果修改器不多（<50条/属性），直接 foreach + if 判断优先级段（分三段计算 flat / pctAdd / mult）
  - 或维护三个分开 List：FlatModifiers / PercentAddModifiers / MultModifiers（最暴力但最快）
- **移除修改器** 时，最好用 **来源标识** 而不是具体 struct
  - 例如：RemoveAllFromSource("Upgrade_001")
  - 加一个 string SourceId 或 object Source 字段
- **Addressables + SO**：UpgradeSO 里不要直接存 StatModifier，而是存 StatModifierData（可序列化），pickup 时转成 struct
- **负值处理**：PercentAdd 可以是负的（-20% 伤害），但 PercentMult 负值要小心（可能导致负伤害）
- **性能目标**（手机 TapTap 发布）
  - 单次重算 < 0.2ms
  - GC Alloc < 0.5KB / 次变化
  - 用 Profiler → Deep Profile 看 RecalculateAll 的耗时和分配
