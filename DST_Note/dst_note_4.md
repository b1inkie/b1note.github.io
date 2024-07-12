## 物品栏逐帧动画

[APNG动图切片(饥荒)(动态物品栏背景图) 工具下载](https://github.com/b1inkie/b1note.github.io/releases/download/DST_TOOLS/APNG_To_InvBgAnim_v3_by_lan.7z)

- 用途:
将你需要的某个动图变成饥荒当中物品栏图片的背景

- 使用指南：
1. 选择需要进行处理的图像文件
2. 点击后等待转换完成
3. 得到一个png文件,请使用TexCreator手动转换成饥荒使用的tex文件
4. 得到一个切好的xml文件(其中tex名称将以纯数字命名,方便循环)
5. 得到一个help_fn.lua文件,里面是一个可以直接调用的函数,详见文件内部
6. 将tex和xml放入你mod的images文件夹中,在合适的地方调用help_fn.lua里的函数即可

***
#### 以下都是旧内容:

![essex_frame_dyn01](../images/essex_frame_dyn01.gif)

顾名思义,多图循环,实现动画
这个函数用来改图片 `inventory:ChangeImageName("")`

找到steam点数商店头像框那一页,复制URL到网页F12,查找一下图片链接,如果是gif那就直接下载,如果是APNG,那可以用这个网址: [在线转换](https://cdkm.com/cn/png-to-gif) 填好URL转好下载,再将gif拖进PS,然后把图做好.

XML写法:
```XML
<Atlas>
	<Texture filename="essex_frame_dyn01.tex"/>
	<Elements>
		<Element name="1.tex" u1="0.0" u2="0.2" v1="0" v2="0.5"/>
		<Element name="2.tex" u1="0.2" u2="0.4" v1="0" v2="0.5"/>
		<Element name="3.tex" u1="0.4" u2="0.6" v1="0" v2="0.5"/>
		<Element name="4.tex" u1="0.6" u2="0.8" v1="0" v2="0.5"/>
		<Element name="5.tex" u1="0.8" u2="2.0" v1="0" v2="0.5"/>
		<Element name="6.tex" u1="0.0" u2="0.2" v1="0.5" v2="1"/>
		<Element name="7.tex" u1="0.2" u2="0.4" v1="0.5" v2="1"/>
	</Elements>
</Atlas>
```
图片的切法:
> <i style="color:aqua;">u1</i>:宽起点 <i style="color:aqua;">u2</i>:宽终点 <i style="color:lime;">v1</i>:高起点 <i style="color:lime;">v2</i>:高终点

最后回到预制物:
```lua
    inst:AddComponent("inventoryitem")
    inst.components.inventoryitem.imagename = "1" 
    inst.components.inventoryitem.atlasname = "images/inventoryimages/essex_frame_dyn01.xml"
    inst.name_num = 1
    inst:DoPeriodicTask(0.05,function(inst)
        if inst.name_num < 7 then
            inst.name_num = inst.name_num + 1   
        else
            inst.name_num = 1
        end
        inst.components.inventoryitem:ChangeImageName(tostring(inst.name_num))
    end)
```
