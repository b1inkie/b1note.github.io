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

## 全自动武器

原理就是检测攻击键是否按住,然后```add_missile```,射速由战场触发器的检测间隔控制.

源码 :

```python
    (game_key_is_down,gk_attack),
    (get_player_agent_no,":player"),
    (agent_get_wielded_item,":weapon_cur",":player",0),

    #(this_or_next|eq,":weapon_cur","itm_bow"), #设置需要连射的武器id
    #(this_or_next|eq,":weapon_cur","itm_pistol"), #设置需要连射的武器id
    #...
    (eq,":weapon_cur","itm_hunting_bow"), #设置需要连射的武器id

    (agent_get_ammo,":ammo",":player", 1),
    (gt,":ammo",0),

    (assign,":missile_id",0),
    (try_for_range, ":item_carry", 0, 4),
      (agent_get_item_slot, ":slot_item", ":player", ":item_carry"),
      (neq,":slot_item",-1)
      (item_get_type, ":slot_item_type", ":slot_item"),
      (item_get_type, ":weapon_cur_type", ":weapon_cur"),
      (try_begin),
        (eq,":weapon_cur_type",itp_type_bow),
        (eq, ":slot_item_type", itp_type_arrows), 
        (assign,":missile_id",":slot_item"),
      (else_try),
        (eq,":weapon_cur_type",itp_type_crossbow),
        (eq, ":slot_item_type", itp_type_bolts),
        (assign,":missile_id",":slot_item"),
      (else_try),
        (eq,":weapon_cur_type",itp_type_pistol),
        (eq, ":slot_item_type", itp_type_bullets),
        (assign,":missile_id",":slot_item"),
      (else_try),
        (eq,":weapon_cur_type",itp_type_musket),
        (eq, ":slot_item_type", itp_type_bullets),
        (assign,":missile_id",":slot_item"),
      (try_end),
    (try_end),

    (val_sub,":ammo",1),
    (agent_set_ammo,":player",":missile_id",":ammo"),

    (agent_get_horse, ":horse", ":player"),
    (agent_get_look_position, pos3, ":player"),
    (agent_get_position, pos4, ":player"),
    (position_copy_rotation, pos4, pos3),
    (try_begin),
        (eq, ":horse", -1),
        (position_move_z, pos4, 170),
    (else_try),
        (position_move_z, pos4, 270),
    (try_end),
    (set_fixed_point_multiplier, 100),
    (store_random_in_range, ":var_3", -35, 100),
    (position_rotate_x_floating, pos4, 9000),
    (position_rotate_y_floating, pos4, ":var_3"),
    (position_rotate_x_floating, pos4, -9000),
    (store_random_in_range, ":var_4", -35, 100),
    (position_rotate_x_floating, pos4, ":var_4"),

    (add_missile, ":player", 4, 13500, ":weapon_cur", 0, ":missile_id", 0),
```

```mission_templates.txt```中搜索```lead_your_men_to_battle```

可以看到如下代码:

```python
You_lead_your_men_to_battle. 

4 1 4160 0 16 12 0  
0 4160 0 16 0 0  
4 8320 0 16 12 0  
4 8320 0 16 0 0  
22 #这里的数字表示触发器总数量
0.200000 0.000000 0.000000  0  42 72 1 6 1700 1 1224979098644774912 1726 3 1224979098644774913 1224979098644774912 0 31 2 1224979098644774913 288230376151712292 1727 3 1224979098644774914 1224979098644774912 1 32 2 1224979098644774914 0 2133 2 1224979098644774915 0 6 3 1224979098644774916 0 4 1804 3 1224979098644774917 1224979098644774912 1224979098644774916 2147483679 2 1224979098644774917 -1 1570 2 1224979098644774918 1224979098644774917 1570 2 1224979098644774919 1224979098644774913 4 0 31 2 1224979098644774919 8 31 2 1224979098644774918 5 2133 2 1224979098644774915 1224979098644774917 5 0 31 2 1224979098644774919 9 31 2 1224979098644774918 6 2133 2 1224979098644774915 1224979098644774917 3 0 3 0 2106 2 1224979098644774914 1 1776 3 1224979098644774912 1224979098644774915 1224979098644774914 1714 2 1224979098644774920 1224979098644774912 1709 2 3 1224979098644774912 1710 2 4 1224979098644774912 718 2 4 3 4 0 31 2 1224979098644774920 -1 722 2 4 170 5 0 722 2 4 270 3 0 2124 1 100 2136 3 1224979098644774921 -35 100 738 2 4 9000 739 2 4 1224979098644774921 738 2 4 -9000 2136 3 1224979098644774922 -35 100 738 2 4 1224979098644774922 1829 7 1224979098644774912 4 13500 1224979098644774913 0 1224979098644774915 0 
-25.000000 0.000000 0.000000  0  15 2071 1 1224979098644774912 1 2 936748722493063549 1224979098644774912 2133 2 1224979098644774913 5000 1718 2 1224979098644774914 1224979098644774912 2171 2 1224979098644774915 1224979098644774914 2107 2 1224979098644774915 35 2105 2 1224979098644774913 1224979098644774915 2136 3 1224979098644774916 0 3000 2105 2 1224979098644774913 1224979098644774916 1716 2 1224979098644774917 1224979098644774912 1671 2 1224979098644774918 1224979098644774917 2121 3 1224979098644774919 1224979098644774918 70 2107 2 1224979098644774919 30 2105 2 1224979098644774913 1224979098644774919 505 3 1224979098644774912 16 1224979098644774913 
-25.000000 0.000000 0.000000  0  3 2071 1 1224979098644774912 1718 2 1224979098644774913 1224979098644774912 1 4 936748722493063580 1729382256910270468 1224979098644774912 1224979098644774913 

```

将上述代码注释行的数字加1,例如显示是```21```,则改为```22```,即添加一个触发器
接着将源码转成txt,粘贴到```22```下面,
并在前面加上```0.200000 0.000000 0.000000  0 行数```,
0.200000表示1秒钟射5次;0.000000表示延迟检测;0.000000表示间隔;0表示触发条件为空;行数在用MBCE转好后有显示,填上即可,

战场触发器中有多个```lead_your_men_to_battle```,请全部修改,不要漏掉.
