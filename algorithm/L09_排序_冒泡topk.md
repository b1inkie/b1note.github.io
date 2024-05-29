## topk

`topk` 是一个常见的编程术语，通常用于表示找到一个数据集中的前 $\color{aqua}k$ 个最大（或最小）的元素。这个操作在数据分析、机器学习和排序算法中非常常见。


## 冒泡排序的topk实现

```python
def bubble_sort_topk(arr, k):  # 冒泡排序算法，找到前k个最大元素
    for i in range(k):
        for j in range(len(arr) - 1 - i):  # 每次冒泡后，最大的元素已经在数组末尾，所以不需要再比较
            if arr[j] > arr[j + 1]:  # 交换相邻元素，使较大的元素在前
                arr[j], arr[j + 1] = arr[j + 1], arr[j]  
    return arr[-k:]  # 返回前k个最大元素
```

时间复杂度为:$$O(nk)$$

## 骑马与砍杀的冒泡排序topk实现

大家用过 $\color{aqua}散弹但每一个弹头都是自瞄$ 的那段代码吗?

先看一段获取距离user最近的k个敌人的代码(注意只是代码片段,有些变量的初始化部分不在代码片段中):
```python
# k表示散弹的弹头数量,由于我们锁定的敌人是离得最近的,所以k也表示距离user最近的k个敌人.

(try_for_range,":i",0,":k"), # 循环k次
    (assign, ":max_range_calc", ":max_range"), # 每次循环后,重置最大距离 
    (try_for_agents, ":enemies"), # 遍历agents

        (assign,":break_1",0), # 标志位,用于跳出循环.

        (try_for_range,":slot_no",":agent_slot_begin",":agent_slot_end"), # 遍历user的slot,检查是否已经存储了该敌人.
            (agent_get_slot,":enemies_id", ":user", ":slot_no"), # 获取slot中的敌人id.
            (eq, ":enemies_id", ":enemies"), # 如果敌人存在了
            (assign,":break_1",1), # 标志位设为true
        (try_end),

        (eq, ":break_1", 0), # 标志位没有变化才继续执行.

        (agent_is_alive, ":enemies"),
        (agent_is_human, ":enemies"),
        (agent_get_team, ":enemy_team", ":enemies"),
        (teams_are_enemies, ":user_team", ":enemy_team"),
        (2076, 0, ":enemies", 9, 1),
        (position_has_line_of_sight_to_position, pos42, pos0),
        (get_distance_between_positions, ":distance_abs", pos0, pos42),

        (lt, ":distance_abs", ":max_range_calc"), # 如果距离小于最大距离,则更新最大距离.
        (assign, ":max_range_calc", ":distance_abs"), # 更新最大距离.
        
        (store_add,":cur_slot",":agent_slot_begin",":i"),
        (agent_set_slot, ":user", ":cur_slot", ":enemies"),
    (try_end),
(try_end),
```

$\color{aqua}Q$:以上这段代码的时间复杂度是多少?
$$O(nk^2)$$

- 如何使用冒泡排序topk方法来降低时间复杂度?
```python

(assign,":enemies_nums",0),
(try_for_agents,":enemies"),
    (agent_is_alive, ":enemies"),
    (agent_is_human, ":enemies"),
    (agent_get_team, ":enemy_team", ":enemies"),
    (teams_are_enemies, ":user_team", ":enemy_team"),
    (2076,0,":enemies",9,1), 
    (position_has_line_of_sight_to_position, pos42, pos0),
    (get_distance_between_positions, ":distance_abs", pos0, pos42),

    (val_add,":enemies_nums",1),
    
    (store_add,":enemies_slot",":agent_slot_begin",":enemies_nums"),
    (store_add,":enemies_distance_slot",":enemies_slot",1),
    
    (agent_set_slot, ":user", ":enemies_slot", ":enemies"),  # 存储敌人的ID到user的槽位中    
    (agent_set_slot, ":user", ":enemies_distance_slot", ":distance_abs"),

(try_end),
(call_script,"script_bubble_sort_topk",":user",":agent_slot_begin",":enemies_nums",":bullets_num"),

#----以下为冒泡排序topk的函数,假设函数名叫bubble_sort_topk----#
(store_script_param,":user",1), # 用户代理的ID
(store_script_param,":agent_slot_begin",2), # 1400
(store_script_param,":enemies_nums",3), # 敌人数量
(store_script_param,":topk",4), # 选择离得最近的k个敌人

(store_add,":slot_begin",":agent_slot_begin",1), # 1401 开始槽位
(store_mul,":double_enemies_nums",":enemies_nums",2), 
(store_add,":slot_end",":agent_slot_begin",":double_enemies_nums"), 

(try_for_range,":i",0,":topk"),
    (store_sub,":iter_times",":enemies_nums",1),
    (val_sub,":iter_times",":i"),

    (try_for_range,":j",0,":iter_times"),
        (store_mul,":db_j",":j",2), # 2*j 槽位偏移量
        (store_add,":cur_slot",":db_j",":agent_slot_begin"), # 当前存距离的槽位
        (store_add,":next_slot",":cur_slot",2) # 下一个存距离的槽位

        (agent_get_slot,":cur_slot_distance" ,":user",":cur_slot"), # 当前槽位距离值
        (agent_get_slot,":next_slot_distance" ,":user",":next_slot"), # 下一个槽位距离值

        (lt, ":cur_slot_distance", ":next_slot_distance"), # 比较距离大小，如果当前距离小于下一个距离，则交换槽位,因为我们要的是距离最近的敌人,所以我们要降序排序,最小的数放后面

        (agent_set_slot, ":user", ":cur_slot", ":next_slot_distance"), # 交换距离
        (agent_set_slot, ":user", ":next_slot", ":cur_slot_distance"), # 交换距离

        (val_sub,":cur_slot",1), # 当前存enemy ID的槽位
        (val_sub,":next_slot",1), # 下一个存enemy ID的槽位 

        (agent_get_slot,":cur_slot_enemy_id",":user",":cur_slot"), # 当前敌人的ID
        (agent_get_slot,":next_slot_enemy_id",":user",":next_slot"),# 下一个敌人的ID
        (agent_set_slot,":user",":cur_slot",":next_slot_enemy_id"), # 交换敌人ID
        (agent_set_slot,":user",":next_slot",":cur_slot_enemy_id"), # 交换敌人ID

    (try_end),
(try_end),

```

以上这段代码的时间复杂度是多少?
$$O(n+nk)$$
也就是
$$=O(n(k+1))$$
当k远小于n时,k+1和k可以当成一个量级的,所以:
$$=O(nk)$$

- 有没有可能继续优化?

观察一下,我们其实先"创建了一张表",然后再进行的排序操作,那么可不可以在创建列表的时候同时找到topk?

这个问题我们先放到一边!

$\color{aqua}※思考$:创建列表时,外层循环复杂度肯定是`O(n)`,如果内层也做了循环并且不发生折半,那么复杂度是`O(k)`,也就是说最终时间复杂度为`O(nk)`.时间复杂度和上面是一样的,也就是说我们必须要或者起码要在内层发生循环折半,才有可能减少复杂度,因为
$$O(n \cdot k)<O(n \cdot logk)$$


这个问题我们暂时还解决不了!以后会讲

