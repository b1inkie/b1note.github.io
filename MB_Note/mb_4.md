## 函数封装与调用(scripts.txt)

这篇可以算是一个前置教程的帖子,有些代码需要提前将函数封装好,以便调用.真的方便很多.

- 你想一行代码给物品添加额外信息吗;
- 你想一行代码自定义战利品吗;
- 你想一行代码去添加各种各样的BUFF效果吗?

* 只需要将你搜集到的代码封装成函数,然后调用即可.

**无论你想自己写代码,还是只是想看函数如何调用,这篇都会有答案** 

ms中,我们调用一个函数(或者叫脚本)的方式是:

```python
(call_script,<script_id>,<第1个参数>,<第2个参数>,<第3个参数>,...),
# Calls specified script with or without parameters. Maximum number of parameters you can pass with the operation is 16.
```

**scripts.txt** 文件中就是一堆封装好的函数,这些函数不会直接运行,它们被MOD根目录的其他文件进行调用从而生效.
**scripts.txt** 文件的第二行的数字指代封装好的函数的总数量. 如果我们要添加一个函数,那么这里就要 **+1** .
那么我们在最下面一行,添加一个自己的函数试试:

```python
fn_mygo -1
 0
```

这样就添加好了,这个函数名字叫`fn_mygo`,次行的第一个数字是代码总行数,后面则填代码.
调用这个函数的方式: `(call_script,"script_fn_mygo"),` , 注意有个 **script_** 前缀.

那么目前这个函数还什么都内容没有.我们来写点什么:

**例如我们写一个:** 根据玩家生涯杀敌数,1比10奖励第纳尔.

```python
(get_player_agent_kill_count, ":kill_count"), # 获取玩家生涯杀敌数.
(val_mul,":kill_count",10), # 乘以10.
(troop_add_gold, "trp_player", ":kill_count"), # 奖励第纳尔.
```

编译好丢进fn_mygo中:

```python
fn_mygo -1
 3 1701 1 1224979098644774912 2107 2 1224979098644774912 10 1528 2 360287970189639680 1224979098644774912 
```

那么以后你在任何地方,`(call_script,"script_fn_mygo"),`,就会按照玩家生涯杀敌数,1比10奖励第纳尔了.

就是这么简单啦! 不过以上是一个没有任何参数的函数. 我们可以添加参数,来让函数更加灵活.

**例如我们写一个:** 根据玩家生涯杀敌数,自定义比例,奖励第纳尔.

```python
(store_script_param, ":rate",1), #第1个参数
#(store_script_param, ":var_2",2), #第2个参数
#(store_script_param, ":var_3",3), #第3个参数

(get_player_agent_kill_count, ":kill_count"), # 获取玩家生涯杀敌数.
(val_mul,":kill_count",":rate"), # 乘以自定义比例.
(troop_add_gold, "trp_player", ":kill_count"), # 奖励第纳尔.
```

编译好丢进`fn_mygo`中;
假如我们想以1比20的比例奖励第纳尔,那么这样写:`(call_script,"script_fn_mygo",20),`
1比30就:`(call_script,"script_fn_mygo",30),`

你可以设置更多变量,让决定奖励第纳尔数量的因素增加.


ok,本篇就到此结束.愉快的去调用大家已经写好的函数吧.



***

附一个网址(若打不开请用chrome或用steam++等加速github):

https://b1inkie.github.io/b1note.github.io/

装备某件物品时增加属性点;添加书籍;全自动武器;自爆步兵;光环:移速BUFF(光写了 没测);散弹 但是每一个弹头都是自瞄;自定义战利品等等功能都可以在这里找到.

若依旧有疑问或需要视频讲解,可以加下面群:
附加半个交流群: 855512521

## 表与遍历

我们在操作号文档中搜索`try_for`,能看到有以下这些操作号:

```python
###正顺序循环全部
try_for_range     = 6 # Works like a for loop from lower-bound up to (upper-bound - 1)
		      # (try_for_range,<destination>,<lower_bound>,<upper_bound>),
###倒顺序循环全部
try_for_range_backwards = 7	# Same as above but starts from (upper-bound - 1) down-to lower bound.
				# (try_for_range_backwards,<destination>,<lower_bound>,<upper_bound>),
###循环所有队伍
try_for_parties   = 11          # (try_for_parties,<destination>),
###循环所有agent
try_for_agents    = 12		# (try_for_agents,<destination>),
```

这些就是ms提供的遍历方式了. 例 可以将所有party看成一个表,挂在party上的slot和对应的值,当成键值对.
举例:
我们可以通过某种方式来给某些兵种升级,一共能升5级,每级都会对该兵种的三围或技能进行强化,那么我们可以用一个slot挂在troop身上,来表示该troop的当前强化等级;后续可以通过`try_for_range`遍历troop,筛选并进行操作

## 潘德自动设置坐镇值和分组修复

本篇仅限于 潘德的预言

本文请配合视频食用(能看懂就不用看视频了) 群号:855512521

潘德的坐镇和分组:是由兵种的4个slot决定的,169,170,171这三个是坐镇,172是分组,0步兵,1弓兵,2骑兵,3游骑,等等

这4个slot,是在函数kt_init_troop_slots中手动设置的,然后被game_start调用以生效.

再说分组,在上述函数中设置分组,实际你会发现根本对不上号,这是因为,设置分组由作者写的另一个函数init_correct_troop_type控制
这个函数我也不想知道他分组规则是怎么写的.我经常说过,像这种自己写一个覆盖掉要比修改别人写的代码要省很多时间.

下面制定一下坐镇和分组的规则:
- 坐镇:
	* 169号slot = 等级 + 力量/2 + 铁骨 + 强击 + 强弓 + 强掷 
	* 170号slot = 169号slot * 1.5
	* 171号slot = 0

- 分组: 有弓有马归为第3组游骑兵,有弓无马归为第1组弓兵,无弓有马归为第2组骑兵,无马无弓归为第0组步兵.

开始添加:

```python
# 在函数 kt_init_troop_slots 后面加上以下代码-------------------------------------

#--------------配置部分如下-----------------------------
(assign,":trp_lower_bound","trp_tournament_participants"), #规定下限兵种id.一般从trp_player后面一个开始
(assign,":trp_upper_bound","trp_allend"), #规定上限兵种id.一般到最后一个兵种的后面一个兵种
#--------------配置部分如上-----------------------------

(try_for_range,":troop_id", ":trp_lower_bound", ":trp_upper_bound"), # 遍历所有兵种,计算坐镇slot的值.
    (assign,":qb",0),

    (store_character_level, ":lv", ":troop_id"), # 等级
    (store_attribute_level, ":strength", ":troop_id", ca_strength), # 力量
    (val_div, ":strength", 2), # 力量/2
    (store_skill_level, ":skl_1", skl_ironflesh, ":troop_id"),
    (store_skill_level, ":skl_2", skl_power_strike, ":troop_id"),
    (store_skill_level, ":skl_3", skl_power_throw, ":troop_id"),
    (store_skill_level, ":skl_4", skl_power_draw, ":troop_id"),

    (val_add, ":qb", ":lv"), # 等级
    (val_add, ":qb", ":strength"), # 力量/2
    (val_add, ":qb", ":skl_1"), # 铁骨
    (val_add, ":qb", ":skl_2"), # 强击
    (val_add, ":qb", ":skl_3"), # 强掷
    (val_add, ":qb", ":skl_4"), # 强弓

    (troop_set_slot, ":troop_id", 169, ":qb"), 
    (val_mul, ":qb", 3), 
    (val_div, ":qb", 2),
    (troop_set_slot, ":troop_id", 170, ":qb"), # 170号slot = 169号slot * 1.5
    (troop_set_slot, ":troop_id", 171, 0), 
    #-------------------------------------------------------------
    (assign,":group",0), # 分组,0步兵,1弓兵,2骑兵,3游骑,等等.

    (assign,":has_ranged",0), # 是否有远程武器,0没有,1有.
    (assign,":has_horse",0), # 是否有马,0没有,1有.

    (try_for_range,":i",0,3), # 遍历武器
        (troop_get_inventory_slot, ":itm_id", ":troop_id", ":i"), # 获取武器id.
        (neq,":itm_id",-1), # 如果武器id不为-1,则有武器.
        (item_get_type, ":weapon_type", ":itm_id"), # 获取武器类型.
        (this_or_next|eq,":weapon_type",itp_type_bow), # 如果是弓,则有远程武器.
        (this_or_next|eq,":weapon_type",itp_type_crossbow), # 如果是弩,则有远程武器.
        (eq,":weapon_type",itp_type_thrown), # 如果是投掷,则有远程武器.
        (assign,":has_ranged",1), # 如果有远程武器,则有远程武器.
    (try_end), # 遍历武器结束.

    (try_begin),
        (troop_get_inventory_slot, ":horse_id", ":troop_id", 8), # 获取坐骑id.
        (neq,":horse_id",-1), # 如果坐骑id不为-1,则有马.
        (assign,":has_horse",1), # 如果有马,则有马.
    (try_end), 

    (try_begin), 
        (eq,":has_ranged",1),
        (eq,":has_horse",1),
        (assign,":group",3), # 有远程武器有马,则归为第3组游骑兵.
    (else_try), 
        (eq,":has_ranged",1),
        (eq,":has_horse",0),
        (assign,":group",1), # 有远程武器无马,则归为第1组弓兵.
    (else_try),
        (eq,":has_ranged",0),
        (eq,":has_horse",1),
        (assign,":group",2), # 无远程武器有马,则归为第2组骑兵.
    (else_try),
        (assign,":group",0), # 无远程武器无马,则归为第0组步兵.
    (try_end), 
    (troop_set_slot, ":troop_id", 172, ":group"), # 设置分组. # 172号slot = 分组.
(try_end), # 遍历所有兵种结束.

#------------------以上4个slot设置完毕----------------------------------
```

那么我们还需修复一下分组:

```python
#在函数game_start最后添加代码

#--------------配置部分如下-----------------------------
(assign,":trp_lower_bound","trp_tournament_participants"), #规定下限兵种id.一般从trp_player后面一个开始
(assign,":trp_upper_bound","trp_allend"), #规定上限兵种id.一般到最后一个兵种的后面一个兵种
#--------------配置部分如上-----------------------------

(try_for_range,":troop_id", ":trp_lower_bound", ":trp_upper_bound"), 
    (troop_get_slot, ":group", ":troop_id", 172), # 获取分组. # 172号slot = 分组.
    (troop_set_class,":troop_id",":group"), # 设置分组. # 172号slot = 分组. # 这里设置分组.
(try_end), # 遍历所有兵种结束.
```

><i style="color:aqua;">我的建议(实际也是这样做的)</i>:单独弄俩函数封装一下,别直接在后面加,这样以后修改坐镇或分组规则,只需要修改这两个函数即可.

><i style="color:red;">特别注意</i>:以后如果你添加了一堆新的兵种,以上自动设置并不会生效,因为没有遍历troop的OP啊.你还是要自己再去填一下配置部分的兵种下界,再编译好覆盖掉.

***

附一个网址(若打不开请用chrome或用steam++等加速github):

https://b1inkie.github.io/b1note.github.io/#/MB_Note/mb_3

装备某件物品时增加属性点;添加书籍;全自动武器;自爆步兵;光环:移速BUFF(光写了 没测);散弹 但是每一个弹头都是自瞄;自定义战利品等等功能都可以在这里找到.

若依旧有疑问或需要视频讲解,可以加下面群:
附加半个交流群: 855512521

