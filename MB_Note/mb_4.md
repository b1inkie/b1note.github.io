## 封装与调用

封装:
```python
(store_script_param, ":var_0", 1), #第一个参数
(store_script_param, ":var_1", 2), #第二个参数
...
...
```
编译好 放入scripts中,记得在第二行更改函数总数量:
```Go
fn_a -1
 行数 代码
```

调用:
```python
(call_script,<script_id>,<第1个参数>,<第2个参数>,<第3个参数>,...),

#注意<script_id>,是封装好的函数前面还要加个前缀 script_
#例:
(call_script,"script_fn_a",<第1个参数>,<第2个参数>,<第3个参数>,...),
```

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
