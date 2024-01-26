# 自定义实体IO

## ``AddOutput``  / ``CustomAddOutput`` / ``AddCustomOutput``

***所有实体可用***  
***``AddOutput``需要参数``ms_entity_io_addoutput`` ``true``***

- ``targetname name``: ``name`` -> **string**
- ``origin x y z``:  ``x``, ``y``, ``z`` -> **float**或 **int**
- ``angles x y z``:  ``x``, ``y``, ``z`` -> **float**或 **int**
- ``max_health value``: ``value`` -> **int**
- ``health value``: ``value`` -> **int**
- ``movetype type``: ``type`` -> **int**
- ``EntityTemplate name``: ``name`` -> **string**

## AddEffects / RemoveEffects

***BaseModelEntity***

- ``flag``: ``flag`` -> **int**

## SetName

***所有实体可用***

- ``name``: ``name`` -> **string**

## SetGravity

***BasePlayerPawn***

- ``gravity``: ``gravity`` -> **float**

## SetMaxHealth

***所有实体可用***

- ``maxHealth``: ``maxHealth`` -> **int**

## SetModel

***BaseModelEntity***

- ``model``: ``model`` -> **string**

## SetAbsAngles

***所有实体可用***

- ``x y z``:  ``x``, ``y``, ``z`` -> **float**或 **int**

## SetAbsOrigin

***所有实体可用***

- ``x y z``:  ``x``, ``y``, ``z`` -> **float**或 **int**

## SetMoveType

***所有实体可用***

- ``type``: ``type`` -> **int**
