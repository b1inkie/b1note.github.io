## pyinstaller 使用教程

跳过非虚拟环境使用pyinstaller的步骤

## 1. 创建虚拟环境

`virtualenv venv`


## 2. 进入虚拟环境

`venv\Scripts\activate`


## 3. 为虚拟环境安装pyinstaller

`pip install pyinstaller`

## 4. 查看安装需要的依赖,等 

1. 查看当前虚拟环境的包:
`pip list`
你也可以生成一个requirements.txt文件,记录当前虚拟环境的包:
`pip freeze > requirements.txt`

2. 安装包
例如`PIL`对应的包: `pip install pillow`
`customtkinter`: `pip install customtkinter`
等等

## 5. 根目录下添加spec文件,例如main.spec

参数已调教好:

```python
# -*- mode: python ; coding: utf-8 -*-
import os

def add_files_to_datas(base_dir, target_dir):
    datas = []
    for root, _, files in os.walk(base_dir):
        # 移除 __pycache__ 目录
        if '__pycache__' in root:
            continue
        
        # 处理文件
        for file in files:
            # 构建源文件路径
            src_file = os.path.join(root, file)
            # 构建目标文件路径
            dest_file = os.path.relpath(os.path.join(root, file), base_dir)
            # 添加到 datas 列表
            datas.append((src_file, os.path.join(target_dir, dest_file)))
    return datas

def gen_datas(folders):
    datas = []
    for folder in folders:
        datas += add_files_to_datas(folder, folder)
    return datas

# 文件夹打包进exe的需要一并在此处填写 - lan
folders = [
    'funcs',
    'images',
    'modresource'
]

datas_add = gen_datas(folders)


a = Analysis(
    ['main.py'],
    pathex=[],
    binaries=[],
    datas=[] + datas_add , 
    hiddenimports=[],
    hookspath=[],
    hooksconfig={},
    runtime_hooks=[],
    excludes=[],
    noarchive=False,
    optimize=0,
)
pyz = PYZ(a.pure)

exe = EXE(
    pyz,
    a.scripts,
    a.binaries,
    a.datas,
    [],
    name='test',
    debug=False,
    bootloader_ignore_signals=False,
    strip=False,
    upx=True,
    upx_exclude=[],
    runtime_tmpdir=None,
    console=False,
    disable_windowed_traceback=False,
    argv_emulation=False,
    target_arch=None,
    codesign_identity=None,
    entitlements_file=None,
)

```

## 6. 将你的项目中的文件夹也一并按照格式添加进去

`datas=[('modresource/*', 'modresource'),('tools/*', 'tools')], # 此处填写你需要一并打包进exe的文件夹`

## 7. 打包exe文件

`pyinstaller main.spec`


## 注意(踩过坑):

1. 当你的项目中,例如tkinter写了个小玩意,加载了一张图片,并且这张图片是要一并打包进exe的,那么图片的路径要这样写:

```python
# 先获取根目录
DIR_BASE = os.path.dirname(os.path.abspath(__file__))
# 然后拼接路径
img_path = os.path.join(DIR_BASE, 'modresource', 'img.png')
```

2. 如果你的项目,使用的文件不在exe内,比如可以使用exe提取一张图片到当前目录,你再对该图片操作,路径就可以这样写:

```python
# 先获取根目录
cur_dir = os.getcwd()
# 然后拼接路径
img_path = os.path.join(cur_dir, 'img.png')
```