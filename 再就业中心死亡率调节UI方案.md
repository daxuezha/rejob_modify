# 再就业中心死亡率调节 UI（日志按钮）方案

> 本文档由调研产出，供后续 session 阅读与维护。
> 目标：为「失业人口再就业中心」（`building_gulag_unemployment_center`）提供一个
> **日志（Journal Entry）+ 可点击按钮** 的界面，让玩家随时调整该建筑在所在州造成的
> 额外死亡率修正。核心要求：**不破坏任何原有功能**。

---

## 1. 需求（已与需求方确认）

| 项 | 结论 |
|---|---|
| 交互形式 | 日志(Journal Entry) 内的可点击按钮，仿原版 Montenegro `je_montenegrin_raiding` |
| 按钮 | `+1 / +10 / +100 / +1000`、`-1 / -10 / -100 / -1000`，外加「重置调节」 |
| 总和定义 | **总和 = 生产方式固定值 + 可调值** |
| 总和下限 | **0**（可调值可降到负数，用于抵消生产方式固定值） |
| 总和上限 | 无限 |
| UI 显示 | **必须显示总和值** |
| 特殊生产方式 | 选用 `pm_rejob_uc_gasking`（9999999）时，**所有按钮失效**（灰掉、不可点） |
| 兼容性 | **零修改原文件**（纯新增文件），不改生产方式、不改建筑、不改原有决议/事件 |

---

## 2. 调研结论（关键机制，均已在游戏自带官方文档与原版脚本中验证）

### 2.1 日志 + 按钮的官方机制

依据游戏自带文档：
`common/journal_entries/journal_entries.md`、`common/scripted_buttons/scripted_buttons.md`、
`common/script_values/script_values.md`、`common/modifier_type_definitions/modifier_types.md`。

- **日志激活方式（重要）**：`is_shown_when_inactive` + `possible` 的「自动激活」机制
  在本 mod 组合下**不可靠**（见 §7.3）。**必须**像 OGAS 那样，由决议或 on_action
  显式调用 `add_journal_entry = { type = ... }` 来添加。本方案同时提供两种途径：
  决议（玩家主动点击）+ on_action（每月脉冲自动兜底）。
- **按钮挂载**：在日志内用 `scripted_button = <key>` 声明，UI 自动渲染。
  按钮支持 `name / desc / visible / possible / effect / ai_chance / cooldown`。
- **动态数值修正**：`add_modifier = { name = X multiplier = <脚本值或变量> }`。
  **同名修正不能叠加**，改值必须先 `remove_modifier` 再重新 `add_modifier`
  （原版范例：`game/common/journal_entries/05_montenegro_je.txt:113-125`）。
- **`state_mortality_mult`**：已定义为 `decimals=0 / color=bad / percent=yes` 的**州级**修正
  （`game/common/modifier_type_definitions/00_modifier_types.txt:2290`）。
  由于是 `percent=yes`，脚本里的 `4` 在游戏内显示为 `+400%`。
- **触发器**：`is_building_type = <建筑>`（building 作用域）、
  `has_active_production_method = <生产方式>`（building 作用域，范例
  `game/common/history/conscription/00_conscription_center.txt:5-7`）。
- **脚本值**支持 `min = <数值或另一个脚本值>`、`if = { limit = {...} add = N }`、
  `value / add / subtract / multiply / divide`，运算**按书写顺序**执行。
- **本地化数值格式**：`[SCOPE.ScriptValue('key')|0%]` 会按百分比显示（`4` → `400%`），
  与游戏内修正提示一致（范例 `game/localization/english/acw_text_l_english.yml:140`）。

> ⚠️ **踩坑记录 1：`ai_chance` 必须用 `value`，不能用 `base`**
> 游戏自带文档 `scripted_buttons.md` 里写的是 `ai_chance = { base = 0 }`，
> 但**实际引擎不认 `base`**，会直接报
> `Unexpected token: base` 并使**整个按钮文件解析失败**（所有按钮全部未定义），
> 进而导致引用这些按钮的日志整条加载失败、游戏里完全不显示。
> 原版全部 296 处 `ai_chance` 用的都是 `ai_chance = { value = 0 }`。
> **正确写法：`ai_chance = { value = 0 }`**。

> ⚠️ **踩坑记录 2：判断变量非零用 `var:x != 0`**
> 原版脚本中 `NOT = { var:x = 0 }` 零使用，标准写法是 `var:x != 0`。

> ⚠️ **踩坑记录 3：`is_ai = no` 在 JE 的 `is_shown_when_inactive` 中不适用**
> 该处应使用 `is_player = yes`（原版 JE 的 `is_shown_when_inactive` 里用的是 `is_player`）。

### 2.2 再就业中心的死亡率来源

再就业中心共 3 个生产方式组，其中 2 个组带有州级 `state_mortality_mult`：

| 组 | 生产方式 | `state_mortality_mult` |
|---|---|---|
| `rejob_uc_camp` | `pm_rejob_uc_shao` | 4 |
| | `pm_rejob_uc_gulag` | 4 |
| | `pm_rejob_uc_oil` | 4 |
| | `pm_rejob_uc_gas` | **400** |
| | `pm_rejob_uc_gasking` | **9999999**（特殊，按钮失效） |
| | `pm_rejob_uc_stop` | 无 |
| `rejob_uc_pop` | `pm_rejob_uc_labour` | 1 |
| | `pm_rejob_uc_barbed_wire_fences` | 2 |
| | `pm_rejob_uc_electric_fencing` | 4 |
| | `pm_rejob_uc_upper` | 无（另有职业级修正，见 §2.3） |
| `rejob_uc_migration` | `pm_rejob_uc_local` / `pm_rejob_uc_country` | 无 |

- **生产方式固定值 = camp 组的值 + pop 组的值**。
  例：`shao(4) + electric_fencing(4) = 8`；常规最大为 `gas(400) + electric_fencing(4) = 404`。
- 依据文件：`rejob_modify/common/production_methods/gulag_unemployment_center.txt`。

### 2.3 「建筑自身自带的」

- `building_gulag_unemployment_center` 与 `building_group gulag_unemployment_center`
  当前**没有** `state_mortality_mult`，故建筑自身贡献为 **0**。
- `pm_rejob_uc_upper` 的 `building_aristocrats_mortality_mult = 40` /
  `building_capitalists_mortality_mult = 40` 属于**职业级**修正
  （只影响该建筑的地主/资本家），**不是州级修正，不纳入本按钮控制范围**。

### 2.4 clamp 数学（本方案核心技巧）

设 `offset` = 玩家可调值，`pm_base` = 当前生产方式组合的固定值。
应用层只追加一个「补偿修正」，其数值为：

```
effective = max(offset, -pm_base)
```

于是：

```
总和 = pm_base + effective = max(pm_base + offset, 0)
```

- `offset = -4`、`pm_base = 4` → `effective = -4` → 总和 `0` ✅
- `offset = -99999`、`pm_base = 4` → `effective = -4` → 总和 `0`（自动封底）✅
- `offset = +100`、`pm_base = 4` → `effective = +100` → 总和 `104` ✅

**结论：完全不需要改动生产方式文件，即可实现「总和下限 0、上限无限」。**

---

## 3. 方案设计

### 3.1 数据模型

- 国家变量 `rejob_uc_mortality_offset`：玩家可调的增量，初始 `0`。
- 无其他持久状态；UI 显示值与生效值全部由脚本值实时算出。

### 3.2 新增文件清单（全部为新增，不覆盖任何原文件）

```
common/static_modifiers/rejob_uc_mortality_modifiers.txt
common/script_values/rejob_uc_mortality_values.txt
common/scripted_triggers/rejob_uc_mortality_triggers.txt
common/scripted_effects/rejob_uc_mortality_effects.txt
common/scripted_buttons/rejob_uc_mortality_buttons.txt
common/journal_entries/rejob_uc_mortality.txt
common/decisions/rejob_uc_mortality_decisions.txt      # §7.3 新增（决议法激活）
common/on_actions/rejob_uc_mortality_on_actions.txt    # §7.2 新增（自动法兜底）
localization/simp_chinese/rejob_uc_mortality_l_simp_chinese.yml
```

### 3.3 代码

以下代码与实际落地文件逐字一致。

#### ① `common/static_modifiers/rejob_uc_mortality_modifiers.txt`

```txt
rejob_uc_mortality_dynamic = {
	icon = gfx/interface/icons/timed_modifier_icons/modifier_fire_negative.dds
	state_mortality_mult = 1
}
```

> 数值固定为 1，实际生效值由 `multiplier = <脚本值>` 缩放。

#### ② `common/script_values/rejob_uc_mortality_values.txt`

```txt
# 州视角：该州再就业中心的生产方式固定死亡率之和
rejob_uc_mortality_pm_base_in_state = {
	value = 0
	if = { limit = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_shao } } add = 4 }
	if = { limit = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_gulag } } add = 4 }
	if = { limit = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_oil } } add = 4 }
	if = { limit = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_gas } } add = 400 }
	if = { limit = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_labour } } add = 1 }
	if = { limit = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_barbed_wire_fences } } add = 2 }
	if = { limit = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_electric_fencing } } add = 4 }
}

# 国家视角：用于 UI 显示基准（取任一拥有该建筑的州）
rejob_uc_mortality_pm_base_country = {
	value = 0
	if = { limit = { any_scope_state = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_shao } } } add = 4 }
	if = { limit = { any_scope_state = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_gulag } } } add = 4 }
	if = { limit = { any_scope_state = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_oil } } } add = 4 }
	if = { limit = { any_scope_state = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_gas } } } add = 400 }
	if = { limit = { any_scope_state = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_labour } } } add = 1 }
	if = { limit = { any_scope_state = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_barbed_wire_fences } } } add = 2 }
	if = { limit = { any_scope_state = { any_scope_building = { is_building_type = building_gulag_unemployment_center has_active_production_method = pm_rejob_uc_electric_fencing } } } add = 4 }
}

# UI 显示用：总和（下限 0）
rejob_uc_mortality_total = {
	value = var:rejob_uc_mortality_offset
	add = rejob_uc_mortality_pm_base_country
	min = 0
}

# 生产方式固定值取负（用于下面的下限比较）
rejob_uc_mortality_pm_base_in_state_negated = {
	value = rejob_uc_mortality_pm_base_in_state
	multiply = -1
}

# 应用层：追加到州上的补偿值 = max(offset, -pm_base)
rejob_uc_mortality_offset_effective = {
	value = owner.var:rejob_uc_mortality_offset
	min = rejob_uc_mortality_pm_base_in_state_negated
}
```


#### ③ `common/scripted_triggers/rejob_uc_mortality_triggers.txt`

```txt
rejob_uc_has_center = {
	any_scope_state = {
		any_scope_building = {
			is_building_type = building_gulag_unemployment_center
		}
	}
}

rejob_uc_has_special_pm = {
	any_scope_state = {
		any_scope_building = {
			is_building_type = building_gulag_unemployment_center
			has_active_production_method = pm_rejob_uc_gasking
		}
	}
}
```

#### ④ `common/scripted_effects/rejob_uc_mortality_effects.txt`

```txt
rejob_uc_refresh_mortality = {
	every_scope_state = {
		limit = {
			any_scope_building = {
				is_building_type = building_gulag_unemployment_center
			}
		}
		remove_modifier = rejob_uc_mortality_dynamic
		if = {
			limit = { rejob_uc_mortality_offset_effective != 0 }
			add_modifier = {
				name = rejob_uc_mortality_dynamic
				multiplier = rejob_uc_mortality_offset_effective
			}
		}
	}
}
```

#### ⑤ `common/scripted_buttons/rejob_uc_mortality_buttons.txt`

共 9 个按钮：`plus_1000 / plus_100 / plus_10 / plus_1 / minus_1 / minus_10 / minus_100 / minus_1000 / reset`。
正数按钮的 `possible` 只判断是否处于特殊生产方式；负数与重置按钮额外要求
`rejob_uc_mortality_total > 0` 或调节值非零。
以下给出 `plus_1000`、`minus_1000`、`reset` 三个代表（其余结构相同，仅 `add` 数值不同）：

```txt
rejob_uc_mortality_plus_1000 = {
	name = "rejob_uc_mortality_plus_1000"
	desc = "rejob_uc_mortality_plus_1000_desc"
	visible = { has_journal_entry = je_rejob_uc_mortality }
	possible = { NOT = { rejob_uc_has_special_pm = yes } }
	ai_chance = { value = 0 }
	effect = {
		change_variable = { name = rejob_uc_mortality_offset add = 1000 }
		rejob_uc_refresh_mortality = yes
	}
}

rejob_uc_mortality_minus_1000 = {
	name = "rejob_uc_mortality_minus_1000"
	desc = "rejob_uc_mortality_minus_1000_desc"
	visible = { has_journal_entry = je_rejob_uc_mortality }
	possible = {
		NOT = { rejob_uc_has_special_pm = yes }
		rejob_uc_mortality_total > 0
	}
	ai_chance = { value = 0 }
	effect = {
		change_variable = { name = rejob_uc_mortality_offset add = -1000 }
		rejob_uc_refresh_mortality = yes
	}
}

rejob_uc_mortality_reset = {
	name = "rejob_uc_mortality_reset"
	desc = "rejob_uc_mortality_reset_desc"
	visible = { has_journal_entry = je_rejob_uc_mortality }
	possible = {
		NOT = { rejob_uc_has_special_pm = yes }
		var:rejob_uc_mortality_offset != 0
	}
	ai_chance = { value = 0 }
	effect = {
		set_variable = { name = rejob_uc_mortality_offset value = 0 }
		rejob_uc_refresh_mortality = yes
	}
}
```


#### ⑥ `common/journal_entries/rejob_uc_mortality.txt`

```txt
je_rejob_uc_mortality = {
	icon = "gfx/interface/icons/event_icons/event_skull.dds"
	group = je_group_internal_affairs

	scripted_button = rejob_uc_mortality_plus_1000
	scripted_button = rejob_uc_mortality_plus_100
	scripted_button = rejob_uc_mortality_plus_10
	scripted_button = rejob_uc_mortality_plus_1
	scripted_button = rejob_uc_mortality_minus_1
	scripted_button = rejob_uc_mortality_minus_10
	scripted_button = rejob_uc_mortality_minus_100
	scripted_button = rejob_uc_mortality_minus_1000
	scripted_button = rejob_uc_mortality_reset

	immediate = {
		if = {
			limit = { NOT = { has_variable = rejob_uc_mortality_offset } }
			set_variable = {
				name = rejob_uc_mortality_offset
				value = 0
			}
		}
		rejob_uc_refresh_mortality = yes
	}

	status_desc = {
		desc = rejob_uc_mortality_status
	}

	on_monthly_pulse = {
		effect = {
			rejob_uc_refresh_mortality = yes
		}
	}

	should_be_pinned_by_default_uninvolved_or_context = yes
	weight = 10000
}
```

> **注意**：本 JE **故意不写** `is_shown_when_inactive` / `possible`。
> 它由 §3.3 ⑦ 的决议、或 ⑧ 的 on_action 显式 `add_journal_entry` 添加（OGAS 同款做法）。

#### ⑦ `common/decisions/rejob_uc_mortality_decisions.txt`

```txt
rejob_uc_mortality_decision = {
	is_shown = {
		is_player = yes
		any_scope_state = {
			has_building = building_gulag_unemployment_center
		}
	}

	possible = {
		always = yes
	}

	when_taken = {
		add_journal_entry = {
			type = je_rejob_uc_mortality
		}
	}
}
```

#### ⑧ `common/on_actions/rejob_uc_mortality_on_actions.txt`（自动兜底）

```txt
on_monthly_pulse_country = {
	on_actions = {
		rejob_uc_mortality_monthly
	}
}

rejob_uc_mortality_monthly = {
	effect = {
		if = {
			limit = {
				NOT = { has_journal_entry = je_rejob_uc_mortality }
				any_scope_state = {
					has_building = building_gulag_unemployment_center
				}
			}
			add_journal_entry = {
				type = je_rejob_uc_mortality
			}
		}
	}
}
```

#### ⑨ `localization/simp_chinese/rejob_uc_mortality_l_simp_chinese.yml`

```yml
l_simp_chinese:
 je_rejob_uc_mortality:0 "再就业中心死亡率调节"
 je_rejob_uc_mortality_reason:0 "调整再就业中心在所在州造成的额外死亡率。生产方式自带的基础值仍然生效，此处增减的是叠加其上的补偿量；合计最低为 0，最高不限。"
 rejob_uc_mortality_status:0 "当前死亡率修正合计：#v [GetPlayer.MakeScope.ScriptValue('rejob_uc_mortality_total')|0%]#!（生产方式基础值 #v [GetPlayer.MakeScope.ScriptValue('rejob_uc_mortality_pm_base_country')|0%]#! ＋ 调节值 #v [GetPlayer.MakeScope.Var('rejob_uc_mortality_offset').GetValue|0%]#!）"
 rejob_uc_mortality_plus_1:0 "+1"
 rejob_uc_mortality_plus_1_desc:0 "将死亡率调节值增加 1。"
 rejob_uc_mortality_plus_10:0 "+10"
 rejob_uc_mortality_plus_10_desc:0 "将死亡率调节值增加 10。"
 rejob_uc_mortality_plus_100:0 "+100"
 rejob_uc_mortality_plus_100_desc:0 "将死亡率调节值增加 100。"
 rejob_uc_mortality_plus_1000:0 "+1000"
 rejob_uc_mortality_plus_1000_desc:0 "将死亡率调节值增加 1000。"
 rejob_uc_mortality_minus_1:0 "-1"
 rejob_uc_mortality_minus_1_desc:0 "将死亡率调节值减少 1。"
 rejob_uc_mortality_minus_10:0 "-10"
 rejob_uc_mortality_minus_10_desc:0 "将死亡率调节值减少 10。"
 rejob_uc_mortality_minus_100:0 "-100"
 rejob_uc_mortality_minus_100_desc:0 "将死亡率调节值减少 100。"
 rejob_uc_mortality_minus_1000:0 "-1000"
 rejob_uc_mortality_minus_1000_desc:0 "将死亡率调节值减少 1000。"
 rejob_uc_mortality_reset:0 "重置调节"
 rejob_uc_mortality_reset_desc:0 "将调节值归零，合计回到生产方式基础值。"
```

> `|0%` 会把数值按百分比显示（`4` → `400%`），与游戏内 `state_mortality_mult` 提示一致。

---

## 4. 兼容性分析（「不破坏原有功能」的论证）

| 关注点 | 结论 |
|---|---|
| 是否修改生产方式文件 | ❌ 完全不改，`state_mortality_mult` 原样保留 |
| 是否修改建筑 / 建筑组 | ❌ 不改 |
| 是否修改原有决议 / 事件 / 脚本值 | ❌ 不改（新增文件独立，ID 全部带前缀） |
| ID 冲突 | 全部使用 `rejob_uc_mortality_*` / `je_rejob_uc_mortality` 前缀，无冲突 |
| 默认状态行为 | 初始 `offset = 0` → `effective = 0` → 总和 = 生产方式基础值，**与原行为完全一致** |
| 对同州其他人口的影响 | 与原有生产方式行为相同（`state_mortality_mult` 本就是州级），**未扩大影响范围** |
| 玩家未建中心 | 日志不显示（`rejob_uc_has_center` 为假），零副作用 |
| 特殊生产方式 | 选 `pm_rejob_uc_gasking` 时所有按钮 `possible` 为假 → 灰掉，原 9999999 行为不受影响 |

---

## 5. 已知限制与风险（需游戏内实测）

1. `multiplier = <脚本值>` 的刷新时机：切换生产方式后由 `on_monthly_pulse` 同步，
   **最长延迟 1 个月**；点击按钮则立即生效。
2. 同国多州各建中心且生产方式不同时，UI 显示取「任一拥有该建筑的州」为基准，
   **不逐一区分**（设计取舍；实际场景通常一致）。
3. `state_mortality_mult` 是**州级**修正，会影响该州**所有人口**
   （这是原模组既有特性，非本方案引入）。
4. `pm_rejob_uc_upper` 的 `building_aristocrats/capitalists_mortality_mult = 40`
   属职业级修正，**不在按钮控制范围内**。
5. 本方案为静态代码推导，**尚未进行游戏内运行验证**。

---

## 6. 测试计划

### 6.1 激活验证（关键，先做这一步）
1. 建造「失业人口再就业中心」。
2. **方式 A（决议法，立即生效）**：在「决议」面板点击
   「设立再就业中心死亡率调节委员会」→ 日志应立刻出现。
3. **方式 B（自动兜底）**：不点决议，等 1 个月脉冲 → 日志应自动出现。
4. 日志出现后应默认置顶（pinned，因 `weight = 10000`）。

### 6.2 功能验证
5. 默认状态：UI 显示值 = 生产方式基础值，与原版一致（**回归验证**）。
6. 依次点击 `+1 / +10 / +100 / +1000`，UI 与州修正同步上升。
7. 点击负数按钮直到 UI 显示 0，继续点负数 → UI 保持 0（**封底验证**）。
8. 切换 `pm_rejob_uc_gas`（400）→ 一个月内 UI 更新为 400；再点负数可归 0。
9. 切换 `pm_rejob_uc_gasking`（9999999）→ 所有按钮灰掉不可点。
10. 点击「重置调节」→ 回到生产方式基础值。
11. 检查 `error.log` 无脚本值 / 触发器报错。

### 6.3 存档验证法（推荐，最可靠）
解包 `save games\autosave_exit.v3` 的 `gamestate`，检索：
- `je_rejob_uc_mortality` 应 ≥ 2
- **`rejob_uc_mortality_offset` 应 ≥ 1**（出现即代表 `immediate` 已执行、日志已激活）

---

## 7. 实施状态与备选方案

**已实施**：§3.2 的 7 个文件已全部创建，编码为 UTF-8 带 BOM，花括号已校验平衡。

### 7.1 首次测试失败与修复记录（2026-10-11）

**现象**：把文件放进游戏后，日志里**完全不显示**该条目，也没有任何按钮。

**排查过程**：
1. 确认文件已同步到 `Documents\Paradox Interactive\Victoria 3\mod\rejob_modify`（MD5 一致）
2. 确认 mod 已启用（`content_load.json` 含该路径；`debug.3.log` 显示
   `Mod 再就业中心2.0 - 个人优化补丁 ... successfully matched game version 1.13.11`
   且 `Mounted Data: .../mod/rejob_modify`）
3. 在 `logs\debug.1.log` 与 `logs\error.3.log` 找到真正原因：

```
Error: "Unexpected token: base, near line: 6" in file: "common/scripted_buttons/rejob_uc_mortality_buttons.txt" near line: 6
（共 9 条，分别对应 9 个按钮的 ai_chance 行）
```

**根因**：游戏自带文档 `common/scripted_buttons/scripted_buttons.md` 中写的
`ai_chance = { base = 0 }` 是**错误的**。实际引擎只接受 `value`，
原版全部 296 处 `ai_chance` 使用的都是 `ai_chance = { value = 0 }`，
`base` 在整个原版 `scripted_buttons` 目录中出现 **0 次**。

由于解析失败，**整个按钮文件失效** → 9 个按钮全部未定义 →
日志中 `scripted_button = ...` 引用到空按钮 → **整条日志加载失败、不显示**。

**修复内容**：
1. `ai_chance = { base = 0 }` → `ai_chance = { value = 0 }`（9 处）
2. `NOT = { var:rejob_uc_mortality_offset = 0 }` → `var:rejob_uc_mortality_offset != 0`
   （原版无前者用法，后者为标准写法）
3. JE 的 `is_shown_when_inactive` 中 `is_ai = no` → `is_player = yes`
   （原版 JE 该处使用 `is_player`）
4. 本地化取值改用 JE 场景的标准写法
   `[SCOPE.GetRootScope.GetCountry.MakeScope.ScriptValue(...)]`
   （参照原版 `je_dreyfus_status`）
5. 为 `rejob_migration_values.txt`、`rejob_migration_decisions.txt`、
   `rejob_migration_events.txt` 补上 UTF-8 BOM（消除 lexer 警告）

**结论**：修复后所有脚本文件 BOM 正确、花括号平衡、无 `base` 残留。
需重新启动游戏验证（见 §6）。

### 7.2 第二次测试失败与修复记录（2026-10-11，根因：日志未被激活）

**现象**：修复 `base` 之后，日志中**已无任何本模组报错**，但游戏里**依然不显示**该日志。

**决定性诊断方法（推荐后续复用）**：
`Documents\Paradox Interactive\Victoria 3\save games\autosave_exit.v3` 实际是
「2162 字节头部 + ZIP」，其中 ZIP 内仅含一个 `gamestate` 条目（约 297MB 纯文本）。
提取后直接检索关键字符串即可判断运行时状态：

```powershell
# 1) 定位 PK 头，切出 zip
$z="$env:USERPROFILE\Documents\Paradox Interactive\Victoria 3\save games\autosave_exit.v3"
$b=[IO.File]::ReadAllBytes($z); $pk=-1
for($i=0;$i -lt 2000000;$i++){ if($b[$i] -eq 80 -and $b[$i+1] -eq 75 -and $b[$i+2] -eq 3 -and $b[$i+3] -eq 4){ $pk=$i; break } }
$o=New-Object byte[] ($b.Length-$pk); [Array]::Copy($b,$pk,$o,0,$o.Length)
[IO.File]::WriteAllBytes("$env:TEMP\s.zip",$o)
# 2) 解出 gamestate 后检索
findstr /C:"je_rejob_uc_mortality" "$env:TEMP\gamestate.txt"
findstr /C:"rejob_uc_mortality_offset" "$env:TEMP\gamestate.txt"
```

**诊断结果**：

| 检索项 | 次数 | 含义 |
|---|---|---|
| `building_gulag_unemployment_center` | 472 | 建筑确实已建造 |
| `je_rejob_uc_mortality` | 2 | 日志仅存在于「定义表」，**未进入任何国家的激活列表** |
| `rejob_uc_mortality_offset` | **0** | `immediate` **从未执行** → 日志从未激活 |
| `je_OGAS_building_weight_manager` | 4 | 对照组：正常工作的日志 |

**根因**：仅靠 `is_shown_when_inactive` + `possible` 的「自动激活」在本 mod 组合
（7 个模组同时启用）下**没有生效**。该自动激活机制要求引擎每帧评估所有未激活日志，
在日志数量庞大时并不可靠。

**修复**：改用官方文档（`_on_actions.md`）明确支持的 **on_action 追加**方式，
由 `on_monthly_pulse_country` 主动调用 `add_journal_entry` 强制添加。
该写法不会覆盖原版 on_action（使用 `on_actions = { ... }` 子列表），
与第三方模组 `mr_arts_artists_on_actions.txt` 的做法一致。

新增文件 `common/on_actions/rejob_uc_mortality_on_actions.txt`：

```txt
on_monthly_pulse_country = {
	on_actions = {
		rejob_uc_mortality_monthly
	}
}

rejob_uc_mortality_monthly = {
	effect = {
		if = {
			limit = {
				NOT = { has_journal_entry = je_rejob_uc_mortality }
				any_scope_state = {
					has_building = building_gulag_unemployment_center
				}
			}
			add_journal_entry = {
				type = je_rejob_uc_mortality
			}
		}
	}
}
```

> 注：`on_monthly_pulse_country` 已被原版与其他模组用于 `on_actions = { ... }` 子列表
> （如原版 `coup_monthly_events`、第三方 `artists_on_monthly_pulse_country`），
> 引擎会把所有来源的子项合并执行，因此本文件不会造成覆盖冲突。

### 7.3 第三次修复：改用 OGAS 模式（2026-10-11，根因：自动激活机制本身不可靠）

**参考对象**：`E:\Project\Mod\victoria3\3092955121`（The OGAS，**已确认游戏内正常工作**）

**关键对比**（这是本次最重要的发现）：

| 项 | OGAS（可用） | 本方案原实现（不可用） |
|---|---|---|
| JE 如何出现 | **决议 `when_taken` → `add_journal_entry`** | `is_shown_when_inactive` 自动激活 |
| JE 是否有 `is_shown_when_inactive` | **完全没有** | 有 |
| JE 是否有 `possible` | **完全没有** | 有 |
| JE 的 `immediate` | 调用 `cnm_default_pm_manager` 初始化变量 | 初始化变量 |
| 按钮是否有 `visible`/`ai_chance` | **都没有** | 都有 |
| 按钮内取值写法 | `root.var:cnm_pm_manage_amount` | `var:rejob_uc_mortality_offset` |
| 按钮内增减 | `change_variable = { add/subtract }`，用 `if` 包住 | 同 |
| 重置按钮 | `reset_pm_manager` → 调用初始化 effect | 同 |
| loc 取值 | `[Scope.GetCountry.MakeScope.Var('x').GetValue]` | `[SCOPE.GetRootScope...]` |
| `weight` | **10000** | 100 |

**OGAS 的 JE 原文**（`common/journal_entries/cnm_pm_manager.txt`）：
```txt
je_cnm_pm_manager = {
	icon = "gfx/interface/icons/event_icons/event_industry.dds"
	group = je_group_internal_affairs

	scripted_button = active_workforce_employment_pm_manager
	scripted_button = pm_manage_frequency_switch
	scripted_button = pm_manage_amount_add
	scripted_button = pm_manage_amount_sub
	scripted_button = reset_pm_manager
	immediate = {
		cnm_default_pm_manager = yes
	}
	status_desc = {
		desc = cnm_pm_manager_status_desc
	}
	...
	should_be_pinned_by_default_uninvolved_or_context = yes
	weight = 10000
}
```

**OGAS 的决议原文**（`common/decisions/cnm_ResetOGAS_decision.txt`）：
```txt
cnm_Reset_decision = {
	is_shown = {
		is_player = yes
	}
	possible = {
		always = yes
	}
	when_taken = {
		add_journal_entry = {
			type = je_cnm_pm_manager
		}
		...
	}
}
```

**结论**：`is_shown_when_inactive` + `possible` 的自动激活机制在本 mod 组合下**确实不工作**
（存档已证明：JE 存在于定义表，但从未进入任何国家的激活列表）。
**必须**像 OGAS 那样，由决议（或 on_action）显式 `add_journal_entry`。

**本次改动**：
1. **新增** `common/decisions/rejob_uc_mortality_decisions.txt` —— 玩家可见的决议，
   `when_taken` 中 `add_journal_entry = { type = je_rejob_uc_mortality }`
2. **删除** JE 中的 `is_shown_when_inactive` 与 `possible`（对齐 OGAS）
3. JE 的 `weight` 改为 `10000`（对齐 OGAS，确保置顶）
4. 按钮改为 OGAS 风格：去掉 `ai_chance`，变量取值改用 `root.var:`，
   负数按钮用 `if = { limit = { root.var:x > -10000 } }` 包住
5. loc 改用 `[Scope.GetCountry.MakeScope...]`（OGAS 验证过的写法）
6. 保留 `common/on_actions/rejob_uc_mortality_on_actions.txt` 作为**双保险**
   （决议没点到时，每月脉冲也会自动补上）

**使用方式（两种任选其一）**：
- **决议法**：在「决议」面板点击「设立再就业中心死亡率调节委员会」→ 日志立即出现
- **自动法**：建造再就业中心后，最迟 1 个月内日志自动出现（on_action 兜底）

### 7.4 备选方案

若 `multiplier` 引用脚本值出现异常，可改为在按钮 `effect` 中用
`if` 分支直接写死 `effective` 常量（按当前生产方式组合枚举），
该退路不依赖 `multiplier` 的脚本值解析。

---

## 附：关键证据索引

| 结论 | 证据文件 |
|---|---|
| 日志语法（is_shown_when_inactive / possible / status_desc / scripted_button / on_monthly_pulse） | `game/common/journal_entries/journal_entries.md` |
| 按钮语法（name / desc / visible / possible / effect / ai_chance） | `game/common/scripted_buttons/scripted_buttons.md` |
| 脚本值语法（min / if / value / add，按序执行） | `game/common/script_values/script_values.md` |
| `state_mortality_mult` 定义（percent=yes, color=bad） | `game/common/modifier_type_definitions/00_modifier_types.txt:2290` |
| `add_modifier` + `multiplier` + 先 remove 后 add 的范例 | `game/common/journal_entries/05_montenegro_je.txt:113-125` |
| `status_desc = { desc = KEY }` 简写范例 | `game/common/journal_entries/05_montenegro_je.txt:213-215` |
| `is_building_type` + `has_active_production_method` 范例 | `game/common/history/conscription/00_conscription_center.txt:5-7` |
| `je_group_internal_affairs` 定义（context = country） | `game/common/journal_entry_groups/00_journal_entries.txt:112` |
| 本地化 `|0%` 百分比格式范例 | `game/localization/english/acw_text_l_english.yml:140` |
| 再就业中心生产方式与死亡率数值 | `rejob_modify/common/production_methods/gulag_unemployment_center.txt` |
| 建筑定义（无 state_mortality_mult） | `rejob_modify/common/buildings/gulag_unemployment_center.txt` |

