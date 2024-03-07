## 简单功能合集

如题

## 装备某件物品时增加属性点

```Go

以下代码丢到简单触发器中,设置0延时

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