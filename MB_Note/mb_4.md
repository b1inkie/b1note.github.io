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
