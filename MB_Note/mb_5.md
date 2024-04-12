## 战场触发器

从本篇开始,可能需要你有一定基础,即看过我写过的基础教程 视频教程,或自己有对游戏脚本做过一定修改.

**我们不以源码,而以txt文件的角度来看战场文件.**

战场文件位置:```mission_templates.txt```

我们拉一条触发器来看看:

```python
-28.000000 0.000000 0.000000 1 2147483679 2 144115188075857435 1 5 2133 2 1224979098644774912 72057594037927936 2071 1 1224979098644774913 2072 1 1224979098644774914 2073 1 1224979098644774915 1 5 936748722493064015 1224979098644774912 1224979098644774913 1224979098644774914 1224979098644774915 
```

解析:

```<正数:触发间隔;负数:触发类型> <延迟多少秒执行> <执行后冷却时间> <判断条件的代码行数> <判断条件> <执行代码的行数> <执行代码>```

例如我们想写一个每0.1秒触发1次的触发器可以这样写:

```python
0.100000 0.000000 0.000000 0 0 
```

由于没有判断条和执行的部分 所以填两个0

那么当第一个参数为负数时,我们有哪些触发类型呢:

```python
#可能不全,具体请于头文件中查阅
# Trigger Param 1: player_no
ti_before_mission_start  = -19.0 #can only be used in module_mission_templates triggers
ti_after_mission_start   = -20.0 #can only be used in module_mission_templates triggers
ti_tab_pressed           = -21.0 #can only be used in module_mission_templates triggers
ti_inventory_key_pressed = -22.0 #can only be used in module_mission_templates triggers
ti_escape_pressed        = -23.0 #can only be used in module_mission_templates triggers
ti_battle_window_opened  = -24.0 #can only be used in module_mission_templates triggers
ti_on_agent_spawn        = -25.0 #can only be used in module_mission_templates triggers
ti_on_agent_killed_or_wounded = -26.0 #can only be used in module_mission_templates triggers
ti_on_agent_knocked_down = -27.0 #can only be used in module_mission_templates triggers
ti_on_agent_hit          = -28.0 #can only be used in module_mission_templates triggers
# Trigger Param 1: damage inflicted agent_id - 受害者
# Trigger Param 2: damage dealer agent_id - 攻击者
# Trigger Param 3: inflicted damage - 造成的伤害
# Register 0: damage dealer item_id - 使用的武器
# Position Register 0: position of the blow
#                      rotation gives the direction of the blow
# Trigger result: if returned result is greater than or equal to zero, inflicted damage is set to the value specified by the module.

ti_on_player_exit        = -29.0 #can only be used in module_mission_templates triggers
# Trigger Param 1: player_id

ti_on_leave_area         = -30.0 #can only be used in module_mission_templates triggers
ti_on_item_picked_up     = -53.0 #can only be used in module_mission_templates triggers
# Trigger Param 1: agent id
# Trigger Param 2: item id
# Trigger Param 3: scene prop id (will be deleted after this trigger)
ti_on_item_dropped       = -54.0 #can only be used in module_mission_templates triggers
# Trigger Param 1: agent id
# Trigger Param 2: item id
# Trigger Param 3: scene prop id
ti_on_agent_mount        = -55.0 #can only be used in module_mission_templates triggers
# Trigger Param 1: agent id
# Trigger Param 2: horse agent id
ti_on_agent_dismount     = -56.0 #can only be used in module_mission_templates triggers
# Trigger Param 1: agent id
# Trigger Param 2: horse agent id
ti_on_item_wielded       = -57.0 #can only be used in module_mission_templates triggers
# Trigger Param 1: agent id
# Trigger Param 2: item id
ti_on_item_unwielded     = -58.0 #can only be used in module_mission_templates triggers
# Trigger Param 1: agent id
# Trigger Param 2: item id
```

例如我们想写一个进入战斗后任何agent受到伤害时就触发的触发器可以这样写:

```python
-28.000000 0.000000 0.000000 0 0 
```

目前判断条件和执行代码都为空白,我们来写点什么,比如当某agent装备某件物品例如短链甲```itm_haubergeon```时,受到伤害减少一半
我们把判断条件留白,判断部分写在执行代码中:

```python
# 注意到ti_on_agent_hit有几个可以用的参数,我们获取一下
(assign, ":itm_id", reg0),
(store_trigger_param_1, ":victim"),
(store_trigger_param_2, ":ataacker"),
(store_trigger_param_3, ":deal_dmg"),
# 
(assign, ":dmg", ":deal_dmg"), #备份一下伤害值

(agent_is_alive, ":victim"),
(agent_is_human, ":victim"),

(agent_has_item_equipped, ":victim", "itm_haubergeon"),
(val_div,":dmg",2),

# ti_on_agent_hit最后还有个输出值,告诉系统这1次攻击的最后伤害是多少
(set_trigger_result, ":dmg"), 

```

例如我想把这个触发器添加到野战场景中,可以在这里添加:

```python
mst_lead_charge lead_charge 65538  8
You_lead_your_men_to_battle.
...
...
...
...

#以上是野战场景
#在最后一行去添加这个触发器 注意触发器总数变化
```

但是如果我们需要在其他场景也触发的话,那么每一个你需要的场景都要去加这个触发器,这里就涉及到了代码维护问题.如果你想进行更改,那么每一处都要进行更改,这非常麻烦.

有没有什么办法可以解决这个问题呢?

其实只需要在```scripts.txt```里把函数封装好,然后在战场触发器中调用这个函数即可.例如刚才写的触发器:

```python
(assign, ":itm_id", reg0),
(store_trigger_param_1, ":victim"),
(store_trigger_param_2, ":ataacker"),
(store_trigger_param_3, ":deal_dmg"),

(call_script,"script_b1_test",":itm_id",":victim",":ataacker",":deal_dmg"),#调用函数,并传入参数
```

执行代码我们只需要以上这段就ok了,然后我们来封装```b1_test```这个函数:

```python
(store_script_param, ":item_id", 1),
(store_script_param, ":victim", 2),
(store_script_param, ":attacker", 3),
(store_script_param, ":get_dmg", 4),
(assign, ":dmg", ":deal_dmg"), #备份一下伤害值

#-----------这段是要实现的功能-----------------
(agent_is_alive, ":victim"),
(agent_is_human, ":victim"),
(agent_has_item_equipped, ":victim", "itm_haubergeon"),
(val_div,":dmg",2),
#------------------------------------------

(set_trigger_result, ":dmg"), #不要忘了设置输出
```

编译好放入```scripts.txt```中即可.

```python
b1_test -1
 10 23 2 1224979098644774912 1 23 2 1224979098644774913 2 23 2 1224979098644774914 3 23 2 1224979098644774915 4 2133 2 1224979098644774916 1224979098644774917 1702 1 1224979098644774913 1704 1 1224979098644774913 1729 2 1224979098644774913 288230376151712151 2108 2 1224979098644774916 2 2075 1 1224979098644774916 
```

我们可以给常用的几个战场触发器例如刚才的这个受击触发还有agent死亡或击晕触发器;agent出生触发器等等,都一一封装好,以后需要进行修改时,只需要修改封装好的函数即可,不需要去修改每一个触发器.






