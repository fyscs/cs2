# Stripper

在地图加载时直接对地图实体Lump进行编辑操作.  
只有**脑残**才会简称为***STP***

> [!CAUTION]
> 该功能为高风险操作, 一旦发生错误, 服务器会立即崩溃.

## 该功能需要 ***ModSharp*** 客户端/服务端方可运行

- ``entity_lump_list``: 查看当前地图的Lump

配置在每次``GameInit``(也就是换图)时**整体重新加载**, 改完配置必须**换图**才生效.  
加载过程中控制台会用青色逐行打印读到的 ``world`` / ``lump`` 和文件相对路径, 用来确认文件名有没有写对.

## 配置文件

一共三种配置文件, 作用域从大到小:

| 文件 | 作用域 |
| --- | --- |
| ``sharp/stripper/global.jsonc`` | 所有``SpawnGroup``的**每一个**Lump |
| ``sharp/stripper/global_default.jsonc`` | 仅**主地图**``world``的``default_ents``这一个Lump |
| ``sharp/stripper/maps/{map}/...`` | 指定的``world`` + ``lump`` |

> [!WARNING]
> ``global.jsonc``里的``add``会在**每一个Lump各创建一次**实体 —— 一张图几十个Lump就是几十份重复实体.  
> 要给全图加实体请写在``global_default.jsonc``里.

> [!NOTE]
> ``global_default.jsonc``的判定是"``lump``名为``default_ents`` **且** ``world``名等于地图名",
> 所以prefab / skybox等**子世界**里的``default_ents``不吃这份配置.

### 地图配置的路径规则

``sharp/stripper/maps/{map}/`` 会被**递归**扫描, 只认``.jsonc``扩展名, 其它文件直接跳过.

- 文件**直接放在地图目录下** → ``world``取**地图名**, ``lump``取**文件名(去掉扩展名)**
  - 例如 ``ze_shroomforest2_i`` 中有 ``item_shroom_template2_50.vents_c``
  - 那么应该创建配置文件 ``sharp/stripper/maps/ze_shroomforest2_i/item_shroom_template2_50.jsonc``
  - ``default_ents.vents_c`` 应使用 ``sharp/stripper/maps/ze_shroomforest2_i/default_ents.jsonc``
- 文件放在**子目录**下 → ``world``取**子目录名**, 用于prefab / skybox等子世界
  - 例如 ``ze_hr_dead_center`` 里的 ``c1m3_mall`` 子世界
  - 那么应该创建配置文件 ``sharp/stripper/maps/ze_hr_dead_center/c1m3_mall/default_ents.jsonc``

> [!IMPORTANT]
> ``Lump``/``SpawnGroup``的匹配**不区分**大小写字母.  
> 同一个``world`` + ``lump``被两个文件同时命中时, 后加载的那份会**整份覆盖**前一份, 且扫描顺序不保证 —— 不要这么写.

### 文件结构

```jsonc
{
  "add": [
    // 往下查看
  ],
  "modify": [
    // 往下查看
  ],
  "remove": [
    // 往下查看
  ]
}
```

### 解析规则

> [!CAUTION]
> 违反下面任何一条, 该文件解析失败, 并且**本次加载的全部配置**(该地图所有文件 + ``global.jsonc`` + ``global_default.jsonc``)
> 会被**整体丢弃**, 只在控制台留一条 ``Provider failed to parse`` 错误. 一个文件写错 = 整张图的Stripper全废.

- ``.jsonc``允许``//``注释, 但**不允许尾逗号**
- 除``delay`` / ``limit``外, **所有值都必须是JSON字符串**: 写 ``"health": 100`` 会炸, 要写 ``"health": "100"``
- ``param`` **不能写``null``**: 要么省略这个键, 要么写 ``""``
- ``delay`` / ``limit`` 必须是JSON数字, 不能写成字符串 ``"0.5"``
- 顶层只读 ``add`` / ``modify`` / ``remove`` 三个块, 其余键(包括拼错的``removes``)**静默忽略**, 不报错也不生效

## 执行顺序

1. 对同一个Lump, 先执行**地图配置**, 再执行``global.jsonc``, 最后(仅主地图``default_ents``)执行``global_default.jsonc``
2. 同一份文件内, 顺序固定为 ``remove`` → ``add`` → ``modify``, **与JSON里的书写顺序无关**
   - 所以本文件``add``出来的实体**会**被本文件的``modify``匹配到
   - 本文件的``remove``**删不掉**本文件``add``的实体
3. ``modify``内部顺序固定为 ``delete`` → ``insert``

## 功能说明

### 匹配规则

- 键名与值的比较**一律不区分大小写**
- 实体名(``targetname``/``classname``), IO名(``output``), IO参数(``param``)允许使用通配符匹配, 通配符仅允许于最后一个字符
  - 例如 ``"targetname": "fys_*"`` 匹配所有targetname以``fys_``开头的实体, 且包含名为``fys_``的实体
  - ``modify``的``delete``块里**任意键**都支持通配符
  - 其余位置(``input`` / ``target`` / ``hammerUniqueId`` 以及``add``/``insert``写入的值)里的``*``是**普通字符**, 不是通配符
  - 单独一个 ``"*"`` 表示"该键存在即匹配", 不看值
- 匹配是**字符串比较**, 非字符串的KeyValues会先转成字符串再比:
  整数 → ``100``, 布尔 → ``true``/``false``, 浮点 → ``1.000000``(固定6位小数), 向量/数组 → ``<KV3_TYPE_ARRAY>``(即匹配不上)
- ``match``里的多个条件是**与**关系, 全中才算匹配
- ``connections``数组里**每一条**都是独立条件, 每条各自要求该实体里"**存在至少一条**IO命中"

> [!CAUTION]
> ``match``写成空对象``{}``, 或者``remove``里放一个空对象``{}``, 会匹配到**该Lump的所有实体**.
> ``remove``的空对象等于删光整个Lump.

- 实体KeyValues都是成对的键值出现
- ``targetname``使用的是vpk中编译好的值, 不一定与vmap匹配, 具体值请使用Source2View查看
  - 例如: 非templdate实体``[PR#]fys_gs_3``  
  - 例如: templdate实体``[PR#]fys_vs_3&0000``  

```jsonc
"classname": "logic_auto"
```

- 实体IO通常为``[]``数组, 里面每个Object代表一个IO

```jsonc
[
  {
    "output": "OnMapSpawn",         // Output名称, 也就是触发的事件, 支持通配符
    "target": "fys_global_command", // 目标实体, 为targetname或 !activator/!caller/!self
    "input":  "Command",            // 目标实体要执行的Input, **不可为空**, 具体参阅VDC或FGD
    "param":  "say 哈哈, 你妈",      // Input附带的参数, 支持通配符, 可省略或写 "", **不可写null**
    "delay":  0.5,                  // 触发延迟, 不填则为0, **不可为负数**
    "limit":  -1                    // 触发次数限制, 默认为-1, 不填也为-1, **不可为0或其他负值**
  }
]
```

- **匹配用**(``match`` / ``remove`` / ``modify.delete``)时: 省略的字段视为**通配**;
  ``delay`` / ``limit`` 一旦写了就必须**完全相等**才算命中
- **写入用**(``add`` / ``modify.insert``)时: ``output`` / ``target`` / ``input`` **三者都不能为空**,
  缺任何一个这条IO会被打WARN跳过, 但实体本身照样会被建出来 / 修改照样会生效

### 新增实体

实体块应该放入``add``数组中

> [!CAUTION]
> ``add``的实体**必须有``classname``**, 缺了会崩服.

```jsonc
{
  // 实体 KeyValues
  "classname": "logic_auto",
  "targetname": "auto_load_config",
  "hammerUniqueId": "hh, 你妈",

  // 实体 IO
  "connections": [
    {
      "output": "OnMapSpawn",         // Output名称, 也就是触发的事件
      "target": "fys_global_command", // 目标实体, 为targetname或 !activator/!caller/!self
      "input":  "Command",            // 目标实体要执行的Input, **不可为空**, 具体参阅VDC或FGD
      "param":  "say 哈哈, 你妈",      // Input附带的参数, 可省略或写 ""
      "delay":  0.5,                  // 触发延迟, 不填则为0, **不可为负数**
      "limit":  -1                    // 触发次数限制, 默认为-1, 不填也为-1, **不可为0或其他负值**
    }
  ]
}
```

### 修改实体

实体块应该放入``modify``数组中

```jsonc
{
  "match": {
    // 实体 KeyValues 匹配可以为任意 键值, 最少要有一项
    "classname":  "prop_dynamic",    // 匹配class
    "targetname": "mg_football_*",   // 匹配targetname以 'mg_football_' 开头
    // 实体IO匹配
    "connections": [
      {
        // connection中最少1项 'output'/'target'/'input'/'param' 其中之一
        "output": "OnBreak"
      }
    ]
  },
  "delete": {
    // 删除 KeyValues, 键存在**且值匹配**才会删, 值支持通配符
    "model": "models/mg_football/ball.vmdl", // 删除模型
    // 删除 IO
    "connections": [
      {
        // 删除所有'OnBreak'的IO
        "output": "OnBreak"
      },
      {
        // 删除所有'OnHealthChanged'中, 目标为 '!activator' 的IO
        "output": "OnHealthChanged",
        "target": "!activator"
      }
    ]
  },
  "insert": {
    // 插入 KeyValues, 键已存在时**直接覆盖**
    "model": "models/fys/football.vmdl", // 插入新的模型
    "connections": [
      {
        "output": "OnBreak",
        "target": "!activator",
        "input" : "SetHealth",
        "param" : "100",
        "delay" : 1.0,
        "limit" : 1
      }
    ]
  }
}
```

> [!TIP]
> ``insert``的KeyValues是"没有就建, 有就覆盖", 所以**改参数只写``insert``就够了**, 不用先``delete``再``insert``.
> 只有"不确定原值是多少, 但只想删掉某个特定值"时才需要``delete``, 这时可以用 ``"model": "*"`` 表示"不管什么值都删".

### 删除实体

实体块应该放入``remove``数组中, 匹配语法与``modify``的``match``完全一致(同样支持``connections``)

```jsonc
{
  // 删除所有'prop_ragdoll'
  "classname": "prop_ragdoll"
}
```

## 不支持的写法

- **没有``replace``块**: 源码里已被关闭, 写了不报错也不生效. 要改IO请用``delete`` + ``insert``
- **``connections``里的``targettype``无效**: 能解析但不会被使用,
  ``add``/``insert``写出去的IO目标类型固定为``ENTITY_IO_TARGET_ENTITYNAME_OR_CLASSNAME``
- **匹配不了数组/表类型的KeyValues**: 这类值转成字符串后是``<KV3_TYPE_ARRAY>`` / ``<KV3_TYPE_TABLE>``, 只能匹配到这串字面量.
  要按``origin``之类的字段筛实体前, 先用`VpkEntities.exe`确认它在vents里到底是字符串还是向量

## 有用的链接

- [Source2Viewer (VRF)](https://valveresourceformat.github.io/)
- [List of entities (Source 2)](https://developer.valvesoftware.com/wiki/List_of_entities_(Source_2))
- [Valve Developer Community](https://developer.valvesoftware.com/wiki)
