## 工具篇

## 1. 一键制作逐帧动画

[点我下载:动图一键转scml LightVer](https://github.com/b1inkie/b1note.github.io/releases/download/dst_tool_gif_to_scml/gif_to_scml_Light.7z)

[上面如果用不了 再下我](https://github.com/b1inkie/b1note.github.io/releases/download/dst_tool_gif_to_scml/gif_to_scml.7z)

- 用途:
将一张动图,转换成scml格式的逐帧动画

- 使用方法:
1.  选择动图文件(支持apng,png,gif)后(请将名字改成你想要的特效名,禁止使用中文)
2.  设置宽度(可不填,高度自适应)(防止图片太大)
3.  设置帧间隔(单位:毫秒 不填则默认为40)
4.  点击开始转换
5.  得到一个exported文件夹,里面包含一个逐帧动画文件夹,文件夹中的scml文件名和build名是一样的,动画名为idle
6.  放入MOD文件夹,自行使用autocompiler打包

## 2. 一键制作动态物品栏边框

[点我下载: APNG动图切片(饥荒)(动态物品栏背景图) LightVer](https://github.com/b1inkie/b1note.github.io/releases/download/DST_TOOLS/TOOL_APNG_To_InvBgAnim_Light.7z)

[上面如果用不了 再下我](https://github.com/b1inkie/b1note.github.io/releases/download/DST_TOOLS/TOOL_APNG_To_InvBgAnim.7z)

- 用途:
将你需要的某个动图变成饥荒当中物品栏图片的背景

- 使用指南：
1. 选择需要进行处理的图像文件
2. 点击后等待转换完成,会得到4个文件
3. 得到一个合并好的png文件
4. 得到一个tex文件(由内置的TexConvertor自动生成)
5. 得到一个切好的xml文件(其中tex名称将以纯数字命名,方便循环)
6. 得到一个help_fn.lua文件,里面是一个可以直接调用的函数,函数下面的示例将会给出调用当前xml的推荐参数
7. 将tex和xml放入你mod的images文件夹中,在合适的地方插入help_fn.lua里的示例函数即可
8. 注意新手请勿随意更改文件名,在modmain中正常注册xml即可

## 3. 一键制作武器

[点我下载: 一键制作武器 LightVer](https://github.com/b1inkie/b1note.github.io/releases/download/DST_gen_weapon/Auto_Gen_Weapon_by_lan.7z)

- 用途:
一键制作武器

- 使用指南：
1. 点击 提取三张素材图片到当前目录, 
    img_ground: 地面图
    img_swap: 手持图
    img_inv: 物品栏图

2. 不更改图片名称和图层大小,替换好以后,填写右边的信息,勾选你需要的功能

3. 点击 注入图片和信息(请耐心等待), 等待完成

4. 还差最后一步,你需要自动打包动画工具,如果是正版,请下载Dont Starve Mod Tools,autocompiler就位于该目录下;学习版请自行搜索下载

5. 你的武器已完成,在武器制作栏你可以找到他,若想修改详细信息,请自行修改scripts/util/prefab_data文件