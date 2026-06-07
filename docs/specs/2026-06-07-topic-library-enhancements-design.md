# 话题灵感库增强：戏曲号 + Deadline + 状态增强 + 拖拽排序

日期：2026-06-07
状态：已确认，待实施

## 概述

对话题灵感库（单文件 `index.html`）做四项增量改动：新增礼乐戏曲号账号、添加 deadline 字段、增强状态筛选与着色、实现卡片可拖拽排序。所有改动向后兼容现有数据和 Supabase 云同步。

## 1. 新增礼乐戏曲号

### 改动

- `ACCOUNTS` 数组从 `["礼乐科普号", "礼乐生活号"]` 改为 `["礼乐科普号", "礼乐生活号", "礼乐戏曲号"]`。
- 表单「归属账号」下拉、右侧账号 tab 自动多出戏曲号选项。
- 「内容支柱」下拉新增四个选项：`戏曲赏析`、`唱腔身段`、`戏里乾坤`、`梨园人物`。

### 兼容性

旧数据无 `account: "礼乐戏曲号"` 记录，不影响现有数据。云端无需改 schema（`data` 列为 JSONB）。

## 2. Deadline 字段

### 表单

在「创作推进」section 中，`负责人` 和 `创建人` 同行区域添加 `<input type="date" id="deadline">`，label 为「截止日期」。

### 数据

`idea.deadline` 存 ISO 日期字符串（如 `"2026-06-15"`），默认空字符串。`normalizeIdea` 给默认值 `""`。

### 卡片显示

在 meta 区域添加 deadline pill，颜色逻辑：

| 条件 | 颜色 | 文字 |
|------|------|------|
| `today < deadline` | 绿色 (`#16a34a` bg `#f0fdf4`) | `截止 6月15日` |
| `today == deadline` | 橙色 (`#b45309` bg `#fff7ed`) | `今日截止` |
| `today > deadline` | 红色 (`#b42318` bg `#fff0ee`) | `已delay · 6月10日` |
| 无 deadline | 不显示 | — |

### 筛选

筛选栏新增 `filterDeadline` 下拉：

- 全部
- 本周到期（deadline 在本周一 ~ 本周日）
- 已 delay（deadline < today）
- 无截止日期（deadline 为空）

## 3. 状态增强

### 状态 pill 按钮组

位置：账号 tab 下方、统计数据上方。样式：比账号 tab 更小的 pill 按钮，每个带颜色圆点和计数 badge。包含「全部」按钮。

点击等价于设置 `filterStatus` 下拉的值，两者联动——按钮选中时同步更新下拉值，下拉改变时也同步高亮对应按钮。

### 卡片状态 pill 着色

| 状态 | 颜色类型 | CSS 参考 |
|------|----------|----------|
| 新灵感 | 灰色 | bg `#f3f4f6` color `#6b7280` |
| 待讨论 / 待验证 | 蓝色 | bg `#eff6ff` color `#2563eb` |
| 已立项 / 写脚本中 | 青绿色 | bg `#e2f5f1` color `#075e57` |
| 拍摄/制作中 | 绿色 | bg `#f0fdf4` color `#16a34a` |
| 已发布 / 复盘完成 | 深绿色 | bg `#dcfce7` color `#15803d` |
| 放弃 | 红色 | bg `#fff0ee` color `#b42318` |

## 4. 可拖拽排序

### 交互

每张卡片左侧显示拖拽手柄 `⠿`，鼠标悬停时显示 `cursor: grab`。拖拽时目标位置显示蓝色虚线占位。

### 实现

使用原生 HTML5 Drag and Drop API（`draggable="true"` + `dragstart` / `dragover` / `drop` 事件），不引入外部库。

### 持久化

拖拽后给每条 idea 写入 `sortOrder` 数字字段（拖拽后重新编号 0, 1, 2, ...），保存到 localStorage 和云端。

### 与排序联动

- 排序下拉新增选项「按手动排序」，使用 `sortOrder` 字段排序。
- 用户拖拽卡片时自动切换到「按手动排序」。
- 选择其他排序（爆款分、时间等）时覆盖手动排序，卡片不可拖拽（手柄隐藏）。

### 兼容性

旧数据无 `sortOrder` 字段，`normalizeIdea` 给默认值 `999999`（排在末尾；不用 `Infinity` 因为 JSON 不支持）。

## 技术要点

- 所有改动在单文件 `index.html` 内完成（CSS + HTML + JS）。
- 新字段 `deadline`（string）和 `sortOrder`（number）加入 `textFields` 或单独处理。
- `collectForm` / `fillForm` / `resetForm` / `normalizeIdea` 均需更新。
- 筛选栏 grid 列数从 6 列调整为 7 列以容纳 deadline 筛选。
- 移动端响应式需测试：状态 pill 按钮组在小屏上应换行显示。

## 不做的事

- 不改 Supabase schema（JSONB 列自动兼容新字段）。
- 不拆分为多文件。
- 不加外部依赖（保持纯 vanilla JS）。
