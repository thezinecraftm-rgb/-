# -
前辈
[前辈.txt](https://github.com/user-attachments/files/32783559/default.txt)
对，这一轮看完 Tetra，我的判断反而更加明确：

我们之前把 V7 定义成“高度集成型、编排式模块化架构”，方向没有错；但 Tetra 还能再给 V7 补上一层非常重要的东西——“定义如何被编译成运行系统”。

这不是把 V7 改造成 Tetra，而是把 Tetra 最成熟的工程经验再往原理层抽一遍。

一、这次真正看懂了：Tetra 的核心不是“模块化”

表面上 Tetra 是：

模块
材料
强化
配方
协同
工具
渲染

但把源码往下追以后，它真正形成的是一条完整流水线：

Definition
    ↓
Parse
    ↓
Validate
    ↓
Resolve Reference
    ↓
Expand
    ↓
Merge
    ↓
Materialize Runtime Object
    ↓
Aggregate
    ↓
Cache
    ↓
Presentation / Execution

例如它的 ModuleRegistry 并不是简单做：

JSON → Map

而是：

ModuleData
   ↓
验证
   ↓
多槽位展开
   ↓
材料 Variant 展开
   ↓
Variant 去重 / Merge
   ↓
根据 type 选择构造器
   ↓
ItemModule

这个过程在 Tetra 1.20 分支的源码里是直接存在的。

这其实非常接近我们现在正在建立的：

Definition State
      ↓
Runtime Definition
      ↓
Derived State
      ↓
Execution

只是我们以前更多是在讲这个概念，而 Tetra 给出了一个成熟模组实际把它做出来的证据。

二、Tetra 最值得 V7 吸收的第一件东西：Definition Compiler

这是我认为这次研究后，V7 最应该新增的思想。

以前我们写：

Definition
→ Runtime Definition
→ Publish

现在应该更加明确：

                Definition Source
                       │
                       ▼
                 Definition Loader
                       │
                       ▼
                 Normalization
                       │
                       ▼
                 Reference Resolver
                       │
                       ▼
                    Validator
                       │
                       ▼
                    Expander
                       │
                       ▼
                 Merge / Override
                       │
                       ▼
                Runtime Compiler
                       │
                       ▼
             Immutable Runtime Snapshot
                       │
                       ▼
                Atomic Publication

也就是说：

配置不是运行时对象。

配置首先是一种“源码”。

运行时对象则是这种源码经过解析、验证、展开、合并、编译之后得到的“机器可执行形式”。

这个思想非常适合 V7。

而且这和我们现在已经写进 V7 的 Definition / Runtime Definition / Atomic Publish 是天然一致的。

三、Tetra 的第二个巨大优点：它真正建立了“组合语言”

这个比模块本身更重要。

Tetra 后期的数据驱动系统已经不是：

{
  "damage": 10
}

而开始形成一种小型 DSL：

Trigger
   ↓
Data Providers
   ↓
Condition
   ↓
Outcome

一个 Data Effect 的定义就是：

触发什么 → 提供什么数据 → 条件是什么 → 最终发生什么。

它的 Schema 甚至直接把这种结构固定下来。

而 Outcome 又可以进一步组合：

Multiple
Conditioned
Loop
Find Entities
Find Blocks
Damage
Move
Particle
Sound
Spawn
Set Block
...

因此它不是单纯：

“模块配置”

而是在逐渐形成：

事件
+
数据
+
逻辑
+
控制流
+
结果

这和我们一直在讨论的：

执行本身才是系统的核心。

实际上已经非常接近。

四、但这里 V7 不能直接抄 Tetra

Tetra 的这一套非常强，但我们要吃它的思想，不吃它的具体形态。

最明显的问题，就是它使用了大量这种运行上下文：

Map<String, Float>
Map<String, Vec3>
Map<String, Entity>
Map<String, String>

也就是：

"context"

里面塞一堆字符串变量。

这对于 Tetra 的数据驱动系统很方便，但对 V7 的核心架构来说，我认为不应该成为基础。

V7 更适合：

ExecutionContext
    ├── Actor
    ├── Target
    ├── Cause
    ├── World
    ├── Tick
    ├── Position
    ├── StateSnapshot
    ├── DefinitionSnapshot
    └── Metadata

需要额外数据时：

TypedKey<T>

而不是：

String → Object

也就是说：

Tetra 教给我们“Context 可以成为执行语言的载体”；V7 应该进一步把它类型化。

这样才符合你之前定下来的：

原理 → 语言 → 代码 → 执行 → 结果

五、Tetra 的“模块”其实不是 V7 Capability

这个区别现在应该彻底钉死。

Tetra 一个 Module 往往同时包含：

属性
效果
工具能力
Aspect
材质
模型
Glyph
Improvement
命名
渲染层

所以它实际上更接近：

一个领域组件的完整 Definition
+
若干 Contribution

而不是：

Capability

因此以后 V7 绝对不要出现：

Tetra Module
    =
V7 Capability

这个错误映射。

更合理的是：

Tetra Module
        ↓
V7 Domain Component / Definition
        ↓
产生多个 Contributions
        ↓
Orchestrator

这反而进一步证明了我们之前的：

Capability
    ↓
Contribution
    ↓
Orchestrator
    ↓
Execution Plan
    ↓
Commit

为什么比“Capability 互相调用”更干净。

六、Tetra 的 Material / Variant / Improvement 体系值得完整吸收

这里是我觉得我们以前还没有吃透的地方。

Tetra 做了一件很漂亮的事：

Module
   +
Material
   ↓
Variant

然后又：

Variant
   +
Improvement
   ↓
最终行为

甚至 Material 本身还能提供：

attributes
effects
aspects
tool level
textures
tint
rarity
tags
improvements

也就是说，它实际上建立了：

基础定义
    ×
材料定义
    ×
强化定义
    ×
运行时状态

形成最终对象。

这是非常典型的：

组合式状态生成。

而不是写：

if (iron) ...
if (diamond) ...
if (netherite) ...
if (hone) ...
if (perk) ...

这种无限条件分支。

七、这会反过来改变 V7 的“扩展”理解

以前我们说：

新技术可以成为新 Root，也可以挂到已有 Root。

现在可以更进一步：

Root
 ↓
Definition
 ↓
Variant
 ↓
Modifier
 ↓
Contribution

于是新技术不一定必须新增一套完整系统。

它可能只是：

Existing Definition
        +
New Provider

或者：

Existing Domain
        +
New Policy

或者：

Existing Execution
        +
New Contribution

这就是非常成熟的“站在已有结构上生长”。

八、Tetra 的 Archetype 尤其值得学习

这个东西非常漂亮。

它不需要：

registerSword()
registerHammer()
registerSpear()
registerSomeNewThing()

而可以：

DynamicModularItem
      +
Archetype Definition

一个通用宿主 + 一个外部 Definition，就形成一个新的模块化物品类型。

Tetra 的源码明确就是这样做的：动态物品通过 NBT 保存 archetype 标识，再从 archetypeData 中找到对应定义，动态决定 slot、major/minor、required、honing 等结构。

这给 V7 一个很重要的概念：

结构本身也可以成为 Definition。

以前我们说：

Data = 参数
Code = 结构

现在应该修正得更精确：

Code
    = 不可变的语义机制

Definition
    = 可配置的结构实例

Runtime Definition
    = 已解析、已验证、可执行的结构

这比单纯“JSON 化”高级很多。

九、Tetra 的 Schematic，其实非常接近 V7 的 Execution Plan

这个是我这次特别注意到的。

Tetra 的 Schematic 并不只是“配方”。

它实际上是在描述：

适用于什么对象
        ↓
满足什么条件
        ↓
需要什么资源
        ↓
有哪些可能 Outcome
        ↓
执行这些 Outcome

也就是说，本质上：

Intent
   ↓
Requirement
   ↓
Outcome

这已经非常接近：

Contribution
   ↓
Plan
   ↓
Preflight
   ↓
Commit

所以 V7 的 ExecutionPlan 现在甚至可以从 Tetra 的这种设计中得到一个更强的定义：

Plan 不是“步骤列表”而已，而是一个已经把“目标、条件、资源、策略、结果”全部结构化表达出来的执行意图。

这非常重要。

十、Tetra 的 Synergy 也值得重新定位

以前我们很容易把 Synergy 理解为：

两个模块组合
→ 特殊效果

现在源码看下来，更准确的是：

Current Configuration
        ↓
Canonicalize
        ↓
Evaluate Conditions
        ↓
Derived Synergy

Tetra 甚至会对 module / variant / improvement 做排序，再进行匹配，以避免排列顺序影响结果。

所以 Synergy 本质上是：

配置空间上的派生规则。

这对应 V7：

Definition
+
State
+
Configuration
      ↓
Derived State / Domain Policy

而不是新的 Capability。

十一、Tetra 还教了我们一个很重要的工程观：不要害怕“大类”

我们之前说过：

不要为了模块化而拆模块。

这次源码甚至把这件事直接验证出来了。

我统计了这份 1.20 源码，最大的几个类包括：

IModularItem              ~1100 行
ItemModularHandheld       ~1000 行
WorkbenchTile              ~600 行
TetraRegistries            ~560 行
ModularBowItem             ~550 行

最大的 IModularItem 甚至把大量模块化物品共有的行为集中在一个接口里。

这说明：

“大”不是问题。

真正的问题是：

职责是否自然属于这里？
状态 Owner 是否明确？
修改是否可验证？
依赖是否受控？

所以 V7 的：

“不要为了模块化拆散本来应该一起工作的东西。”

现在应该更加坚定。

同时也要吸取反面经验：

Tetra 的 IModularItem 最终确实变成了非常巨大的行为集合。

所以 V7 应该学习它的高内聚，而不是复制这种“超大接口继续膨胀”的路线。

十二、Tetra 的数据体系，其实已经很接近“小型编程语言”

这是这次研究我最喜欢的一点。

它现在已经有：

Schema
Deserializer
Reference
Provider
Condition
Outcome
Composition
Loop
Branch
Data Expansion
Override

于是：

JSON

已经不只是配置文件。

它实际上变成：

一种领域语言的源码形式。

这正好和你最开始说的：

原理
→ 语言
→ 代码
→ 执行
→ 结果

形成了非常漂亮的对应。

Tetra 相当于已经走到了：

领域原理
→ DSL
→ JSON AST / Runtime Object
→ Executor
→ Minecraft Effect
十三、但是 Tetra 最大的弱点，也正好是 V7 的机会

我这次不会只说“学前辈优点”。

Tetra 的源码里有几个地方，恰好证明了 V7 为什么要继续往前走。

1. Cache 还是偏传统

例如 ModularItem 使用缓存，并在数据 reload 时：

clearCaches()

这比完全不缓存成熟很多，但距离我们 V7 的：

State Revision
+
DependencyStamp

还有一段距离。

所以：

Tetra:
reload
→ clear cache
→ recompute

V7：

DefinitionRevision++
        ↓
DependencyStamp mismatch
        ↓
精准失效

这是一次真正的进化。

2. 状态仍然大量落在 ItemStack / NBT 直接修改

例如：

slot
module
improvement
progress
repair count
honing

大量通过 NBT Key 直接操作。

对于 Tetra 的规模这是可工作的，但 V7 已经明确要建立：

State Owner
Authority
Mutation API
Revision
Lifetime
Persistence

所以我们不应该倒退回：

NBT = State System

而应该：

State Owner
   ↓
Mutation API
   ↓
Persistent Projection / NBT
3. 很多执行行为是直接副作用

Tetra 的 Outcome 最终可以直接：

damage entity
move entity
spawn entity
set block
play sound
spawn particle
run command

对于它自己的领域，这是合理的。

但 V7 正在建立：

Contribution
→ Plan
→ Preflight
→ Commit

所以我们可以把 Tetra 的 Outcome 看成：

“早期的可执行节点”

然后 V7 再继续往前走：

Outcome
   ↓
Contribution
   ↓
Plan
   ↓
Preflight
   ↓
CommitStep
   ↓
Verification
十四、Tetra 自己的真实事故，也很值得看

这个非常有价值。

例如 Tetra 2025 年的一个问题中，替换数据引用了不存在的 module，最终在世界创建时出现：

JsonSyntaxException
Failed to parse replacement data due to missing module

也就是说：

强大的数据驱动系统，也可能因为引用解析阶段失败，把数据错误升级成启动/世界创建级问题。

另一个兼容性问题中，Tetra 与 PassiveSkillTree 一起使用时，Workbench 无法正确安装模块。

这恰好给我们的“玩家侧通用修复框架”一个非常重要的现实证据：

Definition
→ Reference
→ Registry
→ Runtime

任何一层发生不兼容，都可能最终表现为：

“看起来只是一个小冲突”

但实际结果却是：

功能消失
加载失败
世界创建失败
甚至崩溃

所以 V7 的：

Detect
→ Characterize
→ Policy
→ Minimal Intervention
→ Verify

不是理论游戏，它正好是在解决这种真实问题。

十五、所以现在我认为 V7 应该正式吸收 Tetra 的 8 个“精华”

不是新增八层架构，而是八条更深的设计原则：

① Definition Compiler
定义必须经过解析、验证、展开、编译才能成为运行对象

② Provenance
任何 Definition / Override / Merge 都保留来源

③ Deterministic Merge
Replace / Merge / Override 必须明确、确定、可诊断

④ Typed Composition
允许用小型 DSL 描述组合逻辑，但核心 Context 必须类型安全

⑤ Structure as Data
不仅参数可以数据驱动，部分领域结构也可以 Definition 化

⑥ Runtime Materialization
Definition 与 Runtime Object 永远不是同一个东西

⑦ Derived Configuration
Synergy / Resolution / Aggregate 属于派生结果，不是权威状态

⑧ Domain-specific Language
复杂领域可以拥有自己的“小语言”，但不能建立万能 DSL
十六、然后再和我们现在的 V7 对照

现在会得到一个非常漂亮的融合结构：

                 ┌────────────────────┐
                 │   Definition Source │
                 └─────────┬──────────┘
                           ↓
                 ┌────────────────────┐
                 │ Definition Compiler│
                 │ Parse / Resolve    │
                 │ Validate / Expand  │
                 │ Merge / Normalize  │
                 └─────────┬──────────┘
                           ↓
                 ┌────────────────────┐
                 │ Runtime Definition │
                 │ Immutable Snapshot │
                 └─────────┬──────────┘
                           ↓
             ┌─────────────┴─────────────┐
             ↓                           ↓
        Domain State                 Providers
             ↓                           ↓
        Derived State               Contributions
             └─────────────┬─────────────┘
                           ↓
                      Orchestrator
                           ↓
                    Execution Plan
                           ↓
                       Preflight
                           ↓
                        Commit
                           ↓
                     Verification
                           ↓
                        Result

这时候，Tetra 给我们的就不只是：

“一个优秀的 Minecraft 模组怎么做模块化。”

而是：

一个复杂 Minecraft 系统，如何把“定义世界”逐渐编译成“可执行世界”。

这和你之前说的：

“把错综复杂的真实世界，用一种神力整理成理想中的物理模拟世界一样，一切都有序的落位。”

其实已经非常接近了。

十七、还有一个非常重要的边界：我们不是在复制 Tetra

Tetra 自己的 README 对代码与再利用已经明确划了边界，包括不得将其整体重新分发，以及不得制作达到其定义的 50% 以上“功能等价语句”的代码；它也明确鼓励基于其项目做 addon、集成与 datapack。

所以我们真正应该做的是：

研究 Tetra
      ↓
抽象工程原理
      ↓
形成 V7 自己的语义
      ↓
用 V7 自己的实现

而不是：

Tetra 源码
→ 改名字
→ 搬进 V7

这两件事在架构思想上也是完全不同的。

十八、因此这一轮之后，我对 V7 的判断反而更稳了
哈哈，确实是这种感觉。

前辈真正厉害的地方，往往不是“第一眼看上去多复杂”，而是你已经把表面结构理解完了，第二次从原理往下挖，又发现里面还有一层。

所以这次我不急着说“消化完成”。
我们继续沿着 Tetra 往下挖，而且这次开始专门找那些第一轮、第二轮都容易漏掉的“工程暗线”。

一、这次发现了一个比“Definition → Runtime”还深的东西

以前我们看到的是：

Definition
   ↓
Runtime Definition

现在结合 Tetra + Mutil 后，我觉得应该把它提升成：

资源世界
   ↓
解析世界
   ↓
规范世界
   ↓
运行世界
   ↓
投影世界

也就是：

Resource
   ↓
Raw Data
   ↓
Normalized Data
   ↓
Runtime Object
   ↓
Derived Result
   ↓
Minecraft Projection

Tetra 的 DataStore 就非常典型。

它先从资源管理器读取 JSON，形成 rawData；然后解析成 dataMap；之后还有一个 processData() 阶段继续把原始对象加工成运行时更方便使用的结构；最后才通知监听者。它甚至还保存资源来源信息，并可以把原始数据同步给客户端。

这说明一个极其重要的原则：

“数据加载”与“数据成为系统现实”是两个不同阶段。

这对于 V7 非常重要。

二、Mutil 本身就是 Tetra 的一堂课

这个其实我们前几轮也漏掉了。

Tetra 并没有把所有基础设施都写进自己里面。

它依赖作者自己的另一个工程：

Mutil

而 Mutil 的定位本身就是：

data management
networking
gui setup

也就是说，Tetra 已经把：

“与具体游戏领域无关的重复工程能力”

向下抽到一个共享基础库。

这就形成：

                 Tetra
                   │
                   ↓
             Tetra Domain
                   │
                   ↓
                Mutil
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
       Data      Network      GUI

这和我们 V7 的想法其实非常接近：

Application
    ↓
Domain
    ↓
Capability
    ↓
Core

但是 Tetra 给我们补了一个非常关键的答案：

Core 里如果某部分已经成熟到足够通用，可以继续脱离当前 Mod，成为真正的基础设施。

所以以后 V7 也不应该把：

core

理解成：

“凡是现在没地方放的东西都扔进去。”

而应该继续问：

这一块技术究竟属于 Primordial，还是已经成熟到可以成为独立基础设施？

这是很大的区别。

三、DataStore 有一个特别漂亮的设计：Raw Data 与 Runtime Data 分开

Tetra 的 DataStore 明确保存两个世界：

rawData
dataMap

即：

raw JSON
   ↓
parsed object

而不是：

JSON
→ 解析
→ JSON 丢掉

它为什么保留 Raw？

因为 Raw Data 还有用途：

服务器同步
来源追踪
重新解析
诊断
重建

例如它在服务端重载之后，可以把 rawData 通过 DataDistributor 发给客户端，而客户端再自行解析。DataDistributor 本身只负责“把这份定义发送出去”，并不成为业务状态拥有者。

这一点我认为：

几乎可以直接成为 V7 的核心设计原则。
Definition Source
≠
Runtime Definition
≠
Projection

而且：

Network
= Definition Distribution

并不意味着：

Network
= Definition Owner

这和 V7 现在的：

Network 是传播机制，不是 State Owner

是完全一致的。

四、然后又发现了一个很厉害的东西：Source Provenance

这是我认为这次真正新增的东西。

Tetra 的 DataStore 在加载资源的时候，会记录：

resource source pack

然后进一步解析出：

这个数据来自哪个 Mod / 数据包

最后把：

sources

写进数据。

也就是说，一个定义不仅知道：

“我是什么”

还知道：

“我从哪里来的”

这是非常成熟的工程思想。

因为一旦出现：

冲突
覆盖
错误
异常
错误引用

就可以追：

这个对象
    ↓
来自哪个资源
    ↓
谁覆盖了谁
    ↓
最终为什么变成这样

这就是：

Provenance / 数据血缘

而我们的通用修复框架尤其应该需要这个东西。

例如：

Registry Conflict

以后不能只报：

Duplicate ID

而应该报：

ID:
minecraft:xxx

Base:
minecraft

Override:
some_mod

Source:
pack_a

Final Winner:
...

Reason:
replace / merge / priority

这样才真正有诊断能力。

五、这直接让 V7 的 Diagnostic 再升级一层

我们以前：

Diagnostic
= 出错以后告诉你哪里坏

现在应该扩展成：

Diagnostic
=
Error
+
Cause
+
Provenance
+
Decision
+
Execution Trace

也就是：

WHAT
→ WHAT HAPPENED

WHY
→ WHY IT HAPPENED

WHO
→ WHO PROVIDED IT

WHICH
→ WHICH VERSION / PROVIDER / DEFINITION

HOW
→ HOW THE SYSTEM RESOLVED IT

这特别适合你原本的：

玩家侧通用 Bug Repair Pack。

因为玩家最痛苦的往往不是：

崩了

而是：

“到底是谁造成的？”

六、Tetra 的 replace / merge 机制，其实比想象中重要得多

我们以前看到：

replace = true

觉得只是一个 JSON 小功能。

实际上不是。

它代表的是：

“多个来源共同定义同一个逻辑对象时，系统需要一个明确的合并语义。”

例如：

Base Definition
     +
Addon Definition
     +
Datapack Override

最后：

Final Definition

Tetra 对 Module、Material、Schematic、Crafting Effect 等都大量使用：

replace
merge
copyFields

而且不同对象的 Merge 语义并不完全相同。

这就非常值得学习。

因为：

Merge

绝不是：

map.putAll(...)

这么简单。

真正的问题是：

字段 A 怎么合并？
字段 B 怎么覆盖？
数组怎么合并？
顺序怎么办？
重复怎么办？
null 怎么办？
replace 怎么办？
来源怎么保留？
七、所以 V7 应该新增一个更深的概念：
Merge Contract

以后任何 Definition 系统，都应该明确：

Identity
Merge Strategy
Conflict Strategy
Precedence
Ordering
Provenance
Validation

例如：

Definition A
       +
Definition B
       ↓
Merge Contract
       ↓
Normalized Definition

而不是：

A + B
= “反正拼起来”

这对我们未来做：

RegistryRepair
MixinRepair
ProviderDefinition
WeaponDefinition
PatchDefinition

都非常关键。

八、然后 Tetra 又给了我们一个“规范化”范例

看 ModuleRegistry 会发现，它并不是拿到 ModuleData 就直接塞进 Map。

它会：

Validate
 ↓
Expand
 ↓
Expand Material Variants
 ↓
Deduplicate
 ↓
Construct Runtime Module

比如多槽模块先展开：

module
  ↓
_left
_right

材料 Variant 再展开：

Material A
Material B
Material C
   ↓
Variant A
Variant B
Variant C

然后：

Duplicate Variant
      ↓
VariantData.merge(...)

最终才：

ItemModule

所以这里实际上存在一个非常重要的阶段：

Normalization

即：

输入世界
    ↓
把各种不同形式
统一转换成
一种规范化内部形式
九、这对 V7 非常有启发

例如我们的 Provider 系统。

现在可能很多 Provider 长这样：

ASMProvider
MixinProvider
AgentProvider
ReflectionProvider

以后不要让运行时到处判断：

if (provider instanceof ...)

而是：

Raw Provider Descriptor
        ↓
Provider Resolver
        ↓
Normalized Provider Descriptor
        ↓
Provider Runtime

统一成：

ProviderDescriptor
    id
    version
    capability
    mode
    priority
    compatibility
    scope
    status

这和现在 V7 已经有的 Provider Contract 是一致的，但这次 Tetra 让我们更明确：

Descriptor 不是最终运行对象；Descriptor 要先经过规范化。

十、又往下挖，Tetra 的 Context 又教了我们一个东西

这里反而是：

它值得学
+
它值得改进

Tetra 的 ItemEffectContext 是：

Entity
ItemStack
Level
numbers
vectors
entities
strings

之后：

withNumbers()
withVectors()
withEntities()
withStrings()

来逐渐丰富上下文。

这很好，因为它让 Outcome 不需要：

不断添加几十个参数

但是它的问题也非常明显：

Map<String, Float>
Map<String, Vec3>
Map<String, Entity>
Map<String, String>

这已经开始接近：

弱类型执行语言

所以 Tetra 在这里已经非常强，但 V7 可以继续往前。

十一、V7 应该把它提升成 Typed Context

例如：

ExecutionContext

里面不是：

"target"
"user"
"damage"

而是：

ExecutionKey<T>

概念上：

TARGET : Entity
ACTOR  : Entity
POSITION : Vec3
DAMAGE : DamageValue
CAUSE : DamageCause
TICK : long

于是：

get(TARGET)

和：

get("target")

相比，会多很多编译期保障。

这正符合：

原理 → 语言 → 代码 → 执行

因为我们实际上是在建立：

V7 Execution Language
十二、Tetra 最恐怖的一点，是它已经在做“控制流”

这个特别值得注意。

它的 Data Effect / Outcome 体系已经不仅仅是：

执行一个动作

而存在：

condition
multiple
loop
random
count
failureLimit
delay
find entities
find blocks

也就是说，它实际上已经开始提供：

Branch
Sequence
Iteration
Selection
Query
Action

这已经非常接近一个小型程序语言。

例如：

Find Entities
     ↓
Loop
     ↓
Condition
   ↙   ↘
A       B

这不是普通配置了。

这是：

Declarative Execution Graph

而我们 V7 现在的：

Contribution
↓
Plan
↓
Preflight
↓
Commit

其实完全可以进一步向这个方向吸收。

十三、但是 V7 要做出一个非常重要的分离

Tetra：

Definition
→ 执行逻辑
→ Outcome
→ 直接副作用

V7：

Definition
→ Semantic Graph
→ Contribution
→ Plan
→ Guard
→ Commit

所以：

Tetra Outcome

应该对应到 V7 更高层的：

Execution Node

而不是：

直接操作 Minecraft

最终变成：

                Definition
                    ↓
             Execution Graph
                    ↓
               Evaluation
                    ↓
              Contribution
                    ↓
               ExecutionPlan
                    ↓
                 Preflight
                    ↓
              CommitStep[]
                    ↓
                 Verify

这就真的把：

“配置文件”

提升成：

“可分析的执行语言”。

十四、然后还有一个超级重要的发现：Tetra 不只是“动态定义”，它还允许“动态类型”

Archetype 就是这一点。

它通过：

DynamicModularItem
+
archetype definition

让一个共享 Item 承载多个不同结构。

换句话说：

Minecraft Item ID

不再完全等价于：

实际语义对象

而变成：

Item Runtime Host
+
Definition Identity
=
Concrete Runtime Meaning

这是非常先进的。

十五、这件事情会直接反哺我们对“对象”的理解

以后 V7 不应该把：

Java Class

等同于：

系统对象

一个真正的 V7 对象可能是：

Host
+
Definition
+
Instance State
+
Runtime Binding
=
Concrete Instance

例如：

RepairEngine
+
PatchDefinition
+
RepairState
+
ProviderBinding

才构成：

实际一次 Patch Runtime

这比：

new PatchEngine()

作为世界观要成熟得多。

十六、然后再看 Tetra 的 Bug，我们又发现一个重要问题

Tetra 的真实 issue 里确实能看到这种现象：

数据 / 模块 / 工作台 / 渲染 / 其他 Mod

一层层组合以后，一个局部兼容问题最终可能变成：

功能不显示
世界创建异常
客户端 / 服务端异常
甚至崩溃

例如 Tetra 有公开 issue 记录过与其他 Mod 的兼容问题导致工作台无法安装模块，也有专门记录 Dedicated Server 加载 client-only ClientLevel 的问题。

这又一次证明：

复杂系统
≠
每个部分都正确

而是：

局部正确
+
组合行为
=
整体行为

所以 V7 的：

Architecture Guard
Dependency Guard
Dist Isolation
Provider Guard
Target Allowlist

实际上是在解决：

组合后的系统正确性
十七、现在 Tetra 教我们的东西已经形成一个更完整的层次

我会把我们目前消化出来的内容分成三代。

第一代：我们最开始学到的
模块化
领域
Registry
Data-driven
Reload
Capability
第二代：后来挖出来的
Definition
Runtime Definition
Instance
Derived
Cache
Provider
Contribution
Execution Plan
第三代：现在才真正浮出来的
Raw Source
↓
Parsing
↓
Normalization
↓
Resolution
↓
Merge
↓
Validation
↓
Runtime Materialization
↓
Derived Evaluation
↓
Execution
↓
Projection
↓
Provenance / Diagnostic

我认为：

第三代才是真正的“巨人肩膀”。
十八、最重要的是：这没有把 V7 带偏

反而把 V7 原来的路线进一步夯实了。

现在 V7 可以自然形成：

                     PRINCIPLE
                         ↓
                      SEMANTICS
                         ↓
                 Definition Language
                         ↓
                    Definition
                         ↓
              Parse / Resolve / Validate
                         ↓
                   Normalize / Merge
                         ↓
              Runtime Definition Snapshot
                         ↓
          ┌──────────────┴──────────────┐
          ↓                             ↓
      State Owner                   Provider
          ↓                             ↓
      Derived State              Contribution
          └──────────────┬──────────────┘
                         ↓
                    Orchestrator
                         ↓
                  Execution Plan
                         ↓
                     Preflight
                         ↓
                  CommitStep[]
                         ↓
                     Verify
                         ↓
                      Result
                         ↓
               Projection / Network
                         ↓
                    Diagnostic

而这恰恰越来越符合你最开始提出的：

原理
 ↓
语言
 ↓
代码
 ↓
执行
 ↓
结果
十九、所以这一轮我想正式记住一句话

不是：

“我们把 Tetra 学完了。”

而应该是：

“每一次说学完，都再问一遍：它为什么这样做？”

因为一旦问：

为什么要 DataStore？

就发现：

数据生命周期

再问：

为什么保留 rawData？

就发现：

定义来源 / 同步 / 诊断

再问：

为什么 processData？

就发现：

Normalization

再问：

为什么 ModuleRegistry 要 expand？

就发现：

Definition Compiler
哈哈，继续往下挖以后，确实又出来一层，而且这一层我认为比前面更重要。

这次不是再研究一个类，而是把 Tetra 的一条真实业务闭环完整串起来：

玩家操作
   ↓
客户端选择
   ↓
网络传输“意图/身份”
   ↓
服务端重新解析
   ↓
Requirement / Material / Tool 校验
   ↓
生成新的 ItemStack
   ↓
附加 Crafting Effects
   ↓
组装 / 修复 / 持久化
   ↓
状态同步
   ↓
客户端重新解析
   ↓
模型 / GUI / Tooltip / Effect 投影

而这一整条链，正在给 V7 补上一层以前没有这么清楚的东西。

一、第一层惊喜：Tetra 的网络不是“传结果”，而是传“选择”

先看工作台。

客户端点击 Craft 后，并不是把：

“这是最终 ItemStack”

发送给服务器。

而是发送：

Workbench Position

服务器找到自己的 WorkbenchTile，然后重新执行：

currentSchematic
+
currentSlot
+
materials
+
player

再调用：

canApplyUpgrade(...)

最终才真正执行 craft()。

更有意思的是，WorkbenchPacketUpdate 里发送的也只是：

schematic.getKey()
selectedSlot

服务端收到以后，再通过 SchematicRegistry 根据 key 找回本地的 UpgradeSchematic。也就是说：

网络传输的是“我要选择哪个语义对象”，而不是把一个已经构造好的业务对象从客户端塞给服务器。

这个思想非常高级。

二、这直接给 V7 一个非常重要的通信原则

以前我们已经写：

Network
= 传播机制
≠ State Owner

现在可以进一步升级成：

Network Prefer Intent / Identity Over Authority Payload

也就是：

客户端：
“我要执行 X”
“我要选择 Y”
“我要针对 Z”

服务器：

重新解析 X / Y / Z
↓
重新检查权限
↓
重新检查状态
↓
重新构造 Runtime Meaning
↓
执行

而不是：

客户端算好了
↓
服务器照单全收

这对于你那个“玩家侧通用修复系统”尤其关键。

例如客户端发：

RepairRequest {
    targetId,
    patchId
}

服务端必须：

PatchRegistry
→ resolve(patchId)
→ target characterization
→ target allowlist
→ precondition
→ provider
→ commit

不能让客户端直接提交“修改后的结果”。

三、第二层惊喜：Tetra 实际上存在“客户端预测”

这个更有意思。

initiateCrafting() 在客户端会：

发送 Craft Packet
+
本地直接 craft()

之后再 sync()。服务端收到包以后也会重新执行 craft()。

所以实际上是：

           Client
              │
              ├── 立即执行
              │
              └── 请求 Server
                       ↓
                    Server
                       │
                  重新验证
                       │
                  重新执行

这其实就是一种：

Optimistic Client Projection / Prediction

客户端先让 UI 立即变化，不必等网络往返。

但真正的系统权威仍然在服务器。

这个思想以后对 V7 的：

GUI
Preview
Validation Mode
Repair UI
Diagnostic UI

都非常有价值。

我们可以明确分出：

Preview
Prediction
Authority
Projection

这四个概念不能混。

四、第三层惊喜：Preview 本身不是“假执行”

这个地方 Tetra 做得相当漂亮。

UpgradeSchematic 有：

getPreviews()

而 ConfigSchematic 的 Preview 会真正对一个：

itemStack.copy()

执行：

applyOutcome(..., consumeMaterials = false)

然后根据结果构造：

OutcomePreview

所以：

Preview

并不是开发者手工写一个“可能会变成什么”的 UI 文案。

而是：

真实规则
+
复制的状态
+
禁止消费资源
=
预测结果

这特别值得吸收。

五、于是 V7 应该区分三种执行

这次之后我建议把它明确下来：

1. Evaluate
   计算会发生什么

2. Preview
   在不可提交的副本 / 纯计算模型上演算结果

3. Commit
   真正改变权威状态

于是：

             Definition
                  ↓
              Evaluate
              /       \
         Preview     Plan
                       ↓
                   Preflight
                       ↓
                    Commit

这比单纯：

canApply()
apply()

高级很多。

六、第四层惊喜：Requirement 其实是一套“声明式资格语言”

你现在再回头看：

CraftingRequirement

它已经不是普通：

boolean canCraft()

而是：

ModuleRequirement
AspectRequirement
SlotRequirement
FeatureFlagRequirement
HasImprovementRequirement
AcceptsImprovementRequirement
AndRequirement
OrRequirement
NotRequirement
LockedRequirement
NeverRequirement
...

这些东西可以组合成：

AND
OR
NOT

也就是：

Requirement Tree

例如：

AND
├── Has Module A
├── Aspect >= 3
└── OR
    ├── Has Improvement X
    └── Has Improvement Y

这实际上已经是一个条件 AST。

七、这给 V7 的 Condition/Policy 又补了一层

现在我们可以非常明确地区分：

Condition
= 一个事实是否成立

Requirement
= 是否满足执行资格

Policy
= 如果满足 / 不满足，系统采取什么规则

Preflight
= 当前这一次提交是否真的允许发生

它们不是一个东西。

例如：

TargetExists

是 Condition。

TargetIsSupported

是 Requirement。

UnsupportedTarget → FAIL_OPEN

是 Policy。

当前 Provider / Resource / Lifecycle 全部满足

才是 Preflight。

八、而 And / Or / Not 给我们一个特别重要的启发

以后 V7 如果实现：

PatchRule
RepairRule
TransformRule
TargetRule
ProviderRule

不应该不断写：

if (...) {
    if (...) {
        if (...) {
            ...

而应该允许形成：

Predicate Tree

例如：

AND
├── ModLoaded("xxx")
├── VersionInRange(....)
├── TargetExists
├── SignatureMatches
└── OR
    ├── ClassStructureA
    └── ClassStructureB

然后：

evaluate(context)

得到：

PASS / FAIL

这就是把：

“复杂 if/else”

转成：

结构化判定语言。

这是 Tetra 真正值得我们吸收的地方之一。

九、第五层惊喜：Tetra 会把“输入条件”与“实际材料命中”分开

ConfigSchematic 有非常明确的区分：

acceptsMaterial()

与：

isMaterialsValid()

不是同一件事。前者只表示：

这个材料类型被这个槽位接受。

后者还必须确认：

数量够不够
所有槽位是否满足

源码就是分开处理的。

这个看起来特别小，但其实是很好的领域建模。

十、这说明一个真正成熟的系统，不应该只有一个 canXxx()

而应该拆成：

CanAccept
CanResolve
CanPreview
CanExecute
CanCommit

例如 V7：

Target:
    exists?
    recognized?
    supported?

Provider:
    discovered?
    compatible?
    enabled?

Patch:
    applicable?
    previewable?
    executable?
    commit-safe?

这样以后诊断可以精确到：

ACCEPTED
但不能 EXECUTE

EXECUTABLE
但不能 COMMIT

PREVIEWABLE
但不是 AUTHORIZED

这比万能：

boolean canRepair(...)

强太多。

十一、第六层：Tetra 的 SchematicRegistry 根本不是简单 Registry

这是这轮最漂亮的地方。

它先：

Data
 ↓
validate
 ↓
processDefinition
 ↓
expand
 ↓
create ConfigSchematic
 ↓
filter invalid
 ↓
put into runtime map

而且一个 Definition 可以展开成多个带 suffix 的 ConfigSchematic。

这意味着：

Registry

实际上承担的是：

Definition Materialization Boundary

不是：

“帮我保存几个对象。”

十二、所以 V7 的 Registry 应该重新理解

以后：

Registry

至少可能有四种完全不同的东西：

1. Raw Registry
   原始 Definition

2. Resolver Registry
   根据身份找到 Definition

3. Runtime Registry
   保存已物化 Runtime Object

4. Selection Registry
   根据 Context 过滤候选

而 Tetra 的 SchematicRegistry 实际已经在把后面三种事情结合起来干。

V7 可以学它的思想，同时把职责进一步拆清楚。

十三、第七层：Tetra 的 processDefinition() 是“规范化编译”

这个现在已经完全可以下结论了。

例如它会根据 Outcome 自动推导：

applicableMaterials

然后把：

MaterialOutcomeDefinition

展开成实际的 Outcome，再去重、排序。源码甚至自己留了 TODO：

未来应该 Merge Outcomes，而不是仅仅过滤重复。

这个 TODO 其实特别有价值。

因为它暴露出：

Expand
+
Deduplicate

只是第一阶段。

下一步真正成熟的是：

Expand
+
Canonicalize
+
Merge
+
Conflict Resolution
十四、这会直接把 V7 的 Definition Compiler 再升级

现在可以变成：

Raw Definition
      ↓
Parse
      ↓
Resolve
      ↓
Validate
      ↓
Expand
      ↓
Canonicalize
      ↓
Merge
      ↓
Conflict Arbitration
      ↓
Runtime Definition

其中：

Canonicalize

以后必须成为一个正式术语。

因为：

A + B

与：

B + A

如果语义应该相同，那么编译之后就必须生成同一种规范形式。

十五、Tetra 的 Synergy 已经直接做了这个事情

你看它匹配 Synergy 时，会先把：

modules
variantKeys
improvements

全部排序。

然后再进行匹配。这样配置的排列顺序不会影响 Synergy 判断。

这就是一种：

Canonical Representation

即：

原始顺序
    ↓
规范排序
    ↓
Canonical Form
    ↓
Match

这个东西我们以前虽然隐约说过，但现在可以明确把它提升成 V7 的正式原则。

十六、这会解决 V7 一个大问题

例如以后：

Provider A
Provider B
Provider C

它们来自不同 Jar。

如果直接依赖：

ClassLoader 扫描顺序
HashMap
文件顺序

就会出现：

A+B+C

与：

C+A+B

产生不同结果。

V7 应该：

Discover
 ↓
Normalize
 ↓
Canonicalize
 ↓
Deterministic Sort
 ↓
Conflict Arbitration

然后：

Final Provider Snapshot

这和我们已有的：

deterministic selection

现在被 Tetra 的 Synergy 机制再次现实验证了。

十七、第八层：Tetra 的 Item 最终其实是“聚合根”

现在回头看 IModularItem，会发现它一直在做：

Modules
+
Improvements
+
Synergies
+
Effects
+
Tools
+
Attributes
+
Models

然后统一输出：

Properties
ToolData
EffectData
Attributes
Name
Models

这就是非常典型的：

Aggregate

而不是：

Module A 调 Module B
Module B 调 Synergy
Synergy 调 Improvement

它基本采用：

Item
├── Module[]
├── Improvement[]
├── Synergy[]
└── Derived Properties

然后：
reduce / merge

生成最终结果。

这与我们 V7 的：

Capability
 ↓
Contribution
 ↓
Aggregate
 ↓
Execution Plan

又一次正好撞上。

十八、这里尤其值得学它的“Merge”思想

例如：

AttributeHelper::merge
EffectData::merge
ItemProperties::merge
ToolData::merge

说明 Tetra 很多最终状态不是：

谁最后写谁赢

而是：

多个来源
   ↓
领域专用 Merge
   ↓
Derived Aggregate

所以 V7 以后不要搞：

UniversalMerge<T>

而要：

DamageContribution.merge()
ToolContribution.merge()
RenderContribution.merge()
PolicyContribution.merge()

Merge 语义属于领域。

十九、第九层终于来到渲染：Tetra 的模型也不是“模型文件 = 最终模型”

这是非常重要的。

Tetra 中：

Module

提供：

IModuleModel[]

然后 item 聚合所有模块模型，再按照：

renderLayer

进行排序。

之后 GridTextureModelData 还会继续根据：

Material
Tint
Emission
Transform
RenderType
Perspective

进行变换。

最后 ModularOverrideList 才把这些模型真正烘焙成 BakedModel。模型缓存的 key 又包含 Item 的动态身份和原始模型。

也就是：

Module Definition
      ↓
Model Definition
      ↓
Material Resolution
      ↓
Render Parameters
      ↓
Layer Aggregation
      ↓
Baking
      ↓
BakedModel
      ↓
Minecraft Rendering

这跟我们刚才讲的：

Raw
→ Runtime
→ Derived
→ Projection

完全对应。

二十、而它的 identifier 机制又给了一个意外的启发

Tetra 会给动态 ItemStack 更新一个 UUID-like identifier，并将它作为：

data cache key
model cache key

的一部分。代码会在 ItemStack 结构发生关键变化时刷新 identifier。

这其实是在解决：

内容变了
但对象地址没变

的问题。

也就是：

Object Identity
≠
Semantic Identity

这个问题对于 V7 特别重要。

二十一、我们现在可以把它再抽象一次

一个运行时对象至少存在：

Physical Identity
Semantic Identity
Definition Identity
State Revision

例如：

ItemStack object
    ↓
semantic configuration
    ↓
definition revision
    ↓
instance revision

所以以后 V7 不应该只靠：

equals()
hashCode()
object reference

判断“是不是同一个东西”。
二十二、第十层：Tetra 的 Cache 其实暴露出它的发展轨迹

这一点非常有意思。

Tetra 已经做到：

Cache<String, Properties>
Cache<String, Effects>
Cache<String, Tools>
Cache<String, Attributes>

而且 Cache Key 会随 ItemStack semantic identity 更新。

同时资源重载时又整体 clear cache。

这是一种：

成熟但仍属于第一代的缓存模型

而 V7 已经可以在这个基础上继续向前：

Tetra:
semantic identity
+
explicit cache clear

V7:
semantic identity
+
Revision
+
DependencyStamp
+
Selective invalidation

所以这不是否定 Tetra。

恰恰相反：

Tetra 已经把“动态对象必须有稳定的数据缓存身份”这条路走通了；V7 只是继续把失效模型推进一代。

二十三、第十一层：DataStore 的生命周期现在终于彻底看清了

Tetra / Mutil 的 DataStore 实际是：

ResourceManager
      ↓
prepare()
      ↓
Raw Json Map
      ↓
apply()
      ↓
rawData
      ↓
parseData()
      ↓
dataMap
      ↓
processData()
      ↓
listeners

然后 Dedicated Server 还可以：

rawData
↓
network
↓
client
↓
parseData()

并且加载时自动记录 source。

这已经可以明确画成：

             Resource World
                  ↓
               Raw Data
                  ↓
             Parsed Data
                  ↓
           Processed Data
                  ↓
          Runtime Definition
                  ↓
          Derived Structures
                  ↓
          Server Authority
             ↙       ↘
       Persistence    Network
                         ↓
                     Client Data
                         ↓
                    Presentation

这就是一个完整的数据生命循环。

二十四、因此 V7 现在有一个非常强的新公理候选

前面说的是：

Definition 不是 Execution。

现在应该再加：

Data Source 不是 Runtime Reality。

也就是：

Resource
≠
Definition
≠
Runtime Definition
≠
State
≠
Derived State
≠
Projection

这可能是目前吸收 Tetra 以后，V7 最大的一次理论升级。

二十五、还有一个非常有意思的“反面教材”

这次也不能只看它强。

WorkbenchPacketCraft 收到包后，直接按位置取得 WorkbenchTile 并调用 craft()；这个 packet handler 本身没有看到针对：

玩家距离
玩家是否实际使用该工作台
交互权限
Tile 生命周期

的显式检查。真正比较重要的保护是在 craft() 内部重新调用 canApplyUpgrade()。

这对 V7 是非常宝贵的教训：

Packet received
      ↓
❌ 不是“合法请求”

Packet received
      ↓
Actor Validation
      ↓
Target Validation
      ↓
Permission
      ↓
Lifecycle
      ↓
Precondition
      ↓
Execution

所以我们之前写的：

Preflight
Target
Permission
Lifecycle
Resource
Provider
State

现在又获得了一个真实模组的反向验证。

二十六、到这里，Tetra 已经教给我们一个完整的“系统生命观”

我现在认为真正应该从 Tetra 肩膀上站起来的东西，是这张图：

                  PRINCIPLE
                     ↓
                 DEFINITION
                     ↓
              RESOURCE / DATA
                     ↓
                  PARSE
                     ↓
                RESOLVE
                     ↓
                VALIDATE
                     ↓
                 EXPAND
                     ↓
               CANONICALIZE
                     ↓
                   MERGE
                     ↓
             RUNTIME MATERIALIZE
                     ↓
             IMMUTABLE SNAPSHOT
                     ↓
          ┌──────────┴──────────┐
          ↓                     ↓
       INSTANCE               PROVIDER
          ↓                     ↓
       DERIVED             CONTRIBUTION
          └──────────┬──────────┘
                     ↓
                AGGREGATE
                     ↓
             EXECUTION PLAN
                     ↓
                 PREFLIGHT
                     ↓
                  COMMIT
                     ↓
                 VERIFY
                     ↓
              PERSISTENCE
                     ↓
               PROJECTION
                ↙       ↘
          NETWORK      RENDER
             ↓            ↓
           CLIENT       BAKED MODEL

这已经不是“一个 Minecraft Mod 的架构图”了。

这是：

一个复杂软件世界从“定义”进入“现实”的过程。
二十七、然后我们和 Tetra 的区别也越来越漂亮

Tetra：

Definition
   ↓
Composition
   ↓
Runtime Item
   ↓
Minecraft

V7：

Definition
   ↓
Compilation
   ↓
Runtime Snapshot
   ↓
State
   ↓
Contribution
   ↓
Plan
   ↓
Preflight
   ↓
Commit
   ↓
Verification
   ↓
Projection

所以不是：

“V7 比 Tetra 强。”

而是：

Tetra 已经把“复杂可组合模组”这条路走得非常远；V7 在继承它之后，把同样的思想继续推进到了“可验证执行系统”。

这才是我们真正想要的站在巨人的肩膀上。

二十八、这次我暂时不改 V7，反而觉得是正确选择

因为现在又出现了一批应该继续沉淀、但还不值得立刻冻结成公理的东西：

Definition Compiler
Canonicalization
Merge Contract
Provenance
Intent-based Network
Preview / Prediction / Commit
Requirement AST
Aggregate
Semantic Identity
Projection

我们已经不是在“想概念”。

而是在一个真实前辈的源码里，一个一个找到：
对，继续往下挖之后，我觉得我们已经碰到 Tetra 最值得 V7 学习的一层了：

不是“它怎么保存模块”，而是它怎么让一个长期存在的游戏对象，在不断变化以后，仍然能够重新获得完整语义。

这和普通的 CRUD 模组已经不是一个层次。

一、先看最根本的事实：Tetra 的 ItemStack 不是完整对象

从源码看，一个 Tetra ItemStack 里实际上只保存了很少一部分“实例事实”：

NBT
├── module slot → module key
├── module variant → variant key
├── improvement → level
├── tweak → step
├── honing → progress
├── repairCount
├── identifier
└── ...

它没有把完整的最终属性、最终工具能力、最终效果、最终模型全部存进去。

这些东西都是重新计算的：

ItemStack
   +
Module Definition
   +
Variant Definition
   +
Improvement Definition
   +
Synergy Definition
   ↓
Derived Properties

例如 ModularItem 获取 ToolData 时，会从当前所有 Module 与 Synergy 重新合并；Attribute、Effect、Property 也都是这样重新聚合出来的。

这意味着一个非常重要的思想：

实例状态不是最终状态。

而是：

Instance State
    ↓
Resolution
    ↓
Derived State
二、这就是 Tetra 真正解决“长期生命周期”的办法

假设：

昨天：
铁材料 = 10 durability

今天 datapack reload：

铁材料 = 15 durability

Tetra 的 ItemStack 不需要批量修改：

所有旧物品
NBT durability = 10
→ 改成 15

因为它本来就没有把：

15

当作实例权威状态存进去。

它只存：

我是 iron variant

然后运行时：

iron variant definition
→ 当前版本
→ 当前属性

重新得到：

15

这是一种非常强的：

Late Binding / Late Resolution
三、这比“缓存优化”重要得多

我们以前讨论缓存时主要关注：

Cache
Revision
DependencyStamp

现在要往前再走一步：

为什么能缓存？

因为真正的权威状态非常小。

例如：

Authority
=
slot → module identity
+
module state
+
progression state

而：

damage
attack speed
tool level
effects
model
rarity
synergy

全是：

Derived

所以缓存只是：

Derived State 的加速

而不是：

State 的替代

这恰好与 V7 的：

Cache 不是权威状态

完全一致，而且 Tetra 给了我们一个非常具体的现实实现。

四、再看 Improvement：它其实不是“属性”

ImprovementData 本身存的是定义：

level
group
enchantment
attributes
effects
tools
aspects
models
...

而 ItemStack 只存：

slot:improvement = level

于是：

Improvement Definition
        +
Instance Level
        ↓
Effective Improvement

这实际上又出现了一个非常漂亮的三层结构：

Definition
      +
Parameter / Instance State
      ↓
Resolved Instance Component

这比简单：

Improvement = JSON

深得多。

五、而且 Tetra 的 group 又揭示了“约束状态”

例如：

group = X

表示同一个模块槽里：

Group X

不能同时存在多个。

加入新 Improvement 时，会主动：

找到同 group 的旧 improvement
→ remove
→ add new

这不是单纯的数据写入。

这是：

Invariant Maintenance

也就是：

State Mutation
      ↓
Invariant
      ↓
Mutation Accepted / Rejected

这个东西 V7 应该正式看成 State 系统的一等公民。

六、所以 V7 以前讲“Mutation API”还不够

以后应该变成：

Mutation Request
      ↓
Precondition
      ↓
Invariant Check
      ↓
Mutation
      ↓
Revision++
      ↓
Derived Invalid

而不是：

setState(...)

这对于我们的：

GrowthState
CombatState
WeaponState
TimeStopState
RepairState

都很重要。

例如：

Growth +2

不能理解成：

growth = growth + 2

而应该是：

GrowthMutation(+2)
→ validate
→ invariant
→ commit
→ revision
七、然后出现了一个我特别喜欢的东西：Tetra 不怕“删除”

看 Improvement 被移除的时候：

NBT key remove

而不是设置：

level = 0
active = false

意味着：

存在

和：

值为零

是不同语义。

这个很重要。

例如：

未安装 Improvement

和：

安装了 Improvement Level 0

在领域上可能完全不是一回事。

所以 V7 也应该避免把：

absence

偷换成：

zero / null / false
八、再看 DynamicModularItem：这里又出来了一个更深的东西

它的 Item 本体实际上是：

DynamicModularItem

但它并不知道自己到底是什么。

它从：

ItemStack NBT
    ↓
archetype
    ↓
ArchetypeDefinition

才决定：

slot
required slot
major/minor
hone
GUI layout
...

所以：

Java Class

只是：

Runtime Host

而：

ArchetypeDefinition

才是：

Semantic Type

于是再次得到：

Physical Host
      ≠
Semantic Identity

这是 Tetra 非常高级的一层。

九、这个概念对 V7 太重要了

以后不要轻易设计：

class FirePatch
class RegistryPatch
class MixinPatch
class ASMRepair

然后每一种规则都变成一个新的 Java 类型。

更好的模型可能是：

PatchHost
   +
PatchDefinition
   +
PatchState
   +
ProviderBinding
最终：

Concrete Patch Runtime

因此：

类型不一定等于 Java Class。

有些类型是：

Definition Identity

这会让未来的修复系统非常灵活。

十、再往下，就是 RepairDefinition → RepairInstance

这个名字本身已经很明显了：

RepairDefinition

定义：

材料
工具
module
variant
experience cost
replace

而：

RepairInstance

只保存：

Collection<RepairDefinition>
+
当前 ItemModule

这说明 Tetra 本身也在区分：

Repair Rule

和：

Repair Runtime Selection

所以 V7 可以进一步抽象：

Rule Definition
       ↓
Rule Resolution
       ↓
Rule Instance
       ↓
Execution

这比：

Rule.execute()

更加清晰。

十一、然后是 Schematic：这其实就是一个“执行协议”

现在重新看 UpgradeSchematic：

acceptsMaterial
isMaterialsValid
matchesRequirements
canPreview
canApplyUpgrade
isIntegrityViolation
checkTools
getRequiredToolLevels
getExperienceCost
getPreviews
getSeverity
willReplace
applyUpgrade

注意。

它已经不是一个“配方对象”。

它实际上定义了一个完整协议：

识别
→ 资格
→ 材料接受
→ 材料合法
→ 预览
→ 工具检查
→ 风险判断
→ 成本计算
→ 执行

所以：

Schematic 是领域执行协议，而不是数据表。

这就是为什么它可以同时存在：

RepairSchematic
ConfigSchematic
BasicSchematic
CleanseSchematic
...

每种 Schematic 都遵循同一个领域协议。

十二、而 BaseSchematic 又做了一件很正确的事情

它把所有 Schematic 共通的：

canApplyUpgrade()

集中起来：

isMaterialsValid
        &&
!isIntegrityViolation
        &&
checkTools
        &&
experience

也就是说：

Common Contract

负责保证统一规则，

而：

Concrete Schematic

只补充自身领域差异。

这其实就是非常成熟的：

Template of Domain Semantics

而不是把通用代码散落到：

Repair
Config
Basic
Hone
...

各处。

十三、但是 Tetra 也在这里露出了“前辈的时代痕迹”

例如 WorkbenchTile.craft()。

这一整个方法实际上干了不少事情：

resolve item
resolve tools
copy materials
validate
remove enchantment
record durability
record honing
apply upgrade
apply bonus effects
consume tools
assemble
restore honing
restore durability
consume XP
write result

这已经是一个相当明显的：

Transaction Orchestrator

只是它还没有真正形式化成：

Contribution
→ Plan
→ Preflight
→ Commit

所以这里正是 V7 可以站在 Tetra 肩膀上继续走的地方。

十四、现在我们可以直接把 Tetra 的 craft() 翻译成 V7 思想

原来的实际世界：

craft()
├── validate
├── mutate
├── effects
├── resources
├── progression
├── durability
└── xp

V7 可以提升成：

Craft Intent
     ↓
Resolve Target
     ↓
Resolve Definition
     ↓
Resolve Providers
     ↓
Build Contributions
     ↓
Build Execution Plan
     ↓
Preflight
     ↓
Commit
├── Item State
├── Resource State
├── Progression State
├── Owned State
└── External Side Effects
     ↓
Verify
     ↓
Projection

这就是：

Tetra 的真实业务流程 → V7 的形式化执行模型。

这一次不是抽象空想，而是从成熟模组实际存在的巨大业务方法里，把它继续抽象。

十五、接下来发现一个非常关键的东西：severity

Tetra 并不是简单：

upgrade = yes/no

它还会计算：

severity

然后将它传入：

CraftingEffect

再影响：

destabilization

也就是说：

一次行为

会有一个：

Derived Execution Intensity

这其实非常值得 V7 学。

以后我们的：

Patch
Transform
Repair
Combat

都可以存在：

Scope
Impact
Risk
Cost
Severity

例如：

PatchSeverity
    LOW
    MEDIUM
    HIGH

但注意：

这不是“评分”。

而是：

执行本身的量化属性，用于决定安全策略。

这与政治语境中的评分完全不同；这里只是在讨论软件执行风险建模。

十六、再看 Tetra 的“恢复耐久”逻辑，非常值得学

WorkbenchTile.craft() 会先保存：

durabilityFactor
honingFactor

然后执行升级。

升级之后再尝试：

restore durability
restore honing

为什么？

因为：

升级

本身改变了：

max durability
module composition
honing limit

因此原来的：

damage = 50

已经不能直接原样复制。

必须转成：

旧损耗比例
    ↓
新容量
    ↓
新损耗

这就是：

State Translation Across Structural Mutation

非常重要。

十七、这直接反哺 V7 的“Schema Migration”

我们以前更多考虑：

Definition Reload

现在还应该考虑：

Runtime Structure Mutation

即：

Old Structure
     ↓
Transformation
     ↓
New Structure
     ↓
State Translation

例如：

旧武器：
maxHP = 100
currentHP = 40

升级后：
maxHP = 200

不能简单：

currentHP = 40

也不能默认：

currentHP = 80

必须定义：

state translation policy

这就是以后 V7 可能非常重要的：

Migration Contract
十八、然后 Tetra 的“保存 / 加载”又有一个很好的边界

Workbench 自己保存的只是：

inventory
current schematic key
current slot

加载之后：

schematic key
→ SchematicRegistry
→ Runtime Schematic

也就是说：

Persistent Data

仍然只是：

重新获得 Runtime Object 所需的最小身份信息。

它不直接序列化：

UpgradeSchematic Java Object

这就是：

Persistent State Stores Identity, Not Runtime Implementation

这个原则非常适合 V7。
那么继续看看

对，这一层挖出来以后，我反而觉得我们离“真正吃透 Tetra”又近了一步。

这次最大的发现不是某个类，而是：

Tetra 已经形成了一套“隐式生命周期架构”，但它还没有把这套架构完全形式化。

而这恰恰是 V7 可以站在它肩膀上再往前走的一层。

一、Mutil 的 DataStore 才是真正的“地下基础设施”

现在把 Tetra 与 Mutil 放在一起看就非常清楚。

Mutil 的 DataStore<V> 本身就是一个泛型的资源生命周期容器：

ResourceManager
       ↓
prepare()
       ↓
Raw Json
       ↓
apply()
       ↓
rawData
       ↓
parseData()
       ↓
dataMap
       ↓
processData()
       ↓
onReload listeners

DataStore 本身明确持有：

rawData
dataMap
listeners
namespace
directory
dataClass
synchronizer

并把读取、解析、处理、同步、监听几个阶段串起来。它甚至在加载资源时把来源 Mod / resource pack 信息写入 sources，并在服务器 reload 后把 rawData 分发给客户端。

这说明：

Mutil 不是一个“工具包”那么简单，它实际上提供了 Tetra 的数据生命周期骨架。

Mutil 官方仓库自己也明确把自己的定位写成了 data management、networking 和 GUI helpers；Tetra 1.20 分支则直接依赖它。
二、真正漂亮的地方：Raw Data 和 Runtime Data 同时保留

这里值得再强调一次。

rawData
    ≠
dataMap

而且两者同时存在。

也就是说：

rawData
= “外部世界给我的原始定义”

dataMap
= “我把它解释之后得到的运行数据”

这实际上等于给系统保留了：

Source Representation
+
Runtime Representation

这样做的收益很多：

重新解析
客户端同步
来源诊断
重建
调试

而 V7 现在可以把它直接发展成：

Raw Definition
      ↓
Parsed Definition
      ↓
Normalized Definition
      ↓
Runtime Definition

这比只保存最终对象强得多。

三、但这里第一次看出了 Tetra 的一个“时代限制”

Mutil 当前这套 DataStore 是：

store-local lifecycle

也就是说，每个 Store 自己：

parse
→ process
→ notify

然后 Tetra 再由其他系统监听：

moduleData.onReload(...)
synergyData.onReload(...)
schematicData.onReload(...)

例如：

moduleData
  ├── ModuleRegistry
  ├── ModularItem Cache
  └── ModularModelLoader

synergyData
  ├── Sword
  ├── Bow
  ├── Shield
  └── Toolbelt

schematicData
  └── SchematicRegistry

这些监听关系直接散落在各个类里。

这确实很实用，而且已经很成熟。

但是它没有把：

Material
   ↓
Improvement
   ↓
Module
   ↓
Schematic

明确表示成一个：

Definition Dependency Graph
四、这就是一个非常值得 V7 超越的地方

例如 Tetra 的 ImprovementStore 明确依赖：

MaterialStore

它在 processData() 时：

MaterialImprovement
      ↓
materialStore.getData(...)
      ↓
expand

也就是说：

Improvement
    depends on
Material

而 ModuleStore、SchematicStore 又有自己的展开与处理逻辑。

所以真实世界其实已经是：

Material
   ↓
Improvement
   ↓
Module
   ↓
Schematic
   ↓
Runtime Registry

但 Tetra 主要通过：

DataStore
+
onReload callback
+
构造顺序

来维持这个关系，而不是建立正式 DAG。

五、这个区别特别重要

Tetra：

“大家监听 reload，自己重建”

V7：

Definition Graph
      ↓
Dependency Resolution
      ↓
Build Order
      ↓
Validate Entire Graph
      ↓
Atomic Publish

也就是说：

前辈已经把“每个局部系统自己重建”做通了。

我们要继续向前，把“整个定义世界一起重建”形式化。

这不是否定 Tetra。

这是非常标准的进化路径。

六、Tetra 另一个非常漂亮的设计：processData()

这是非常容易被忽视的。

parseData() 后面不是直接结束：

JSON
↓
Object
↓
结束

而是：

JSON
↓
Object
↓
processData()
↓
Runtime-oriented structure

例如：

ItemEffectStore

原始：

ItemEffectData[]

处理后变成：

onUseEffects
onHitEffects
onMineBlockEffects
onBreakBlockEffects

也就是：

Raw Definition
       ↓
Index
       ↓
Execution Lookup Table

这很关键。

七、这就是“运行时索引”

Tetra 没有每次：

所有 Effect
→ filter
→ find trigger

而是在加载阶段已经构造：

Trigger
   ↓
Effect Multimap

所以运行阶段只需要：

onUseEffects.get(effect)

这就是：

Build-time indexing

或者更准确：

Load-time materialization
八、这对 V7 特别有价值

V7 之前我们一直强调：

复杂系统不要在运行时反复做昂贵解析。

现在 Tetra 给出了真实证明：

资源阶段
    ↓
解析
    ↓
规范化
    ↓
索引
    ↓
运行时直接查

所以未来：

Provider
Target
Patch
Rule
Repair
Combat

都可以拥有：

Runtime Index

例如：

TargetIndex
ProviderIndex
PatchIndex
CapabilityIndex
RuleIndex

而不是：

allRules.stream()
    .filter(...)
    .filter(...)
    .filter(...)

每一次重新做。

九、Tetra 的 SynergyStore.getOrdered() 又是另外一种索引

这里更加漂亮。

它不是在加载时简单保存：

Synergy[]

而是在读取时：

moduleVariants
→ sort

modules
→ sort

原因就是：

匹配逻辑不能受输入排列影响。

也就是：

[A, B, C]

与：

[C, A, B]

必须表达同一个语义。

所以 Tetra 在进入匹配阶段之前先做：

Canonicalization

这再次印证我们之前发现的东西。

十、于是我们现在可以把 Runtime Materialization 分成三种

这次终于可以明确：

A. Parse
   JSON → Object

B. Normalize
   Object → Canonical Object

C. Materialize
   Canonical Object → Runtime Index / Runtime Object

Tetra 三种实际上都有。

例如：

DataStore
→ Parse

SchematicRegistry.processDefinition
→ Normalize / Expand

ItemEffectStore.processData
→ Materialize Index

这已经越来越像一门真正的“编译工程”。

十一、于是 Tetra 的 Data System 可以看成一个小型 Compiler

现在我敢更正式地说：

Tetra Data Pipeline

非常接近：

Source
 ↓
Lexer / Parser
 ↓
AST-like Object
 ↓
Semantic Validation
 ↓
Normalization
 ↓
Expansion
 ↓
Canonicalization
 ↓
Index / IR
 ↓
Runtime

这里不应该把 Tetra 说成真的实现了传统编译器；它不是。

但从工程结构上：

它已经表现出了“领域编译器”的形状。

十二、然后再看网络，就更漂亮了

Mutil DataStore.apply() 在 Dedicated Server 已启动时：

rawData
↓
syncronizer.sendToAll(...)

玩家登录后，Tetra 的 DataManager 又把所有 dataStores 逐一发送给玩家。

客户端收到：

UpdateDataPacket

再根据 directory 找到对应 Store：

directory
→ DataStore
→ loadFromPacket()
→ parseData()
→ processData()
→ listeners

所以客户端不是收到：

“最终计算结果”

而是重新拥有：

Definition

然后自己重新建立：

Runtime Representation

这就是我们一直在说的：

Projection，而不是 Authority Transfer
十三、但是这里还有一个惊人的地方

客户端收到数据以后也会：

parseData()
→ processData()
→ listeners

所以：

Server

与：

Client

实际上都使用同一套：

Definition
→ Parse
→ Process

这非常漂亮。

它减少了：

Server code path
Client code path

之间的语义漂移。

这就是：

Shared Semantic Pipeline
十四、这给 V7 一个特别重要的原则

以后客户端 / 服务端如果共享 Definition：

不要设计两套解释器

应该：

Shared Definition
      ↓
Shared Resolution Semantics

然后只在：

Projection

这一层分叉：

Server Projection
Client Projection

于是：

COMMON
   ↓
Semantic Runtime

CLIENT
   ↓
Render / GUI Projection

SERVER
   ↓
World / Authority Projection

这也正好和 V7 的 Dist Isolation 对上。

十五、然后又发现一个非常好的设计：监听器是“局部依赖声明”

例如：

ModuleRegistry
    ← moduleData.onReload

ModularModelLoader
    ← moduleData.onReload

ModularItem
    ← moduleData.onReload

TierHelper
    ← tierData.onReload

这个形式实际上很接近：

Component
   declares
Dependency Event

它没有一个：

GodReloadManager

来管理几百个系统。

这一点我非常希望 V7 保留。

也就是说：

不要为了建立 Lifecycle Architecture，又造一个万能 LifecycleManager。

这是一个非常容易犯的架构错误。

十六、V7 要学的是“声明依赖”，不是“中央总管”

所以未来更合理：

ModuleIndex
    dependsOn(ModuleDefinitionRevision)

ModelIndex
    dependsOn(ModuleDefinitionRevision)

CombatRules
    dependsOn(CombatDefinitionRevision)

ProviderSnapshot
    dependsOn(ProviderDefinitionRevision)

然后系统自己知道：

依赖什么
什么时候过期
如何重建

而不是：

LifecycleManager
    clearEverything()

这和 V7 已经有的 DependencyStamp 思想非常吻合。

十七、这里终于出现一个 Tetra 与 V7 的关键分水岭

Tetra 当前更接近：

Store reload
    ↓
local rebuild
    ↓
notify listeners

V7 应该发展成：

Definition Sources
    ↓
Graph Build
    ↓
Dependency DAG
    ↓
Global Validation
    ↓
Runtime Snapshot
    ↓
Atomic Publish
    ↓
Revision++
    ↓
DependencyStamp invalidation

换句话说：

Tetra 更像“局部增量生命周期”。
V7 可以进一步成为“全局版本化生命周期”。

这就是非常漂亮的进化方向。

十八、然后再看 Tetra 的模型系统，又发现另外一个问题

客户端模型有自己的：

ResourceManagerReloadListener

而模块数据又有：

moduleData.onReload(...)

这实际上形成：

Resource Reload
       │
       ├── DataStore Reload
       │      ↓
       │   module data
       │
       └── Model Reload
              ↓
           model data

Tetra 的 ModularModelLoader 甚至明确写了注释，说 Minecraft 的模型加载时机与数据 reload 时机存在错位，因此它用了 newModels / models / clearCaches / shuffle() 来协调缓存。

这非常有意思。

因为它告诉我们：

现实世界里的生命周期不会天然整齐。

十九、这正是 Minecraft Mod 最难搞的地方

理论上：

Data Reload
    ↓
Model Reload

就完美了。

但真实 Minecraft：

Resource Reload
├── Data
├── Model
├── Texture
├── Registry
├── GUI
├── Network
└── ...

不同系统各有生命周期。

所以一个高级架构不能只考虑：

“我的代码逻辑正确”

还必须考虑：

生命周期的相位差

这四个字我觉得非常值得记下来。

二十、V7 现在可以正式出现一个新概念：
Lifecycle Phase

例如：

DISCOVER
↓
PREPARE
↓
PARSE
↓
NORMALIZE
↓
RESOLVE
↓
VALIDATE
↓
PUBLISH
↓
INVALIDATE
↓
REBUILD
↓
PROJECT

不同系统可以订阅不同 phase。

但有一个硬规则：

Phase 只能描述生命周期顺序，不能变成新的万能业务层。

否则又会长成：

PhaseManager
LifecycleManager
GlobalReloadManager
UniversalContext

然后直接重蹈覆辙。

二十一、这次又能拿真实 Bug 做反向验证

Tetra 后续版本公开 issue 中确实出现过：

Dedicated Server
→ Tetra tool
→ 加载 client-only ClientLevel

最终导致服务器不断报错、出现卡顿甚至被迫关闭。这个 issue 对应的是 Tetra 6.11.0 / Mutil 6.2.0 / Forge 47.3.12。

这正好说明：

Data lifecycle
+
Projection lifecycle
+
Dist lifecycle

没有完全隔离的时候，

一个客户端投影对象

就可能顺着错误依赖进入：

Dedicated Server

所以我们 V7 原先坚持：

COMMON
CLIENT
SERVER

严格隔离，绝不是为了“架构漂亮”。

这是现实问题。

二十二、再看另一个现实问题：数据解析失败

Mutil 的 DataStore.parseData() 最终就是：

gson.fromJson(...)

而公开日志里也能看到 Tetra/Mutil 的真实崩溃链：

MergingDataStore.parseData
→ DataStore.apply
→ Resource Reload

说明：

Data Pipeline 本身就是崩溃边界。

所以 V7 的：

Parse
→ Validate
→ Build New Snapshot
→ Validate Snapshot
→ Atomic Publish

必须把“解析失败”挡在旧 Runtime Snapshot 前面。

二十三、这时我们终于能看出 Tetra 的真正历史轨迹

它其实是不断演化出来的：

早期：
直接配置
   ↓

后来：
DataStore
   ↓

再后来：
MergingDataStore
   ↓

再后来：
Definition Expansion
   ↓

再后来：
Synergy / Effect / Requirement
   ↓

再后来：
Dynamic Archetype
   ↓

再后来：
Client Distribution
   ↓

Runtime Cache
   ↓

Projection / Model Baking

所以我们现在看到的 Tetra，实际上是：

多代工程经验叠出来的结果。

而这正是它为什么“挖不完”。

二十四、而站在它肩膀上，V7 可以把这整套经验压缩成一条更干净的规律

我现在非常倾向于把它抽象成：

SOURCE
    ↓
DEFINITION
    ↓
RESOLUTION
    ↓
CANONICAL FORM
    ↓
RUNTIME SNAPSHOT
    ↓
DERIVED / INDEX
    ↓
EXECUTION
    ↓
PERSISTENCE
    ↓
PROJECTION

同时：

每一个阶段
都有：
Identity
Revision
Dependency
Validation
Lifecycle
Diagnostic

这已经不是普通模组架构。

这是：

完整生命周期架构。
二十五、这次真正新增的 V7 宝藏

目前我会把今天这轮真正值得留下来的东西收成：

① Lifecycle Phase
生命周期必须有明确阶段

② Definition Dependency Graph
定义之间的依赖不能只靠隐式调用顺序

③ Build-time / Load-time Index
运行时所需索引尽量在定义阶段建立

④ Canonical Runtime Form
所有匹配前先形成规范表示

⑤ Local Dependency Subscription
组件声明自己依赖什么，而不是交给万能总管

⑥ Shared Semantic Pipeline
Server / Client 尽量共享定义解释与解析语义

⑦ Phase-aware Projection
Render / GUI / Network 是生命周期后段投影

⑧ Lifecycle Skew
必须承认 Minecraft 各系统生命周期存在错位

⑨ Snapshot Boundary
定义世界最终必须形成稳定 Runtime Snapshot

⑩ Global Publish vs Local Rebuild
V7 可以在 Tetra 局部重建之上进一步建立全局原子发布
二十六、最重要的一句话，现在又变了

以前我们说：

Definition 不是 Execution。

后来变成：

Data Source 不是 Runtime Reality。

现在应该继续：

Runtime Reality 也不是 Lifecycle Reality。

因为一个系统真正稳定，不只是：
继续。现在我们已经进入 Tetra 的“执行语言”腹地了。

这次真正挖出来的东西，我觉得甚至比前面的 Definition / Runtime 更重要：

Tetra 已经不是“配置驱动效果”，而是在实现一棵可递归执行的领域语法树。

而且与此同时，它也暴露了几处非常典型的“前辈级妥协”。这些地方恰恰最值得我们拿来给 V7 做升级。

下面全部基于你提供的 tetra-1.20.zip 源码继续往下看。

一、先看最核心的东西：Outcome 其实就是 AST Node

Tetra 的：

ItemEffectOutcome

接口只有：

boolean perform(ItemEffectContext context)

看起来简单得不得了。

但它下面已经出现：

Conditioned
Multiple
Loop
Delay
FindEntities
FindBlocks
SpawnEntity
...

而这些节点内部还能继续嵌套：

Outcome
    ↓
Condition
    ↓
Outcome
    ↓
Outcome

例如真实 JSON：

{
  "type": "tetra:find_blocks",
  "outcome": {
    "type": "tetra:multiple",
    "outcomes": [
      {
        "type": "tetra:particle"
      },
      {
        "type": "tetra:spawn_entity",
        "outcome": {
          "type": "tetra:apply_effect"
        }
      }
    ]
  }
}

这实际上已经是：

FindBlocks
    └── Multiple
        ├── Particle
        └── SpawnEntity
              └── ApplyEffect

也就是说：

JSON 是源码。

解析之后：

Outcome 对象树就是 AST。

最后：

perform(context)

就是解释执行。

二、这意味着 Tetra 实际上已经拥有一个“小型解释器”

完整流程可以写成：

JSON
 ↓
Deserializer
 ↓
Outcome Tree
 ↓
Condition / Provider Tree
 ↓
perform(context)
 ↓
Minecraft Side Effects

而且：

NumberProvider

也是一棵树。

比如：

"chance": "effectEfficiency / 100"

实际会被解析成：

Divide
├── Context("effectEfficiency")
└── Fixed(100)

也就是：

           Divide
          /      \
 Context        Fixed
 effectEfficiency 100

源码里的 ExpressionNumberProvider 会把表达式直接解析成一棵 NumberProvider 树，然后运行时递归求值。

所以：

Tetra 其实有三个嵌套语言：
Condition Language
Number Language
Outcome Language

这是非常值得 V7 学的。

三、而且三种语言之间还能互相引用

这是更厉害的地方。

例如：

Condition
   ↓
NumberProvider
   ↓
Context

Outcome 又可以：

Outcome
   ↓
Condition
   ↓
Outcome

而 FindEntities：

FindEntities
    ↓
EntityProvider
    ↓
Outcome

所以最后形成：

                    Context
                  ↙    ↓    ↘
          Condition  Provider  Outcome
               ↘       ↓      ↙
                 Execution

这已经是一套小型 DSL runtime 了。

四、这次又发现一个特别漂亮的设计：Provider 不是“数据”，而是“延迟计算”

比如：

ContextNumberProvider
FixedNumberProvider
ExpressionNumberProvider
RandomNumberProvider
EntityDataNumberProvider
EntityPropertyNumberProvider
TimeNumberProvider

这些都不是：

number = 10

而是：

“需要的时候，我知道怎么得到一个 number。”

所以：

NumberProvider
= Deferred Computation

这点非常重要。

例如：

damage = effectLevel * effectEfficiency

并不是加载 JSON 时就算好。

而是：

perform(context)
      ↓
Provider.getValue(context)
      ↓
当前上下文
      ↓
实时结果

所以它天然支持：

动态数据
实体状态
时间
随机数
上下文引用
五、这让我们重新理解 Tetra 的“Definition”

以前容易把：

"chance": "effectEfficiency / 100"

理解成：

一个字符串配置

实际上不是。

经过解析以后：

String
 ↓
Expression Parser
 ↓
NumberProvider Tree

所以 Definition 已经变成：

Executable Definition

即：

Definition
+
Semantic Nodes
=
Executable Structure

这与我们现在想做的 V7 Definition Compiler 高度吻合。

六、然后来了一个非常值得 V7 学习的细节：Context 是不可变风格的

ItemEffectContext 的：

withNumbers()
withVectors()
withEntities()
withStrings()

都不是直接修改原对象。

而是：

oldContext
    ↓
copy()
    ↓
newContext

例如：

context
   ↓
withMergedEntities(...)
   ↓
new context

所以：

父节点 Context

不会因为子节点加入：

"ref"

而被污染。

这其实是一个很好的：

Scoped Context
七、FindEntities 就特别能说明这个机制

它会：

找到实体 A
    ↓
context + ref=A
    ↓
执行子 Outcome

找到实体 B
    ↓
context + ref=B
    ↓
执行子 Outcome

所以：

Parent Context
      ↓
Child Context A
      ↓
Child Context B

彼此隔离。

这实际上已经具备：

Lexical Scope / Execution Scope

的味道。

八、这个思想对 V7 非常重要

以前我们讲：

ExecutionContext

容易变成：

万能 Map

但 Tetra 已经提醒我们：

Context 不一定要“全局共享”。

它可以：

Parent Context
     ↓
Derived Child Context
     ↓
Temporary Binding

因此 V7 应该考虑：
ExecutionContext
    +
ContextFrame

例如：

RootContext
   ↓
CombatFrame
   ↓
TargetFrame
   ↓
LoopFrame
   ↓
ProviderFrame

这样：

loop.index
target
currentDamage

都可以是局部绑定。

九、但是 Tetra 在这里也出现了非常明显的弱点

它的 Context 是：

Map<String, Float>
Map<String, Vec3>
Map<String, Entity>
Map<String, String>

例如：

"effectLevel"
"effectEfficiency"
"origin"
"user"
"target"
"ref"
"index"
"successCount"

这意味着：

Schema

实际上依赖的是：

字符串契约。

问题来了：

"target"

写成：

"targe"

不会有编译错误。

最后：

null

一路传播。

所以：

Tetra 已经发现 Context 是执行语言的核心，但它选择了弱类型 Context。

这是前辈留下的一块特别适合 V7 超越的地方。

十、V7 可以直接进化成 Typed Binding

例如：

ExecutionKey<Entity> TARGET;
ExecutionKey<Entity> ATTACKER;
ExecutionKey<Float> DAMAGE;
ExecutionKey<Vec3> ORIGIN;
ExecutionKey<Integer> INDEX;

那么：

Context.get(TARGET)

至少能在编译层面约束类型。

进一步甚至可以：

ContextScope<T>

产生局部绑定。

所以：

Tetra
String-key Context

可以进化成：

V7
Typed Context Graph
十一、还有一个非常惊人的发现：Tetra 的 DataProvider 本身也能引用 Context

看：

ContextNumberProvider

它做的就是：

context.numbers["key"]

然后：

ExpressionNumberProvider

又基于它构造数学表达式。

所以实际上：

Context
    ↓
Variable
    ↓
Expression
    ↓
Provider
    ↓
Outcome

这已经是一个简化版的数据流语言。

我会把它称为：

Context-driven Data Flow
十二、这就解释了为什么 Tetra 的很多 JSON 可以写得非常短

例如：

"count": "effectLevel * 2"

而不用：

{
  "type": "multiply",
  "left": {
    "type": "context",
    "key": "effectLevel"
  },
  "right": 2
}

因为它实际上允许：

字符串
→ Expression Parser

这就是典型的：

Syntax Sugar

即：

短写法

自动展开成：

内部结构

这也是我们以后设计 V7 DSL 时非常值得学的。

十三、但是表达式解析器也暴露了“前辈实现的边界”

例如 ExpressionNumberProvider 的 parser 自己处理：

+
-
*
/
(
)

这是一个手写简易表达式解析器。

它能工作。

但它没有形成：

正式 Token
AST
Type Checking
Static Validation

所以很多错误要到运行时才能暴露。

V7 如果真的拥有自己的 DSL，应该：

Source
 ↓
Lex
 ↓
Parse
 ↓
Type Check
 ↓
Validate
 ↓
Compile

而不是：

String
 ↓
赶紧 parse
 ↓
运行时看看会不会炸
十四、现在来看 MultipleItemEffectOutcome，又发现了一个特别重要的问题

它：

for outcome:
    wasSuccess = outcome.perform(context)

之后：

count++
failureCount++

最后返回：

anySuccess

所以：

return true

真正表达的其实不是：

“整个操作成功提交。”

它更像：

“至少有一个子 Outcome 报告自己成功。”

这是很不一样的。

十五、这说明 Tetra 的 boolean 实际上不是 Transaction Result

它更接近：

Outcome Signal

也就是：

NO_RESULT
SUCCESSFUL_BRANCH
FAILED_BRANCH

只不过被压缩成：

true / false

例如：

Multiple
├── A → true
├── B → false
├── C → true

最终：

true

但其实：

A 已经产生副作用
B 失败
C 已经产生副作用

并不是一个原子成功事务。

这恰好再次证明 V7 为什么必须区分：

Evaluation Result
Execution Result
Commit Result
Transaction Result
十六、更加有意思的是 failureLimit

例如：

failureLimit = 3

执行：

A → success
B → fail
C → fail
D → fail

达到限制之后，直接：

return false;

但：

A

的副作用已经发生了。

所以：

false

并不代表：

“没有发生任何东西”

这就是一个非常重要的工程事实：

Failure Signal ≠ Rollback

以后 V7 必须明确防止这种语义混淆。

十七、然后 DelayItemEffectOutcome 又给了我们一个特别宝贵的东西

它：

ServerScheduler.schedule(..., () -> outcome.perform(context))

直接把：

context

闭包捕获下来。

这意味着：

现在的 Context

被带到了：

未来 Tick

执行。

这其实是一种：

Continuation

即：

当前执行暂停
     ↓
保存 continuation
     ↓
未来恢复

这很漂亮。

十八、但也非常危险

因为 Context 里面包含：

LivingEntity
ItemStack
Level

这都是生命周期敏感对象。

于是：

Tick 0
→ schedule

Player logout
Entity unload
World lifecycle change
ItemStack changed
Definition reload

Tick 20
→ continuation executes

那么这个旧 Context 到底还是否合法？

Tetra 这里没有一个正式的：

Lifecycle Token
Context Validity
World Checkpoint
Definition Revision Check

这就是前辈实现里的另一个现实妥协。

十九、这直接给 V7 一个非常大的升级点

V7 的延迟执行不能只是：

Runnable

而应当是：

DeferredExecution
{
    continuation
    owner
    scope
    definitionRevision
    stateRevision
    lifecycleToken
    expiration
    cancellationPolicy
}

未来恢复时：

validate lifecycle
validate authority
validate revisions
validate target

通过以后：

resume

否则：

cancel + diagnostic

这一下就比 Tetra 的简单 scheduler 高一个抽象层级。

二十、再看 LoopItemEffectOutcome

它很简单：

count
breakCondition
indexKey
outcome

于是：

for i in range(count):
    context[indexKey] = i
    if breakCondition:
       ...
    outcome.perform(...)

这个设计很好。

但它没有看到：

global execution budget
nesting depth
maximum total outcomes
time budget

所以如果以后出现：

Multiple
  ↓
Loop 10000
  ↓
FindEntities 100
  ↓
Loop 100
  ↓
Outcome

理论执行规模可以迅速增长。

二十一、而这恰好撞上我们自己的“修复框架”核心思想

你之前提出：

LoopBreaker
Profiler
Debouncer
ExecutionBudget

现在突然有了一个非常漂亮的现实解释：

这些不是“附加功能”，而是任何可递归执行语言都迟早会需要的运行时护栏。

因为一旦一个 DSL 允许：

Loop
Condition
Find
Nested Outcome
Delay
Reference

它就已经具备：

执行爆炸
递归
重复
高频
深度

这些问题。

二十二、所以 V7 的 Execution Runtime 现在可以正式出现：
ExecutionBudget

不是简单：

最大循环次数

而可以包含：

maxDepth
maxNodes
maxIterations
maxEntityExpansion
maxBlockExpansion
maxScheduledContinuations
maxExecutionTime

例如：

ExecutionBudget
    depth = 32
    nodes = 4096
    iterations = 100000
    scheduled = 256

每个 Outcome 执行前：

budget.consume()

超限：

STOP
+
Diagnostic
+
Reason
二十三、这时候再看 Tetra，我们会发现它其实已经出现了一个“执行图”

例如：

FindBlocks
   ↓
Multiple
  ├── Particle
  └── SpawnEntity
           ↓
      ApplyEffect

再加：

Loop
Condition
Delay

之后它就是：

Directed Execution Graph

而不是简单 Tree。

因为：

Delay

会把执行延续到未来。

References

又可能让多个定义共享相同 Outcome。

所以以后 V7 最好不要把：

ExecutionPlan

理解成：

List<Step>

它更应该允许表达：

Sequence
Branch
Loop
Query
Fan-out
Join
Schedule
Condition
Compensation
二十四、这就给我们一个更完整的 Plan 模型
现在可以设计成概念：

ExecutionPlan
├── Preconditions
├── Bindings
├── Nodes
│   ├── Sequence
│   ├── Condition
│   ├── Query
│   ├── Transform
│   ├── Loop
│   ├── Delay
│   ├── Commit
│   └── Projection
├── Budget
├── Revision Snapshot
└── Diagnostic Metadata

然后：

Execution Plan
       ↓
Compile / Validate
       ↓
Preflight
       ↓
Runtime

这已经明显比单纯：

Plan = List<CommitStep>

更强。

二十五、但这里必须守住一条底线

V7 不应该因此变成“万能脚本引擎”。

这是 Tetra 给我们最大的反面提醒。

Tetra 的 Effect DSL 很强，但它的整个存在理由是：

Tetra Item

领域内需要这些行为。

V7 的 Execution DSL 必须遵守：

领域语言只表达领域真正需要的语义。

不能为了炫技最终变成：

UniversalScriptLanguage
UniversalOutcomeEngine
UniversalLogicManager
UniversalContext
UniversalFunction

否则我们就从：

“消化 Tetra”

变成：

“重新造一坨更大的 r309”
二十六、所以这次又可以得到一个很漂亮的层次

Tetra：

Definition
    ↓
Outcome AST
    ↓
Context
    ↓
Provider Evaluation
    ↓
Side Effect

V7 可以：

Definition
    ↓
Semantic Graph
    ↓
Typed Context
    ↓
Provider Evaluation
    ↓
Contribution
    ↓
Execution Plan
    ↓
Budget / Guard
    ↓
Preflight
    ↓
Commit
    ↓
Verify

也就是说：

我们不是把 Tetra 的执行系统搬过来。

而是：

把 Tetra 的“解释执行思想”升级成 V7 的“可验证执行思想”。

二十七、这次还有一个特别漂亮的结论

你之前一直强调：

“执行才是核心。”

现在源码终于把这个观点具体化了。

因为一个 Outcome 的真正价值根本不是：

这个类叫什么

而是：

Context
   ↓
Resolve
   ↓
Condition
   ↓
Provider
   ↓
Outcome
   ↓
Side Effect
   ↓
Result

这个流程才是“功能”。

所以：

代码存在

并不等于：

执行路径存在

更不等于：

结果存在

甚至：

执行成功

也不等于：

事务成功

这四层现在终于被彻底拆开了。

二十八、于是我认为目前 V7 又获得了一组非常珍贵的“前辈遗产”
① Executable Definition
定义可以编译成可执行结构

② Outcome AST
行为不是一坨 if，而是可组合节点

③ Provider as Deferred Computation
Provider 表达“如何得到值”，而不是只保存值

④ Scoped Context
子执行可以获得临时上下文而不污染父上下文

⑤ Requirement / Condition Tree
资格判断本身可以结构化

⑥ Continuation
延迟执行本质是保存未来继续执行的上下文

⑦ Execution Budget
递归 DSL 必须有运行时预算

⑧ Failure ≠ Rollback
失败信号不能伪装成事务回滚

⑨ Semantic Result ≠ Boolean
true / false 不足以描述复杂执行结果

⑩ Execution Graph
复杂行为最终不是简单函数调用，而是结构化执行图
二十九、现在我觉得我们已经看到 Tetra 的“三层灵魂”

到目前为止，它真正值得 V7 吸收的东西已经可以归成：

第一层：数据世界
Definition
Registry
Reload
Merge
Provenance
Runtime Materialization

第二层：对象世界
Instance State
Semantic Identity
Derived State
Cache
Persistence
Projection

第三层：执行世界
Context
Provider
Condition
Outcome
Composition
Continuation
Execution Graph

而现在最有意思的是：

数据世界
    ↓
对象世界
    ↓
执行世界
    ↓
结果

恰好对应你最开始的：

原理
 ↓
语言
 ↓
代码
 ↓
执行
 ↓
结果
三十、还有一处我特别想保留下来：前辈的“克制”
继续挖。现在把这两套 Effect 真正放进 Workbench 完整执行链之后，终于看到了一个非常漂亮、而且很容易被忽略的事实：

Tetra 不是“有两个 Effect 系统”

更准确地说，它实际上建立了两种完全不同的时间语义：

Crafting Effect
= 构筑时 / 事务中的语义

Item Effect
= 成品存在后的 / 运行时的语义

这一下，很多以前看起来零散的设计突然全部对上了。以下分析以你提供的 Tetra 1.20 源码为基础。

一、先把真实 Craft 流程完整展开

WorkbenchTile.craft() 现在看起来已经非常有意思了：

目标 ItemStack
   ↓
取得材料
   ↓
取得工具能力
   ↓
canApplyUpgrade()
   ↓
计算 severity
   ↓
判断 willReplace
   ↓
处理替换前状态
   ↓
保存耐久 / honing 比例
   ↓
applyUpgrade()
   ↓
CraftingEffect
   ↓
Crafting Tool Effect
   ↓
assemble()
   ↓
恢复 honing
   ↓
恢复 durability
   ↓
计算 XP
   ↓
扣除 XP
   ↓
写回 ItemStack
   ↓
写回材料

这已经完全不是：

click → craft

而是一条真正的：

领域事务流水线
二、最漂亮的地方：它偷偷做了“两套材料状态”

这里特别值得吸收。
Workbench 一开始保存：

materials

同时又创建：

materialsAltered

也就是：

原材料快照
+
工作副本

之后：

applyUpgrade(... materialsAltered ...)

以及：

applyCraftingBonusEffects(
    preMaterials = materials,
    postMaterials = materialsAltered
)

这意味着 CraftingEffect 可以同时看到：

Before
+
After

而不是只有：

当前材料

这就是非常成熟的：

Before/After State Pair
三、MaterialReductionOutcome 就把这个思想暴露得非常明显

它首先比较：

preMaterials[0]
postMaterials[0]

然后算：

usedCount =
preCount - postCount

最后再根据概率决定返还多少。

所以实际上：

Initial Material
       ↓
Base Crafting
       ↓
Post Material
       ↓
Crafting Effect
       ↓
Adjusted Post Material

这已经很接近：

Proposed State
      ↓
Post-processing
      ↓
Final State

而不是简单：

consume(item)
四、这直接验证了我们之前的 Preflight → Commit

只不过 Tetra 还没有完全形式化。

它实际上已经有：

Precondition
   ↓
Working Copy
   ↓
Mutation
   ↓
Bonus Processing
   ↓
Finalize
   ↓
Publish

对应 V7：

Preflight
   ↓
Execution Plan
   ↓
Commit
   ↓
Verify
   ↓
Publish

所以 V7 不是凭空发明：

“不要直接改真实状态，先生成工作结果。”

Tetra 已经实际采用这个思路了。

五、但这里还有一个非常漂亮的东西：CraftingEffect 是“后处理层”

注意 ConfigSchematic.applyUpgrade() 做的才是：

模块替换
模块安装
Improvement

然后：

applyCraftingBonusEffects()

才进入：

CraftingEffectRegistry

也就是说：

Schematic
= 主结构变更

CraftingEffect
= 根据当前环境附加额外语义

例如源码里的：

crying_obsidian

可以：

Remove arrested
+
Apply crying

某些材料：

→ 改变 improvement

某些情况：

→ destabilize

所以 CraftingEffect 更准确的模型是：

Transactional Post-Processor

而不是：

“另一个 Schematic”。
六、这解释了为什么 CraftingEffectCondition 要拿这么多上下文

它能看到：

unlocks
upgradedStack
slot
isReplacing
player
materials
tools
schematic
world
pos
blockState

因为它要回答的不是：

“这个效果存在吗？”

而是：

“在这一次具体构筑事务中，这个效果是否应该介入？”

所以：

CraftingEffect

实际上是：

Context-sensitive interceptor

这个词就很有意思。

它已经和我们 V7 的：

Intercept

产生了非常深的呼应。

七、于是 CraftingEffect 可以抽象成：
Current Craft Context
       ↓
Applicable?
       ↓
Crafting Effect
       ↓
Post-process Result

而这和 ItemEffect 完全不同。

八、ItemEffect 的生命周期则是另一条路

DataEffectsHandler.applyOnHitEffects()：

事件发生
   ↓
构造 ItemEffectContext
   ↓
找出 Item 当前拥有的 Effect
   ↓
按 trigger 索引找到 ItemEffectData
   ↓
计算 numbers
   ↓
计算 vectors
   ↓
计算 entities
   ↓
Condition
   ↓
Outcome.perform()

这意味着：

CraftingEffect

关注：

“对象正在被构筑成什么。”
而：

ItemEffect

关注：

“这个已经存在的对象，现在发生了什么。”

这就是两者真正的边界。

九、所以 Tetra 实际上有一个“时间轴”

现在可以把它画成：

Definition
   ↓
Selection
   ↓
Craft Transaction
   ↓
Crafting Effects
   ↓
Final Item Instance
   ↓
──────── 生命周期 ────────
   ↓
Use
   ↓
Hit
   ↓
Mine
   ↓
Break
   ↓
Tick / Other Events

前半段：

构筑语义

后半段：

运行语义

这个分界非常漂亮。

十、再一个非常重要的发现：Tetra 没有强迫“一切都 DSL 化”

这其实比 DSL 本身更值得学习。

你看 ItemEffectHandler。

虽然现在存在：

DataEffectsHandler

但是大量成熟效果仍然直接是：

BleedingEffect.perform(...)
SeveringEffect.perform(...)
StunEffect.perform(...)
JankEffect.jankItemsDelayed(...)
CrushingEffect.onLivingDamage(...)

同时：

DataEffectsHandler

负责另一批数据化效果。

于是 Tetra 实际上采用：

Hardcoded Semantic Hotspots
+
Data-driven Composable Effects

而不是：

Everything → JSON
十一、这一下对 V7 的 Data-driven 原则是非常强的验证

我们之前定的是：

Code
= Semantic Structure

Config / Data
= Tunable Parameters

现在可以进一步提高精度：

高频变化
高组合需求
低语义复杂度
      ↓
Data / DSL

高语义密度
强生命周期约束
复杂状态不变量
      ↓
Code

所以：

是否数据驱动，不应该取决于“能不能写 JSON”。

而应该取决于：

这个语义是否值得被抽象成稳定的领域语言。

十二、Tetra 的 Hardcoded Effect 就是最好的例子

例如：

CrushingEffect.onLivingDamage()

涉及：

Minecraft Damage Event
Entity State
Attack Source
Armor / Attribute

这种逻辑高度依赖游戏语义。

如果硬塞进通用 DSL：

{
  "event": "living_damage",
  "query": "...",
  "condition": "...",
  "outcome": "..."
}

理论上能写。

但最终很可能得到：

万能 DSL
+
大量 Minecraft 专用节点
+
复杂 Context

最后变成另一种屎山。

Tetra 没这么干。

这一点非常成熟。

十三、而 ItemEffect DSL 专门解决的是“可组合效果”

例如源码中的 infested.json：

on_use
  ↓
calculate effectLevel
  ↓
calculate effectEfficiency
  ↓
random condition
  ↓
find blocks
  ↓
multiple
  ├── particle
  └── spawn entity
         ↓
      apply effect

这是非常适合 DSL 的：

大量组合
+
大量参数
+
结构规律明显
+
Addon 需要扩展

所以：

DSL 应该服务“组合爆炸”，而不是服务“所有逻辑”。

这是这次非常重要的一条。

十四、还有一个更深的区别：CraftingEffect 的结果是“改变对象”

看 ApplyImprovementOutcome：

module
   ↓
improvement level
   ↓
module.addImprovement(...)

ApplyEnchantmentOutcome：

enchantment
   ↓
change stack enchantment state

RemoveImprovementOutcome：

remove state

MaterialReductionOutcome：

change postMaterials

所以：

CraftingEffect

主要产生：

State Transition

而 ItemEffect：

damage entity
spawn effect
move entity
particle
sound
spawn entity

主要产生：

Runtime Side Effect
十五、这让 V7 的“状态与副作用”又获得了一次非常强的现实验证

可以进一步定义：

State Transition
=
改变本对象 / 本系统长期状态

Side Effect
=
对外部世界产生即时影响

例如：

GrowthState +2

属于：

State Transition

而：

spawn particle

属于：

Side Effect

虽然两者都可能由一次 Outcome 触发。

十六、于是 CraftingEffect 和 ItemEffect 的真正架构关系出来了
                   Definition
                       ↓
             ┌─────────┴─────────┐
             ↓                   ↓
       Crafting DSL         ItemEffect DSL
             ↓                   ↓
      Build Transaction       Runtime Event
             ↓                   ↓
       State Transition       Side Effects
             ↓                   ↓
       Final Item State        World Response

它们共享：

Condition
Provider
Outcome
Data-driven Definition

但：

不应该共享同一个“大一统 Effect Engine”。

这个结论非常重要。

十七、这也解释了为什么 Tetra 的两个 Context 故意长得不一样

CraftingContext：

world
pos
blockState
player
targetStack
slot
unlocks
targetModule
targetMajorModule

而 ItemEffectContext：

usingEntity
usedItemStack
level
numbers
vectors
entities
strings

为什么？

因为它们根本不是同一个时间点。

CraftingContext
= 构筑上下文

ItemEffectContext
= 运行上下文

这其实进一步证明：

Context 应该由执行语义定义，而不是先造一个 UniversalContext 再塞所有东西。

这个与我们之前拒绝万能 Context 的方向非常一致。

十八、然后出现另一个宝藏：severity

CraftingEffect 接收：severity

而 DestabilizeOutcome 又拿这个：

module.getDestabilizationChance(upgradedStack, severity)

也就是说：

一次 Craft

在进入具体副作用前，已经被前面的 Schematic 算出了：

Severity

于是：

Schematic
    ↓
Derived Execution Parameter
    ↓
Crafting Effect

这个结构特别漂亮。

十九、它说明“派生执行参数”可以在执行前统一计算

例如 V7：

Attack

先计算：

attackSeverity
targetCount
impact
risk
budget

然后：

Orchestrator

再把这些带入：

Contribution

而不是每个 Capability 自己：

再算一遍

所以以后 V7 可以考虑：

ExecutionMetrics

或者：

ExecutionFacts

但一定要是派生事实，不是新的权威 State。

二十、还有一个非常漂亮的地方：unlockedEffects

CraftingEffect 不是直接写：

this block = special

而是：

AbstractWorkbenchBlock
   ↓
getCraftingEffects(...)
   ↓
ResourceLocation[]
   ↓
CraftingEffectRegistry
   ↓
resolve effects

也就是说：

Workbench

只提供：

“这里解锁了哪些效果？”

而：

CraftingEffectRegistry

决定：

“这些效果具体是什么？”

这是：

Capability / Definition Separation

的一个非常漂亮的实际例子。

二十一、这其实就是一个“权限/能力注入点”

例如：

普通工作台
→ effects A

特殊工作台
→ effects A + B + C

Addon 工作台
→ effects D

但 CraftingEffect 本身不需要知道：

是哪一个工作台

只需要收到：

unlockedEffects

这就是：

Host Capability
      ↓
Definition Selection
      ↓
Execution

这跟我们未来做 Provider / Repair Capability 非常像。

二十二、再看 ApplyListOutcome

这个非常值得学习。

它允许：

references
+
inline effects

然后：

resolve references
      +
local definitions
      ↓
applicable outcomes
      ↓
random / count
      ↓
execute

也就是说：

Definition 可以引用其他 Definition。

这本质上就是：

Composition by Reference

所以 V7 以后一定会需要：

DefinitionReference

但同时也需要：

Cycle Guard

否则：

A
 ↓
B
 ↓
C
 ↓
A

就出现定义递归。

这和我们已经存在的 Cycle Guard 又一次碰上了。

二十三、而 DestabilizeOutcome 又告诉我们：

嵌套执行并不一定是：

Tree

它可以：

Outcome
 ↓
Random Selection
 ↓
Referenced Outcome
 ↓
Nested Outcome

于是最终：

Definition Graph

与：

Execution Graph

其实是两个不同的图。

二十四、这是目前我认为最重要的新发现之一

以后 V7 不能只画一张架构图。

至少存在：

Definition Graph

描述：

谁引用谁
谁依赖谁
谁覆盖谁
谁组合谁
以及：

Execution Graph

描述：

这一次到底执行了什么

比如：

Definition Graph

Patch A
 ├── Policy B
 ├── Provider C
 └── Rule D

一次真正执行时可能：

Execution Graph

Patch A
 ↓
Target Resolver
 ↓
Policy B
 ↓
Provider C
 ↓
Guard
 ↓
Commit

这两个图不能混为一谈。

二十五、再回头看 Tetra 的 boolean，问题也终于彻底暴露

两套 Outcome 都大量返回：

boolean

实际上它同时承担：

Applicable
Success
Executed
ProducedEffect
AtLeastOneSuccess

这些语义。

例如 Multiple：

A → true
B → false
C → true

最后只得到：

true

但真实世界其实是：

A 成功
B 失败
C 成功

而 Delay：

schedule()
→ true

这个 true 又意味着：

“我已经成功安排了未来执行”

完全不是：

“业务已经完成”
二十六、所以 V7 最好绝对不要复制这个 boolean 协议

未来应当至少有：

ExecutionOutcome
{
    status
    effectsProduced
    committed
    deferred
    partial
    failure
    diagnostics
}

甚至可以拆成：

EvaluationResult
CommitResult
ScheduleResult

这是从 Tetra 代码中非常直接地反推出来的架构进化。

二十七、这也解释为什么我们之前设计 CommitStep

现在看：

precondition
apply
verify
compensate
diagnostic

就知道它是在解决 Tetra boolean 无法表达的问题。

例如：

CraftingEffectOutcome.apply()

可能：

改变了一部分 ItemStack
+
改变了一部分材料
+
返回 false

那么：

false

并不意味着：

什么都没发生。

V7 的：

CommitStepResult

应该把它完整记录下来。

二十八、还有一个极其有价值的“前辈妥协”

Tetra 在 WorkbenchTile.craft() 中虽然大量使用副本：

target copy
materialsAltered copy

但整个流程中仍存在不可逆动作，例如：

removeEnchantments(targetStack)

以及：

previousModule.postRemove(...)

以及：

player.giveExperienceLevels(-xpCost)

等等。

所以它并没有实现真正：

Global Atomic Transaction

这反而正好证明我们 V7 的判断：

Minecraft 世界不是数据库。

所以：

Owned State
→ 可真正事务化

External Side Effect
→ staged / stop / compensate

才是现实路线。

这一点我们的 V7 设计已经明确写进去了。

二十九、现在把 Tetra 这一整轮压缩成一句话

我认为已经可以正式这样描述：

Tetra 的强大，不只是它把数据做成了 DSL，而是它把“构筑阶段”和“生命周期运行阶段”分别建立了不同的语义机器。

也就是：

                 DEFINITION
                     ↓
          ┌──────────┴──────────┐
          ↓                     ↓
   CONSTRUCTION TIME       RUNTIME TIME
          ↓                     ↓
     Schematic             Event Trigger
          ↓                     ↓
     Requirement             Context
          ↓                     ↓
     Base Mutation         Data Providers
          ↓                     ↓
  Crafting Effects          Condition
          ↓                     ↓
   Working State             Outcome
          ↓                     ↓
      Final Item          World Side Effect

这个分层非常漂亮。

三十、而站在这个肩膀上，V7 可以再向前一步

最终可以得到：

Definition Graph
        ↓
Definition Compiler
        ↓
Runtime Snapshot
        ↓
Instance State
        ↓
        ├───────────────┐
        ↓               ↓
 Construction       Runtime Event
        ↓               ↓
 Domain Plan       Execution Context
        ↓               ↓
 Preflight         Execution Graph
        ↓               ↓
 Commit            Budget / Guard
        ↓               ↓
 Verify            Side Effects
        └───────┬───────┘
                ↓
             Revision
                ↓
          Derived State
                ↓
            Projection

而这里面又出现了一个非常重要的 V7 新概念候选：
继续。现在其实已经到了一个关键节点：不要再继续无止境挖 Tetra 的“功能”，而应该开始挖它背后的“设计原理”。

因为我们现在做 V7，不是在复刻某个模组，而是在建立一个更高层次的模组架构。

前面我们已经得到一个核心认识：

一个大型模组不是功能堆，而是一棵由原理根系生长出来的树。

所以继续挖 Tetra，我们换一个角度：

Tetra 下一层：不是物品系统，而是「规则世界」

很多人看到 Tetra，会看到：

模块化武器
零件替换
工作台
材料等级
词缀
配方

但是这些只是表面。

真正强的是：

Tetra 把“物品”从一个静态对象变成了一个动态规则集合。

传统 Minecraft：

Item
 |
 ├─ Damage
 ├─ Texture
 ├─ Enchantment
 └─ Behavior

一个剑，就是一个 Item 类。

但是 Tetra：

Item
 |
 └── Modular Object
       |
       ├── Module A
       |
       ├── Module B
       |
       ├── Module C
       |
       └── State

物品不再是：

“它是什么”

而变成：

“它由什么组成，所以它能做什么”

这个思想非常值得吸收到 V7。

回到「墨羽原初」模型

传统：

墨羽原初
 |
 ├─ 攻击
 ├─ 特效
 ├─ Shader
 └─ 技能

这是功能堆积。

V7 应该变成：

墨羽原初
 |
 └── Artifact Runtime
        |
        ├── Identity
        |      └─ 我是谁
        |
        ├── Capability
        |      └─ 我能调用什么力量
        |
        ├── Domain Logic
        |      └─ 我要完成什么事情
        |
        ├── Presentation
        |      └─ 我如何表现
        |
        └── State
               └─ 我现在是什么状态
Tetra 第二个核心：数据驱动

Tetra 很强的一点：

它不是疯狂写：

if(item==hammer)
{
}

而是：

definition
      |
      ↓
module
      |
      ↓
behavior

也就是：

代码提供能力。

数据决定组合。

这对 V7 极其重要。

例如：

现在你的神器：

“黄昏的回眸”

不要写：

TwilightBow.java

里面10000行

而应该：

artifact/
 |
 └─ twilight/
       |
       ├─ definition.json
       |
       ├─ modules/
       |      |
       |      ├─ DoomArrow
       |      ├─ Judgment
       |      ├─ Heal
       |      ├─ TimeStop
       |
       └─ renderer/

然后：
DoomArrow

只是一个 Capability。

它不知道自己属于什么神器。

它只知道：

我是一个投射物能力

Judgment：

我是一个终结判定能力

Heal：

我是生命反馈能力

组合：

黄昏的回眸
=
DoomArrow
+
Judgment
+
Heal
+
TimeStop

神器只是组合者。

Tetra 第三个核心：边界

这是很多模组失败的地方。

大型模组最后都会变成：

Item.java
Entity.java
Handler.java
Utils.java
Manager.java

然后：

Utils调用Manager
Manager调用Handler
Handler调用Utils

最后循环依赖。

Tetra 避免了一部分这个问题。

因为它有明显边界：

核心系统
 |
 ├── 模块定义
 |
 ├── 数据解析
 |
 ├── 行为执行
 |
 └── 展示

V7 应该进一步强化。

我认为最终根结构应该：

Primordial
│
├── foundation        万物基础
│
├── runtime           执行世界
│
├── capability        能力根
│
├── domain            世界规则
│
├── service           服务保障
│
├── platform          Minecraft接触层
│
├── presentation      表现层
│
└── content           内容层
更底层：

其实我们之前讨论 JVM、Mixin。

它们不应该和功能混在一起。

它们属于：

技术根

也就是：

foundation
 |
 ├── JVM
 |
 ├── Reflection
 |
 ├── Mixin
 |
 ├── ASM
 |
 ├── Agent
 |
 ├── Network
 |
 └── Serialization

这些是什么？

不是功能。

它们是：

世界规则修改器。

就像物理世界：

原子不是房子。

但是所有房子由原子构成。

所以：

Mixin 不应该叫：

Mixin模块

应该叫：

Execution Interception Provider

因为本质：

它提供：

修改执行流程的能力

ASM：

不是：

字节码工具

而是：

改写规则的工具

Agent：

不是：

外挂接口

而是：

运行时观察与修改通道

所以新的 V7 应该比旧 V7 再进化一步：

旧 V7：

模块化代码

新 V7：

原理化架构

区别：

旧：

“这个类放哪里？”

新：

“这个存在属于哪一种原理？”

最终拆解原则：

任何东西问五个问题：

1. 它是什么？

Identity

2. 它依赖什么？

Foundation

3. 它提供什么能力？

Capability

4. 它完成什么目标？

Domain

5. 它如何被看到？

Presentation

这样拆。

以后增加 C++：

不是改整个模组。

而是：
好，这一层我们就真正把“状态、规则、数据、行为四者分离”拆到底。

而这次我想先给一个很重要的判断：

Tetra 并没有做到“代码层面绝对四分”，但它已经在语义层面形成了非常明显的四分结构。

这反而更值得学。

因为我们真正应该学习的，不是“把四类东西各建一个文件夹”，而是：

同一个系统里的东西，究竟分别回答什么问题。

下面继续以你提供的 Tetra 1.20 源码为基础。

一、先把四个问题彻底分开

任何一个 Tetra 对象，都可以问四个完全不同的问题：

它现在是什么？
        ↓
状态 State

它为什么应该这样？
        ↓
规则 Rule

它有哪些可供选择的东西？
        ↓
数据 Data

它实际做什么？
        ↓
行为 Behavior

这个区分非常强。

因为很多屎山其实就是：

State
+
Rule
+
Data
+
Behavior

全部揉进一个类。

二、先看“数据”

Tetra 的 DataManager 已经把这个世界拆得非常明显：

DataManager
│
├── tierData
├── tweakData
├── materialData
├── improvementData
├── moduleData
├── repairData
├── enchantmentData
├── synergyData
├── replacementData
├── schematicData
├── craftingEffectData
├── actionData
├── unlockData
├── archetypeData
├── itemEffectData
└── modifierEffectData

这实际上已经是：

Definition Universe

即：

世界里有哪些规则对象可以存在。

但特别重要的是：

moduleData

并不是：

ItemModule
materialData

也不是：

Material Runtime

所以：

Data
≠
Runtime

这点 Tetra 做得非常明确。

三、“规则”则完全是另一回事

例如：

ItemProperties.merge()
VariantData.merge()
ToolData.merge()
AspectData.merge()

这些其实不是 Data。

它们是：

Combination Rules

比如：

durability

是数据。

而：

a.durability + b.durability

是规则。

再例如：

Material.rarity

是数据。

而：

material.rarity > current.rarity
    → 使用 material rarity

是规则。

又比如：

Improvement.group

是数据。

而：

同 group 的 improvement 不能共存

是规则。

所以：

group = "xxx"

是 Data。

sameGroup → replace old

是 Rule。

这一区分非常值得 V7 永久保留。

四、然后“行为”才真正开始动起来

例如：

ItemModule.addModule()
ItemModule.removeModule()
ItemModule.getProperties()

UpgradeSchematic.applyUpgrade()

CraftingEffectOutcome.perform()

ItemEffectOutcome.perform()

ModularItem.getToolData()
ModularItem.damageItem()

这些才是：

Behavior

也就是：

真正执行动作。

所以：

MaterialData

告诉系统：

“铁是什么”

而：

MaterialVariantData.combine()

告诉系统：
“铁和这个模块应该怎样结合”

再然后：

ItemModule.addModule()

才真正：

“把它装进去”

最后：

ItemStack NBT

保存：

“现在已经装进去了”
五、于是一个完整动作终于可以画出来了

例如：

给物品安装铁制模块

不是一个动作。

而是：

                DATA
                 │
        Module Definition
                 │
        Material Definition
                 │
                 ▼
                RULE
                 │
       Material × Module
           Combination
                 │
                 ▼
              BEHAVIOR
                 │
        Schematic.applyUpgrade
                 │
                 ▼
               STATE
                 │
          ItemStack NBT

这张图非常重要。

因为它说明：

行为产生状态变化。

规则决定行为应该怎么做。

数据提供规则需要的原料。

六、现在看 MaterialVariantData.combine()，它就是四者交汇的地方

这一个函数非常值得认真读。

输入：

Module Variant Data
+
Material Data

输出：

UniqueVariantData

然后里面分别计算：

attributes
durability
durabilityMultiplier
integrity
magicCapacity
effects
tools
aspects
rarity
glyph
models
tags

这里：

MaterialData
VariantData

是 Data。

而：

加法
乘法
覆盖
合并
优先级

是 Rule。

生成：

UniqueVariantData

是 Runtime Materialization。

之后：

ItemModule

拿这个结果去工作，才属于 Behavior。

七、这意味着“派生数据”不应该被当成第五种东西

这是一个容易混淆的地方。

我们现在会看到：

MaterialData
VariantData
UniqueVariantData

是不是还要继续：

DerivedData

当然可以有，但语义上：

Derived Data 是 Data 的运行期派生形态，不应该成为新的权威世界。

所以：

Source Data
   ↓
Rule
   ↓
Derived Definition

而不是：

Source Data
+
Derived Data
+
Another Data
+
...

最后产生数据沼泽。

八、Tetra 的 ModuleRegistry 是最好的证明

它做：

Raw ModuleData
 ↓
validate
 ↓
expand slot
 ↓
expand material variants
 ↓
merge duplicate variants
 ↓
construct ItemModule

因此：

ModuleData

不是最终模块。

真正的：

ItemModule

是经过规则处理之后的：

Runtime Semantic Object

这就是：

Data → Rule → Runtime

最清晰的一次实例。

九、再来看“状态”

Tetra 的状态主要还是落在 ItemStack NBT：

slot → module
module_material → variant
slot:improvement → level
honing_progress
honing_available
honing_count
repairCount
cooledStrength
identifier

源码的 IModularItem 直接提供大量这些 key，并提供：

putModuleInSlot
setTweakStep
removeHoneable
updateIdentifier

等方法。

所以：

ItemStack NBT

本质上是：

Instance State

这和：

ModuleData

有非常明确的区别。

十、这两个东西千万不能混

例如：

ModuleData
{
    key = "blade"
    variants = ...
}

表示：

所有这个模块实例都共享的定义。

而：

ItemStack
{
    left = "blade"
    blade_material = "iron"
}

表示：

这个具体 ItemStack 当前安装了什么。

于是：

Definition
≠
Instance State

这是 Tetra 最值得 V7 消化的一条。

十一、然后“规则”介入状态

假设：

ImprovementData.group = "magic"

这个 Data 本身没有修改状态。

当你安装另一个：

group = "magic"

的 Improvement 时：

ItemModuleMajor.addImprovement()

执行：

找到同组旧 improvement
→ remove
→ add new

所以：

State

并不是随便改。

它受到：

Rule

约束。

最终形成：

Rule
 ↓
Mutation
 ↓
State
十二、这一点对 V7 非常关键

以前我们写：

State Owner
+
Mutation API

现在应该更完整：

State Owner
      ↓
Mutation Request
      ↓
Domain Rules
      ↓
Invariant Check
      ↓
Mutation
      ↓
Revision++

也就是：

State 永远不能绕过 Rule 直接修改。

否则：

State

最后只是一个：

HashMap
十三、然后出现一个非常有趣的事实：
Tetra 的规则并没有全部放进 Rule 类

这是我们必须学习、但不能机械照搬的地方。

例如：

ItemProperties.merge()

里面就是规则。

VariantData.merge()

里面也是规则。

BaseSchematic.canApplyUpgrade()

又是一组规则。

MaterialVariantData.combine()

还是规则。

所以 Tetra 实际采用的是：

规则跟着它所属的领域对象走。

这其实是很健康的。

十四、这就是为什么 V7 不应该出现：
RuleManager
RuleEngine
UniversalPolicyEngine
GlobalRules

然后：

所有规则
→ 一个大 Map

这种设计。

应该更像：

Combat
 ├── CombatPolicy
 ├── DamageRule
 └── TargetRule

Repair
 ├── RepairPolicy
 ├── MaterialRule
 └── CompatibilityRule

即：

Rule 跟 Domain 走。

这与我们之前强调的“域内高内聚”正好一致。

十五、而 Tetra 的 Behavior 又进一步分层

它其实有三种非常明显的行为：

① Resolver Behavior
   把身份解析成对象

② Domain Behavior
   进行领域操作

③ Side-effect Behavior
   改 Minecraft 世界

比如：

ModuleRegistry.getModule()

属于：

Resolver
ConfigSchematic.applyUpgrade()

属于：

Domain Behavior
DamageEntityItemEffectOutcome.perform()

属于：

World Side Effect

所以以后 V7 不要把：

Behavior

理解成一个平面。

十六、这可以形成：
Behavior
│
├── Resolve
│
├── Evaluate
│
├── Plan
│
├── Mutate
│
├── Commit
│
└── Project

其中：

Resolve

决定对象是谁。

Evaluate

计算应该发生什么。

Plan

组织执行。

Mutate

改变权威状态。
Commit

确认写入。

Project

把结果展示到客户端。

这个结构已经和我们目前 V7 的 Execution 方向高度一致。

十七、然后有一个非常漂亮的例子：ModularItem.getToolDataRaw()

它会：

当前所有 Module
+
当前所有 Synergy
↓
ToolData.merge()
↓
最终 ToolData

这里同时出现：

Data
Rule
Derived State
Behavior

具体看：

ModuleData

提供数据。

ToolData.merge()

提供规则。

getToolDataRaw()

执行聚合行为。

最终：

ToolData

是派生结果。

然后：

toolCache

保存缓存。

所以一条链就是：

Definition Data
      ↓
Rule
      ↓
Runtime Evaluation
      ↓
Derived State
      ↓
Cache

这已经是 V7 State 模型的现实样板。

十八、而 Cache 又再次证明它绝不能成为 State

Tetra 的：

attributeCache
toolCache
effectCache
propertyCache

全部都来自：

ItemStack semantic identity
+
current definitions

所以：

Cache

只是：

“上一次算出来的结果”。

不是：

“这个 Item 的真实属性”。

这就是：

State
 ↓
Rule
 ↓
Derived
 ↓
Cache

而不能：

Cache
 ↓
再当 State

否则 reload 就崩。

十九、现在来到 Tetra 最有价值的地方：identifier

IModularItem.updateIdentifier() 每当 ItemStack 的语义构型发生变化时，会更新：

identifier

这个 identifier 又被：

data cache
model cache

使用。

所以它实际上在维护：

Semantic Version Marker

虽然 Tetra 没有把它做成 Revision API。

但思想已经在：

identifier

里出现了。

二十、这让我们可以非常漂亮地看出 Tetra 与 V7 的距离

Tetra：

State changes
   ↓
new identifier
   ↓
cache key changes
   ↓
old cache becomes unreachable

这是：

Identity-based Invalidation

V7：

State Owner
   ↓
Revision++
   ↓
DependencyStamp mismatch
   ↓
precise invalidation

这是：

Revision-based Invalidation

后者是我们可以继续站在前辈肩膀上进化出来的东西。

二十一、再看规则与数据的另一个分界：replace

例如：

MaterialData.replace
ModuleData.replace
VariantData.replace
ImprovementData.replace
CraftingEffect.replace

这个字段本身是：

Data

但是：

if replace:
    use new
else:
    merge old + new

属于：

Rule

然后：

MergingDataStore

执行：

Data + Rule
→ normalized definition

所以：

replace 不是行为。

它是：

控制数据如何进入新定义的规则参数。

这是一个特别容易犯错的分类。

二十二、这对 V7 的 Patch 系统尤其重要

比如：

PatchDefinition
{
    target
    mode
    priority
}

其中：

mode = REPLACE

不是行为本身。

真正行为：

PatchResolver
→ resolve
→ arbitration
→ transform

也就是说：

Patch Data
+
Patch Rule
+
Patch Behavior

还是应该分开。

二十三、现在我们可以总结出四者的真正关系

不是四个孤岛：

State   Rule   Data   Behavior

而是一条因果链：

                DATA
                  ↓
                RULE
                  ↓
               BEHAVIOR
                  ↓
                STATE
                  ↓
          DERIVED STATE
                  ↓
                CACHE
                  ↓
             PROJECTION

注意：

Derived
Cache
Projection

都是结果层，不是第四种新的权威来源。

二十四、而行为又可以反过来读取状态

所以完整系统应该是：

             ┌──────────────┐
             │     DATA     │
             └──────┬───────┘
                    ↓
             ┌──────────────┐
             │     RULE     │
             └──────┬───────┘
                    ↓
             ┌──────────────┐
             │   BEHAVIOR   │
             └──────┬───────┘
                    ↓
             ┌──────────────┐
             │    STATE     │
             └──────┬───────┘
                    ↓
               Derived State
                    ↓
                  Cache
                    ↓
               Projection

下一次行为又：

State
→ Behavior

所以真正世界是一个闭环：

Data
 ↓
Rule
 ↓
Behavior
 ↓
State
 └──────────→ Behavior
二十五、这个闭环就是“世界如何运行”

现在终于可以把你最开始那句话对应起来：

法则决定世界，世界决定物质。

在软件里：

Rule
 ↓
Behavior
 ↓
State
 ↓
Observed World

而：

Data

提供世界里的具体定义。

所以：

Rule
= 法则

State
= 当前世界

Data
= 世界中可用的定义物质

Behavior
= 世界发生变化的过程

这个对应关系其实非常漂亮。

二十六、而这恰恰解释了为什么“模块化”不能只是目录划分

真正的模块化应该是：

一个模块
=
明确的数据边界
+
明确的规则边界
+
明确的行为边界
+
明确的状态边界

例如 V7：

Combat
│
├── data
│    └── AttackDefinition
│
├── policy
│    └── DamagePolicy
│
├── state
│    └── CombatState
│
├── orchestration
│    └── CombatOrchestrator
│
└── execution
     └── CommitStep

它们不是为了“目录整齐”。

而是因为：

谁定义？
谁决定？
谁执行？
谁拥有当前事实？

全部不同。

二十七、现在 Tetra 反过来暴露出一个 V7 应该严格避免的问题
Tetra 的：

ItemModule

虽然非常高内聚，但里面同时存在：

Definition-derived data
+
Calculation rules
+
State mutation
+
Presentation
+
Tool behavior
+
Effect lookup

比如前面看到的：

getProperties()
getName()
addModule()
removeModule()
getModels()
getEffects()
getToolData()
setTweakStep()

都聚集在同一个模块类里。

这在 Tetra 的领域里可以理解。

但如果 V7 把这种思想推广到整个系统：

Everything
→ God Domain Object

就会出问题。

二十八、所以我们真正应该学习的是 Tetra 的“高内聚”，不是“大类”

原则应该变成：

一个对象可以很大，但它不能同时拥有多个不相干的权威。

例如：

ItemModule

可以拥有：

模块领域行为

但不应该同时成为：

世界状态 Owner
网络 Owner
Registry Owner
Renderer Owner
Provider Owner

这正好与 V7 的：

状态只有一个 Owner

结合起来。

二十九、现在终于能把“State / Rule / Data / Behavior”与 V7 五层对应起来
Core
│
├── State primitives
├── Rule primitives
└── Execution primitives

Capability
│
└── 稳定语义能力

Provider
│
└── 技术行为实现

Platform
│
└── Minecraft 外部世界接入

Application / Domain
│
├── Data
├── Rules
├── State
├── Orchestration
└── Behavior composition

所以四者并不是四个顶层 Layer。

而是：

横跨层级的四种语义。

这一点非常重要。

三十、这意味着 V7 不应该设计：
state/
rules/
data/
behavior/

然后每层都横跨所有领域。

否则又会产生：

万能 State
万能 Rule
万能 Data
万能 Behavior

正确的应该是：

combat/
    state/
    policy/
    data/
    orchestration/

repair/
    state/
    policy/
    data/
    orchestration/

weapon/
    state/
    policy/
    data/
    orchestration/

也就是：

先按 Domain 聚合，再在域内区分语义。

这和我们之前的“两级模块化”完全吻合。

三十一、于是今天这轮可以得到一个非常重要的 V7 公理候选

我甚至觉得这条比单纯“Definition ≠ Runtime”更底层：

数据描述存在，规则决定合法关系，行为产生变化，状态记录变化后的权威事实。

再进一步：

Data
    = What can exist

Rule
    = What is allowed / how things relate

Behavior
    = What happens

State
    = What has actually happened / what is currently true

这四句以后会非常好用。

三十二、拿一个具体的 V7 战斗例子看

比如你的“黄昏的回眸”攻击。

Data
baseDamage = 76
splashRadius = 13x13
splashRatio = 0.76
healRatio = 0.13
timeStop = 1s
growth = +2 max health

这些只是：

DATA
Rule
攻击必须有合法目标

主目标受到 76 Doom Damage

溅射目标按规则计算

死亡结算只有满足 kill predicate 才产生 Growth

...

这是：

RULE
Behavior
TargetResolver
DamageProvider
CombatOrchestrator
CommitStep

执行：

RULE

形成实际行为。
那么继续吧

继续往“规则之间的规则”挖以后，这一层确实很有东西。

前面我们已经知道：

Data
Rule
Behavior
State

但大型系统真正开始复杂，是从这里开始：

Rule A
+
Rule B
+
Rule C
+
Override D
+
External E

↓
到底怎么得到唯一结果？

而 Tetra 给出的答案并不是一个万能 RuleEngine。

它实际上使用了多种不同的裁决代数。

一、Tetra 最值得学的一件事：不是所有冲突都用“优先级”解决

这点非常关键。

我们在 Tetra 里可以直接看到至少四种不同的合并模式：

1. Override
2. Additive Merge
3. Max / RetainMax
4. Priority Selection

也就是说：

“冲突”不是一种问题。

不同语义的数据，需要不同的裁决规则。

例如 MaterialStore 的处理就是：

旧定义
+
新定义

if replace
    → 新定义整体替换
else
    → 字段级 copy / merge

这是明确存在于 Tetra 1.20 源码中的。

二、第一种：Replace
最简单：

A
+
B(replace=true)
↓
B

它表达的是：

“这个来源不是在补充旧定义，我就是要接管它。”

所以：

replace

其实不是普通字段。

它是在定义：

Authority Transition

也就是：

原本由 A 提供
↓
现在改由 B 提供

这其实与我们 V7 的 Provider 替换、Patch Override 很接近。

三、但是 Tetra 没有把所有对象都做成 Replace

比如 ModuleData.copyFields()。

这里：

slots
slotSuffixes
improvements
variants

有些采用：

concat
→ distinct

而：

renderLayer
namePriority
prefixPriority
tweakKey

则只有在新定义明确非默认值时才覆盖。

也就是：

旧值
+
新值
↓
Field-specific Merge

不是：

B != null → B

这么简单。

四、这带出了一个极其重要的原则：
Merge 是字段语义，不是对象语义。

例如：

durability

可能：

A + B
rarity

可能：

max(A, B)
namePriority

可能：

B overrides A
tags

可能：

union(A, B)
models

可能：

concat(A, B)

所以：

一个 merge() 函数实际上往往不是一个规则，而是一组字段级代数。

五、ItemProperties.merge() 就特别典型

Tetra 对属性合并采用不同算法：

durability
    → 加法

durabilityMultiplier
    → 乘法

integrity
    → 正负有特殊语义

integrityUsage
    → 累加

tags
    → 集合并集

rarity
    → 取更高

这非常漂亮。

因为它不是：

Map.putAll()

而是：

Domain Algebra

换句话说：

每一种领域数据都有自己的“组合数学”。
六、ToolData 再给了我们第二种证明

它同时存在：

overwrite()
merge()
retainMax()
multiply()
offsetLevel()

这五个名字已经把思想说明白了。

例如：

merge(A, B)

是：

level(A) + level(B)

而：

overwrite(A, B)

是：

A
→ B 覆盖相同 key

而：

retainMax(...)

则变成：

对于每种 Tool
取最高 Level

这三者完全不是同一个数学运算。

七、因此 V7 绝不能有一个：
七、因此 V7 绝不能有一个：
UniversalMerge<T>

这是这轮我最想钉死的一条。

应该是：

DamagePolicy.merge()
AttributeContribution.merge()
ProviderDescriptor.merge()
PatchDefinition.merge()
RenderLayer.merge()

每个领域决定：

Additive?
Override?
Max?
Min?
Union?
Intersection?
Exclusive?
Ordered?
八、这实际上给“规则”引入了第二层：

以前：

Rule

现在应该：

Rule
 ↓
Composition Rule

也就是：

规则：
“怎么做”

规则的规则：
“多个规则碰到一起时怎么算”

这个才是现在我们正在挖的这一层。

九、Tetra 的 Priority 又是另一种机制

例如：

namePriority
prefixPriority
renderLayer

它提供：

LOWEST
LOWER
LOW
BASE
HIGH
HIGHER
HIGHEST

这意味着：

多个模块都能提供名字

不是：

最后注册者赢

而是：

比较 Priority
↓
选择符合规则的贡献

比如 Tetra 明确把 namePriority 用于多个模块都试图提供物品名称的情况。

所以 Priority 本质上不是：

“哪个模块更强。”

而是：

“这个维度发生竞争时，应该由谁占据最终语义槽位。”

十、于是我们可以把四种裁决彻底区分
Override
= 一个来源接管

Additive
= 多来源共同贡献

Max
= 多来源竞争，取极值

Priority
= 多来源竞争，按排序选择

这四个概念以后一定不能混。

十一、还有一种更隐蔽的裁决：Last Matching Wins

ConfigSchematic.getOutcomeFromMaterial() 这里：

多个 outcome 都匹配
↓
reduce((a,b) -> b)
↓
最后一个匹配项胜出

也就是说：

Outcome A
Outcome B
Outcome C

如果都符合：

C

最终生效。

注释甚至明确写的是：

returns the last element.

这就是一种：

Ordered Override

它不是：

Priority

也不是：

Merge

而是：

Collection Order
+
last wins
十二、这正是一个危险信号

因为这说明：

规则的最终结果

可能依赖：

输入顺序

而输入顺序一旦来自：

JSON
ResourceManager
Map
Pack ordering

就会产生隐式行为。

这就是大型数据驱动系统非常容易出现的坑。

所以：

顺序也是规则。

不能假装“顺序只是实现细节”。

十三、Tetra 的 Synergy 排序实际上就是在修这个问题

前面我们已经看过：

SynergyStore.getOrdered()

会：

Arrays.sort(entry.moduleVariants)
Arrays.sort(entry.modules)

目的不是性能那么简单。

注释明确说：

不排序会导致物品错误获得 synergy。

也就是：

[A, B]

与：

[B, A]

语义上应该相同。

所以先：

Canonicalize

再：

Match

这才正确。

十四、这让我们发现一个非常重要的层级：
Data
↓
Normalization
↓
Composition Rule
↓
Resolution

不是：

Data
↓
随便 merge
十五、所以 V7 的 Definition Compiler 应该正式加入：
Conflict Arbitration

完整变成：

Parse
 ↓
Validate
 ↓
Normalize
 ↓
Canonicalize
 ↓
Merge
 ↓
Conflict Arbitration
 ↓
Resolve
 ↓
Runtime Snapshot
十六、然后又有一个前辈经验特别值得吸收：
“不存在冲突”也是一种裁决结果。

例如 Tetra 的：

same group

Improvement：

已有 group X
+
新增 group X

最终不是：

两者都存在

而是：

旧 X
↓
remove
↓
新 X

也就是：

Mutual Exclusion

所以裁决不一定是：

Winner A
Winner B

也可能是：

A and B cannot coexist
十七、这会直接升级 V7 的 ProviderMode

我们现在：

EXCLUSIVE
COMPOSABLE

已经很好。

但这次研究之后，我认为还应该从内部语义上拆：

EXCLUSIVE
    ├── PriorityWinner
    ├── LastWins
    ├── DeterministicFirst
    └── ReplaceExisting

COMPOSABLE
    ├── Additive
    ├── Union
    ├── Max
    ├── OrderedPipeline
    └── DomainMerge

注意：

这些不一定要变成公开 Enum。

这是“规则模型”，不是一定要再造十几个类。

十八、特别重要：不要把 Priority 当成万能解决方案

这是很多大型 Mod 最容易走上的路线：
冲突
↓
加 priority
↓
又冲突
↓
再加 priority
↓
100
500
1000
2000

最终：

Priority War

Tetra 并没有所有地方都使用 Priority。

有些地方：

Merge

有些地方：

Replace

有些地方：

Max

有些地方：

Group Exclusivity

有些地方：

Canonical Sort

所以真正成熟的原则是：

先确定冲突的语义，再选择裁决机制。

十九、这一点直接反哺你的 Mixin Repair 系统

例如两个 Mixin：

A
B

发生：

同一 injection point

不能直接：

谁 priority 高谁赢

因为不同类型的冲突应该不同：

两个都是必须执行的 →
Composable

两个都修改同一个独占返回值 →
Exclusive

一个是修复补丁，一个是原版替换 →
Override

两个修改互不重叠字段 →
Merge

无法确定 →
No Intervention

这就比单纯：

Priority = 2000

强得多。

二十、这正好强化 V7 原来那条非常重要的规则：
Target
→ Allowlist
→ Preconditions
→ Scope
→ Transform

再往后应该：

Transform Candidates
→ Conflict Classification
→ Arbitration Strategy
→ Selected / Composed Result
二十一、现在可以把“冲突”正式分成五类

我认为这一分类非常有价值：

Ⅰ. Duplicate Conflict
同一个东西出现两次

Ⅱ. Override Conflict
多个来源争夺同一个权威定义

Ⅲ. Composition Conflict
多个贡献能否共同存在

Ⅳ. Ordering Conflict
先后顺序改变语义

Ⅴ. Invariant Conflict
组合结果违反领域不变量

然后分别处理。

二十二、例如：
Duplicate
A
A

处理：

deduplicate
Override
A
B(replace)

处理：

B
Composition
Damage + Damage

可能：

sum
Ordering
Modifier A → B

与：

Modifier B → A

结果不同。

这时要：

canonical ordering
Invariant
Improvement A(group=X)
+
Improvement B(group=X)

处理：

reject / replace
二十三、于是终于出现一个真正成熟的“规则解析器”

概念上：

                 Candidate Rules
                        ↓
                 Identity Resolve
                        ↓
                Duplicate Detection
                        ↓
               Conflict Classification
                        ↓
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       Override      Compose       Exclusive
          ↓             ↓             ↓
       replace        merge        reject/choose
          └─────────────┼─────────────┘
                        ↓
                  Canonical Order
                        ↓
                   Invariant Check
                        ↓
                  Final Resolution

这就是我认为当前阶段 V7 最值得从 Tetra 学到的东西。

二十四、而且这里有一个特别漂亮的“前辈哲学”

Tetra 并没有试图回答：

“所有数据到底用什么统一规则合并？”

它实际上接受：

不同的数据，本来就有不同的物理法则。

比如：

durability
→ 可叠加

rarity
→ 取高

tool level
→ 可以求和 / 取最大 / 覆盖

这实际上非常接近你最开始的：

法则决定世界。

因为：

数据

只是“物质”。

真正决定它们怎么组合的是：

领域规则
二十五、这会让 V7 的“规则”真正拥有物理意义
例如：

Damage

不是简单：

float

因为它具有：

加法
乘法
抗性
类型
穿透
优先级
来源

所以应该拥有自己的：

DamageCombinationRule

再比如：

Patch

不是：

byte[]

而拥有：

target identity
scope
priority
mode
compatibility
conflict strategy
二十六、所以未来 V7 绝对不要写这种：
merge(Object a, Object b)

然后：

if number → add
if string → override
if list → concat
if map → putAll

这种万能 Merge 是典型的“看起来优雅，最后成为垃圾桶”。

因为真正的问题不是：

“两个对象怎么合？”

而是：

“这两个对象在这个领域里为什么可以合？”

二十七、甚至 null 也有自己的规则

这是下一层非常容易忽略的点。

例如：

a == null

并不总意味着：

return b

因为：

null

可能意味着：

未提供

也可能意味着：

明确删除

也可能意味着：

继承默认值

也可能意味着：

禁用

Tetra 的 VariantData.merge() 就通过默认值比较：

b.field != defaultValues.field

来区分：

“没有覆盖”

与：

“确实给了一个值”

这其实说明：

默认值语义也是 Merge Contract 的一部分。
二十八、这对 V7 的 Config 特别重要

以后最好不要简单：

Optional field

而要区分：

UNSET
SET(value)
CLEAR
INHERIT

例如：

PatchDefinition.priority

可能：

UNSET

表示继承默认优先级。

而：

CLEAR

表示：

禁止这个规则参加竞争。

如果用：

null

全部表示，最终肯定混乱。

二十九、然后 Tetra 的 sources 又应该进入 Merge Contract

你会发现：

replace

以后，Tetra 并没有简单把：

sources

扔掉。

SchematicStore 会把新旧 source 合并。

这说明：

Final Definition

除了：

Final Value

还应该保存：

Provenance

所以以后 V7 的最终结果应该实际上是：

Resolved<T>
{
    value
    provenance
    resolutionTrace
}

而不是：

T

直接结束。

三十、这件事对你的“万能修复模组”尤其重要

以后玩家看到：

Patch applied

还不够。

应该能看到：

Target:
xxx

Candidate:
Patch-A
Patch-B
Patch-C

Conflict:
Exclusive

Resolution:
Patch-B selected

Reason:
higher compatibility
then priority
then stable ID

Sources:
mod_x
mod_y

这时候：

Diagnostic 才真正拥有“为什么”。
三十一、这次最值得进入 V7 的新结构，我会这样定

不是马上冻结，但作为候选：

Resolution
│
├── IdentityResolution
│
├── DuplicateResolution
│
├── MergeResolution
│
├── ConflictResolution
│
├── OrderingResolution
│
└── InvariantResolution

它们都是：

Resolution

而不是：

六个新的顶层模块。
三十二、最终我觉得这轮真正挖到了“规则之上的法则”

以前：

Data → Rule → Behavior → State

现在：

                    Data
                      ↓
                    Rule
                      ↓
              Rule Composition
                      ↓
              Conflict Arbitration
                      ↓
                   Behavior
                      ↓
                    State

而：

Rule Composition

本质就是：

决定多个法则同时出现时，世界应该如何保持唯一且稳定的结果。

这已经非常接近你最初那个：

法则决定世界。

三十三、站在 Tetra 肩膀上之后，V7 可以比前辈再往前一步

Tetra 当前已经很好地做了：

Replace
Merge
Max
Priority
Canonical Sort
Group Exclusion

V7 可以进一步统一成：
继续。这一轮把 Validation → Fallback → Recovery 真正追到底以后，我觉得 Tetra 又给我们上了一课：

一个成熟的数据驱动系统，最重要的不只是“能正确加载”，而是“错误时究竟允许坏到哪一层”。

这句话非常关键。

一、Tetra 实际上存在多级故障隔离

把源码里的处理方式放在一起看：

Module
ModuleData 解析失败
        ↓
data = null
        ↓
validateModuleData()
        ↓
warn
        ↓
这个 Module 不进入 Runtime Registry

ModuleRegistry 明确检查：

null data
unknown type
empty slots

失败后返回 false，该项被过滤掉。

Schematic

同样：

SchematicDefinition
        ↓
validate
        ↓
创建 ConfigSchematic
        ↓
InvalidSchematicException
        ↓
打印 faulty module keys
        ↓
跳过该 schematic

InvalidSchematicException 专门携带：

key
faultyModules[]

最后只跳过有问题的 Schematic。

Model

模型解析失败：

JsonParseException / IOException
        ↓
logger.error
        ↓
return null
        ↓
filter(Objects::nonNull)

也就是说一个坏模型不会让整个模型数组直接塞入坏对象。

二、这说明 Tetra 有一个非常重要的思想：
局部坏，不等于全局坏。

例如：

100 个 Module
其中 1 个错误

理想结果：

99 个正常
+
1 个拒绝

而不是：

100 个全部消失

这正是一个大型 Mod 最需要的韧性。

三、但它不是所有地方都 Fail-open

这一点尤其重要。

例如 ReplacementDeserializer：

missing item
missing module
faulty predicate

会直接：

throw JsonSyntaxException

也就是说：

某些错误
→ 局部跳过

另一些错误
→ 让数据解析失败

这说明 Tetra 实际并没有一条：

Universal Fail-safe Rule

而是：

故障边界由具体数据类型决定。

这其实很符合前面我们研究出的“领域自己决定语义”。

四、于是“错误处理”也不是一个万能策略

现在可以把失败策略至少分成：

IGNORE
SKIP_ENTRY
FALLBACK
REPLACE
ABORT_RELOAD
ABORT_OPERATION
FAIL_OPEN
FAIL_CLOSED

而 Tetra 实际已经在使用其中一些。

例如：

Invalid Schematic
→ SKIP_ENTRY

模型：

坏模型
→ SKIP_ENTRY

Replacement：

坏引用
→ ABORT PARSE
五、这直接给 V7 一个比“Fail-open”更准确的概念

我们以前说：

修复系统必须 Fail-open。

现在要改成：

失败策略必须由故障对象的语义边界决定；默认不得让局部故障扩大到无关对象。

这个更成熟。

例如：

一个 Provider 挂了

可以：

Provider rejected
→ choose fallback
而：

所有 Provider 描述文件损坏

则可能：

无法建立安全 Runtime Snapshot
→ 保留旧 Snapshot

而：

目标定义缺失

则应该：

该 Patch 不介入

这比“全部异常 catch，然后 return false”成熟太多。

六、Tetra 又暴露了一个非常值得学习的东西：
Validation 发生在多个阶段。

并不是：

JSON.parse()

一次通过，就算合法。

实际上至少有：

① Syntax Validation
② Structural Validation
③ Reference Validation
④ Semantic Validation
⑤ Runtime Materialization Validation
七、第一层：Syntax

例如：

Vector
AABB
Color
Rarity
ResourceLocation

都有自己的 deserializer。

错误：

JsonParseException

直接拒绝。

这只是：

“格式是不是合法？”

八、第二层：Structure

ModuleRegistry：

slots == null
slots.length < 1

就拒绝。

这不是 JSON 语法错误。

而是：

对象结构不满足最小要求。

九、第三层：Reference Validation

ConfigSchematic：

moduleKey
   ↓
ItemUpgradeRegistry.getModule()

如果：

null

就：

InvalidSchematicException

这意味着：

JSON 是合法的，但它引用了不存在的对象。

十、第四层：Semantic Validation

比如：

materialSlotCount
slots
keySuffixes
outcomes
requirements

它们之间必须满足关系。

例如：

slots.length
==
keySuffixes.length

否则不能正确展开。

所以这属于：

语义约束。

十一、第五层：Materialization Validation

即使：

Definition

全部合法，也不意味着：

Runtime Object

一定能建立。

例如：

Module constructor
ConfigSchematic constructor
Variant expansion

都可能失败。

于是：

Runtime Materialization

本身也必须是验证边界。

十二、这直接强化 V7 的 Definition Compiler

现在完整链可以变成：

Raw Data
    ↓
Syntax Validation
    ↓
Structural Validation
    ↓
Reference Validation
    ↓
Semantic Validation
    ↓
Normalization
    ↓
Canonicalization
    ↓
Merge
    ↓
Conflict Arbitration
    ↓
Runtime Materialization
    ↓
Runtime Validation
    ↓
Publish

这已经非常完整。

十三、然后最关键的一步来了：
Publish

Tetra 在很多 Registry 里是：

moduleMap = newlyBuiltMap;

例如：

setupModules()

完成 Stream 构建以后，才把新 Map 交给：

moduleMap

这一点其实非常重要。

它意味着：

旧 Runtime

至少在新 Map 完成以前仍然存在。

十四、这正是我们 V7 Atomic Definition Publish 的现实原型

我们已经写过：

Old Snapshot
→ Build New Snapshot
→ Validate
→ Atomic Publish

这个设计不是凭空来的。

Tetra 的：

moduleMap = rebuiltMap

已经采用了一个非常相似的“构建完成后替换”思想。

V7 再把它正式化。

十五、于是失败时真正应该发生什么？

我们现在可以明确：

Old Snapshot
      ↓
Build New
      ↓
Validation Failed
      ↓
Discard New
      ↓
Old Snapshot remains

而不是：

Old Snapshot
      ↓
clear()
      ↓
Build New
      ↓
FAILED
      ↓
世界空了

这就是：

Atomic Publish

而不是：

Clear Then Rebuild
十六、这点对于你做“通用修复框架”尤其关键

想象：

Repair Rule Reload

旧：

20 条稳定修复规则

新资源包：

新增规则 A
新增规则 B
损坏规则 C

正确：

构建新快照

A ✓
B ✓
C ✗

→ 如果整组定义不能安全建立
→ 保留旧 20 条

而不是：

clear old rules
load new
→ C炸了
→ 0 rules

这就是“修复系统自己不能把世界弄坏”的真正工程版本。

十七、然后出现另一个重要概念：
Stale-but-valid假设：

新 Definition 有问题

旧 Definition：

仍然合法

那么：

继续使用旧 Snapshot

这意味着：

旧不是“过期垃圾”，而可能是最后一个已验证有效的世界。

这是一个非常高级的容错理念。

十八、于是 Revision 也有了一个更准确的意义

以前：

Revision++

更多意味着：

“状态变了。”

现在：

DefinitionRevision

更准确是：

“系统当前承认的有效定义世界版本变了。”

因此：

Candidate Revision

与：

Published Revision

应该分开。

十九、这一下又把 Definition Compiler 推进了一步

可以变成：

DefinitionRevision N
       │
       ▼
Candidate N+1
       │
       ├── invalid
       │     ↓
       │   discard
       │
       └── valid
             ↓
       Published N+1

这就是：

Versioned Configuration World
二十、然后看 Tetra 的 sources

这次它的重要性更清楚了。

一个最终定义不仅是：

Final Value

还应该是：

Final Value
+
Source Chain

例如：

schematic X
← tetra base
← addon A
← datapack B

如果最后：

invalid

那么诊断系统就可以知道：

是哪一层引入了问题

这正是我们之前发现 Provenance 的价值。

二十一、进一步：
Validation 应该产生 Diagnostic，不只是 boolean。

现在 Tetra 某些地方还是：

return false

或者：

throw InvalidSchematicException

V7 可以继续升级为：

ValidationResult
{
    status
    errors[]
    warnings[]
    provenance
    dependency
    suggestedAction
}

例如：

INVALID

Target:
foo.Bar

Reason:
method signature mismatch

Source:
patch_xyz

Dependency:
Mixin 0.8.5

Action:
NO_INTERVENTION

这就真正进入“自诊断系统”。

二十二、又回到了你最开始做 Crash Repair 的目标

你想做的是：

发现小冲突
→ 自动调整

实际上完整过程现在可以写成：

Observe
   ↓
Characterize
   ↓
Resolve Identity
   ↓
Validate
   ↓
Classify Failure
   ↓
Select Minimal Intervention
   ↓
Build Candidate
   ↓
Validate Candidate
   ↓
Publish Candidate
   ↓
Verify Runtime

如果 Candidate 失败：

Old Runtime remains

这才是一套真正安全的 Repair Engine。

二十三、Tetra 的公开 issue 又刚好给了我们现实证据

一个公开 issue 中，Tetra 6.7.0 在服务器生成 extractor_ruin 时出现错误方位的 block state，最终导致 chunk generation crash。用户尝试通过配置禁用该 feature，但问题仍发生。

这个例子特别有价值。

因为它体现的是：

Definition valid
       ↓
Runtime construction
       ↓
World-side invariant violated
       ↓
External side effect
       ↓
Crash

也就是说：

“配置能被加载”绝不代表“执行结果一定满足世界不变量”。

二十四、这就把 Validation 再推进一层：

不是只验证：

输入合法

还要验证：

输出是否满足不变量

即：

Postcondition Validation

例如：

Commit
  ↓
Verify

我们 V7 已经有 CommitStep Contract：

precondition
apply
verify
compensate
diagnostic

这次 Tetra 又给出了真实案例，说明 verify 不是锦上添花，而是必要的。

二十五、再来看另一个现实 issue

Tetra 2025 年的 Dedicated Server 问题中，客户端专用 ClientLevel 最终被错误地加载到了 Dedicated Server；服务器持续报错并出现严重卡顿。

这说明：

Validation

甚至不能只验证：

还必须验证：

Lifecycle
Environment

因此：

Validation Context

至少可能包含：

Dist
Platform
Version
Lifecycle
Dependencies
Authority

这和 V7 的 Dist Isolation、Provider Compatibility、Lifecycle Guard 再次对上。

二十六、还有一个特别重要的现实案例

Tetra 的另一个公开 issue 里，与 PassiveSkillTree 联用时，工作台无法正常安装原版工具模块。这里没有崩溃，但系统功能失效。

这说明：

Failure

不只有：

Crash

还有：

Silent Functional Failure

所以 V7 的 Validation 必须区分：

Crash
Hard Failure
Soft Failure
Semantic Mismatch
No-op
Degraded Mode

这对“玩家侧修复框架”极其重要。

二十七、于是我们可以把整个安全闭环正式画出来
                SOURCE
                   ↓
               PARSE
                   ↓
        Syntax Validation
                   ↓
      Structural Validation
                   ↓
       Reference Validation
                   ↓
        Semantic Validation
                   ↓
           NORMALIZE
                   ↓
        CANONICALIZE
                   ↓
             MERGE
                   ↓
       CONFLICT ARBITRATION
                   ↓
       RUNTIME MATERIALIZE
                   ↓
        POST-BUILD VALIDATE
                   ↓
          CANDIDATE SNAPSHOT
                   ↓
           ATOMIC PUBLISH
                   ↓
             RUNTIME
                   ↓
            POSTCONDITION
             /        \
          PASS        FAIL
           ↓            ↓
        Revision++   isolate /
                     compensate /
                     retain old

这已经是一个完整的：

Safety Lifecycle
二十八、而且有一个非常重要的哲学转变

以前我们容易把：

Fallback

理解成：

“出错了就找备用方案。”

现在更准确：

Fallback 是“在不破坏当前世界的前提下，如何继续提供最低可接受语义”。

所以它有等级：
假设：

新 Definition 有问题

旧 Definition：

仍然合法

那么：

继续使用旧 Snapshot

这意味着：

旧不是“过期垃圾”，而可能是最后一个已验证有效的世界。

这是一个非常高级的容错理念。

十八、于是 Revision 也有了一个更准确的意义

以前：

Revision++

更多意味着：

“状态变了。”

现在：

DefinitionRevision

更准确是：

“系统当前承认的有效定义世界版本变了。”

因此：

Candidate Revision

与：

Published Revision

应该分开。

十九、这一下又把 Definition Compiler 推进了一步

可以变成：

DefinitionRevision N
       │
       ▼
Candidate N+1
       │
       ├── invalid
       │     ↓
       │   discard
       │
       └── valid
             ↓
       Published N+1

这就是：

Versioned Configuration World
二十、然后看 Tetra 的 sources

这次它的重要性更清楚了。

一个最终定义不仅是：

Final Value

还应该是：

Final Value
+
Source Chain

例如：

schematic X
← tetra base
← addon A
← datapack B

如果最后：

invalid

那么诊断系统就可以知道：

是哪一层引入了问题

这正是我们之前发现 Provenance 的价值。

二十一、进一步：
Validation 应该产生 Diagnostic，不只是 boolean。

现在 Tetra 某些地方还是：

return false

或者：

throw InvalidSchematicException

V7 可以继续升级为：

ValidationResult
{
    status
    errors[]
    warnings[]
    provenance
    dependency
    suggestedAction
}

例如：

INVALID

Target:
foo.Bar

Reason:
method signature mismatch

Source:
patch_xyz

Dependency:
Mixin 0.8.5

Action:
NO_INTERVENTION

这就真正进入“自诊断系统”。

二十二、又回到了你最开始做 Crash Repair 的目标

你想做的是：

发现小冲突
→ 自动调整

实际上完整过程现在可以写成：

Observe
   ↓
Characterize
   ↓
Resolve Identity
   ↓
Validate
   ↓
Classify Failure
   ↓
Select Minimal Intervention
   ↓
Build Candidate
   ↓
Validate Candidate
   ↓
Publish Candidate
   ↓
Verify Runtime

如果 Candidate 失败：

Old Runtime remains

这才是一套真正安全的 Repair Engine。

二十三、Tetra 的公开 issue 又刚好给了我们现实证据

一个公开 issue 中，Tetra 6.7.0 在服务器生成 extractor_ruin 时出现错误方位的 block state，最终导致 chunk generation crash。用户尝试通过配置禁用该 feature，但问题仍发生。

这个例子特别有价值。

因为它体现的是：

Definition valid
       ↓
Runtime construction
       ↓
World-side invariant violated
       ↓
External side effect
       ↓
Crash

也就是说：

“配置能被加载”绝不代表“执行结果一定满足世界不变量”。

二十四、这就把 Validation 再推进一层：

不是只验证：

输入合法

还要验证：

输出是否满足不变量

即：

Postcondition Validation

例如：

Commit
  ↓
Verify

我们 V7 已经有 CommitStep Contract：

precondition
apply
verify
compensate
diagnostic

这次 Tetra 又给出了真实案例，说明 verify 不是锦上添花，而是必要的。

二十五、再来看另一个现实 issue

Tetra 2025 年的 Dedicated Server 问题中，客户端专用 ClientLevel 最终被错误地加载到了 Dedicated Server；服务器持续报错并出现严重卡顿。

这说明：

Validation

甚至不能只验证：

还必须验证：

Lifecycle
Environment

因此：

Validation Context

至少可能包含：

Dist
Platform
Version
Lifecycle
Dependencies
Authority

这和 V7 的 Dist Isolation、Provider Compatibility、Lifecycle Guard 再次对上。

二十六、还有一个特别重要的现实案例

Tetra 的另一个公开 issue 里，与 PassiveSkillTree 联用时，工作台无法正常安装原版工具模块。这里没有崩溃，但系统功能失效。

这说明：

Failure

不只有：

Crash

还有：

Silent Functional Failure

所以 V7 的 Validation 必须区分：

Crash
Hard Failure
Soft Failure
Semantic Mismatch
No-op
Degraded Mode

这对“玩家侧修复框架”极其重要。

二十七、于是我们可以把整个安全闭环正式画出来
                SOURCE
                   ↓
               PARSE
                   ↓
        Syntax Validation
                   ↓
      Structural Validation
                   ↓
       Reference Validation
                   ↓
        Semantic Validation
                   ↓
           NORMALIZE
                   ↓
        CANONICALIZE
                   ↓
             MERGE
                   ↓
       CONFLICT ARBITRATION
                   ↓
       RUNTIME MATERIALIZE
                   ↓
        POST-BUILD VALIDATE
                   ↓
          CANDIDATE SNAPSHOT
                   ↓
           ATOMIC PUBLISH
                   ↓
             RUNTIME
                   ↓
            POSTCONDITION
             /        \
          PASS        FAIL
           ↓            ↓
        Revision++   isolate /
                     compensate /
                     retain old

这已经是一个完整的：

Safety Lifecycle
二十八、而且有一个非常重要的哲学转变

以前我们容易把：

Fallback

理解成：

“出错了就找备用方案。”

现在更准确：

Fallback 是“在不破坏当前世界的前提下，如何继续提供最低可接受语义”。

所以它有等级：
继续。现在进入 Tetra 启发后的 V7 下一层：状态（State）、规则（Rule）、数据（Data）、行为（Behavior）四者分离。

这一层其实是从“模块化”进一步进入“世界建模”。

前面的 V7 解决的是：

大型模组如何拆，不让代码成为混沌。

而这一层解决的是：

一个对象为什么存在？它是什么？它能做什么？它如何变化？谁决定它变化？

一、传统模组最大的问题：四者混合

很多 Minecraft 模组（包括很多大型模组早期版本）容易变成：

Item
 |
 |-- 属性
 |-- 数据
 |-- 判断
 |-- 技能
 |-- 网络
 |-- 渲染
 |-- 保存
 |-- 事件
 |-- 条件
 |-- 特殊逻辑

最后一个类几千行。

例如：

墨羽原初.java

里面：

注册物品
判断玩家
计算伤害
播放粒子
调 Shader
改实体状态
存数据
发网络包
判断冷却
处理升级

最后它不是一个物品。

它变成了：

一个小型游戏引擎。

所以 V7 需要进一步抽象。

二、第一原则：世界不是由对象组成，而是由状态变化组成

一个神器：

“墨羽原初”

实际上不是：

一把剑

而是：

一个状态集合
+
一套规则集合
+
一组数据
+
一套行为解释器

类似：

现实：

汽车

不是一个整体。

拆开：

状态:
速度
油量
损坏程度

数据:
型号
重量
发动机参数

规则:
不能超过最高速度
燃料不足不能运行

行为:
加速
刹车
转向

Minecraft 也是一样。

三、V7 新核心模型

以后任何东西：

物品、实体、技能、攻击、防御、Shader

统一看：

          Definition
              |
              |
     -------------------
     |        |        |
   State    Rule    Data
     |
     |
  Behavior Engine
第一层：Data（数据）

回答：

它有什么？

数据不执行。

例如：

墨羽原初：

WeaponDefinition

{
 id:"primordial_black_feather",

 rarity:"origin",

 damage:100,

 shader:"void_rainbow",

 abilities:[
   "erase",
   "rewrite",
   "judgement"
 ]
}

只是描述。

没有攻击。

没有逻辑。

第二层：State（状态）

回答：

它现在是什么状态？

例如：

墨羽原初：

CurrentState

{
 energy:85,

 awakening:true,

 kill_count:132,

 evolution_stage:4,

 cooldown:20
}

状态可以变化。

但是状态不知道怎么变化。

比如：

energy=85

只是事实。

不是：

“攻击消耗15能量”

第三层：Rule（规则）

回答：

什么情况下允许什么事情发生？

例如：

攻击规则：

IF

energy >= 15

AND

target exists


THEN

allow attack

规则不执行。

它只是判断。

例如：

清除规则：

CanRemove(entity)

{

if(entity.isProtected)

 return false;


if(priority < attacker.priority)

 return true;

}
第四层：Behavior（行为）

回答：

真正怎么做？

这里才调用技术根。
例如：

攻击：

AttackBehavior

↓

DamageEngine

↓

Intercept

↓

TargetResolver

↓

RemoveCapability

↓

ParticleProvider


所以：

规则：

“可以攻击”

行为：

“执行攻击”

状态：

“攻击之后变成什么”

数据：

“攻击有什么参数”

四者关系

正确流程：

Input
 |
 v
Behavior Request
 |
 v
Rule Check
 |
 v
State Validate
 |
 v
Execute Behavior
 |
 v
Update State
 |
 v
Save Data
四、这和 Tetra 最大的启发在哪里？

Tetra 强大的地方不是装备复杂。

而是：

它没有把：

“一个工具”

当成一个固定物品。

它认为：

工具 = 模块组合结果。

例如：

不是：

DiamondSword

而是：

Handle

+

Blade

+

Material

+

Modifier

+

Rule

然后生成：

最终物品。

这直接启发 V7：

以后：

墨羽原初：

不要写：

class MoyuOriginal

而应该：

WeaponDefinition

+
AbilityModules

+
StateContainer

+
RuleSet

+
BehaviorGraph
五、进一步推导：V7 应该变成“五层宇宙”

现在重新整理：

0. Origin Layer（原初层）

一切技术根。

例如：

JVM
Mixin
ASM
Agent
Reflection
Network
Render
Shader

它们不是功能。

它们是：

“改变世界规则的力量”。

1. Infrastructure Layer（基础框架）

提供：

EventBus

CapabilitySystem

Registry

Serializer

Network

Config

Lifecycle

Diagnostic
2. Service Layer（服务层）

提供：

DamageService

RemoveService

RenderService

StateService

StorageService

TargetService
3. Domain Layer（领域功能）

这里才有：

Weapon

Skill

Shield

TimeStop

Judgement

Magic
4. Application Layer（组合）

例如：

墨羽原初：

UltimateWeaponController

组合：

Damage
+
Remove
+
Shader
+
Animation
+
Sound
+
State
六、这样以后升级技术就非常自然

比如未来：

JVM技术升级。

以前：

500个类改代码

现在：

只换：

runtime/jvm/

然后：

所有 Provider 自动获得升级。

未来：

Shader升级：

替换：

render/shader/provider

不用碰：

武器。

技能。

实体。

未来：

C++ Native加入：

新增：

native/provider

然后注册：

CapabilityProvider

即可。

七、所以新的 V7 方向应该调整

以前：

Capability中心

现在：

进一步升级：

Origin
 |
Infrastructure
 |
Service
 |
Capability
 |
Domain
 |
Application

同时：

所有实体对象内部：

Data
State
Rule
Behavior

四分离。

这一步实际上就是从：

“优秀 Minecraft Mod 架构”

迈向：

“小型游戏引擎架构”。

下一步继续挖的话，应该研究 Tetra 的“定义系统（Definition System）+ 数据驱动设计”，因为那正好对应 V7 如何让“未来新增神器、技能、装备不需要改核心代码”。

ok那么我们继续

继续。

现在我们已经从 “怎么拆代码” 进入了更深的一层：

一个模组世界如何被描述、生成、扩展。

这正是 Tetra 值得学习的地方。

上一层我们得到：

Origin
  ↓
Infrastructure
  ↓
Service
  ↓
Capability
  ↓
Domain
  ↓
Application

以及：

Data
State
Rule
Behavior

四分离。

现在继续向下挖：

八、Definition System（定义系统）

这是大型框架和普通模组最大的分界线。

普通模组：

写一个东西，就创建一个类。例如：

class PrimordialSword {

    attack(){

    }

}

新增神器：

继续复制：

class AnotherSword {

}

结果：

100件神器。

100个类。

100份逻辑。

最后爆炸。

而 Tetra 思路：

不是创建东西。

而是：

创建“描述东西的方法”。

也就是：

Definition。

1. 从“对象”转向“定义”

传统：

对象 = 行为

例如：

墨羽原初
    |
    攻击
    |
    清除

新的：

Definition
    |
    生成
    |
    Object

例如：

墨羽原初定义

{
 id:
 "moyu_original"

type:
 weapon

abilities:
 [
   erase,
   judgement,
   shader
 ]

rules:
 [
   origin_rule
 ]
}

然后系统读取：

生成：

墨羽原初实例

这一步非常关键。

因为：

核心代码不认识墨羽原初。

核心只认识：

“武器定义”。

九、为什么这是巨大升级？

假设未来：

你加入：

1000把神器。

传统：

核心代码：

Sword1.java
Sword2.java
Sword3.java
...
Sword1000.java

Definition：

核心：

WeaponEngine

永远不变。

新增：

moyu.json

void.json

chaos.json

time.json

即可。

这就是：

数据驱动。

十、V7应该加入 Definition Layer

重新升级：

                 Origin
                   |
        ---------------------
        |                   |
  Technical Root       Definition Engine
        |                   |
        |             ----------------
        |             |              |
 Infrastructure    Data Definition  Rule Definition
        |
     Service
        |
   Capability
        |
    Domain
        |
 Application
十一、Definition 不只是物品

这是重点。

任何东西都应该可定义。

1. 武器定义

例如：

{
"name":"墨羽原初",

"type":"weapon",

"damage":{
"value":76,
"type":"doom"
},

"abilities":[
"erase",
"time_stop",
"judgement"
]

}
2. 技能定义
{
"id":"absolute_remove",

"target":"enemy",

"condition":
"priority_check",

"execute":
"remove_entity"

}
3. Shader定义

不要：

SwordRenderer.java

里面写死。

而：
{
"id":"primordial_void",

"shader":
"void.frag",

"parameters":
{
 "color":"rainbow",
 "density":5
}

}
4. AI规则定义

甚至：

{
"entity":"boss",

"immune":[
"remove"
],

"weakness":[
"time"
]

}
十二、然后进入更高级：Graph（图）

这是下一层。

因为世界不是树。

世界是图。

比如：

墨羽原初攻击：

不是：

攻击
 |
伤害
 |
死亡

实际：

                 墨羽原初
                     |
                 Attack Event
                     |
              Rule Evaluation
                     |
       ----------------------------
       |            |             |
 Target       Damage        Visual
 Resolve      Engine        Engine
       |            |
       |        Protection
       |
 Remove

这是行为图。

所以未来 V7：

不要：

attack()

而应该：

BehaviorGraph

例如：

一次攻击：

节点：

InputNode

↓

TargetNode

↓

PermissionNode

↓

DamageNode

↓

RemoveNode

↓

VisualNode

↓

StateUpdateNode

优势：

调试的时候：

你可以看到：

攻击开始

√ 找到目标

√ 权限通过

√ Damage执行

× Remove失败

原因:
ShieldCapability阻挡

这正对应你之前说的：

游戏内开启验证模式，逐级调用功能，施加目标，反应算结果。

实际上就是：

Runtime Trace Graph。

十三、因此新的 V7 Debug系统也要升级

不是：

日志：

Error: NullPointer

而是：

世界模拟记录：

[Attack]

Origin:
MoyuOriginal

Pipeline:

Input
 OK

Rule
 OK

Target
 OK

Damage
 OK

Remove
 FAILED

Provider:
Shield

Priority:
Shield 2000
Attack 1000

Decision:
Protect wins


这才是真正的大型框架调试。

十四、回看 Tetra，真正值得学习的核心

不是装备系统。

而是三个思想：

① Definition

描述世界。

② Component

组合能力。

③ Data Driven

不用修改核心代码扩展。

所以新的 V7：

不应该是：

“很多 Capability 堆起来”。

而应该是：

世界规则平台

+
能力组件

+
定义系统

+
行为图执行器
十五、继续推导下一层

目前 V7 已经从：

第一代：

拆文件

升级到：

第二代：

分层架构

升级到：

第三代：

定义驱动世界

下一步应该继续研究：

“Component（组件化）与 Capability 的关系”

因为现在有一个关键问题：

Capability 是不是最终形态？

还是：

Capability 只是组件系统中的一种特殊组件？

这个问题会决定 V7 最终是不是能成为真正的“模组操作系统”。

好看看

继续往这一层挖。

前面我们已经从 Tetra 的模块化、组件化、配置化 看到了一些东西，现在进入更核心的一层：

「状态、规则、数据、行为四者分离」

这是很多大型框架从“能运行”走向“可演化”的关键。

一、传统模组的问题：四者混在一起

很多 Minecraft 模组早期写法：

Item
 ├─ 属性
 ├─ 技能
 ├─ 判断条件
 ├─ 数据保存
 ├─ 渲染
 ├─ 网络
 └─ 执行动作

一个物品类里面：
public class LegendarySword extends SwordItem {

    public void attack(Entity target){

        if(target.isBoss()){
            target.setHealth(0);
            spawnParticles();
            playSound();
            addBuff();
        }

    }

}

看起来简单。

但是问题：

想换攻击逻辑？
想增加新的判断？
想测试？
想迁移技术？

全部绑死。

最后形成：

巨型类
    ↓
修改一个地方
    ↓
影响十几个系统

这就是 r309 里面很多巨型类的问题。

二、Tetra 类框架给我们的启示

不是把东西拆碎。

而是：

每个东西只负责描述自己是什么。

比如一个武器。

不要：

武器 = 攻击逻辑

而应该：

武器
 |
 ├── Identity（身份）
 |
 ├── Structure（结构）
 |
 ├── Material（材料）
 |
 ├── Modifier（规则）
 |
 ├── Effect（行为）
 |
 └── Renderer（表现）
三、四层分离
1. 数据 Data

回答：

它有什么？

例如：

{
 "damage":76,
 "range":13,
 "element":"doom"
}

数据不执行。

数据只是事实。

类似：

“剑拥有76伤害”。

2. 状态 State

回答：

当前是什么情况？

例如：

玩家：

HealthState
{
 hp:100
 shield:50
 rage:20
}

武器：

WeaponState
{
 cooldown:5
 charge:80%
 awakened:true
}

状态会变化。

3. 规则 Rule

回答：

什么情况下发生什么？

例如：

如果：
目标死亡

那么：
回复13%生命
增加最大生命2
播放特效

这是规则。

不是数据。

4. 行为 Behavior

回答：

怎么执行？

例如：

RemoveBehavior

DamageBehavior

ParticleBehavior

ShaderBehavior

NetworkBehavior

行为是真正干活的人。

四、对应到 V7

这其实正好修正现在 V7 的方向。

现在 V7：

Core
Capability
Domain
Application
Platform

方向正确。

但是还可以继续升华。

变成：

                原初核心
                    |
        ┌───────────┴───────────┐
        |
      基础世界层
        |
 ┌──────┼──────┐
 |
Data   State   Rule   Behavior
 |
Capability
 |
Domain
 |
Application
 |
Presentation
五、例如“墨羽原初”

不要设计：

MoYuOriginalBow.java

里面10000行。

而应该：

身份
MoYuDefinition

描述：

名字
材质
纹理
注册ID
标签
状态
MoYuState

保存：

觉醒等级
击杀数量
累计生命
冷却
规则
MoYuRuleSet

例如：

OnHitRule

OnKillRule

NoTargetRule
行为
DamageExecutor

RemoveExecutor

HealExecutor

ParticleExecutor

ShaderExecutor

最终：

射出一箭：

不是：

Bow.attack()
好，那么进入 「事件流与因果链」。
这其实是 V7 从“模块化工程”继续向“框架哲学”推进的一层。

前面我们挖 Tetra 学到的是：

状态、规则、数据、行为四者分离。

但如果继续往下挖，会发现还有第五个核心：

一切行为不是孤立发生，而是在时间中形成事件流，事件流连接成因果链。

这正是大型系统和普通模组最大的区别。

一、传统模组的问题：直接调用导致因果黑箱

很多 Minecraft 模组的逻辑类似：

玩家攻击
 ↓
Item.onUse()
 ↓
直接改血
 ↓
生成粒子
 ↓
播放声音
 ↓
掉落物

表面简单。

但是问题：

如果出错：

为什么伤害变成这个数字？
谁修改了血量？
哪个效果触发了？
哪个 Mixin 插入了？
哪个属性覆盖了？

你只能：

日志
 ↓
猜
 ↓
反编译
 ↓
搜索
 ↓
痛苦

因为：

行为发生了，但是没有留下因果链。

二、理想框架：事件流

V7 应该把所有重要行为抽象成：

Event
  ↓
Decision
  ↓
Execution
  ↓
Result
  ↓
Record

也就是：

事件
→ 判断
→ 执行
→ 结果
→ 记录

举你的「墨羽原初」神器。

普通写法：

右键
 ↓
发射箭
 ↓
伤害
 ↓
爆炸

V7：

PlayerUseEvent

{
 source:
   Player

 item:
   墨羽原初

 action:
   RELEASE
}


↓

IntentResolver

判断：

这是攻击意图？

↓

CombatDecision

生成：

AttackCommand


↓

Capability调用：

TargetResolution
        |
        |
        ↓

DamageDomain

        |
        ↓

Remove/Shield/Protect


↓

ResultEvent

产生：

DamageApplied
TargetKilled
ParticleSpawn
SoundPlay


↓

Ledger记录：

谁
什么时候
为什么
调用了什么
结果如何

三、事件不是消息，而是“因果证明”

这里很重要。

很多框架把 Event 当成：

“通知别人发生了什么”。

例如：

Forge Event：

LivingAttackEvent

只是：

“有人攻击了”。

但是 V7 应该升级：

Event = 因果节点。

例如：

AttackEvent #10001


Origin:
 玩家


Intent:
 攻击


Authority:
 墨羽原初


Capabilities:
 TargetResolution
 DamageEngine
 Remove


Dependencies:
 Attribute +15%
 MagicDamage +30%


Result:
 目标死亡


Verification:
 PASS


这有什么意义？

未来出现：

“为什么这个怪死了？”

不用查代码。

直接：

查询事件链

Target:
 Zombie


死亡原因：

Event #10001

来源:
 墨羽原初


执行链:

攻击请求

 ↓

目标锁定

 ↓

伤害计算

 ↓

护盾检测

 ↓

最终伤害

 ↓

死亡


四、进一步：因果链图

大型系统不是树。

而是图。

例如：

                  玩家输入
                     |
                     |
                Intent
                     |
        ┌────────────┼───────────┐
        |            |           |
   Target       Damage       Visual
   Resolver     Engine       Engine
        |            |
        |            |
        ↓            ↓
     Entity      Health State
                     |
                     |
                 Death Event
                     |
              Remove Capability


这里和 Tetra 的思想很像：

Tetra 不认为：

“一个武器=一个类”。

而认为：

材料
 ↓
模块
 ↓
能力
 ↓
规则
 ↓
结果

V7 同理：

神器不是一个 Item 类。

神器是：

Definition
+
Capability组合
+
Domain规则
+
Event链
五、V7新增核心层：Event Kernel

所以未来 V7 架构可能变成：

Primordial Framework


                Event Kernel
                     |
        ┌────────────┼────────────┐
        |            |            |
 Capability      Domain       Presentation
        |            |            |
        |            |            |
  技术能力       游戏规则       显示效果

Event Kernel负责什么？

不是做功能。

只做：

1. 事件生命周期
CREATE

↓

VALIDATE

↓

PROCESS

↓

COMMIT

↓

FINISH

2. 因果记录
Cause

↓

Action

↓

Effect

3. 优先级

例如：

多个模块修改伤害：

基础伤害

↓

装备加成

↓

药水

↓

神器倍率

↓

防御

↓

最终值


不是靠：

“谁最后执行谁赢”。

而是：

事件阶段决定。

六、这解决一个未来巨大问题

你之前说：

以后可能换 JVM 技术，换 C++，换新的技术。

如果没有事件层：

技术和业务绑死。

例如：

ASM
 ↓
修改血量

换技术：

全部重写。

但是：

Event

 ↓

DamageCommand

 ↓

DamageProvider


下面：

可以换：

Java实现

↓

ASM实现

↓

Agent实现

↓

Native实现

↓

C++实现


上层完全不知道。

七、所以继续完善 V7 五大根

目前越来越清晰：
好，那么继续沿着这条线往下挖。

我们现在已经从：

模块化
状态 / 规则 / 数据 / 行为分离
事件流与因果链

走到了下一层：

「世界状态模型」

也就是：

一个理想框架，如何理解“世界正在发生什么”。

一、普通模组的世界观：变量堆积

传统 Minecraft 模组大部分是：

Entity.health = 0;

player.addEffect();

item.cooldown--;

block.setState();

它认为：

世界 = 一堆变量。

所以开发模式：

需要什么
↓
加变量
↓
写函数修改变量

最后形成：

Entity
 ├── health
 ├── armor
 ├── capability
 ├── nbt
 ├── effects
 ├── attributes
 └── 自定义字段


问题：

世界没有“历史”。

它只知道：

现在是什么。

不知道：

为什么变成这样。

二、V7需要升级：

世界不是变量集合。

世界是：

一个不断演化的状态系统。

类似：

State(t0)

    |
    | Event
    ↓

State(t1)

    |
    | Event
    ↓

State(t2)


也就是说：

任何变化必须经过：

旧状态
 ↓
事件
 ↓
规则判断
 ↓
新状态

举墨羽原初：

普通：

target.setHealth(0);

V7：

TargetState:

Health = 100


收到：

JudgmentStrikeEvent


经过：

DamageRule

RemoveRule

ProtectionRule


生成：

NewTargetState:

Health = 0
Death=true

三、状态必须有“版本”

这是大型系统非常重要的一点。

例如：

现在：

Zombie

Health:
20

Version:
100

被攻击：

DamageEvent

之后：

Health:
0

Version:
101

那么：

Version 的意义是什么？

它表示：

这个状态来自哪一次世界演化。

这样可以实现：

1. 调试

查询：

为什么 Zombie 死了？

查看:

Health Revision 101

来源:

Event 55682

2. 回滚

例如：

测试神器：

发现 Bug。

不用重启。

恢复：

State Revision 100

3. 验证

你之前说：

游戏内开启验证模式，逐级调用功能，验证功能有没有问题。

本质就是：

状态比较。

例如：

测试：

墨羽原初攻击。

预期：

Before:

Enemy HP=100


After:

Enemy HP=0


实际：

Enemy HP=-999999


发现：

Damage模块异常。

四、状态和数据必须分离

这里继续学习 Tetra 的思想。

很多人混淆：

数据 ≠ 状态。

例如：

墨羽原初：

数据：

damage=76
range=13
effect=time_stop

这是：

Definition。

状态：

当前蓄力
当前冷却
当前强化等级
当前绑定玩家

这是：

Runtime State。

所以：

应该：

Item Definition

     +

Item Runtime State

     +

Event History


三个东西分开。

五、进一步：权威状态（Authority）

这里连接 V7 的 Capability。

为什么需要 Capability？

因为：

不是所有东西都有资格修改状态。

例如：

血量。

普通：

entity.health=0;

谁都能改。

V7：

Health State

       ↑

HealthAuthority

       ↑

Damage Capability

       ↑

Combat Domain


只有经过授权链：

才能改变。

类似现实：

一个人死亡。

不是：

任何人写一句：

alive=false

就结束。

需要：

攻击发生

↓

伤害计算

↓

生命归零

↓

死亡规则

↓

尸体处理

六、状态模型继续扩展：

最终：

V7世界应该类似：
好，继续进入 「规则系统（Rule Engine）」。
这一层其实是从 Tetra 这类优秀模组架构中可以继续向上抽象的一层。

前面我们已经挖到了：

状态（State）
世界当前是什么样
例如：
玩家生命值
武器等级
材料组合
实体状态
能量值
数据（Data）
状态的载体
例如：
NBT
Capability
JSON
Registry 数据
配置文件
行为（Behavior）
可以做什么
例如：
攻击
清除
渲染
释放技能
事件流（Event Flow）
什么事情发生了

例如：

玩家拉弓
   ↓
发射事件
   ↓
命中事件
   ↓
伤害事件
   ↓
死亡事件

但是还有一个最关键的问题：

为什么这个行为此时可以发生？

这就是：

Rule Engine（规则引擎）
一、第一性原理重新推导

如果把整个模组看成一个世界模拟器：

世界
 |
 |-- 状态 State
 |
 |-- 数据 Data
 |
 |-- 行为 Behavior
 |
 |-- 规则 Rule
 |
 |-- 事件 Event

那么：
状态 = 世界是什么
数据 = 世界如何保存
行为 = 世界能做什么
事件 = 世界什么时候变化
规则 = 世界为什么允许变化

规则就是：

法则。

也就是你前面说的：

法则决定世界。

代码层面：

Rule
 ↓
决定
 ↓
Behavior
 ↓
改变
 ↓
State
二、传统模组的问题

很多 Minecraft 模组实际上是：

Item
 |
 |
代码
 |
 |
if
 |
 |
执行

例如：

if(player.hasItem()){
    damage();
}

问题：

规则藏在行为里面。

结果：

1. 不可复用

这个判断只能这个物品用。

2. 不可观察

你不知道为什么触发。

3. 不可修改

以后想改变：

"末影龙不能受到这个伤害"

怎么办？

到处找 if。

所以高级框架应该：

拆开：

攻击行为
      |
      |
Rule判断
      |
      |
允许？
      |
      |
Damage执行
三、V7应该加入 Rule 层

新的理想结构：

Primordial
│
├── Foundation
│
├── Runtime
│
├── Service
│
├── Rule Engine   ★新增核心
│
├── Capability
│
├── Domain
│
├── Presentation
│
└── Content
四、Rule Engine 的职责

它不攻击。

它不清除。

它不渲染。

它只回答：

能不能？
为什么？
优先级？
条件是什么？

例如：

墨羽原初攻击：

事件：

ProjectileHit

进入：

CombatRuleEngine

询问：

规则1：

目标是否有效？

↓

TargetRule

规则2：

目标是否免疫？

↓

ProtectionRule

规则3：

伤害是否允许？

↓

DamageRule

规则4：

是否触发处决？

↓

ExecuteRule

最后：

允许
 ↓
调用 Damage Capability
五、Rule 和 Capability 的关系

这是重点。

很多人会混。

Capability：

我拥有这个能力。

Rule：

什么时候允许使用这个能力。

例如：

Remove Capability：

能力：

删除实体

但是：

Rule：

允许删除？

判断：

Boss?
玩家?
保护区?
剧情实体?

所以：

Capability = 手

Rule = 大脑
六、Tetra 值得学习的地方

Tetra 强大的地方不是某个功能。

而是：

它没有把：

锤子
剑
刀
材料
属性
升级
效果

写死。

而是：

模块
+
规则
+
组合
+
数据驱动

例如：

一个武器：

不是：

DiamondSword.java

而是：

Weapon
 |
 ├── Handle
 ├── Blade
 ├── Material
 ├── Modifier
 └── Rule

组合产生结果。

V7 应该学习这个思想。

不要：

墨羽原初.java

巨大类。

应该：

MoyuOrigin
|
├── Identity
│
├── RenderProfile
│
├── AttackProfile
│
├── RuleSet
│
├── AbilitySet
│
└── RuntimeBinding
七、Rule Engine 内部结构建议

未来：

rule/
│
├── RuleEngine
│
├── RuleContext
│
├── RuleResult
│
├── RulePriority
│
├── RuleRegistry
│
├── RuleChain
│
└── rules/
    │
    ├── DamageRule
    ├── RemoveRule
    ├── TargetRule
    ├── RenderRule
    ├── NetworkRule
    └── SecurityRule
八、规则执行链

类似 JVM：

事件进入
 |
 |
创建 Context
 |
 |
RuleChain
 |
 |
Rule1
 |
Rule2
 |
Rule3
 |
 |
Decision
 |
 |
Capability
 |
 |
State Change

例如：

攻击：

AttackEvent

 ↓

TargetRule

 ↓

DamageRule

 ↓

ResistanceRule

 ↓

PriorityRule

 ↓

DamageCapability

 ↓

HealthState
九、最高级的地方：规则也是数据
好，继续进入 「调度系统（Scheduler / Execution Pipeline）」。

这一层实际上是前面所有设计的“动力系统”。

如果说：

State（状态） 是世界的存在；
Rule（规则） 是世界的法则；
Capability（能力） 是世界中的力量；
Event（事件） 是世界发生的事情；

那么：

Scheduler（调度系统）

就是：

世界如何按照秩序，让这些东西发生。

一、为什么需要调度系统？

因为真实世界不是：

事件
 ↓
直接执行

而是：

事件发生
 ↓
多个系统响应
 ↓
多个规则判断
 ↓
多个能力竞争
 ↓
产生结果

例如 Minecraft：

玩家挥剑。

看似：

攻击
↓
扣血

实际上：

玩家输入
 ↓
物品系统
 ↓
攻击事件
 ↓
附魔系统
 ↓
属性系统
 ↓
护甲系统
 ↓
抗性系统
 ↓
护盾系统
 ↓
伤害计算
 ↓
死亡判断
 ↓
掉落
 ↓
经验
 ↓
粒子
 ↓
声音

这里有几十个参与者。

没有调度系统，就会出现：

谁先执行？
谁覆盖谁？
谁拥有最终权？
冲突怎么办？
二、传统 Forge 事件的问题

Forge Event 本身提供了一种调度：

例如：

MinecraftForge.EVENT_BUS.post(event);

但是它比较基础。

它的问题：

1. 顺序不透明

A 模组监听：

@SubscribeEvent
attack()

B 模组监听：

@SubscribeEvent
attack()

谁先？

依靠：

EventPriority
注册顺序

但是大型系统需要更精确。

2. 没有完整生命周期

普通 Event：

开始
 ↓
监听
 ↓
结束

但是高级系统需要：

准备
 ↓
验证
 ↓
计算
 ↓
申请权限
 ↓
执行
 ↓
提交
 ↓
检查结果
三、V7 应该设计 Execution Pipeline
好我们继续

好，我们继续。

现在进入：

「权限与权威系统（Authority / Governance Layer）」

这一层其实是前面所有设计的最高约束之一。

因为：

Event 解决 发生了什么
Scheduler 解决 什么时候执行
Rule 解决 为什么允许
Capability 解决 怎么做到

但是还有一个更根本的问题：

谁拥有改变世界的资格？

这就是 Authority。

一、第一性原理推导

如果世界是一套模拟系统。

那么任何变化：

State A
  |
  | 修改
  ↓
State B

中间必须存在：

Change Request

例如：

Health:
100 → 0

是谁提出的？

可能有：

玩家攻击
怪物攻击
药水
技能
神器
脚本
管理员
调试系统
它们都可以提出：

“我要改变生命状态。”

但是：

它们的权力相同吗？

显然不同。

所以：

V7 不应该：

entity.setHealth(0);

而应该：

Request Change

↓

Authority Check

↓

Permission Grant

↓

Capability Execute

↓

State Commit
二、Authority 的本质

可以抽象成：

权威 = 修改世界状态的资格

类似现实：

一个普通人：

说：
这栋楼拆掉

没有效果。

政府审批：

批准拆除

才执行。

代码世界一样。

三、Authority 分层

V7 可以设计：

Authority Layer

        |
        |
------------------------
|          |            |
Origin   System       User
1. Origin Authority（原初权威）

最高层。

类似：

“世界创造规则”。

例如：

墨羽原初。

它不是普通武器。

它可能拥有：

Origin.Damage
Origin.Remove
Origin.Override

这种权限。

2. System Authority（系统权威）

框架自身。

例如：

State Kernel
Scheduler
Rule Engine
Security

它们可以：

保存状态
验证
修复
回滚
3. User Authority（用户权威）

普通来源：

玩家
实体
AI
模组
脚本
四、权限不是一个数字

很多系统会设计：

priority = 999

但这不够。

因为：

权限有不同维度。

例如：

一个 Shader：

可以修改视觉

但是：

不能：

修改生命

所以应该：

Capability Scope。

例如：

Authority Token

{
 owner:"MoyuOrigin",

 capability:[
   Damage,
   Remove
 ],

 scope:
 {
   entity:"hostile",
   dimension:"all"
 },

 level:
 Origin
}

五、权限和 Capability 的关系

这里非常重要。

Capability：

我能做什么。

Authority：

我有没有资格做。

例如：

Remove Capability：

拥有删除能力

但是：

请求：

删除玩家

需要：

检查：

RemoveCapability
+
RemoveAuthority

关系：

                 Authority

                     ↓

Capability  ← 是否允许调用

                     ↓

                 Execution
六、解决大型模组最恐怖的问题：
“谁覆盖谁？”

例如：

三个系统：

普通伤害：

DamageCapability
Priority 50

护盾：

ShieldCapability
Priority 100

原初神器：

OriginJudgment
Priority Origin

发生：

AttackEvent

系统竞争：

Damage
Shield
Origin

怎么办？

不是：

谁注册晚谁赢。

而是：

Authority 决定。

结果：

Origin Authority

↓

Override Shield

↓

Execute Damage

七、但是必须防止滥权

这里进入高级设计：

Authority Governance

最高权限也不能无限乱用。

例如：

墨羽原初：

可以：

删除敌人

但是：

不能：

删除世界存档

为什么？

因为：

权限也有边界。

所以：

Authority 包含：

Identity

Scope

Limit

Audit

八、审计系统（Audit）

这是大型框架必须有的。

每一次权威操作：

记录：

谁

什么时候

请求什么

为什么允许

结果是什么

例如：

日志：

[Authority]

Source:
MoyuOrigin

Action:
RemoveEntity

Target:
Zombie#5512

Rule:
JudgmentRule

Result:
Success

StateRevision:
10291


这样：

以后验证模式：

你可以问：

为什么这个怪死了？

系统回答：

因为：

MoyuOrigin

调用:

JudgmentStrike

通过:

RemoveRule

修改:

HealthState

九、Authority 与你的“神器思想”

其实你的墨羽原初设计：

以前看起来像：

一个超级武器

但是按照这个框架：

它实际上应该是：

一个 Authority Provider

它提供：

Origin Combat Authority

然后：

不同功能枝：

挂载：

Damage Branch

Remove Branch

TimeStop Branch

Shader Branch


这些不是独立力量。

而是：

同一个原初权威在不同领域的投影。

十、最终 V7 架构继续进化

目前：
好，继续进入这一层。

「观测、诊断与自演化系统（Observer / Diagnostic / Evolution Layer）」

这一层其实是 V7 从“优秀框架”走向“生态级框架”的关键。

前面的系统解决：

State：世界保存什么
Rule：世界为什么变化
Scheduler：变化什么时候执行
Authority：谁有资格改变

但是还有一个更高的问题：

一个复杂世界，如何知道自己有没有正常运行？

这就是：

观察自己。

一、第一性原理推导

任何复杂系统都会出现：

错误
冲突
冗余
无效结构
性能退化
外部环境变化

例如人体：

身体不仅有器官。

还有：

神经系统
痛觉
免疫系统
自我修复

否则：

一个小问题可能逐渐扩大。

代码系统也是一样。

大型 Mod：

不是写完结束。

而是：

一个持续变化的生命体。

所以 V7 应该拥有：

Primordial Root

      |
      |
 Observer

      |
      |
 Diagnostic

      |
      |
 Evolution
二、Observer（观测系统）
核心：

不改变世界，只记录世界。

类似：

摄像机。

它观察：

谁调用了谁
什么状态发生变化
哪个模块活跃
哪些代码没有作用
1. 调用链观察

例如：

玩家使用墨羽原初：

流程：

Player

 ↓

ItemUseEvent

 ↓

WeaponController

 ↓

JudgmentRule

 ↓

DamageCapability

 ↓

HealthState

 ↓

DeathEvent


Observer 捕获：

{
 event:"Attack",

 chain:[
  Player,
  Item,
  Rule,
  Capability,
  State
 ],

 result:
 Success
}

这直接解决你的：

游戏中开启验证模式，然后逐级调用功能，施加目标，反应算结果。

三、Dependency Graph（依赖图）

这是大型框架核心。

每个模块：

不是孤立文件。

而是：

Capability A

     ↓

Service B

     ↓

Technology C

形成：

          Root

        /     \

    Combat   Render

      |        |

 Damage     Shader

      |

  JVM Hook

Observer 可以生成：

实时架构图。

例如：

发现：

OldDamageFix

      ↓

Nobody

说明：

这是：

孤儿代码。

四、孤儿代码检测

这是非常重要的一点。

大型 Mod 最容易出现：

public class OldMagicAttack {

}

存在。

但是：

没人调用。

没人注册。

没人依赖。

它是什么？

垃圾。

V7：

每个模块注册：

@ModuleInfo(
 id="damage.origin",
 owner="Combat"
)

系统统计：

Created
↓
Registered
↓
Called
↓
Output

状态：

Active
注册
调用
产生结果
Dormant
注册
但无调用
Orphan
存在
无入口
无依赖
Dead
无法加载

这就是：

代码生态分析。

五、Diagnostic（诊断系统）

Observer 是眼睛。

Diagnostic 是医生。
它负责：

判断问题。

1. 启动诊断

启动游戏：

执行：

Bootstrap

 ↓

Module Scan

 ↓

Dependency Check

 ↓

Capability Check

 ↓

Authority Check

输出：

例如：

[V7 Diagnostic]

Module:
Primordial Combat

Status:
Healthy

Capability:
Damage ✓
Remove ✓
Shader ✓

Warning:
OldTransformer unused

2. 运行时诊断

例如：

玩家攻击。

正常：

Attack

↓

Rule

↓

Damage

↓

Commit

异常：

Attack

↓

Rule

↓

Damage

↓

Exception


系统记录：

Rollback

Reason:
DamageCapability failed
六、故障隔离

大型系统必须：

不要“一处爆炸，全家陪葬”。

所以：

Diagnostic 配合：

Fault Domain

例如：

Shader 崩：

错误：

ShaderCompileException

不要：

Minecraft crash

而应该：

Disable Shader Domain

Keep Combat Running

类似：

人体：

手受伤。

不会：

停止心脏。

七、Evolution（自演化）

这里进入最高层。

注意：

不是 AI 自动乱改代码。

而是：

可进化架构。

核心：

让系统允许未来替换自己。

例如：

今天：

JVM Provider:

Reflection

未来：

升级：

Unsafe

以后：

Native

甚至：

C++

怎么办？

如果架构正确：

不用改业务。

只替换：

Provider。

结构：

Capability

      |

Interface

      |

Provider

-------------
JVM Provider

Native Provider

CPP Provider


所以：

你的想法：

JVM 技术升级，直接把 JVM 文件夹抛出来。

这是正确方向。

但需要：

隔离接口。

八、自演化三原则
1. Replace

替换。

例如：

旧：

OldRenderer

换：

NewShaderRenderer

业务不知道。

2. Extend

扩展。

例如：

新增：

C++ Acceleration Provider

不影响：

Damage Domain。

3. Adapt

适配。

例如：

旧 Mod：

Forge Event

新系统：

V7 Event Bus

中间：

Adapter。

九、版本演化系统

大型框架必须记录：

不是：

代码版本。

而是：

能力版本。

例如：

Damage Capability

v1:
Basic Damage

v2:
Armor Ignore

v3:
Authority Support

v4:
Parallel Execution

状态迁移：

Old State

 ↓

Migration

 ↓

New State

类似：

游戏存档升级。

十、最终 V7 进化后的完整思想

现在：

                 Primordial Root

                       |

                  Authority

                       |

                 Scheduler

                       |

        ----------------------------

        Rule              Capability


                       |

                    State


                       |

                    Data



                       ↑

                       |

          Observer / Diagnostic / Evolution

十一、这一层对应你的“框架艺术”

你之前说：

把错综复杂真实世界整理成理想物理模拟世界。

那么 Observer 就是：

世界认识自己。
很好，现在进入 「通信与边界系统（Communication / Boundary Layer）」，这一层其实是前面所有研究的一个关键闭环。

如果把前面几层比作一个生命体：

根技术层 = 神经、骨骼、基础物理规则
Capability / 技术能力层 = 器官
Rule Engine = 思维规则
Scheduler / Pipeline = 行动流程
Observer / Diagnostic = 感知与自省
Meta / Definition = 基因与描述自身的语言

那么：

Communication / Boundary Layer，就是这个生命体与外界、与自身不同区域之间的“接口与边界法则”。

它决定：

谁可以和谁交流？
数据如何流动？
权限如何隔离？
不同平台如何连接？
不同版本如何兼容？
客户端、服务端、第三方模组如何交互？

这也是很多大型模组最容易腐化的地方。

一、第一物性原则重新推导通信层

从最基础：

存在
 ↓
状态
 ↓
变化
 ↓
作用
 ↓
传播

任何“作用”都必须跨越某个边界。

例如：

墨羽原初：

玩家
 ↓
手持物品
 ↓
Item实例
 ↓
能力模块
 ↓
规则系统
 ↓
执行管线
 ↓
世界状态改变

这里每一步其实都是一次通信。

如果没有边界：

所有东西直接互相调用：

A.class
    调 B.class
        调 C.class
            调 D.class

最终：

屎山

因为没有宇宙法则。

所以：

边界不是限制，而是创造秩序。

二、V7应该如何理解通信层

不是简单的：

Network

而应该拆成：

Communication Layer

├── Internal Communication
│
├── External Communication
│
├── Platform Boundary
│
├── Version Boundary
│
├── Data Boundary
│
└── Security Boundary
三、Internal Communication（内部通信）

这是模组内部模块之间交流。

例如：

攻击能力：

SwordAttack
      |
      ↓
DamageCapability
      |
      ↓
RuleEngine
      |
      ↓
ExecutionPipeline

不要：

DamageCapability
直接new
SwordAttack

应该：

CapabilityBus.request(
    DamageEvent
)

类似：

现实：

人不会直接控制心脏。

而是：

大脑 → 神经 → 器官

V7：

增加：

core.communication

例如：

communication

├── EventBus
├── MessageBus
├── RequestBus
├── Signal
└── Context
四、Event（事件）

这是Minecraft最熟悉的。

但是很多模组错误理解事件。

事件不是：

“发生事情通知一下”。

更高层：

事件是宇宙状态变化的描述。

例如：

不是：

EntityAttackEvent

而是：

ActionIntent

{
 actor:
 target:
 action:
 reason:
 timestamp:
}

然后规则决定：

是否允许。

例如：

木剑：

攻击意图产生

↓

Rule Engine检查

↓

Damage Pipeline

↓

Health State改变

神器：

攻击意图

↓

Intercept Capability

↓

替换规则

↓

执行特殊攻击

本质一样。

五、Request / Command 系统

事件是：

“发生了什么”

Command：

“请求做什么”

例如：

RemoveEntityCommand

DamageCommand

TeleportCommand

ModifyStateCommand

非常重要。

因为未来：

C++模块加入：

GPU计算模块加入：

AI模块加入：

它们只需要发送Command。

不用知道内部。

六、Context（上下文）

这是高级框架必须有的。

任何执行都需要环境。

例如：

伤害：

不是：

damage(entity,100)

而是：

DamageContext

{
 source
 target
 amount
 type
 rule
 priority
 cause
 timestamp
}

为什么？

因为未来：

你加入：

魔法伤害
灵魂伤害
代码伤害
原初伤害
很好，这一层其实是前面所有层的最终归宿。

前面的：

根技术层（JVM / Mixin / ASM / Native）
Capability（能力）
Rule Engine（规则）
Scheduler（执行）
Observer（观测）
Definition（描述）
Communication（通信）

它们全部是在回答：

“如何改变世界？”

而 World Model / State Universe Layer 回答的是：

“这个世界到底是什么？”

这也是从“工程设计”进入“框架哲学”的最后一步。

一、第一物性原则重新推导

从最基础：

01
 ↓
信息
 ↓
结构
 ↓
状态
 ↓
变化
 ↓
规律
 ↓
世界

代码世界也是一样。

一个程序不是因为有函数才存在。

它首先需要：

对象存在
状态存在
关系存在
变化存在

否则：

函数没有意义。

例如：

普通代码：

entity.setHealth(0);

看似简单。

但它隐藏了大量问题：

entity是什么？
health是什么？
谁拥有health？
改变是否合法？
改变后谁知道？
死亡意味着什么？
世界如何响应？

所以真正高级框架不会直接操作：

setHealth()

而是操作：

世界模型中的状态变化
二、Minecraft本身的问题

Minecraft其实已经有一个世界模型：

World

├── Block
├── Entity
├── Item
├── Player
├── Dimension
├── Capability
├── NBT
└── Event

但是它的问题：

它是为游戏运行设计的。

不是为大型模组生态设计的。

例如：

原版：

Entity
 |
Health
 |
Damage
 |
Death

这是线性的。

但是大型模组：

Entity

同时拥有：

生命状态
灵魂状态
护盾状态
等级状态
契约状态
诅咒状态
时间状态
空间状态
代码状态

原版模型开始不足。

三、V7世界模型的核心思想

不要把世界理解为：

一堆对象

而应该：

状态宇宙

结构：

World Model

├── Entity Model
│
├── State Model
│
├── Relation Model
│
├── Event Model
│
├── Time Model
│
├── Rule Model
│
└── History Model
四、Entity Model（存在模型）

回答：

“什么东西存在？”

例如：

墨羽原初。

不要只是：

ItemStack

而是：

Artifact Entity

{
 Identity
 Definition
 Capability
 State
 History
 Authority
}

例如：

一把神器：

墨羽原初

Identity:
  id = primordial:black_feather

Definition:
  神器定义

Capability:
  清除
  攻击
  时间
  着色

State:
  当前等级
  使用次数
  绑定者

History:
  曾经击杀
  曾经升级

这就是为什么以后可以：

“升级神器”。

因为神器不是一个Item。

而是一个存在。

五、State Model（状态模型）

这是最重要的一层。

前面研究：

状态、规则、数据、行为分离。

这里正式落地。

不要：

class Entity {

health;
damage;
speed;

}

因为全部混在一起。

应该：

Entity

拥有：

State Container

例如：

LivingEntity

State:

├── HealthState
├── CombatState
├── ShieldState
├── MagicState
├── TimeState
└── CustomState

每个状态：

State

{
 Value
 Version
 Owner
 Authority
 Revision
}

为什么需要Revision？

因为：

未来多人同步。

例如：

服务器：

HealthState revision 10

客户端：

revision 8

发现：

不同步。

重新同步。

六、Relation Model（关系模型）

这是很多模组缺失的。

世界不是对象组成。
好，那么到了最后一个隐藏层：

Evolution Layer（演化与自适应系统）

前面的所有层，其实都在回答：

世界是什么？ → World Model
状态是什么？ → State
规则如何决定结果？ → Rule Engine
行为如何执行？ → Execution Pipeline
如何被发现与修正？ → Observer / Diagnostic
如何描述和扩展？ → Meta / Schema
如何跨边界交流？ → Communication
如何组织能力？ → Capability / Domain

但是还有一个更高层的问题：

如果世界变化了，技术升级了，需求变化了，这个系统如何不被时代淘汰？

这就是 Evolution Layer。

一、Evolution Layer 的本质

从第一物性原则推导：

任何系统都有三个阶段：

诞生
 ↓
运行
 ↓
变化

普通代码只能处理前两个：

定义 → 执行

高级框架必须处理：

定义 → 执行 → 反馈 → 改变 → 新定义 → 新执行

所以：

Evolution Layer 不是让代码自己乱改代码，而是让系统具备“有规则的变化能力”。

二、Evolution Layer 在 V7 中的位置

重新画整个宇宙结构：

                 Evolution Layer
                       ▲
                       |
             Observer / Diagnostic
                       ▲
                       |
              World Model Layer
                       ▲
                       |
        --------------------------------
        |              |               |
   Rule Engine   Execution       Communication
        |              |
        |              |
   Capability      Provider
        |
        |
   Domain Feature

注意：

Evolution 不直接控制攻击、清除、Shader。

它控制：

能力如何诞生、替换、升级、退化、验证。

三、Evolution Layer 第一原则
不改变过去，而是替换未来

这是很多大型工程失败的原因。

错误：

旧代码
 ↓
不断修改
 ↓
越来越复杂
 ↓
屎山

正确：

旧版本 Capability

        ↓

Evolution Layer

        ↓

新版本 Capability

        ↓

切换 Provider

例如：

现在：

DamageProvider_V1

未来：

加入 C++ 高性能计算：

DamageProvider_V2_NATIVE

系统：

发现 V2
 ↓
验证
 ↓
测试
 ↓
切换
 ↓
保留V1作为fallback
四、Evolution Layer 的核心模块
1. Version System（版本演化）

所有东西必须有版本。

例如：

Capability:
    RemoveCapability

version:
    1.0

升级：

RemoveCapability

1.1

再升级：

RemoveCapability

2.0

系统知道：

当前运行：
Remove 1.1

可用：
Remove 2.0

状态：
compatible
2. Migration System（迁移系统）

最大的问题：

升级不是换代码。

升级意味着：

旧世界 → 新世界

例如：

旧：

HealthData

{
 hp:100
}

新：

HealthState

{
 current:100,
 max:100,
 revision:5
}

怎么办？

Migration：

Old State

      ↓

Migration Rule

      ↓

New State

所以：

玩家存档不会炸。

3. Capability Replacement

这是 V7 最重要的一点。

比如：

现在：

RemoveCapability

Provider:
 JVM Reflection

未来：

RemoveCapability

Provider:
 Unsafe

再未来：

RemoveCapability

Provider:
 Native C++

但是：

上层不用变化。

调用：

Remove.execute(target)

永远不变。

下面随便换。

这就是：

语义稳定，技术演化。

4. Experiment Layer（实验系统）

高级框架必须允许：

“不确定的新技术存在”。

例如：

开发：

DamageAlgorithm_V3

不能直接上线。

流程：

Candidate

 ↓

Sandbox

 ↓

Observer

 ↓

Compare

 ↓

Approve

 ↓

Release

类似：

游戏测试服。

五、自适应（Adaptive）

注意：

不是 AI 自己写代码。

而是：

根据环境调整策略。

例如：

发现：

Mixin冲突率增加

Evolution：

调整：

Intercept Provider Priority

2000
 ↓
1800

或者：

发现：

Shader性能下降

调整：

Particle Density

100%
 ↓
70%
六、Evolution 与你的“神器”思想对应

以墨羽原初为例：

现在：

墨羽原初

不是一堆技能。

它是一套：

Definition
+
Capability
+
Rule
+
Provider

未来：

加入：

星辰模式

墨羽原初 V2

不是重写武器。

而是：

新增：

Evolution Profile

改变：

Shader Provider

Damage Rule

Target Resolver

Execution Strategy

核心身份不变。

七、Evolution Layer 防止什么？
1. 防止技术锁死

例如：

现在：

Java

未来：

C++

甚至：

GPU Compute Shader

不会推倒重来。

2. 防止功能膨胀

没有 Evolution：

墨羽原初.java

10000行

有 Evolution：

Definition

+
Capability

+
Version
3. 防止老代码死亡

旧功能：

Legacy Provider

不会删除。

进入：

Archive

等待重新启用。

八、最终 V7 宇宙结构

经过所有层，现在完整模型：

                     Evolution
                         |
                  Observer System
                         |
                    Meta Schema
                         |
                   World Model
                         |
        --------------------------------
        |              |              |
     Rule          Execution     Communication
        |              |
        |              |
    Capability     Provider
        |
    Domain Feature
        |
    Item / Weapon / Entity

最底层：

Minecraft

只是：

World Provider。

九、最终得到的设计哲学

一句话：

不制造一个强大的模组，而制造一个可以不断诞生强大模组的世界。

这也是为什么之前研究 Tetra 有价值。

它真正厉害的不是某一个武器。

而是：

定义系统
模块组合
数据驱动
行为分离
可扩展结构

V7 如果吸收到这里，就不再只是“重构 Primordial”。

而是在建立：

一个 Minecraft Mod Framework。

下一步其实就应该回到现实：

把这些理想层映射回 r309：

哪些已有？
哪些缺失？
哪些应该作为 V7 根模块？
哪些只是未来 Evolution 插槽？

然后重新画 V7 最终目录树。

好，那么进行一轮回溯吧，同时最后再尊重的检查一轮前辈模组,看看是否成功的站在了巨人的肩膀之上，继承了所有
  

好，现在进行这一轮回溯审查（Retrospective Audit）。

这一步非常重要，因为大型架构最危险的不是“不够强”，而是：

在追求理想的时候，忘记了最初为什么出发，把已有优秀结构丢掉。

所以我们现在不继续扩张，而是回头检查：

V7 最初目标有没有丢？
r309 老代码中的优秀资产有没有被错误舍弃？
Tetra 等优秀前辈模组的精髓有没有真正吸收？
新的“世界模型”思想有没有脱离 Minecraft 实际？
最终架构是否真的进化，而不是重新造轮子？
一、回溯最初目标

最初 V7 不是为了创造一个抽象理论。

它来源于一个现实问题：

r309 已经成长为巨大复杂模组，功能强，但是结构开始接近极限，需要重新整理。

最初目标：

① 模块化

解决：

巨型类
功能纠缠
修改困难

例如：

旧：
PrimordialEndEvents
    |
    ├ 攻击
    ├ 渲染
    ├ 清除
    ├ 时停
    ├ 注册
    ├ 网络
    └ 调试

问题：

一个文件承担一个宇宙。

V7：

Feature

↓

Domain

↓

Capability

↓

Provider


保留了功能，但拆开。

✅ 继承。

二、回溯 r309 的核心资产

现在检查：

哪些东西不能丢。

1. Mixin / ASM / Agent 技术根

这是 r309 最核心的“力量来源”。

不是普通 Forge API。

包括：

Mixin
ASM
Transformer
JVMTI Agent
Reflection Bridge

以前：

它们混在一起。

V7：

重新定位：

foundation

├── mixin
├── asm
├── agent
├── reflection
└── native

它们变成：

技术根。

不是功能。

✅ 保留。

2. Remove / Clear 系统

这是 r309 最强资产之一。

旧：

EntityClearerService
DeepRemoval
KillTracker
RemovalService

里面其实包含：

删除
清理
记录
恢复
防护

以前混合。

V7：

拆：

Remove Capability

负责：
目标消失

↓

History
记录

↓

Cleanup Domain
善后


注意：

没有删除它。

只是让它回归正确位置。

✅ 保留并升华。

3. Health / Damage 系统

r309 最大的问题之一：

血量不是简单血量。

存在：

强制修改
锁血
覆盖
投影
读取
写入

以前容易形成：

Health God Class。

V7：

拆：

State

HealthState

↓

StateAccess

↓

Damage Domain

↓

Rule Engine


这其实比原版更接近大型游戏设计。

✅ 保留核心思想。

4. Shield / Protection

r309：

PrimordialShieldTransformer
IRunicShieldCapability

这里其实已经接近 V7。

因为它已经不是：

if伤害减少。

而是：

能力。

V7：

Shield Capability

Provider:
Mixin
ASM

Rule:
Protection


基本是自然演化。

✅ 完整继承。

三、检查 Tetra 学习成果

现在进入前辈审查。

这里不是复制 Tetra。

而是检查：

有没有吸收它的设计精神。

Tetra 第一核心：
① 数据驱动，而不是代码驱动

优秀模组：

不是：

每个武器一个class

而是：

Definition

+
Component

+
Rule


V7：

现在：

Definition Layer

Meta Schema

Capability

Feature

对应：

✅ 已吸收。

Tetra 第二核心：
组合，而不是继承

传统：

Sword
 |
FireSword
 |
IceFireSword
 |
UltimateSword

爆炸。

Tetra：

Component A

+
Component B

+
Component C

生成。

V7：

墨羽原初：

不是：

PrimordialBow.java

巨大类。

而：

Artifact Definition

+
Damage Capability

+
Remove Capability

+
Shader Capability

+
Evolution Profile


组合。

✅ 已吸收。

Tetra 第三核心：
状态和行为分离

这个我们已经深入。

最终：

State

描述是什么

Behavior

描述做什么

Rule

描述什么时候做


✅ 已吸收。

Tetra 第四核心：
可扩展性

Tetra为什么能长久？

因为新增内容：

不是修改核心。

而：

新增组件。

V7：

Evolution Layer。

目标：

未来：

C++

GPU

AI

新渲染

新物理

都是：

Provider替换。

✅ 已吸收。

四、检查是否过度理想化

这是最重要的一项。

因为架构艺术最大风险：

“设计天堂，无法落地。”

检查：

World Model 是否过度？

答案：

需要降级。

不能真的重造Minecraft世界。

所以：

V7不能：

替换Minecraft Entity。

正确：

Minecraft World

↓

Adapter

↓

V7 World Projection


也就是：

建立自己的世界认知层。

不是取代MC。

Evolution 是否过度？

同样。

不能让模组自己改自己。

应该：

版本管理

+
Provider替换

+
Schema迁移

+
实验系统


而不是：

AI自动修改代码。

五、最终V7进化后的真实定位

现在重新定义：

以前：

一个高度模块化的 Minecraft 模组。

现在：

一个建立在 Minecraft 上的可演化领域框架。

但仍然：

服务于模组。

不是脱离游戏创造操作系统。

六、最终架构草图（回溯版）
                    Evolution
                         |
                  Observer/Diagnostic
                         |
                    Meta Schema
                         |
                  World Projection
                         |
        ---------------------------------
        |              |                |
    Domain        Rule Engine     Execution
        |
    Capability
        |
    Provider
        |
Communication Boundary
        |
Platform Adapter
        |
Forge/Minecraft
        |
Technical Root
(JVM/Mixin/ASM/Native)
七、最终检查表
项目	是否继承
r309 技术力量	✅
Mixin/ASM 根技术	✅
Remove体系	✅
Health体系	✅
Shield体系	✅
数据驱动思想	✅
Tetra组合思想	✅
状态/行为分离	✅
规则系统	✅
执行流水线	✅
诊断系统	✅
扩展未来技术	✅
防止巨型类	✅
八、最终结论

这次回溯后，可以确认一个关键点：

V7不是推翻r309。

它更像：

r309
  |
  | 提炼核心力量
  |
V7 Framework
  |
  | 吸收优秀前辈思想
  |
未来模组生态

真正站在巨人的肩膀上，不是复制巨人的外形。

而是理解：

为什么巨人能站在那里。

Tetra的强大不是因为它有多少代码。

而是它建立了一套允许未来继续生长的秩序。

而现在V7追求的，也是同一个方向：

不是创造一个完成的神器，而是创造一个能不断诞生神器的世界。

下一步现实工作就应该进入：

V7最终目录树设计 + r309功能迁移地图。

也就是把这个“宇宙法则”真正落回文件夹和代码。

