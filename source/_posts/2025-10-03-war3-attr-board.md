---
title: 魔兽3 属性面板的设计
date: 2025-10-4 03:34:45 +0800
tags: [war3]
---

需求：
* 当鼠标浮动到一个按钮区域时，显示属性面板
* 显示的同时立即刷新属性面板的属性
* 每1秒更新属性面板内各条属性的实时状态

几个要点：
* 百分比属性的渲染
* 实数属性的渲染 - 保留小数位数
* 刷新的契机 - 开启时刷新 + 间隔时间异步刷新
* 判定属性面板是否正在显示
* 同类属性归类
* 每一个区域各自保持一定颜色规则确保清晰

关键点：
* 每一个属性的数值获取和渲染都有其单独规则，因此建议以配置表的方式明确定义每一个属性的规则。
* 通常属性面板有明确布局，可以通过拆分配置，分别为“属性功能表”和“属性布局表”，前者定义渲染规则，后者定义布局定位。

分块计算属性区位置：

```lua
local pos = {}
for i = 1, 12 do
    local x, y, w, h, gap_x, gap_y

    x = 12
    y = 12
    w = 240
    h = 160
    gap_x = w + 20
    gap_y = h + 20

    local ix, iy
    ix = (i - 1) % 4 + 1
    iy = (i - 1) // 4 + 1

    pos[i] = {
        x = x + (ix - 1) * gap_x,
        y = y + (iy - 1) * gap_y,
    }
end
```

读取布局数据并应用属性：

```lua
local layout_settings = {}
local attr_settings = {}

for i = 1, 12 do
    local layout = layout_settings[i]
    local piece = panel.c_container[i]
    for j = 1, 9 do
        local label = layout[j]
        local def_data = attr_settings[label]
        if def_data then
            local rendered_title
            local value
            local rendered_value
            local method = def_data.method
            local mode = def_data.mode

            if method == FETCH_METHOD_DEFAULT then
                value = get_unit_attr_data(hero, def_data.symbol)
            else
                print('unknown attr method: ', method, 'label: ', label)
                error('...')
            end

            if mode == 'r' then
                rendered_value = r2s_real(value)
            elseif mode == 'p' then
                rendered_value = r2s_pct(value)
            else
                rendered_value = value
            end

            rendered_title = string.format('%s%s|r', def_data.color, def_data.title)
            piece:x_set_text(j, rendered_title, rendered_value)
        else
            local rendered_title = ''
            local rendered_value = ''
            piece:x_set_text(j, rendered_title, rendered_value)
        end
    end
end
```
