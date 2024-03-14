# Stripper

在地图加载时直接对地图实体Lump进行编辑操作.  
只有**脑残**才会简称为***STP***

> [!CAUTION]
> 该功能为高风险操作, 一旦发生错误, 服务器会立即崩溃.

## 该功能需要 ***ModSharp*** 客户端/服务端方可运行

- ``ms_list_strippers``: 查看当前已经加载的stripper
- ``entity_lump_list``: 查看当前地图的Lump

## 配置文件

- 所有``SpawnGroup``都将加载一个全局配置文件``sharp/stripper/global.jsonc``
- 针对特定的``SpawnGroup``需要遵循路径 ``sharp/stripper/maps/{map}/{path}.jsonc``
  - 例如 ``ze_shroomforest2_i`` 中有 ``item_shroom_template2_50.vents_c``
  - 那么应该创建配置文件 ``sharp/stripper/maps/ze_shroomforest2_i/item_shroom_template2_50.jsonc``
  - ``default_ents.vents_c`` 应使用 ``sharp/stripper/maps/ze_shroomforest2_i/default_ents.jsonc``
- 文件结构如下

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

## 功能说明

- 实体名(``targetname``/``classname``), IO名(``output``), IO参数(``param``)允许使用通配符匹配, 通配符仅允许于最后一个字符
  - 例如 ``"targetname" "fys_*"`` 匹配所有targetname以``fys_``开头的实体, 且包含名为``fys_``的实体
- 实体KeyValues都是成对的键值出现

```jsonc
"classname": "logic_auto"
```

- 实体IO通常为``[]``数组, 里面每个Object代表一个IO

```jsonc
{
  {
    "output": "OnMapSpawn",         // Output名称, 也就是触发的事件
    "target": "fys_global_command", // 目标实体, 为targetname或 !activator/!caller/!self
    "input":  "Command",            // 目标实体要执行的Input, **不可为空**, 具体参阅VDC或FGD
    "param":  "say 哈哈, 你妈",      // Input附带的参数, 可为空, null 或 ""
    "delay":  0.5,                  // 触发延迟, 不填则为0, **不可为负数**
    "limit":  -1                    // 触发次数限制, 默认为-1, 不填也为-1, **不可为0或其他负值**
  }
}
```

### 新增实体

实体块应该放入``add``数组中

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
      "param":  "say 哈哈, 你妈",      // Input附带的参数, 可为空, null 或 ""
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
    // 删除 KeyValues
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
    // 插入 KeyValues
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

### 删除实体

实体块应该放入``remove``数组中

```jsonc
{
  // 删除所有'prop_ragdoll'
  "classname": "prop_ragdoll"
}
```

## 有用的链接

- [List of entities (Source 2)](https://developer.valvesoftware.com/wiki/List_of_entities_(Source_2))
- [Valve Developer Community](https://developer.valvesoftware.com/wiki)
