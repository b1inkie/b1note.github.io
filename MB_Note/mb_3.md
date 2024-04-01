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
        #若是手枪或火枪 可以在此处添加烟雾和枪声
      (else_try),
        (eq,":weapon_cur_type",itp_type_musket),
        (eq, ":slot_item_type", itp_type_bullets),
        (assign,":missile_id",":slot_item"),
        #若是手枪或火枪 可以在此处添加烟雾和枪声
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

## 自爆步兵

进场后提高移速,贴近敌人后自爆

```python
(assign,":blow_dmg",1000), #设置自爆伤害
(assign,":blow_trpid","trp_rhodok_blow"), #设置自爆兵种ID

(assign, ":distance_closest", 1000),
(try_for_agents, ":blow_agent"),
    (agent_is_alive, ":blow_agent"),
    (agent_is_human, ":blow_agent"),
    (agent_get_troop_id, ":tar_troop", ":blow_agent"),
    (eq, ":tar_troop", ":blow_trpid"),
    (agent_get_team, ":blow_team", ":blow_agent"), 
    (agent_get_position,pos2,":blow_agent"),
    (agent_set_speed_modifier, ":blow_agent", 300),

    (assign, ":blowed", 0),

    (try_for_agents, ":enemies"),
        (agent_is_alive, ":enemies"),
        (agent_is_human, ":enemies"),

        (agent_get_position, pos3, ":enemies"),
        (agent_get_team, ":enemies_team", ":enemies"),
        (teams_are_enemies, ":blow_team", ":enemies_team"),

        (get_distance_between_positions, ":distance_abs", pos2, pos3),
        (lt, ":distance_abs", ":distance_closest"),
        (agent_deliver_damage_to_agent, ":blow_agent", ":enemies", ":blow_dmg"),
        (assign, ":blowed", 1),
    (try_end),

    (eq, ":blowed", 1)
    (agent_play_sound, ":blow_agent", "snd_pistol_shot"),
    (particle_system_burst,"psys_pistol_smoke",pos2,55),
    (agent_deliver_damage_to_agent, ":blow_agent", ":blow_agent", 1000),
(try_end),
```

依旧是战场触发器,和上一个案例一样,不过自爆不用检测间隔很短,设置个两三秒都可以
比如```2.000000 0.000000 0.000000  0 行数```两秒检测一次

## 光环:移速BUFF(光写了 没测)

为一个兵种设置移速光环,在该兵种附近的友军会得到一个10秒的双倍移速BUFF.

```python
(assign,":aura_troop_id","trp_rhodok_blow"), #设置光环士兵ID

(assign, ":distance_closest", 3000),
(try_for_agents, ":need_speed_agent"),
    (agent_is_alive, ":need_speed_agent"),
    (agent_is_human, ":need_speed_agent"),
    
    (agent_get_party_id, ":need_speed_agent_party", ":need_speed_agent"), 
    (agent_get_position,pos2,":need_speed_agent"),

    (assign,":in_range",0),
    (try_for_agents, ":ally"),
        (eq, ":in_range", 0),

        (agent_get_troop_id, ":tar_troop", ":ally"),
        (eq, ":tar_troop", ":aura_troop_id"),
        (agent_is_alive, ":ally"),
        (agent_is_human, ":ally"),

        (agent_get_position, pos3, ":ally"),
        (agent_get_party_id,":ally_party",":ally"),
        (eq, ":need_speed_agent_party", ":ally_party"),

        (get_distance_between_positions, ":distance_abs", pos2, pos3),
        (lt, ":distance_abs", ":distance_closest"),

        (assign,":in_range",1),
    (try_end),

    (try_begin),
        (eq, ":in_range",1),
        (agent_set_speed_modifier, ":need_speed_agent", 200),
    (else_try),
        (eq, ":in_range",0),
        (agent_set_speed_modifier, ":need_speed_agent", 100),
    (try_end),
(try_end),
```

依旧是战场触发器,和上一个案例一样,请参照修改
注意设置触发器的检测间隔为10秒```10.000000 0.000000 0.000000  0 行数```
