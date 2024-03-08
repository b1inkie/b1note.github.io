## 简单功能合集

如题

## 装备某件物品时增加属性点

><i style="color:aqua;">以下代码丢到简单触发器中,设置0延时</i>

```Go
//配置部分(必填)
(assign,":max",63) //声明属性的上限
(assign,":which_arm","itm_godz_zlzt_low") //哪件装备
(assign,":which_attribute",ca_agility) //增加哪个属性 ca_agility是敏捷
(assign,":buff",70) //增加多少点
(assign,":which_slot_to_show_equipped", 2024),//用trp的2024号slot表示是否装备该物品 1是 0否
(assign,":which_slot_to_store_value_of_real_buff", 2025),//用trp的2025号slot存储实际获得的属性加成
//主体
(troop_get_slot, ":equipped", "trp_player", ":which_slot_to_show_equipped"), 
(try_begin),
    (neg|eq, ":equipped", 1),

    (troop_has_item_equipped, "trp_player", ":which_arm"),
    (troop_set_slot, "trp_player", ":which_slot_to_show_equipped", 1),

    (store_attribute_level, ":agility_cur", "trp_player", ":which_attribute"),
    (store_add,":agility_after", ":agility_cur", ":buff"),

    (assign,":real_buff",":buff")
    (try_begin),
        (ge, ":agility_after", ":max"),
        (store_sub,":real_buff", ":max", ":agility_cur"),
    (else_try),
        (store_sub,":real_buff", ":agility_after", ":agility_cur"),
    (try_end),

    (troop_set_slot, "trp_player", ":which_slot_to_store_value_of_real_buff", ":real_buff"),
    (troop_raise_attribute, "trp_player", ":which_attribute", ":real_buff"),
(else_try),
    (eq, ":equipped", 1),
    (neg|troop_has_item_equipped, "trp_player", ":which_arm"),
    (troop_set_slot, "trp_player", ":which_slot_to_show_equipped", 0),

    (troop_get_slot, ":get_real_buff", "trp_player", ":which_slot_to_store_value_of_real_buff"),

    (store_attribute_level, ":agility_now", "trp_player", ":which_attribute"),
    (store_sub,":get_real_buff", 0, ":get_real_buff"),

    (troop_raise_attribute, "trp_player", ":which_attribute", ":get_real_buff"),
    
(try_end),
```

## 添加书籍

><i style="color:aqua;">添加一本书籍,当玩家拥有时,增加多少点技能</i>

以潘德395为例,打开scripts,找到`game_get_skill_modifier_for_troop`这一行,拉出来反编译:

```python
(store_script_param, ":var_0", 1),
(store_script_param, ":var_1", 2),
(assign, ":var_2", 0),
(try_begin),
    (eq, ":var_1", 11),
    (call_script, "script_get_troop_item_amount", ":var_0", "itm_book_wound_treatment_reference"),
    (gt, reg0, 0),
    (val_add, ":var_2", 1),
(else_try),
    (eq, ":var_1", 17),
    (call_script, "script_get_troop_item_amount", ":var_0", "itm_book_training_reference"),
    (gt, reg0, 0),
    (val_add, ":var_2", 2),
(else_try),
    (eq, ":var_1", 10),
    (call_script, "script_get_troop_item_amount", ":var_0", "itm_book_surgery_reference"),
    (gt, reg0, 0),
    (val_add, ":var_2", 1),
(try_end),
(set_trigger_result, ":var_2"),
```

仿着写就可以了

```python
(store_script_param, ":var_0", 1),
(store_script_param, ":var_1", 2),
(assign, ":var_2", 0),
(try_begin),
    (eq, ":var_1", 11),
    (call_script, "script_get_troop_item_amount", ":var_0", "itm_book_wound_treatment_reference"),
    (gt, reg0, 0),
    (val_add, ":var_2", 1),
(else_try),
    (eq, ":var_1", 17),
    (call_script, "script_get_troop_item_amount", ":var_0", "itm_book_training_reference"),
    (gt, reg0, 0),
    (val_add, ":var_2", 2),
(else_try),
    (eq, ":var_1", 10),
    (call_script, "script_get_troop_item_amount", ":var_0", "itm_book_surgery_reference"),
    (gt, reg0, 0),
    (val_add, ":var_2", 1),
//-------------------------仿写部分
(else_try),
    (eq, ":var_1", 17), //这里17是教练技能的ID,自行尝试其他技能

    (call_script, "script_get_troop_item_amount", ":var_0", "itm_book_id"), //填物品ID
    (gt, reg0, 0),
    //如果你的MOD没有封装get_troop_item_amount这个函数,请自行判断玩家背包是否有一个及以上该物品

    (val_add, ":var_2", 5), //具体提升多少点,这里5点教练
//--------------------------
(try_end),
(set_trigger_result, ":var_2"),
```

><i style="color:aqua;">编译成txt,覆盖回去即可,注意行数的变化</i>
