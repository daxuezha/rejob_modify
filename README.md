# 再就业中心 整合版

面向《Victoria 3》的中文模组。把 Steam 创意工坊模组「再就业中心 2.0」与本仓库原有的优化补丁**合并为一个自包含模组**：只订阅本模组即可获得两者的全部内容，无需再启用原模组。

当前版本：**2.0.0** ｜ 支持游戏版本：**1.13.\*** ｜ 语言：简体中文

---

## 目录

- [1. 模组做了什么](#1-模组做了什么)
- [2. 快速上手](#2-快速上手)
- [3. 核心界面：失业人群管理中心](#3-核心界面失业人群管理中心)
- [4. 两座建筑](#4-两座建筑)
- [5. 不可接触者（delicious）](#5-不可接触者delicious)
- [6. 死亡率调节机制](#6-死亡率调节机制)
- [7. 决议按钮](#7-决议按钮)
- [8. 其他内容](#8-其他内容)
- [9. 与旧版/原模组的差异](#9-与旧版原模组的差异)
- [10. 目录结构](#10-目录结构)
- [11. 常见问题](#11-常见问题)
- [12. 开发记录](#12-开发记录)
- [13. 修复记录（2026-10-11 第二轮）](#13-修复记录2026-10-11-第二轮)

---

## 1. 模组做了什么

本模组提供一套「社会清洗 + 人口管理」玩法，围绕两座建筑展开：

| 建筑 | 定位 | 招募对象 |
|---|---|---|
| **再就业中心** | 面向失业人口的正规就业设施 | 失业的普通职业人口 |
| **集中营** | 关押并处理「不可接触者」 | 不可接触者（delicious） |

两座建筑都可以通过生产方式切换产出、岗位与服务范围，并且**都能被同一个调节面板统一调整其在所在州造成的额外死亡率**。

主要功能一览：

- **失业人群管理中心**（日志条目）：一个面板同时管理死亡率调节、生效开关与全部决议按钮。
- **死亡率调节**：以 10% 为步长上下调整，合计最低为 0，最高不限。
- **双开关机制**：可分别决定调节值是否作用于「再就业中心」「集中营」，支持同时、单独或都不作用。
- **决议按钮化**：原本在「决议」面板里的指令（66 号、67 号、迁移失业人口）全部搬进日志，做成按钮。
- **不可接触者**：新增 `delicious` 人口类型，配套转化与恢复机制。
- **大迁徙法令**：提高州的移民吸引力。
- **文化社区权重调整**：与原模组一致的整套权重脚本。

---

## 2. 快速上手

1. 订阅 / 安装本模组，在启动器中**只启用本模组**（不要同时启用原「再就业中心 2.0」，两者内容重复）。
2. 进入游戏，建造「再就业中心」或「集中营」。
3. 打开**决议**面板，点击 **「设立失业人群管理中心」**，日志即会出现。
   - 如果没点这个决议，建成建筑后**最迟一个月**日志也会自动出现。
4. 在日志里按需调节。

> 提示：日志默认会固定在右侧追踪栏（pinned），因为它的 `weight` 设得很高。


---

## 3. 核心界面：失业人群管理中心

日志条目 `je_rejob_management`，界面上的按钮按下面三组从上到下排列：

```
┌─────────────────────────────────────────────┐
│  A 组 · 死亡率调节                          │
│  [+1000%] [+100%] [+10%]                    │
│  [-10%] [-100%] [-1000%] [重置调节值]        │
├─────────────────────────────────────────────┤
│  B 组 · 生效开关（默认均为关闭状态）          │
│  [调节作用于：再就业中心] [调节作用于：集中营] │
├─────────────────────────────────────────────┤
│  C 组 · 决议按钮                            │
│  [66号指令] [67号指令] [迁移失业人口至首都]   │
└─────────────────────────────────────────────┘
```

日志的状态栏会显示：

- **调节值**：当前设置的百分比（带正负号），对所有建筑统一。
- **再就业中心**：生产方式基准值、调节值，以及**合计死亡率**
  （`max(基准值 + 调节值, 0)`）。该修正为**建筑级**，只影响本建筑雇员。
- **集中营**：同上。该修正为**州级**，只影响不可接触者。
- 各建筑生产方式**不一致**时，显示为**范围值**（`min% ~ max%`）。
- 末尾附「作用范围」说明，注明建筑级 / 州级的区别与受影响职业。

### 置灰规则

| 情形 | 结果 |
|---|---|
| 两个生效开关都关闭（**默认**） | 所有调节按钮置灰 |
| 某建筑不存在 | 对应的生效开关置灰 |
| 某建筑处于「特殊生产方式」 | 对应开关仍可点，但调节按钮对它是无效的（见 §6.4） |

---

## 4. 两座建筑

### 4.1 再就业中心（`building_rejob_unemployment_center`）

- 建筑组 `rejob_unemployment_center`，父组为政府建筑组。
- 只允许玩家建造，禁止私人建造，**只从失业人口池招募**（`hires_unemployed_only = yes`）。
- 三组生产方式：**生产项目**、**岗位构成**、**服务范围**。

生产项目（`rejob_uc_camp`）：

| 生产方式 | 投入 | 产出 |
|---|---|---|
| 瓷器生产 | 煤 200 | 瓷器 500 |
| 肉类加工 | 弹药 100、轻武器 100 | 肉类 4000 |
| 石油加工 | 工具 25、电力 150、发动机 25 | 石油 2000 |
| 化肥生产 | 电力 50、煤炭 150 | 化肥 1000 |
| 停止配套生产 | — | 压缩奢侈品产出 |

> **★ 2026-10-11 第九轮变更**：生产项目组**不再提供任何死亡率修正**
> （原瓷器/肉类/石油/化肥各带 400%，重型化学工艺带 9999999，均已删除）。
> 死亡率现在**只由「岗位构成」组**的生产方式提供。
> 「重型化学工艺」生产方式已按需求**删除**。

岗位构成（`rejob_uc_pop`）：

| 生产方式 | 岗位 | 建筑死亡率（仅本建筑内该职业） |
|---|---|---|
| 基础再就业岗位 | 劳动者 +100,000/级 | +100% |
| 铁丝网生产计划 | 劳动者 +200,000/级 | +200% |
| 电子围栏生产计划 | 劳动者 +400,000/级 | +400% |
| 专业阶层岗位 | 资本家 / 地主 各 +50,000/级 | +4000% |

服务范围（`rejob_uc_migration`）：服务当地（运输 20 + 移民吸引力 -99）、服务全国（运输 200，需铁路）。

> **说明（第四轮改造后）**：生产方式自带的死亡率**已按需求恢复为原模组值**，
> 它会与日志面板的调节值**相加**，构成建筑面板上显示的合计值。
> 详见 §6.5（500% 的来历）、§6.6（作用域）、§6.7（合计值）。

### 4.2 集中营（`building_gulag_logging`）

即原模组的「再就业中心」，本整合版按建筑本意改称**集中营**，相关生产方式、生产方式组文本同步改为集中营语境。

- 建筑组 `gulag_industry`，父组为政府建筑组。
- 只允许玩家建造，且**排除四种法律**：种族隔离、文化排斥、多元文化、臣属关系。
- 三组生产方式：**切换运转方式**、**切换关押人群**、**切换服务区域**。

生产项目（`gulag_camp`）：

| 生产方式 | 投入 | 产出 |
|---|---|---|
| 集中营:瓷器生产 | 煤 200 | 瓷器 500 |
| 集中营:肉类生产 | 弹药 100、轻武器 100 | 肉类 4000 |
| 集中营:炼油产业 | 工具 25、电力 150、发动机 25 | 石油 2000 |
| 集中营:化肥生产 | 电力 50、煤炭 150 | 化肥 1000 |
| 停止运转 | — | 压缩奢侈品产出 |

> **★ 2026-10-11 第九轮变更**：与再就业中心对称 ——
> 生产项目组**不再提供任何死亡率修正**，死亡率只由「切换关押人群」组提供。
> 「集中营:终极解决」生产方式已按需求**删除**；
> 「集中营:环保产业」改名为「**集中营:化肥生产**」，构成改为 50 电力 + 150 煤炭 → 1000 化肥。

关押人群（`gulag_pop`）：

| 生产方式 | 关押人数 | 州死亡率（不可接触者） |
|---|---|---|
| 基础关押 | 不可接触者 +100,000/级 | +100% |
| 集中营:铁丝网 | 不可接触者 +200,000/级 | +200% |
| 集中营:电子围栏 | 不可接触者 +400,000/级 | +400% |
| 上层关押 | 资本家 / 地主 各 +50,000/级 | 建筑级 +4000% |

> **说明**：集中营的死亡率走的是 `state_delicious_mortality_mult`，**只影响不可接触者**；而再就业中心走 `building_<职业>_mortality_mult`，**只影响该建筑内的对应职业**。这是两座建筑最本质的区别。

---

## 5. 不可接触者（delicious）

新增的失业型人口类型：

- 起始生活水平 1，工资权重 0，不领私人工资，工作年龄人口比 0.75。
- 基础政治参与度 0，但会受饥饿影响（轻度 +0.25，严重 +1）。
- 资格增长脚本与原模组一致：仅在所在州对该文化的接纳度低于 4 时生效。

---

## 6. 死亡率调节机制

### 6.1 单位

调节值使用修正类型的**原始单位**。`state_mortality_mult` 与 `state_delicious_mortality_mult` 的定义均为 `percent = yes`，引擎显示时会 ×100 并补 `%` 号：

| 变量值 | 界面显示 |
|---|---|
| `0.1` | +10% |
| `1.0` | +100% |
| `10.0` | +1000% |

即按钮上的 **`+10%` 就是真的加 10%**。

### 6.2 调节下限（合计最低为 0）

**合计不得为负**是本模组的硬性设计。实现方式：

```txt
单座建筑合计 = max( 生产方式基准值 + 调节值 , 0 )
```

- 调节值本身在写入修正前先做 `max(offset, 0)`，负值不产生效果。
- 所有合计脚本值都带 `min = 0`，即使基准值与调节值相加为负，也会截断为 0。

举例：生产方式基准值 400%，调节值 -1000% → 合计 `max(400 + (-1000), 0) = 0`，不会变成负数。

### 6.3 生效开关（双开关）

两个开关**默认都是关闭的**，需要玩家自行点亮后调节值才会生效。

| 再就业中心 | 集中营 | 效果 |
|---|---|---|
| 亮 | 亮 | 调节值同时作用于两座建筑 |
| 亮 | 灭 | 只作用于再就业中心 |
| 灭 | 亮 | 只作用于集中营 |
| 灭 | 灭 | 都不作用（调节值保留，但不产生修正）**（默认状态）** |

**开关只控制「修改动作是否生效」**：点击开关不会改变调节值本身，也不会让已经写好的数值「变形」——对被关闭的建筑是撤销其动态修正，对被开启的建筑是重新施加。

### 6.4 特殊生产方式

以下生产方式自带 `state_mortality_mult = 9999999`（保留未移除，作为「终极手段」的设定值）：

- 再就业中心：**重型化学工艺**（`pm_rejob_uc_gasking`）
- 集中营：**终极解决**（`pm_Auschviking_gasking`）

此时：

1. **状态栏直接显示 9999999%**（真实生效值优先，而不是只显示调节值）。
2. **该建筑的调节通道被禁用** —— 调节按钮置灰，避免与这个极端值叠加。

### 6.5 「500%」到底从哪来？—— 它是算出来的，不是建筑自带的

**先给结论**：那 500% **不是建筑本身的属性**，而是**当前激活的生产方式各自带的
死亡率修正值相加**得到的。

`common/buildings/*.txt`（建筑定义）和 `common/building_groups/*.txt`（建筑组定义）
里**根本没有任何 mortality 字段**（已逐字核对）。所以：

```
建筑面板显示的死亡率（该建筑内对应职业）
    = 当前激活的各个生产方式的 mortality 修正之和
```

> ★ 第六轮起，再就业中心的生产方式把死亡率写在
> `building_modifiers.unscaled` 下的 `building_<职业>_mortality_mult`
> （**建筑级 · 按职业**），而不是 `state_modifiers`。

默认生产方式组合的算式：

| 生产方式组 | 默认选中 | 贡献 |
|---|---|---|
| 生产项目 `rejob_uc_camp` | 瓷器生产 `pm_rejob_uc_shao` | `building_laborers/aristocrats/capitalists_mortality_mult = 4` → **400%** |
| 岗位构成 `rejob_uc_pop` | 基础再就业岗位 `pm_rejob_uc_labour` | `building_laborers_mortality_mult = 1` → **100%** |
| 服务范围 `rejob_uc_migration` | 服务当地 `pm_rejob_uc_local` | 无 |
| **合计** | | **500%** |

集中营完全对称（`pm_Auschviking_shao` 400% + `pm_gulag_labour` 100% = **500%**）。

**你说的「800%」正好能证明它是加出来的**：只要把两个生产方式组各选一个高值的，和就会叠加。

| 生产项目 | 岗位构成 | 面板显示 |
|---|---|---|
| 瓷器生产（400%） | 基础岗位（100%） | 500% |
| 瓷器生产（400%） | 铁丝网（200%） | 600% |
| 瓷器生产（400%） | 电子围栏（400%） | **800%** |
| 化肥投入（40000%） | 电子围栏（400%） | 40400% |
| 停止配套生产（0） | 专业阶层岗位（0） | 0% |

可见面板上的数字**完全等于各生产方式修正值的算术和**，建筑本身没有任何贡献。

**为什么叫 `unscaled`（无等级修正）**：生产方式里 `building_modifiers` 分三层——
`workforce_scaled`（随雇员规模缩放）、`level_scaled`（随建筑等级缩放）、
`unscaled`（**两者都不随，是固定值**）。所以这 500% 是一个**不随建筑等级变化的固定值**。

> **注意**：这些值在第四轮改造中已**按需求恢复**（见 §6.7），
> 因为新的设计需要「生产方式基准值 + 日志调节值」共同构成建筑面板上的合计。

---

### 6.6 作用域：日志调节的到底是什么修正？

**结论（第六轮重做后）**：日志调节的是**建筑级的「按职业」死亡率修正**，
使用 `building_<职业>_mortality_mult` 系列。

| | `state_<职业>_mortality_mult` | `building_<职业>_mortality_mult`（本 mod 采用） |
|---|---|---|
| 挂载对象 | 州（state） | **建筑（building）** |
| 作用范围 | 该州**所有**该职业人口 | **仅在该建筑就业**的该职业人口 |
| 能否做到「只影响招募进来的人」 | **不能** | **能** |

**为什么必须用建筑级**（需求 1c）：

再就业中心雇佣的是 `laborers` / `aristocrats` / `capitalists`，
这些职业在州里**到处都是**。若挂州级修正，全州所有劳工/地主/资本家的
死亡率都会被改 —— 严重超范围。只有建筑级修正才等价于
「只影响被这个建筑招募的那部分人」。

**官方原生就在用这套机制**：

| 键 | 对应职业 | 官方用例 |
|---|---|---|
| `building_laborers_mortality_mult` | 劳工 | `common/production_methods/03_mines.txt:145` = `0.3` |
| `building_machinists_mortality_mult` | 机械师 | 同上 `:146` = `0.2` |
| `building_engineers_mortality_mult` | 工程师 | 同上 `:147` = `0.1` |
| `building_aristocrats_mortality_mult` | 地主 | 本 mod 登记（vanilla 未登记） |
| `building_capitalists_mortality_mult` | 资本家 | 本 mod 登记（vanilla 未登记） |

即官方语义就是「**这座煤矿里**的劳工死亡率 +30%」——正是我们要的。

> ⚠️ **一个必须记住的坑**：`state_<职业>_mortality_mult` 在**代码**里存在，
> 但官方**脚本**只给 8 个职业写了 `modifier_type_definitions`
> （`laborers` / `aristocrats` / `machinists` / `academics` / `farmers` /
> `peasants` / `slaves` / `soldiers`）。其余职业
> （`clerks` / `bureaucrats` / `capitalists` / `engineers` 等）
> 直接使用会报 `Unknown modifier type`，并**污染整个修正数据库**。
> 详见 §13.13 与 §11 的 FAQ。

**两座建筑的修正方案**：

| 建筑 | 修正类型 | 作用范围 |
|---|---|---|
| 再就业中心 | `building_laborers / aristocrats / capitalists _mortality_mult` | **仅该建筑内**被招募的这几类职业人口 |
| 集中营 | `state_delicious_mortality_mult` | 所在州的**不可接触者** —— 只有集中营会雇佣，故等价于只影响营内人群 |

> 集中营之所以仍是州级：`delicious` 是本 mod 自定义人口类型，
> 没有对应的 `building_delicious_mortality_mult`；而它**只**被集中营雇佣，
> 所以州级效果恰好等于「只影响营内人群」（原模组一直如此，工作正常）。

> **注意**：建筑**未雇佣**的职业不受影响。例如选了「基础再就业岗位」（只雇佣劳工），
> 则地主/资本家即使容器里有对应键，也不会产生效果（因为该建筑里没有这些人口）。

---

### 6.7 建筑面板上的合计值（生产方式基准 + 日志调节）

**需求**：建筑面板上显示的死亡率 = **当前生产方式自带的死亡率** + **日志调节值**，
且**合计不得为负**（最小为 0）。

**实现**（对应你的第 5 点思路）：

```txt
合计 = 生产方式基准值 + 调节值
若合计 < 0 → 合计 = 0                    ← 由 min = 0 实现
写入州的补偿量 = max(调节值, -生产方式基准值)  ← 由 min = -pm_base 实现
```

**两条公式各司其职**：

| 公式 | 作用 | 若写错的后果 |
|---|---|---|
| `min = 0`（合计值） | 保证**显示**的合计不为负 | 会出现负死亡率显示 |
| `min = -pm_base`（补偿量） | 保证**负调节**能抵消 PM 自带值 | **负调节完全失效，按钮点了没反应**（第四轮的 bug） |

举例（生产方式基准值 400%）：

| 调节值 | 补偿量 | 合计 |
|---|---|---|
| +100% | +100% | 500% |
| -200% | -200% | 200% |
| -400% | -400% | 0% |
| -1000% | **-400%**（被 clamp） | 0% |

**日志状态栏的显示规则**：

| 情形 | 显示 |
|---|---|
| 所有建筑生产方式**一致** | 显示**单一**合计值：`生产方式 X%　合计 Z%` |
| 各建筑生产方式**不一致** | 显示**范围值**：`合计范围 Z_min% ~ Z_max%` |

判断方式：比较 `rejob_uc_total_min_country` 与 `rejob_uc_total_country` 是否相等
（用 `EqualTo_CFixedPoint`）。范围值的实现是先对基准值取 `min`，再加调节值并 clamp(0)。

**状态栏示例**：

```
调节值：+50%

再就业中心（州级 · 按职业）
　　生产方式：400%　合计：450%

集中营（州级 · 不可接触者）
　　各建筑生产方式不一致，合计范围：100% ~ 450%

作用范围：再就业中心的修正挂在建筑本身，只影响被它招募的劳工 / 地主 / 资本家，
不会波及同州其他建筑的同类职业；集中营作用于所在州的不可接触者。
合计 = 生产方式自带值 + 本面板调节值，最低为 0。
```

---

### 6.8 生产方式修正值对照（代码）

> **历史说明**：第三轮曾按需求把 PM 自带死亡率删成 0；
> 第四轮按需求「保持和一开始一样」**全部恢复**为原模组值；
> **第六轮按需求 1a 把再就业中心的死亡率从「州级不分职业」改为「建筑级按职业」**。
> 数值本身未变，只改了修正类型与所在区块。

#### `common/production_methods/rejob_unemployment_center_pm.txt`

> 第六轮起：死亡率从 `state_modifiers.unscaled` 移到
> `building_modifiers.unscaled`，并使用 `building_<职业>_mortality_mult`。

```txt
# --- pm_rejob_uc_shao（瓷器生产）---
	building_modifiers = {
		unscaled = {
			building_laborers_mortality_mult = 4
			building_aristocrats_mortality_mult = 4
			building_capitalists_mortality_mult = 4
		}
	}

# --- pm_rejob_uc_gulag（肉类加工）---
	building_modifiers.unscaled = { 同上三键 = 4 }

# --- pm_rejob_uc_oil（石油加工）---
	building_modifiers.unscaled = { 同上三键 = 4 }

# --- pm_rejob_uc_gas（化肥投入）---
	building_modifiers.unscaled = { 同上三键 = 400 }

# --- pm_rejob_uc_gasking（重型化学工艺）★特殊★ ---
	building_modifiers.unscaled = { 同上三键 = 9999999 }

# --- pm_rejob_uc_labour（基础再就业岗位）---
	building_modifiers.unscaled = { building_laborers_mortality_mult = 1 }

# --- pm_rejob_uc_barbed_wire_fences（铁丝网生产计划）---
	building_modifiers.unscaled = { building_laborers_mortality_mult = 2 }

# --- pm_rejob_uc_electric_fencing（电子围栏生产计划）---
	building_modifiers.unscaled = { building_laborers_mortality_mult = 4 }

# --- pm_rejob_uc_upper（专业阶层岗位）---
	building_modifiers = {
		level_scaled = {
			building_employment_aristocrats_add = 50000
			building_employment_capitalists_add = 50000
		}
		unscaled = {
			building_aristocrats_mortality_mult = 40
			building_capitalists_mortality_mult = 40
		}
	}

# --- pm_rejob_uc_local（服务当地）---
	state_modifiers.unscaled = { state_migration_pull_mult = -99 }
```

#### `common/production_methods/rejob_old_building_pm.txt`

> 集中营保持原样（`state_delicious_mortality_mult`，州级）——
> `delicious` 是自定义人口，没有建筑级版本；且只有集中营会雇佣它。

```txt
# --- pm_Auschviking_shao / pm_gulag / pm_gulag_oil（瓷器/肉类/炼油）---
+			state_delicious_mortality_mult = 4

# --- pm_Auschviking_gas（环保产业）---
+			state_delicious_mortality_mult = 400

# --- pm_Auschviking_gasking（终极解决）★特殊★ ---
 			state_mortality_mult = 9999999

# --- pm_Auschviking_barbed_wire_fences（铁丝网）---
+			state_delicious_mortality_mult = 2

# --- pm_Auschviking_electric_fencing（电子围栏）---
+			state_delicious_mortality_mult = 4

# --- pm_gulag_labour（基础关押岗位）---
+			state_delicious_mortality_mult = 1

# --- pm_gulag_upper（专业阶层岗位）---
		unscaled = {
+			building_aristocrats_mortality_mult = 40
+			building_capitalists_mortality_mult = 40
		}

# --- pm_gulag_local（服务当地）---
+			state_migration_pull_mult = -99
```

> **注意**：两座建筑的数值完全对称（4 / 4 / 4 / 400 / 1 / 2 / 4），
> 只是键名不同 —— 再就业中心用 `state_mortality_mult`（影响全州人口），
> 集中营用 `state_delicious_mortality_mult`（只影响不可接触者）。

---

## 7. 决议按钮

原本位于「决议」面板的指令，现已全部改为日志按钮。条件不满足时按钮自动置灰。

### 66 号指令 - 不可接触者

- **花费**：50,000 国库、10 恶名。
- **条件**：玩家国家；未采用种族隔离 / 文化排斥 / 多元文化 / 臣属关系；存在基础文化接纳度低于 4 的文化。
- **效果**：把人口规模小于 3,000 且所在州对其文化接纳度低于 4 的群体转为不可接触者并迁往首都；在首都建立一座集中营；清除首都 1% 的不可接触者。

### 67 号指令 - 冤假错案

- **花费**：5,000 国库。
- **条件**：存在所在州文化接纳度高于 3 的不可接触者。
- **效果**：这些人恢复为劳动者，并获得财富分布 10 的安置。

### 迁移失业人口至首都

- **条件**：非首都州的失业人口估算值 > 0。
- **效果**：打开迁移批次选择窗口，可选择迁移 100 万 ~ 1000 万人或全部。
- **费用**：按实际迁移量每人 1 国库单位。

> 该估算值为「非首都州总人口 × 失业率」的近似值，实际合格人数可能更少（事件内还会排除不可接触者）。

---

## 8. 其他内容

### 大迁徙（法令）

- **需求**：浪漫主义科技。
- **成本**：200 权威。
- **效果**：州迁徙吸引力 +20、倍率 +1。
- 同时会把文化社区的生成权重大幅提高。

### 未并入州处理

州所有权变更后，若新所有者是玩家且该州未并入，自动移除军营、征兵中心、海军基地。

### 文化社区权重

与原模组一致的整套 `cultural_community_creation_weight` 脚本，并叠加「大迁徙」法令的 +1 权重。


---

## 9. 与旧版/原模组的差异

### 相对原模组「再就业中心 2.0」

| 项目 | 原模组 | 本整合版 |
|---|---|---|
| 建筑名称 | 再就业中心 | **集中营**（按建筑本意更名） |
| 依赖关系 | — | **完全自包含**，不再依赖原模组 |
| 死亡率调节 | 无 | **有**，统一面板 + 双开关 |
| 决议位置 | 决议面板 | **日志按钮** |
| 生产方式数值 | 原始值 | **与原模组一致**（第四轮已恢复） |
| 修正作用域 | 生产方式自带，作用于州 | 日志调节为**州级按职业**（再就业中心）/ 州级 delicious（集中营） |
| 面板合计值 | 仅生产方式值 | **生产方式值 + 日志调节值**，下限 0，可显示范围 |

### 相对本仓库旧补丁版

| 项目 | 旧补丁版 | 本整合版 |
|---|---|---|
| 是否需要原模组 | 需要 | **不需要** |
| 新建筑名称 | 失业人口再就业中心 | **再就业中心** |
| 脚本 ID 前缀 | `gulag_unemployment_center` | **`rejob_unemployment_center`** |
| 调节按钮 | +1/+10/+100/+1000 | **+10%/+100%/+1000% 与对应负值** |
| 调节单位 | 修正原始值（显示为 ×100） | **百分点（+10 即 +10%）** |
| 生效范围 | 只能作用于新建筑 | **双开关，可分别作用于两座建筑** |
| 决议 | 独立决议 | **全部按钮化** |

---

## 10. 目录结构

```
rejob_modify/
├── .metadata/
│   └── metadata.json                    # 模组元数据（自包含，无外部依赖）
├── common/
│   ├── building_groups/
│   │   ├── rejob_unemployment_center.txt    # 再就业中心建筑组
│   │   └── rejob_old_gulag_industry.txt     # 集中营建筑组
│   ├── buildings/
│   │   ├── rejob_unemployment_center.txt    # 再就业中心
│   │   └── rejob_old_gulag_logging.txt      # 集中营
│   ├── decisions/
│   │   └── rejob_management_decisions.txt   # 开启管理面板的引导决议
│   ├── decrees/
│   │   └── rejob_old_greener_grass_decree.txt   # 大迁徙法令
│   ├── defines/
│   │   └── rejob_old_defines.txt            # 迁徙上限定义
│   ├── journal_entries/
│   │   └── rejob_management.txt             # 失业人群管理中心
│   ├── modifier_type_definitions/
│   │   └── rejob_old_mortality.txt          # 自定义修正类型
│   ├── on_actions/
│   │   ├── rejob_management_on_actions.txt  # 日志自动开启
│   │   └── rejob_old_bingyin_on_actions.txt # 未并入州建筑移除
│   ├── pop_types/
│   │   └── rejob_old_delicious.txt          # 不可接触者
│   ├── production_method_groups/
│   │   ├── rejob_unemployment_center.txt    # 再就业中心 PM 组
│   │   └── rejob_old_gulag_method_group.txt # 集中营 PM 组
│   ├── production_methods/
│   │   ├── rejob_unemployment_center_pm.txt # 再就业中心 PM
│   │   └── rejob_old_building_pm.txt        # 集中营 PM
│   ├── script_values/
│   │   ├── rejob_mortality_values.txt       # 死亡率脚本值（含 clamp 数学）
│   │   ├── rejob_migration_values.txt       # 迁移人口估算
│   │   └── rejob_old_cultural_community_values.txt
│   ├── scripted_buttons/
│   │   └── rejob_mortality_buttons.txt      # 全部日志按钮
│   ├── scripted_effects/
│   │   ├── rejob_mortality_effects.txt      # 刷新与重置效果
│   │   └── rejob_old_decision_effects.txt   # 66/67 号指令效果
│   ├── scripted_triggers/
│   │   ├── rejob_mortality_triggers.txt     # 建筑存在性 / 特殊 PM 判断
│   │   └── rejob_old_delicious_clothes.txt
│   ├── social_classes/
│   │   └── rejob_old_social_classes.txt     # 社会阶级（含不可接触者）
│   └── static_modifiers/
│       ├── rejob_mortality_modifiers.txt    # 动态修正容器
│       └── rejob_old_conquered_building_static_modifiers.txt
├── events/
│   └── rejob_migration_events.txt           # 迁移失业人口事件
└── localization/
    └── simp_chinese/
        ├── rejob_management_l_simp_chinese.yml      # 日志与按钮
        ├── rejob_migration_l_simp_chinese.yml       # 迁移事件
        ├── rejob_old_gulag_l_simp_chinese.yml       # 集中营与不可接触者
        └── rejob_unemployment_center_l_simp_chinese.yml  # 再就业中心
```

**命名约定**：文件名带 `rejob_old_` 前缀的表示内容来自原模组；其余为本仓库自有内容。


---

## 11. 常见问题

**Q：需要同时启用原「再就业中心 2.0」吗？**
A：不需要，也不要。本模组已包含原模组全部内容，同时启用会导致定义冲突。

**Q：日志没有出现？**
A：先在决议面板点一次「设立失业人群管理中心」。若该决议也不显示，说明境内还没有两座建筑中的任何一座。

**Q：点了 `+10%`，界面显示的是加 10% 还是 1000%？**
A：加 10%。本整合版已修正单位问题。

**Q：为什么调不到负数？**
A：这是设计如此。合计死亡率最低为 0，调节值可以继续往下点，但不会产生低于 0 的效果。

**Q：为什么调节按钮点不动了？**
A：可能的原因有两种：
1. 两个生效开关都关着（**默认就是关着的**，需要先点亮）；
2. 对应的建筑还没建（境内没有「再就业中心」或「集中营」）。

> **★ 第九轮变更**：「特殊生产方式」判定已取消。
> 「重型化学工艺」与「终极解决」两个 PM 已按需求**删除**，
> `rejob_uc_has_special_pm` / `rejob_camp_has_special_pm` 现恒为 `no`，
> 因此不再存在「被特殊 PM 禁用」这一情况。

**Q：生产方式还会自带死亡率吗？**
A：**会，但只来自「岗位构成 / 关押人群」这一组**。
第九轮已按需求把「生产项目」大类（瓷器/肉类/石油/化肥）自带的死亡率**全部删除**，
现在生产项目只负责商品投入产出，不再影响死亡率。
其余非死亡率的 PM 原始值（含 `state_migration_pull_mult = -99`）保持与原模组一致。

**Q：建筑面板上的 100% / 200% / 400% 到底是什么？**
A：那是**当前激活的「岗位构成」PM 的 `building_modifiers.unscaled` 值**，
不是建筑自带属性（建筑定义文件里没有任何 mortality 字段）。例如：

- 基础再就业岗位（+100%）→ 面板显示 **100%**
- 铁丝网生产计划（+200%）→ 面板显示 **200%**
- 电子围栏生产计划（+400%）→ 面板显示 **400%**
- 专业阶层岗位（地主/资本家 各 +4000%）→ 面板显示 **4000%**

日志面板的调节值会**叠加**在这个基准值之上。详见 §6.5 / §6.6 / §6.7。

**Q：日志调节的修正会影响整个州吗？**
A：**不会 —— 只影响被该建筑招募的那部分人**。
第六轮重做后，再就业中心改用**建筑级**修正
（`building_laborers / aristocrats / capitalists _mortality_mult`），
修正挂在建筑对象上，因此只作用于在该建筑就业的人口，
**不会**波及同州其他建筑的同类职业。详见 §6.6。

**Q：为什么启用本 mod 后，所有建筑界面的生产/产出/消耗都只显示初始生产方式的值？**
A：先检查**本 mod 到底有没有被启用**。实测中遇到过一种情况：
`content_load.json` 里启用的是**原模组「再就业中心 2.0」（3266813717）**，
而 `rejob_modify` 根本没进启用列表 —— 此时游戏跑的是原模组的脚本，
自然会出现下面这些症状。请到启动器播放集里确认
「再就业中心 整合版」已勾选、且原模组已取消勾选。

若确认已启用本 mod 仍出现该症状，则根因有两条，都在日志里能查到：
1. `common/static_modifiers/rejob_mortality_modifiers.txt` 使用了官方脚本**未登记**的
   修正类型（`state_clerks_mortality_mult` / `state_capitalists_mortality_mult` /
   `state_engineers_mortality_mult`）→ 日志报
   `Unknown modifier type ...` 与 `Invalid category prefix for modifier corruption`，
   **污染了整个修正数据库**，导致建筑面板的数据查询失败
   （`BuildingProductionMethodsEntry.GetBuildingType` 取不到上下文）。
2. 建筑组写了**非法** `lens = government`（官方合法值只有
   `development` / `mine` / `trade` / `special` / `agriculture` / `fishing` /
   `forestry` / `heavy_industry` / `light_industry` / `plantation`），
   使建筑浏览器的分组异常。
修复后：修正类型全部登记、`lens` 改为 `development`。详见 §13.13 B。

**Q：哪些职业会被调节？**
A：取决于该建筑所选的生产方式。再就业中心覆盖
**劳工 / 地主 / 资本家**（这三种是该建筑实际会雇佣的职业）；
建筑**未雇佣**的职业不受影响。集中营影响**不可接触者**。

**Q：为什么负调节按钮「点了没反应」？**
A：这是第四轮的 bug，**已在第五轮修复**，第六轮沿用。根因是补偿量被误写成 `min = 0`，
导致负值被截断为 0，永远抵消不掉 PM 自带的死亡率。
现在改为 `min = -生产方式基准值`：负调节最多抵消 PM 自带部分，合计最低 0。
详见 §13.12 / §13.13。

**Q：建筑拆除后修正会残留吗？**
A：**不会**。这是第四轮的 bug，**已在第六轮彻底修复**。
第四轮的根因是清除阶段带了「该州必须存在某建筑类型」的 limit，建筑一拆就永远清不掉。
现在改为：`every_scope_building` 遍历时只用**静态定义**的 `is_building_group` 收窄
（建筑组定义不会因建筑拆除而消失），再加 `has_modifier` 守卫。
由于修正本就挂在建筑对象上，建筑被拆除时修正会随之消失，不存在跨对象残留。
详见 §13.13 A。

**Q：日志里显示的是单一值还是范围值？**
A：若境内所有同类建筑的生产方式**一致** → 显示**单一合计值**；
若**不一致** → 显示**范围值**（如 `100% ~ 450%`）。详见 §6.7。

**Q：合计值会变成负数吗？**
A：**不会**。所有合计都带 `min = 0`，最小就是 0%。

**Q：可以只调一座建筑的死亡率吗？**
A：可以。用两个生效开关控制，只打开你想调的那一座即可。若某座建筑还没建（例如采用文化多元法律时无法建造集中营），对应的开关会自动置灰。

**Q：存档兼容性？**
A：本模组 2.0 相比 1.x 重命名了大量脚本 ID，**旧存档中的建筑与变量不会自动迁移**。建议开新档。

---

## 12. 开发记录

### 12.1 日志激活方式

本模组的日志条目**不使用** `is_shown_when_inactive` + `possible` 的自动激活机制。实测在多个模组同时启用的环境下该机制不可靠（日志已定义却从未被激活）。因此改为由决议或 `on_action` 显式调用：

```txt
add_journal_entry = {
    type = je_rejob_management
}
```

这与 The OGAS 模组的做法一致。**请勿**给本日志添加 `is_shown_when_inactive` / `possible`。

### 12.2 死亡率单位

游戏官方文档 `modifier_types.md` 原文：

> `percent = no` — determines whether the modifier value should be considered a percentage when displaying entries of this type (i.e. **multiply by 100 and append a % sign, the value underneath is not affected**)

由于 `state_mortality_mult` 定义为 `percent = yes`，脚本值 `4` 会显示为 `400%`。故本模组统一使用**百分点**作为调节单位：按钮 `+10%` 对应变量 `0.1`。

### 12.3 clamp 数学

实现「合计最低为 0」：

```txt
单座建筑合计 = max( 生产方式基准值 + 调节值 , 0 )
```

其中调节值在写入修正前先做 `max(offset, 0)`（负值不生效），
所有合计脚本值再带 `min = 0`（双重保险）。

### 12.4 动态修正的移除守卫

移除动态修正前先判断 `has_modifier`，避免产生 `remove_modifier effect [ Timed modifier ... not found ]` 报错刷日志。

### 12.5 脚本 ID 命名规范

| 前缀 | 含义 |
|---|---|
| `rejob_uc_` | 再就业中心（新建筑）相关 |
| `rejob_camp_` | 集中营（旧建筑）相关 |
| `rejob_old_` | 文件名前缀，表示内容来自原模组 |
| `rejob_btn_` | 日志中的决议按钮 |

**脚本 ID 本身保持原模组原名**（如 `building_gulag_logging`、`pm_Auschviking_shao`、`delicious`），以确保存档与内部引用兼容；仅**文件名**加前缀以便人工维护时区分来源。

---

## 13. 修复记录（2026-10-11 第二轮）

### 13.1 按钮缺失 —— 定义块未闭合

**现象**：日志中只能看到 7 个死亡率调节按钮，**生效开关（2 个）与决议按钮（3 个）全部消失**。

**根因**：`rejob_mortality_reset` 这个按钮定义**漏了结尾的 `}`**，导致其后的 B 组、C 组共 5 个按钮被**嵌套进了它的 `effect` 内部**，因而从未成为顶层定义、不会被引擎注册。

**修复**：补齐 `}`。修复后按钮文件含 **12 个顶层定义**，与日志引用的 12 个按钮完全对应。

> **教训**：脚本文件里每个顶层定义都必须顶格（第 0 列）书写。若某定义被意外嵌套，引擎**不会报错**，只会静默忽略 —— 表现为「按钮凭空消失」。建议每次改动后用「顶层定义数 = 引用数」做交叉校验。

### 13.2 脚本值不存在 —— 同样的问题

**现象**：`error.log` 被以下报错刷屏（每秒 2 条）：

```
[jomini_eventscope.cpp:39]: Referencing a named script value that does not exist 'rejob_camp_pm_base_country'
[jomini_eventscope.cpp:39]: Referencing a named script value that does not exist 'rejob_camp_total_country'
```

**根因**：`rejob_uc_offset_effective_in_state` 定义**漏了 `min` 与结尾的 `}`**，导致其后 `rejob_camp_*` 的 5 个脚本值全被嵌套进去。

**修复**：重建脚本值文件，所有定义顶格。现在共 **4 个顶层定义**，全部可被引用。

### 13.3 生效开关默认值改为「关闭」

**需求**：两个生效开关（`rejob_apply_uc` / `rejob_apply_camp`）**默认不打开**。

**修复**：`rejob_default_mortality` 中两者的初始值由 `1` 改为 `0`。玩家需自行点亮开关，调节值才会生效。

### 13.4 修正对象搞反 —— 已纠正

> **最终状态提示**：本节记录的是第三轮的认识过程。第四轮按需求
> 「PM 原始值保持和一开始一样」，这些值**已全部恢复**。当前最终状态见 §13.10。

**第三轮的认识**：需求当时要删除的「**无等级修正**」——
`state_modifiers` 下的 `unscaled` 区块**本身就是无等级修正**（不随建筑等级缩放），
它叠加出的「默认 400% + 100% = **500%**」正是玩家在建筑面板上看到的那个 500%。
换成「瓷器生产 400% + 电子围栏 400%」则会显示 **800%**，
换成「化肥投入 40000% + 电子围栏 400%」则是 **40400%** ——
面板数字**完全等于各生产方式 `unscaled` 值的算术和**，建筑本身零贡献
（`common/buildings/*.txt` 与 `common/building_groups/*.txt` 中无任何 mortality 字段）。
详见 §6.5 / §6.6。因此最终处理如下：

| 修正位置 | 键名 | 含义 | 处理 |
|---|---|---|---|
| PM → `state_modifiers` → `unscaled` | `state_mortality_mult`<br>`state_delicious_mortality_mult` | **生产方式**的州死亡率（无等级修正） | ✅ **已删除**（14 处，两座建筑各 7 处） |
| PM → `building_modifiers` → `unscaled` | `building_aristocrats_mortality_mult`<br>`building_capitalists_mortality_mult` | **建筑自带**的无等级修正死亡率 | ✅ **已删除**（4 处） |
| PM → `state_modifiers` → `unscaled` | `state_migration_pull_mult` | 生产方式带来的移民吸引力 | ✅ **已删除**（2 处） |

**删除明细**（两座建筑完全对称）：

| 建筑 | 生产方式 | 删除值 |
|---|---|---|
| 再就业中心 | 瓷器生产 / 肉类加工 / 石油加工 | `state_mortality_mult = 4` |
| 再就业中心 | 化肥投入 | `state_mortality_mult = 400` |
| 再就业中心 | 基础再就业岗位 / 电子围栏 | `state_mortality_mult = 1` / `4` |
| 再就业中心 | 铁丝网生产计划 | `state_mortality_mult = 2` |
| 集中营 | 瓷器/肉类/炼油 | `state_delicious_mortality_mult = 4` |
| 集中营 | 环保产业 | `state_delicious_mortality_mult = 400` |
| 集中营 | 基础关押岗位 / 电子围栏 | `state_delicious_mortality_mult = 1` / `4` |
| 集中营 | 铁丝网 | `state_delicious_mortality_mult = 2` |

**保留**：`pm_rejob_uc_gasking` / `pm_Auschviking_gasking` 的 `state_mortality_mult = 9999999`（「终极手段」的设定值）。

**连带简化**：生产方式基础值清零后，脚本值的 clamp 数学简化为：

```txt
合计死亡率 = max(调节值, 0)
```

不再需要 `pm_base` 系列基准值（`rejob_uc_pm_base_in_state` 等 6 个脚本值已删除），
状态栏也不再显示「（生产方式 X%）」。

### 13.5 特殊生产方式时状态栏显示真实值

**现象**：当某座建筑处于「重型化学工艺 / 终极解决」（自带 9,999,999 死亡率）时，状态栏只显示调节值的合计，**没有反映这个真实生效的 9999999**，且调节按钮仍可点击。

**修复**：

1. **状态栏**：`rejob_uc_total_country` / `rejob_camp_total_country` 在检测到特殊生产方式时**直接返回 9999999**，优先显示真实值。
2. **调节按钮**：`rejob_mortality_buttons_possible` 中新增 `rejob_has_unemployment_center` / `rejob_has_camp` 与 `NOT = { ..._has_special_pm }` 判断 —— 建筑不存在、或处于特殊生产方式时，该建筑对应的调节通道被禁用。

### 13.6 关于「子 agent」

本次任务按要求尝试了「开子 agent 分片处理」的思路，但当前环境**只提供文件读写、代码搜索、命令执行三类工具**，没有子 agent 调度能力。因此改用**分批高密度并行读取 + 把结论固化进代码注释与本文档**的方式控制上下文占用，效果上等效。

### 13.7 引导决议改为「可重复点击」

**需求**：决议里的「打开再就业中心的日志条目」要变成**可重复点击**——
之前没打开就打开；如果当前已打开，则点击后该日志条目先删除、再生成新的，
并且所有值变成默认状态。

**关键限制**：Victoria 3 的**脚本效果里没有删除日志条目的能力**。
经三处交叉验证：

1. `victoria3.exe` 的字符串表里确实存在 `invalidate_journal_entry`、
   `complete_journal_entry`、`fail_journal_entry`、`activate_journal_entry`、
   `timeout_journal_entry`，但它们的说明都是
   *"…for the **player country**"*，属于**控制台命令**而非脚本效果。
2. CWTools 的权威配置 `effects.cwt` / `list_effects.cwt` 中**完全没有**这些名字，
   日志相关只有 `add_journal_entry` 与 `add_contextless_journal_entry`。
3. 游戏本体与全部已装 mod 的脚本中**没有任何一处**使用过它们。

**解决方案**：用「就地复位为默认状态」实现等价效果。因为本日志**除三个变量外
没有任何持久状态**，所以「复位」与「删掉重建」对玩家而言结果完全一致。

`common/decisions/rejob_management_decisions.txt`：

```txt
when_taken = {
    if = {
        limit = { NOT = { has_journal_entry = je_rejob_management } }
        add_journal_entry = { type = je_rejob_management }
    }
    else = {
        rejob_reset_all = yes
    }
}
```

`common/scripted_effects/rejob_mortality_effects.txt` 中新增的 `rejob_reset_all`：
先 `remove_variable` 掉三个变量，再调用 `rejob_default_mortality` 走一遍全新初始化
（调节值 0、两个开关关闭，并刷新动态修正）。

| 情形 | 点击后的行为 |
|---|---|
| 日志**未**开启 | 直接 `add_journal_entry`，按 `immediate` 初始化 |
| 日志**已**开启 | `rejob_reset_all`：三个变量全部复位为默认值，并刷新修正 |

**对玩家的效果**：等于「日志被重置为一条全新的」，所有值回到默认状态。

### 13.8 第四轮改造：修正作用域改为「建筑级 · 按职业」

**需求**：日志调节的修正**不应该是州级**（`state_mortality_mult` 会无差别影响所在州
全体人口），而应**只影响入职了该建筑的劳工、职员、官僚、资本家等**，
即「类似方案 B」的建筑级修正，且只对建筑内部人群生效。

**改造内容**：

| 项目 | 改造前 | 改造后 |
|---|---|---|
| 再就业中心修正 | `state_mortality_mult`（州级，影响全州） | `building_laborers/clerks/bureaucrats/capitalists/aristocrats/machinists/engineers_mortality_mult`（**建筑级 · 按职业**） |
| 挂载方式 | `every_scope_state` + `add_modifier` | `every_scope_state` → `every_scope_building` + `add_modifier` |
| 集中营修正 | `state_delicious_mortality_mult` | **保持不变**（引擎无建筑级 delicious 键） |

**关键技术依据**：

1. `building_*_mortality_mult` 系列确实存在于引擎 —— CWTools 权威配置
   `modifiers.gen.cwt` 中列有 15 个（`laborers` / `clerks` / `bureaucrats` /
   `capitalists` / `aristocrats` / `machinists` / `engineers` / `farmers` /
   `peasants` / `shopkeepers` / `clergymen` / `academics` / `officers` /
   `soldiers` / `slaves`），且作用域标注为 `= building`。
2. `add_modifier` 可以挂到**建筑**作用域 —— 游戏本体 `coup_events.txt:883`
   就有 `scope:barracks_theft_building = { add_modifier = { name = ... } }`。
3. `every_scope_building` 在**脚本值与效果**中均可用 —— 游戏本体用了 20 处
   （`00_victoria_scripted_effects.txt`、`je_values.txt` 等），
   其他已装 mod 用了 48 处。

**关于「只影响建筑内部人群」**：这一点由建筑级修正本身保证 ——
`building_laborers_mortality_mult` 只作用于**该建筑雇佣的劳工**，
不会波及同州其他建筑的人口。建筑未雇佣的职业自然不受影响。

**关于集中营**：CWTools 中**不存在** `building_delicious_mortality_mult`
（delicious 是不可接触者专属人口类型，其「职业」就是 delicious 本身，
引擎没有为它提供建筑级修正键）。原模组作者也曾在 PM 里写过
`building_delicious_mortality_mult`，但**全部注释掉了**，印证该键不可用。
因此集中营只能继续用州级 `state_delicious_mortality_mult`；
但因 delicious 的排他性（只有集中营会雇佣），**实际效果与建筑级等价**。

### 13.9 第四轮改造：日志 UI 改为显示「修正值 + 合计 + 范围值」

**需求**：
1. 日志显示的不再只是最终合计，还要**注明是哪些职业**的修正。
2. 建筑面板显示的值 = **当前生产方式产生的死亡率** + **日志调节值**，
   且**合计不得为负**（最小 0）。
3. 若所有建筑生产方式**一致** → 显示单一合计值；
   若**不一致** → 显示**范围值**。

**实现**：

```txt
单座建筑合计 = max( 生产方式基准值 + 调节值 , 0 )
```

- 所有合计值都带 `min = 0`，保证「最终总和值不得为负数，最小就是 0」。
- 范围值：先对基准值取 `min` / `max`，再加调节值并 clamp(0)，
  得到 `rejob_uc_total_min_country` ~ `rejob_uc_total_max_country`。
- 本地化用 `SelectLocalization(EqualTo_CFixedPoint(min, max), '单一', '范围')`
  自动切换显示形式。

**状态栏显示效果**：

```
调节值：+50%

再就业中心（建筑级 · 仅影响本建筑雇员）
　　生产方式：400%　调节值：+50%
　　合计死亡率：450%

集中营（州级 · 仅影响不可接触者）
　　各建筑生产方式不一致，合计死亡率范围：100% ~ 450%

作用范围：再就业中心 为建筑级修正，只影响该建筑内部雇员（劳工 / 职员 /
官僚 / 资本家 / 地主 / 机械师 / 工程师，取决于所选生产方式）；集中营 为
州级不可接触者修正，只影响营内的不可接触者。
```

**日志说明文本**（`je_rejob_management_reason`）也已加入上述细节，
明确区分「建筑级」与「州级」两种修正及其作用人群。

### 13.10 第四轮改造：PM 原始值恢复

**需求**：「原有的移民率的生产方式和死亡率的生产方式修正的值保持和一开始一样」，
即**恢复原模组 3266813717 的原始值**。

**恢复清单**（与 §6.8 一致，两座建筑各 11 处）：

| 位置 | 恢复的值 |
|---|---|
| UC 生产项目（3 个 PM） | `state_mortality_mult = 4` |
| UC 化肥投入 | `state_mortality_mult = 400` |
| UC 基础岗位 / 电子围栏 | `state_mortality_mult = 1` / `4` |
| UC 铁丝网 | `state_mortality_mult = 2` |
| UC 专业阶层岗位 | `building_aristocrats/capitalists_mortality_mult = 40` |
| UC 服务当地 | `state_migration_pull_mult = -99` |
| CAMP 生产项目（3 个 PM） | `state_delicious_mortality_mult = 4` |
| CAMP 环保产业 | `state_delicious_mortality_mult = 400` |
| CAMP 基础关押 / 电子围栏 | `state_delicious_mortality_mult = 1` / `4` |
| CAMP 铁丝网 | `state_delicious_mortality_mult = 2` |
| CAMP 专业阶层岗位 | `building_aristocrats/capitalists_mortality_mult = 40` |
| CAMP 服务当地 | `state_migration_pull_mult = -99` |

**逐项比对结果**：与原模组 `gulag_method_group.txt` 的
`state_delicious_mortality_mult` / `state_mortality_mult` /
`state_migration_pull_mult` / `building_*_mortality_mult` 完全一致。

### 13.11 关于「多开子 agent」

本轮需求建议「多开子 agent 并行执行」。**当前环境只提供文件读写、代码搜索、
命令执行、编辑、网页抓取、提问六类工具，没有子 agent 调度能力**。
因此改用**批量并行命令 + 精准 grep 定位 + 只读必要片段**的方式压缩上下文占用，
等效达成「加快处理、节省上下文」的目标。

### 13.12 第五轮重置：建筑级方案全面废弃，改为「州级 + 按职业」

**背景**：第四轮的建筑级方案（`building_*_mortality_mult` + `every_scope_building`）
引发了三个**严重 bug**：

| # | 现象 | 根因 |
|---|---|---|
| 1 | 按钮完全无效，死亡率无法变化 | ①往「建筑」作用域挂修正这条路径不可靠；②`offset_effective` 被误写成 `min = 0`，导致**负调节值被截断为 0，永远抵消不掉 PM 自带死亡率**（这是最致命的一条） |
| 2 | 建筑拆除后死亡率残留、地区死亡率错乱 | 清除阶段写成 `every_scope_state → every_scope_building → limit{ is_building_type... }`。**建筑一拆，该州就再也匹配不到 limit，`remove_modifier` 永远不执行** |
| 3 | 建筑 UI 显示错乱 | 往建筑挂修正与建筑面板渲染相互干扰 |

**关键发现（你的第 4 点完全正确）**：官方**原生就有按职业的州级死亡率修正**，
根本不需要自己造建筑级机制：

| 键 | 作用域 | 原生用例 |
|---|---|---|
| `state_laborers_mortality_mult` | state | `common/laws/02_childrens_rights.txt:13` = `0.05` |
| `state_machinists_mortality_mult` | state | 同上 `:14` |
| `state_aristocrats_mortality_mult` | state | `content_4_modifiers.txt:325` = `0.50` |
| `state_clerks` / `bureaucrats` / `capitalists` / `engineers` | state | CWTools 权威列表（共 16 个） |

它们的定义与 `state_mortality_mult` **完全相同**（`decimals=0 / color=bad / percent=yes`）。
原模组作者自定义的 `state_delicious_mortality_mult` 也正是照抄这套写法。

**重置方案**（对应你的第 4、5 点）：

1. **修正改回州级**，用官方原生按职业修正：
   ```
   rejob_uc_mortality_dynamic = {
       state_laborers_mortality_mult = 1
       state_clerks_mortality_mult = 1
       state_bureaucrats_mortality_mult = 1
       state_capitalists_mortality_mult = 1
       state_aristocrats_mortality_mult = 1
       state_machinists_mortality_mult = 1
       state_engineers_mortality_mult = 1
   }
   ```
   这样「只影响指定职业」，且**不碰建筑** → 不会与建筑面板冲突。

2. **清除阶段改为无条件遍历**（修复残留 bug）：
   ```
   every_scope_state = {
       if = { limit = { has_modifier = rejob_uc_mortality_dynamic }
              remove_modifier = rejob_uc_mortality_dynamic }
       if = { limit = { has_modifier = rejob_camp_mortality_dynamic }
              remove_modifier = rejob_camp_mortality_dynamic }
   }
   ```
   **绝不在清除阶段要求「该州存在某建筑」** —— 否则建筑一拆，修正就清不掉。

3. **补偿量恢复为 `max(调节值, -生产方式基准值)`**（修复按钮无效）：
   ```
   rejob_uc_offset_effective = {
       value = owner.var:rejob_offset
       min = rejob_uc_pm_base_in_state_negated   # 基准值取负
   }
   ```
   正调节直接生效；负调节最多抵消 PM 自带部分。**合计由 `min = 0` 保证不为负**。

**与旧版工作正常实现的对照**：

| 项目 | 旧版（正常） | 第四轮（有 bug） | 第五轮（重置后） |
|---|---|---|---|
| 修正作用域 | 州级 | 建筑级 ❌ | **州级** ✅ |
| 修正类型 | `state_mortality_mult`（全州） | `building_*` ❌ | **`state_<职业>_mortality_mult`**（按职业）✅ |
| 清除阶段 | 无条件遍历 | 带建筑 limit ❌ | **无条件遍历** ✅ |
| 补偿量 | `max(offset, -pm_base)` | `max(offset, 0)` ❌ | **`max(offset, -pm_base)`** ✅ |

**结果**：既实现了「只影响指定职业」，又回到了经过验证的稳定实现路径。

> ⚠️ **本轮的结论在第六轮被推翻**：第五轮误以为「州级 + 按职业」能满足需求，
> 但州级修正无法做到「只影响招募进来的人」。详见 §13.13。

---

### 13.13 第六轮重做：改回「建筑级 · 按职业」，并修复一批全局 bug（2026-10-11）

本轮依据**真实日志**（`logs\debug.log` / `error.log` / `game.log`）定位问题，
所有结论均有日志行号佐证。

#### A. 需求 1 重做：再就业中心改为「建筑级 · 按职业」

| 项目 | 第五轮（错误） | 第六轮（现行） |
|---|---|---|
| 修正类型 | `state_laborers/clerks/bureaucrats/capitalists/aristocrats/machinists/engineers_mortality_mult` | **`building_laborers / aristocrats / capitalists _mortality_mult`** |
| 挂载对象 | 州 | **建筑** |
| 实际影响范围 | ❌ 全州同职业人口 | ✅ **仅该建筑招募的人** |
| 脚本值 root | 州（`state`） | **建筑（`building`）** |

**第五轮为什么错**：`state_<职业>_mortality_mult` 作用域是**州**。
再就业中心雇佣的 `laborers` / `aristocrats` / `capitalists` 在州里到处都是，
挂州级修正 = 把全州所有劳工/地主/资本家的死亡率一起改掉，严重超范围。

**PM 也同步改为按职业**（需求 1a）。以 `pm_rejob_uc_shao` 为例：

```txt
# 改前（第五轮）：州级 · 不分职业
state_modifiers = {
    unscaled = { state_mortality_mult = 4 }      # 全州所有人口
}

# 改后（第六轮）：建筑级 · 按职业
building_modifiers = {
    unscaled = {
        building_laborers_mortality_mult = 4     # 只影响本建筑的劳工
        building_aristocrats_mortality_mult = 4  # 只影响本建筑的地主
        building_capitalists_mortality_mult = 4  # 只影响本建筑的资本家
    }
}
```

**第四轮的建筑级方案为何失败（本轮已修掉根因）**：

| 第四轮的缺陷 | 本轮修法 |
|---|---|
| 清除阶段 `limit` 里带 `is_building_type` 守卫 → 建筑一拆就清不掉 | `limit` 只用**静态定义**的 `is_building_group` 收窄 + `has_modifier` 守卫 |
| 用了官方脚本**未登记**的 `state_clerks/bureaucrats/engineers` → `Unknown modifier type`，污染修正数据库 | 只用**已登记**的键（见下方 B） |
| 建筑组写了**非法** `lens = government` → 建筑浏览器分组异常 | 改为合法的 `lens = development` |

#### B. 修正数据库污染（**这是「所有建筑 UI 显示错乱」的根因**）

日志硬证据（`logs\debug.log` / `error.4.log`）：

```
[17:47:52][pdx_persistent_reader.cpp:268]: Error: "Unknown modifier type:
  state_clerks_mortality_mult. It is either invalid, or a potential dynamic
  modifier type definition missing from the database.: state_clerks_mortality_mult,
  near line: 40" in file: "common/static_modifiers/rejob_mortality_modifiers.txt" near line: 40
（state_capitalists_mortality_mult / state_engineers_mortality_mult 同样报错）

[17:48:11][modifier_types.cpp:624]: Invalid category prefix for modifier corruption
```

**根因**：`state_<职业>_mortality_mult` 在**代码**里存在
（CWTools `modifiers.gen.cwt` 有 `state_clerks_mortality_mult = state`），
但官方**脚本**只给 8 个职业写了 `modifier_type_definitions`：

| 已登记（可安全使用） | 未登记（使用即报 Unknown） |
|---|---|
| `laborers` / `aristocrats` / `machinists` / `academics` / `farmers` / `peasants` / `slaves` / `soldiers` | `clerks` / `bureaucrats` / `capitalists` / `engineers` |

修正类型未登记 → 修正数据库被污染 → **建筑面板的数据查询失败**，
于是出现「所有建筑的生产/产出/消耗都只显示初始生产方式」的现象
（对应日志：`No context supplied ... wanted context of type 'BuildingProductionMethodsEntry'`
+ `PdxDataFetchLocalizedData failed for 'PRODUCTION_METHOD_OPTIONS_BULK'`）。

**修法**：不再使用未登记的键。本轮改用建筑级后，
`building_aristocrats/capitalists_mortality_mult` 由本 mod 自行登记
（`common/modifier_type_definitions/rejob_old_mortality.txt`），
`building_laborers_mortality_mult` 官方已登记。

#### C. 其余一批日志确证的 bug

| # | 日志证据 | 文件 | 修法 |
|---|---|---|---|
| 1 | `Unknown trigger type: every_scope_pop`（在 `possible` 里） | `scripted_buttons/rejob_mortality_buttons.txt:230` | trigger 上下文不能用 effect，改为 `any_scope_pop` |
| 2 | `Unknown trigger type: has_port` | `script_values/rejob_old_cultural_community_values.txt:29` | 正确名为 `has_port_state` |
| 3 | `PostValidate of effect 'remove_building' returned false` | `on_actions/rejob_old_bingyin_on_actions.txt:26` | `building_naval_base` **不存在**，改为 `building_naval_fortification` |
| 4 | `There is more than one 'effect' defined` | 同上 `:2` | 不能直接覆盖 vanilla 的 `on_state_owner_change`；改用官方 `on_actions = { ... }` 列表追加新名 |
| 5 | `Duplicated key upper_class/middle_class/lower_class` | `social_classes/rejob_old_social_classes.txt` | 原 mod 用同名文件 `00_default.txt`（整文件替换）才能生效；改名后必须用 `REPLACE:` 条目模式 |
| 6 | `Duplicated key slaves_clothes_pop_trigger` | `scripted_triggers/rejob_old_delicious_clothes.txt:1` | 同上，改用 `REPLACE:` |
| 7 | `Badly read script value 6666666666666666` | `pop_types/rejob_old_delicious.txt:64` | 数值超出定点数范围，改为 `1000000` |
| 8 | 建筑组 `lens = government`（官方无此 lens） | `building_groups/*.txt` | 改为合法的 `development` |
| 9 | PM 内**重复的 `workforce_scaled` 键**（后者覆盖前者，产出凭空消失） | `production_methods/rejob_old_building_pm.txt:242` | 删除重复空块 |

> **关于 `REPLACE:` / `INJECT:` 等前缀**：这是官方支持的「数据库条目模式」
> （见 CWTools `definition_injections.cwt`）：
> `INJECT` / `REPLACE` / `TRY_INJECT` / `TRY_REPLACE` / `INJECT_OR_CREATE` / `REPLACE_OR_CREATE`。
> **同名条目在不同文件中重复定义会被判定为冲突并丢弃**（这就是 bug 5、6 的原因）；
> 想覆盖官方条目必须显式使用 `REPLACE:` 前缀。

#### D. 本轮与旧版（正常）实现的最终对照

| 项目 | 旧版（正常） | 第五轮 | 第六轮（现行） |
|---|---|---|---|
| 修正作用域 | 州级 | 州级 | **建筑级** ✅ |
| 修正类型 | `state_delicious_mortality_mult`（只影响营内人群） | `state_<职业>_mortality_mult`（❌ 波及全州） | **`building_<职业>_mortality_mult`** ✅ |
| 清除阶段 | 无条件 | 无条件 | **无条件（`is_building_group` 收窄 + `has_modifier` 守卫）** ✅ |
| 补偿量 | `max(offset, -pm_base)` | `max(offset, -pm_base)` | **`max(offset, -pm_base)`** ✅ |
| 修正类型是否全部登记 | 是 | ❌ 否（3 个未登记） | **是** ✅ |
| `lens` 是否合法 | 是 | ❌ `government` | **`development`** ✅ |

**结论**：本轮把「只影响招募进来的人」（需求 1c）与「不污染修正数据库」
（需求 2 的根因）同时解决 —— 这两点正是前几轮反复出问题的真正原因。

### 13.14 第七轮改造：修正「跟随生产方式」+ 按职业拆分容器

**需求（用户原话归纳）**：
> 用户发现 `rejob_uc_mortality_dynamic` 一个容器同时挂了 3 个职业的死亡率，
> 导致三种职业的死亡率**同时**变化（在有任何建筑的州）。
> 而**切换建筑的生产方式会改变实际入职的职业**。
> 用户希望：修正能「单独作用于建筑当前生产方式对应的职业」，
> 这样建筑界面的状态显示也能正常。

**先回答一个关键问题：能不能把调节值直接写进 PM 代码？**

**不能。** 证据：

| 验证项 | 结果 |
|---|---|
| CWTools schema `aliases.cwt:138-143` | `<modifier_type> = value_field` —— `value_field` 是**数值原语** |
| 全游戏 `.txt` 扫描：modifier 值用过脚本值/变量/块 | **0 处** |
| 官方 PM 的 `building_modifiers` 值 | **全部是字面数字** |

结论：**PM 里的修正值是加载时固化的常量**，读不到 `rejob_offset`。
所以 `add_modifier` + 静态修正容器这一层**无法取消** ——
它能做到的只是「让容器的内容跟随生产方式变化」。

**本轮实现**：

| 项目 | 第六轮 | 第七轮（现行） |
|---|---|---|
| 容器结构 | **1 个**容器塞 3 个职业 | **3 个**独立容器（每职业一个） |
| 挂载依据 | 无条件全挂 | 按 `has_active_production_method` 判断**实际雇佣的职业** |
| 切换生产方式 | ❌ 不响应 | ✅ `on_production_method_changed` 自动重挂 |
| 负调节下限 | 三种职业共用（可能压成负数） | **每职业各自截断** |
| 特殊 PM 判断粒度 | 国家级（任一建筑触发则全国屏蔽） | **逐建筑判断** |

**关键技术依据（全部经核实）**：

1. **`on_production_method_changed` 存在且 Root = Building**
   ```
   common/on_actions/00_code_on_actions.txt:4426
   # Root = Building
   on_production_method_changed = { effect = { ... } }
   ```
   CWTools `on_actions.cwt:633-636`：
   ```
   ### Root = Building
   ## replace_scopes = { this = building root = building }
   ## event_type = scopeless
   on_production_method_changed
   ```

2. **追加写法**（官方文档 `_on_actions.md:117-128` 明确给出）：
   ```
   some_vanilla_on_action = { on_actions = { some_modded_on_action } }
   some_modded_on_action = { effect = { ... } }
   ```
   ⚠️ 文档同时警告（`:115`）：
   *"you cannot append an effect block directly to an on_action which already
   has an effect block, as this creates a conflict"*。

3. **`has_active_production_method` 作用域 = building**
   CWTools `triggers.cwt:1155`：`## scopes = { building }`

**雇佣关系映射**（依据 PM 的 `building_employment_<职业>_add`）：

| 生产方式（组：`rejob_uc_pop`） | 实际雇佣 |
|---|---|
| `pm_rejob_uc_labour` / `_barbed_wire_fences` / `_electric_fencing` | `laborers` |
| `pm_rejob_uc_upper` | `aristocrats` + `capitalists` |

**为什么必须按职业拆容器**：
若仍用 1 个容器塞 3 个职业，建筑面板会把 3 个职业全部列出来 ——
但该建筑可能只雇佣劳工（选了「基础再就业岗位」），显示就不准。
拆成 3 个容器后，脚本只挂「真正雇佣的职业」，面板显示即与实际一致。

**负调节下限也必须按职业拆**（这是本轮新发现的隐患）：

```
劳工基准 4、地主基准 44
共用下限 -44 时：对劳工施加 -44 → 4 + (-44) = -40  ← 负死亡率，错误
按职业下限后：对劳工施加 -4  → 4 + (-4)  = 0    ← 正确
```

**新增/改名的脚本值**：

| 名称 | 作用 |
|---|---|
| `rejob_uc_mortality_laborers/aristocrats/capitalists` | 3 个独立静态修正容器 |
| `rejob_uc_employs_laborers/aristocrats/capitalists` | 判断本建筑是否雇佣该职业 |
| `rejob_uc_this_building_has_special_pm` | 建筑作用域版特殊 PM 判断 |
| `rejob_uc_pm_base_production` | 生产项目组基准值（三职业相同） |
| `rejob_uc_pm_base_laborers` / `_upper` | 按职业统计的基准值 |
| `rejob_uc_pm_base_laborers_negated` / `_upper_negated` | 按职业的负下限 |
| `rejob_uc_offset_effective_laborers/aristocrats/capitalists` | 按职业截断后的实际施加值 |
| `rejob_refresh_uc_building_mortality` | 刷新单座建筑（供 on_action 调用） |
| `rejob_apply_uc_mortality_to_building` | 按雇佣职业挂容器 |
| `rejob_on_production_method_changed` | PM 变更钩子 |

**新增 on_action**（`common/on_actions/rejob_management_on_actions.txt`）：

```txt
on_production_method_changed = {
	on_actions = {
		rejob_on_production_method_changed
	}
}

rejob_on_production_method_changed = {
	effect = {
		if = {
			limit = { is_building_group = rejob_unemployment_center }
			rejob_refresh_uc_building_mortality = yes
		}
	}
}
```

**性能说明**：PM 变更时只刷新**这一座**建筑（`rejob_refresh_uc_building_mortality`
直接以建筑为 root），不做全国重算。

### 13.15 第八轮改造：拆除后的残留自愈 + 多建筑叠加说明

**用户提问**：
> 1. 这个修正是跟随建筑的吗？在游戏里把当前建筑拆掉，补偿还会留在本 state 吗？
>    会不会随着一起去掉？
> 2. 如果同一个 state 建了很多这个建筑，死亡率修正是叠加的吗？
>    如果是叠加的，能不能改成「生产方式带来的死亡率修正只生效一个」，
>    不产生多个再就业中心叠加的情况；动态调节值也不能因多建筑而叠加。

#### A. 结论：两座建筑的机制**不同**

| 建筑 | 修正类型 | 挂载对象 | 拆除后 |
|---|---|---|---|
| **再就业中心** | `building_<职业>_mortality_mult`（**建筑级**） | **建筑对象** | ✅ **随之消失**，不残留 |
| **集中营** | `state_delicious_mortality_mult`（**州级**） | **州对象** | ⚠️ **会残留** |

**为什么不同**（依据 `modifier_types.md:1-3`）：
> *"the key determines which category the modifier falls into and thereby its
> flow through the modifier graph... a modifier definition key that starts with
> `state_` ... will flow through countries to all states in that country"*

- 建筑级修正挂在**建筑对象**上 → 建筑被拆除，对象销毁 → 修正一起消失
- 州级修正挂在**州对象**上 → 州不会因拆建筑而消失 → 修正残留

> ⚠️ **第五轮之所以会残留，正是因为那时用的是州级。**
> 第六轮起再就业中心已改建筑级，该问题已消除。

#### B. 关于「多个建筑是否叠加」

**再就业中心（建筑级）：不会按人头叠加。**

```
building_laborers_mortality_mult = 4
→ 只作用于【这座建筑】里就业的劳工
→ 3 座中心 = 各自覆盖自己的雇员，同一个人只被算一次
```

对比「真叠加」会长什么样：
```
# 真叠加（州级 + 每个建筑挂一次）
state_laborers_mortality_mult = 4  × 3 个建筑 → 州级 +1200%  ❌
```

建筑级是「每座建筑各自 +400%」，州整体死亡率上升只是因为**被覆盖的人更多了**，
不是每个人被乘了 3 次。

**集中营（州级）：⚠️ 会叠加。**

`state_modifiers.unscaled` 是「建筑激活该 PM 时施加到州」，
同一州 N 个营 → 施加 N 次 → 州级修正叠加。
（这是原模组固有行为；`delicious` 是自定义人口，
引擎没有 `building_delicious_mortality_mult`，无法改为建筑级 ——
原模组作者试过并**全部注释掉**，见 `3266813717/gulag_method_group.txt:12`）

#### C. 本轮实际改动：州级残留自愈

新增 `on_monthly_pulse_state` 钩子（Root = State，
依据 `00_code_on_actions.txt:2038`），每月检查：

```txt
on_monthly_pulse_state = {
	on_actions = { rejob_state_monthly_mortality_check }
}

rejob_state_monthly_mortality_check = {
	effect = {
		if = {
			limit = {
				has_modifier = rejob_camp_mortality_dynamic
				NOT = {
					any_scope_building = {
						is_building_type = building_gulag_logging
					}
				}
			}
			remove_modifier = rejob_camp_mortality_dynamic
		}
	}
}
```

**效果**：集中营被拆除后，最多 **1 个月**自动清掉残留的州级修正，
无需玩家手动点按钮。

> 官方原生就在 `on_monthly_pulse_state` 里追加子 on_action：
> `00_code_on_actions.txt:2038`
> `on_monthly_pulse_state = { on_actions = { colonial_claims_check } ... }`

#### D. 为什么不把集中营也改成建筑级

| 尝试 | 结果 |
|---|---|
| `building_delicious_mortality_mult` | ❌ 引擎无此修正类型（CWTools `modifiers.gen.cwt` 未收录） |
| 原模组作者的尝试 | ❌ 写了 7 处，**全部注释掉**（`gulag_method_group.txt:12/45/75/108/209/239/268`） |
| 官方 `building_<职业>_mortality_mult` | 只有 5 个（academics/laborers/slaves/machinists/engineers），**无 delicious** |

因此集中营只能保持州级 + 本轮新增的月度自愈。

---

### 13.16 第九轮改造：生产项目与死亡率彻底解耦 + 作用域陷阱根治（2026-10-11）

#### A. 需求：生产项目组不再提供死亡率

按需求把「生产项目 / 运转方式」这一大类生产方式自带的死亡率**全部删除**，
死亡率现在**只由「岗位构成 / 关押人群」组**提供。

| 建筑 | 生产项目组 | 删除的死亡率 |
|---|---|---|
| 再就业中心 | `rejob_uc_camp` | 瓷器 400% / 肉类 400% / 石油 400% / 化肥 40000% / 重型化学工艺 9999999 |
| 集中营 | `gulag_camp` | 瓷器 400% / 肉类 400% / 石油 400% / 环保 40000% / 终极解决 9999999 |

同时删除两个「特殊生产方式」及其 PMG 引用：

- `pm_rejob_uc_gasking`（重型化学工艺）
- `pm_Auschviking_gasking`（集中营:终极解决）

并把化肥类 PM 改造成真正的「生产」：

- `pm_rejob_uc_gas`：化肥投入 → **化肥生产**，投入 50 电力 + 150 煤炭，产出 1000 化肥
- `pm_Auschviking_gas`：集中营:环保产业 → **集中营:化肥生产**，同上构成

> 副作用：`rejob_uc_has_special_pm` / `rejob_uc_this_building_has_special_pm` /
> `rejob_camp_has_special_pm` 三个触发器**保留定义但恒为 `always = no`**。
> 不能直接删定义 —— 引用方会报 `unknown scripted trigger` 并使整条效果链静默失效。

#### B. 根治「list 子句内裸名脚本值」的作用域陷阱（160 条 error.log）

**症状**：`error.log` 出现 160 条

```
Error: has_active_production_method trigger
       [ Wrong scope for trigger: country, expected building ]
Script location: common/script_values/rejob_mortality_values.txt:59-63,69-71
```

**根因**（引擎规则，已用原版代码验证）：

```txt
# 在 list 子句（every_scope_*）内，add / max / min 若跟一个【裸名】脚本值，
# 该脚本值会在【外层作用域】求值，而不是被迭代的对象！
every_scope_building = {
    limit = { ... }
    max = rejob_uc_pm_base_building   # ← 在 country 作用域求值！
}
# 原版正确写法（common/script_values/je_values.txt:81）：
every_scope_state = { add = this.level }   # ← 必须写 this.
```

于是 `rejob_uc_pm_base_building` 内部的 `has_active_production_method`
（作用域仅 `building`，见 CWTools `triggers.cwt:1155`）在 country 作用域被求值 → 报错。

**修法**：`rejob_uc_pm_base_country` / `_min_country` /
`rejob_camp_pm_base_country` / `_min_country` 全部改为
**「逐个 PM 判断」**的对称写法，用 `any_scope_state` + `any_scope_building`
（合法作用域 `{ country state }`，见 `lists.cwt:1738`）直接检测
「境内是否存在使用某 PM 的建筑」，只用字面量做 `max` / `min`。

```txt
rejob_uc_pm_base_country = {
    value = 0
    if = {
        limit = {
            any_scope_state = {
                any_scope_building = {
                    is_building_group = rejob_unemployment_center
                    has_active_production_method = pm_rejob_uc_labour
                }
            }
        }
        max = 1        # ← 字面量，无作用域问题
    }
    ...
}
```

#### C. 修复 `add_modifier` 的 multiplier 作用域错误

**症状**：按钮点了没反应，动态死亡率完全不生效（问题 2）。

**根因**：`multiplier` 在**效果链顶层作用域**求值。原写法

```txt
add_modifier = {
    name = rejob_uc_mortality_laborers
    multiplier = rejob_uc_offset_effective_laborers   # ← 该值内部要读 building 作用域
}
```

被 `rejob_refresh_mortality`（root = country）调用时，`multiplier` 在 country 求值，
撞上同样的 `has_active_production_method` 作用域错误，修正链中断。

**修法**：`multiplier` 只用「变量 + 字面量」，作用域无关：

```txt
add_modifier = {
    name = rejob_uc_mortality_laborers
    multiplier = { value = owner.var:rejob_offset  min = -1 }   # 下限按 PM 基准值逐 PM 给出
}
```

（`min = <字面量>` 在 `multiplier = { }` 内是合法的，原版即如此使用：
`min = 1` / `min = -25`。）

#### D. 修复哨兵值泄漏导致的「999999900%」

原写法把哨兵值写在 `if` **外面**：

```txt
rejob_uc_pm_base_min_country = {
    value = 9999999        # ← 无条件先赋值
    if = { limit = { ... } min = 1 }
}
```

一旦没有任何 PM 命中，哨兵值就残留下来 → 日志显示 `999999900%`，
且「各建筑是否一致」的判等永远为假（永远显示范围值）。

**修法**：把 `value = 9999999` **移进 `if` 内部**，只有确认境内存在该类建筑时才置哨兵：

```txt
rejob_uc_pm_base_min_country = {
    value = 0
    if = {
        limit = { rejob_has_unemployment_center = yes }
        value = 9999999
        if = { limit = { ... } min = 1 }
        ...
    }
}
```

#### E. 清理死代码

以下脚本值已无引用，全部删除：

| 已删除 | 原因 |
|---|---|
| `rejob_uc_offset_effective`（含 `_laborers` / `_aristocrats` / `_capitalists`） | 被内联的 `multiplier = { }` 取代 |
| `rejob_camp_pm_base_in_state_negated` | 被 `rejob_camp_offset_effective` 内联的逐 PM 下限取代 |
| `rejob_uc_pm_base_production` | 生产项目组已无死亡率，不再需要 |

#### F. ★ 问题 1（建筑面板商品 I/O 不跟随 PM）的决定性根因 ★

排查结论：**不是脚本 bug，而是本 mod 在游戏里根本没有被启用。**

证据链（全部为磁盘上的实证文件）：

| 证据 | 内容 |
|---|---|
| `content_load.json`（**22:34:06** 写入，游戏 **22:35:01** 启动前） | 启用列表只有 6 项：`2893069455` / `3211725977` / **`3266813717`（原模组「再就业中心 2.0」）** / `3790567715` / `daoyu_cheat_ra` / `3092955121`（OGAS）—— **不含 `rejob_modify`** |
| `logs/game.log` 全文 | 搜索 `rejob` / `gulag` / `Auschviking` / `delicious` → **各 0 次命中** |
| `logs/error.log` 全文 | 搜索 `rejob` → **0 次命中**（第九轮修复的 160 条错误也因此「消失」，但这是假象） |
| 用户 mod 目录 | 只有 `daoyu_cheat_ra` 与 `rejob_modify` 两个 |

**为什么会出现「面板只显示初始 PM 的 I/O」**：

游戏实际加载的是**原模组 3266813717**，而原模组里：

- `gulag_camp` 组引用 `pm_Auschviking_gasking`（本 mod 已删除该 PM）
- 原模组的 `building_groups/gulag.txt:5` 用的是 **`lens = government`**（官方非法值）

两者都会让建筑面板的数据查询失败 —— 这正是 README §11 里描述的
`BuildingProductionMethodsEntry.GetBuildingType` 取不到上下文、
面板退化为「只显示初始生产方式」的现象。

**结论与操作**：这是**环境/启用问题，不是代码问题**。

1. 在启动器的播放集（playset）中**启用 `rejob_modify`**；
2. 同时**禁用原模组「再就业中心 2.0」（3266813717）** ——
   本整合版已自包含其全部内容，同时启用必然定义冲突；
3. 重新启动游戏后，`content_load.json` 中应出现 `...\mod\rejob_modify`，
   且 `game.log` 应能搜到 `rejob`。

> 验证命令（PowerShell）：
> ```powershell
> (Get-Content "$env:USERPROFILE\Documents\Paradox Interactive\Victoria 3\content_load.json" -Raw)
> # 期望能看到 rejob_modify 路径，且看不到 3266813717
> ```

**本次实测记录**（2026-10-11）：

```txt
content_load.json 最后写入 : 22:34:06
游戏进程启动               : 22:35:01
error.log 最后写入         : 22:47:43   ← 整局游戏期间

enabledMods =
  2893069455  Realism Ai Historical Flavor Mod
  3211725977  World War BGM
  3266813717  再就业中心2.0            ← ★ 原模组仍在启用
  3790567715  Daoyu Cheat — Continued
  daoyu_cheat_ra
  3092955121  The OGAS
  （无 rejob_modify）
```

`launcher-v2.sqlite` 中 `rejob_modify` 的状态是 `ready_to_play` + `applied`，
说明**启动器认识这个 mod，只是没被勾进当前播放集**。

**备选方案（直接改启用列表）**：若不便操作启动器，可手动编辑
`content_load.json` 的 `enabledMods`：加入

```json
{"path":"C:\\Users\\CPSU-ZS\\Documents\\Paradox Interactive\\Victoria 3\\mod\\rejob_modify"}
```

并移除 `{"path":"F:\\SteamLibrary\\steamapps\\workshop\\content\\529340\\3266813717"}`。

> ⚠️ 注意：`content_load.json` 由启动器在每次启动时**根据播放集重写**，
> 手动改动会在下次通过启动器启动时被覆盖。因此**首选方案仍是改播放集**。

#### G. 汉化补全：动态修正容器 + 建筑组名 + 自定义修正类型（2026-10-11）

用户反馈两处未汉化：

1. `rejob_uc_mortality_laborers` / `rejob_uc_mortality_aristocrats` /
   `rejob_uc_mortality_capitalists`（还有 `rejob_camp_mortality_dynamic`）——
   建筑面板上显示原始英文 ID；
2. 建筑面板**右上角的建筑组名**显示原始 ID（`rejob_unemployment_center`）。

**根因：两处都写错了 loc key 的命名空间。**

##### ① 静态修正的 loc key = 修正名**本身**（无前缀）

香草实证：

```txt
common/static_modifiers/00_static_modifiers.txt:
    recently_conquered_state = { ... }

localization/simp_chinese/*.yml:
    recently_conquered_state: "最近被征服"        ← 裸名，无前缀
```

本 mod 的 4 个容器此前**完全没有 loc**，因此显示原始 ID。已补齐：

```yml
rejob_uc_mortality_laborers:0 "$laborers$[Concept('concept_pop_growth', '$concept_pop_growth_mortality$')]"
rejob_uc_mortality_laborers_desc:0 "对在本建筑就业的$laborers$[...]的增益或惩罚。"
rejob_uc_mortality_aristocrats:0 "..."
rejob_uc_mortality_capitalists:0 "..."
rejob_camp_mortality_dynamic:0 "$delicious$[...]"
```

显示名格式**对齐香草** `state_laborers_mortality_mult`
（`"$laborers$[Concept('concept_pop_growth', '$concept_pop_growth_mortality$')]"`）。

##### ② 建筑组名的 loc key = **组名本身**（不是 `building_group_<组名>`）

香草实证：**69 个建筑组 ↔ loc 里 72 条 `bg_*` 裸名条目**，一一对应：

```txt
common/building_groups/00_building_groups.txt:  bg_government = { ... }
localization/simp_chinese/*.yml:                bg_government: "政府建筑"     ← 裸名
```

而 `building_group_<组名>_<后缀>` 是**修正类型**的命名空间，例如：

```txt
building_group_bg_government_throughput_add: "$bg_government$[concept_throughput]"
building_group_bg_government_tax_mult: "政府税收"
```

本 mod 原先误写成 `building_group_rejob_unemployment_center`（修正类型的格式），
所以建筑面板右上角取不到组名，退化为显示原始 ID。已改为裸名：

```yml
rejob_unemployment_center:0 "再就业中心"    # ← 正确
gulag_industry:0 "集中营"                  # ← 正确
```

并删除错误的 `building_group_rejob_unemployment_center` / `_desc`。

##### ③ 顺带补全的修正类型 loc（22 条）

`modifier_type_definitions/rejob_old_mortality.txt` 里登记的自定义修正类型
此前大部分**没有 loc**，tooltip 会显示原始 ID。已全部补齐：

| 类别 | 数量 | 示例 |
|---|---|---|
| delicious 人口相关 | 5 | `building_delicious_mortality_mult` / `building_employment_delicious_add` / `country_delicious_pol_str_mult` / `country_delicious_voting_power_add` |
| 建筑组 × delicious 死亡率 | 14 | `building_group_bg_mining_delicious_mortality_mult` … |
| 建筑组 × delicious 生活水平 | 2 | `building_group_bg_mining_delicious_standard_of_living_add` … |

> 注：`concept_voting_power` 在香草中**不存在**，改用香草既有的
> `concept_voting_rights`（"投票权"）。

##### ④ 新增文件

`localization/simp_chinese/rejob_mortality_l_simp_chinese.yml`
（4 个动态容器 + 2 个建筑组名），UTF-8 BOM，与其他 loc 文件一致。

##### ⑤ 校验结果

| 检查项 | 结果 |
|---|---|
| loc key 缺失（`MISSING BOTH`） | **0** |
| loc key 重复 | **0** |
| `$变量$` 引用有效性 | 全部有效 |
| `[concept_xxx]` 引用有效性 | 全部有效 |
| YAML 格式（`key:0 "value"`） | 全部合规 |
| UTF-8 BOM | 5/5 文件均有 |

#### H. ★ 问题 1（建筑面板不随生产方式更新）的真正根因 —— 默认 PM 被科技锁死 ★

**用户观察（关键线索）**：
> 我创建该建筑的时候，不同于默认建筑，新建筑会自动选择初始化的生产方式，不一定是默认，就是每个生产方式的第一个。
> 但是其他建筑在一个新地块建立的时候，会自动选择统一的生产方式（如果建筑界面当前是统一的生产方式）。

**根因**：本 mod 的 6 个 PMG 中，**每个组的第一个 PM（即引擎的默认 PM）都被科技锁死**，
而两个建筑本身**没有** `unlocking_technologies`（开局即可建造）——
形成「建筑可建、默认 PM 却不可用」的矛盾状态，导致新建建筑无法正常初始化 PM。

##### 官方文档依据（游戏自带，比 Wiki 权威）

> ⚠️ `vic3.paradoxwikis.com` 被 Cloudflare 拦截（返回 `Client Challenge`），
> 但**游戏安装目录自带官方 modding 文档**，且更权威、与当前版本严格对应：
> `game/common/production_methods/production_methods.md` 等 80+ 个 `.md`。

```txt
# game/common/production_methods/production_methods.md
pm_key = {
    is_default = yes/no   # is this the default pm for its group?
                          # if no pm in a group is designated as the default,
                          # the first pm is considered the default.
    ...
}

# game/common/production_method_groups/production_method_groups.md
pm_group_key = {
    production_methods = { ... }   # 组内 PM 互斥，任意时刻有且仅有一个激活
    is_hidden_when_unavailable = no
    ai_selection = most_productive   # most_profitable / most_productive
    texture = "..."
}
```

##### 香草实证（197 个 PMG 全量统计）

| 指标 | 数值 |
|---|---|
| 香草 PMG 总数 | **197** |
| 第一个 PM **不需要**科技 | **193**（98%） |
| 第一个 PM **需要**科技 | **4** |

那 4 个例外全部「建筑本身也需对应科技才能建造」：

| PMG | 第一个 PM | 科技 | 所属建筑 | 建筑自身科技 |
|---|---|---|---|---|
| `pmg_explosives_building_chemical_plant` | `pm_leblanc_process` | `nitroglycerin` | `building_chemical_plant` | `intensive_agriculture` |
| `pmg_automobile_production` | `pm_automobile_production` | `combustion_engine` | `building_automotive_industry` | `combustion_engine` |
| `pmg_foundries` | `pm_cannons` | `artillery` | `building_arms_industry` | `gunsmithing` |
| `pmg_base_building_power_plant` | `pm_early_power_plant` | `steam_turbine` | `building_power_plant` | `electrical_generation` |

##### 本 mod 修复前的状态（全部违反该规律）

| PMG | 第一个 PM（引擎默认） | 修复前科技 | 修复前 |
|---|---|---|---|
| `rejob_uc_camp` | `pm_rejob_uc_shao` | `chemical_bleaching`（era_2） | ❌ |
| `rejob_uc_pop` | `pm_rejob_uc_labour` | `central_archives`（era_2） | ❌ |
| `gulag_camp` | `pm_Auschviking_shao` | `chemical_bleaching`（era_2） | ❌ |
| `gulag_pop` | `pm_gulag_labour` | `central_archives`（era_2） | ❌ |
| `rejob_uc_migration` | `pm_rejob_uc_local` | — | ✅ |
| `gulag_migration` | `pm_gulag_local` | — | ✅ |

而两个建筑 `building_rejob_unemployment_center` / `building_gulag_logging`
**都没有** `unlocking_technologies` → 开局即可建造。

##### 日志硬证据

`logs/error.log` 与 `logs/debug.log` 中反复出现（每次打开建筑面板触发）：

```
[pdx_data_callstack.cpp:16]: No context supplied (Use SetDataContext),
    wanted context of type 'BuildingProductionMethodsEntry'
    for 'BuildingProductionMethodsEntry.GetBuildingType'
[pdx_data_localize_helper.cpp:286]: FetchData failed for
    'ProductionMethodGroup.GetNumAvailableOptionsForBuildingType(
        BuildingProductionMethodsEntry.GetBuildingType)'
[pdx_gui_localize.cpp:140]: PdxDataFetchLocalizedData failed for
    'PRODUCTION_METHOD_OPTIONS_BULK'
```

对应 GUI 源码 `game/gui/production_methods.gui:581-711`：

```txt
type old_buildings_production_method_item = flowcontainer {
    block "visible" {
        visible = "[Not(IsDataModelEmpty(BuildingProductionMethodsEntry.GetBuildings))]"
    }
    block "datacontext" {
        datacontext = "[BuildingProductionMethodsEntry.GetBuildingType]"
    }
    ...
    raw_text = "#bold [ProductionMethodGroup.GetNumAvailableOptionsForBuildingType(
                    BuildingProductionMethodsEntry.GetBuildingType)]#!"
}
```

即：**面板取不到当前建筑的 `BuildingType` 上下文** → 退回显示建筑类型级
（而非实例级）的 PM 数据 → 表现为「面板 I/O 不跟随生产方式更新」。

##### 修复

对 4 个「首个 PM」显式加 `is_default = yes` 并**移除其科技要求**，
使默认 PM 与建筑同步可用：

```txt
pm_rejob_uc_shao = {
    texture = "..."
    is_default = yes                 # ← 新增：显式声明为组内默认
    # unlocking_technologies = {     # ← 移除：默认 PM 不得被科技锁死
    #     chemical_bleaching
    # }
    building_modifiers = { ... }
}
```

同理修复 `pm_rejob_uc_labour` / `pm_Auschviking_shao` / `pm_gulag_labour`。

##### 保留的科技要求（正确模式，与香草一致）

非首个 PM 继续保留科技门槛，这正是香草的标准设计
（「默认 PM 免费，升级 PM 需科技」）：

| PM | 科技 | 所在 PMG 位置 |
|---|---|---|
| `pm_rejob_uc_barbed_wire_fences` / `pm_Auschviking_barbed_wire_fences` | `field_works`（era_2） | 第 2 位 |
| `pm_rejob_uc_gas` / `pm_Auschviking_gas` | `nitrogen_fixation`（era_4） | 第 4 位 |
| `pm_rejob_uc_country` / `pm_gulag_country` | `railways`（era_2） | 第 2 位 |

##### 排查过程中排除的假设

| 假设 | 结论 | 依据 |
|---|---|---|
| PMG 缺 `pmg_` 前缀 | ❌ 非硬性要求 | `victoria3.exe` 中 `pmg_` 出现 **0 次**；Morgenroete 用 `manzoni_pmg_` 正常工作 |
| 本 mod 未被启用 | ❌ 已启用 | `content_load.json`（23:06:49）已含 `rejob_modify`，且原模组已移除 |
| `lens` / `parent_group` 非法 | ❌ 已合法 | 与香草合法值集合一致 |
| 修正类型未登记 | ❌ 全部已登记 | 2387 个类型全量比对通过 |
| `on_actions` 追加写法错误 | ❌ 写法正确 | 游戏自带 `_on_actions.md:117-128` 明确给出该写法 |
| 商品键 / gfx 资源非法 | ❌ 全部合法 | 与香草 `00_goods.txt` 及 gfx 目录比对通过 |
| 其他 mod 覆盖 GUI | ❌ 未启用 | 仅 Cold War Project / CMF 覆盖 `production_methods.gui`，但均未启用 |

> **关于 Wiki**：`https://vic3.paradoxwikis.com/Modding` 及其子页
> 全部被 Cloudflare 的 `Client Challenge` 拦截（含 `?action=raw`、`r.jina.ai` 代理、
> `web.archive.org` 均失败）。**替代方案**：游戏安装目录自带的
> `common/*/**.md` 官方文档（80+ 个），与当前版本严格对应，权威性更高。

## 许可与致谢


- 原模组「再就业中心 2.0」（Steam 创意工坊 ID `3266813717`）内容版权归其原作者所有。
- 日志激活方式参考 [The OGAS](https://steamcommunity.com/sharedfiles/filedetails/?id=3092955121)（Steam ID `3092955121`）。
- 本整合版由 `daxuezha` 维护。
