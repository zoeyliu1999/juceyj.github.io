---
date: 2019-08-07 21:30:24
layout: archive
title: '服务器使用经验汇总'
author_profile: true
permalink: /blog/server/
---

# Server Tips

- 服务器使用jupyter notebook

  ```bash
  # 服务器上运行
  jupyter notebook --no-browser --port=8889
  # 本地运行
  ssh -N -f -L 127.0.0.1:8889:127.0.0.1:8889 username@address
  # 本地打开 http://localhost:8889/token=...即可
  ```

- 配置环境

  ```bash
  conda create -n py3 python=3.6
  source activate py3
  
  # 可以将指令写在一个sh脚本当中就可以使用nohup后台安装packet
  # test.sh
  echo start
  while read requirement; do conda install --yes $requirement || pip install $requirement; done < requirements.txt
  echo finish
  # run
  nohup ./test.sh &
  ```

- 手动指定conda env位置和pkt cache位置

  ```bash
  # 在用户目录下面创建.condarc文件，并指定
  envs_dirs:
    - /l/vision/v6/kaichen/anaconda/envs
  pkgs_dirs:
    - /l/vision/v6/kaichen/anaconda/pkgs
  ```

- bashrc

  ```bash
  # 登陆的默认操作
  vim .bashrc
  ```

- 服务器下载并使用dropbox

  ```bash
  $ wget https://raw.github.com/andreafabrizi/Dropbox-Uploader/master/dropbox_uploader.sh
  $ chmod +x dropbox_uploader.sh
  
  查询用户信息
  $ ./dropbox_uploader.sh info
  显示根目录内容
  $ ./dropbox_uploader.sh list
  要列出某个特定文件夹中的所有内容
  $ ./dropbox_uploader.sh list Documents/manuals
  上传一个本地文件到一个远程的 Dropbox 文件夹
  $ ./dropbox_uploader.sh upload snort.pdf Documents/manuals
  从 Dropbox 下载一个远程的文件到本地
  $ ./dropbox_uploader.sh download Documents/manuals/mysql.pdf ./mysql.pdf
  从 Dropbox 下载一个完整的远程文件夹到一个本地的文件夹
  $ ./dropbox_uploader.sh download Documents/manuals ./manuals
  在 Dropbox 上创建一个新的远程文件夹
  $ ./dropbox_uploader.sh mkdir Documents/whitepapers
  完全删除 Dropbox 中某个远程的文件夹（包括它所含的所有内容）
  $ ./dropbox_uploader.sh delete Documents/manuals
  ```

- screen

  ```bash
  # 建立新的screen，打开一个新的bash
  $ screen
  # ctrl + a + d 将screen置入后台运行，期间可以logout服务器
  
  $ screen -ls
  $ screen -r # process_id
  $ screen -L # Log screen
  ```

- python添加环境变量

  ```python
  import sys
  sys.path.append('your path')
  ```

- 进程前后台切换

  ```bash
  $ nohup python train.py &
  
  # 1. 运行程序时，先按ctrl z使其进入后台
  # 2. 使用jobs查看对应的job 编号（和PID不同）
  jobs
  # 3. fg是前台，bg是后台，但是只有nohup是忽略输出
  fg/bg %1
  ```

- 指定GPU

  ```bash
  export CUDA_VISIBLE_DEVICES=ID
  os.environ["CUDA_VISIBLE_DEVICES"] = "0,1,2,3"
  ```

- 清楚process结束之后仍然继续占用GPU的进程

  ```bash
  fuser -v /dev/nvidia*
  kill process_id
  ```

- pytorch要求gcc版本>4.9.0，因此在安装pytorch之前我们要先用conda安装gcc

  ```bash
  conda install -c serge-sans-paille gcc_49
  
  # 增加软链
  ln -s /home/chenjoya/opt/anaconda3/envs/graph-retina-rcnn/bin/gcc-4.9 /home/chenjoya/opt/anaconda3/envs/graph-retina-rcnn/bin/gcc  
  ln -s /home/chenjoya/opt/anaconda3/envs/graph-retina-rcnn/bin/g++-4.9 /home/chenjoya/opt/anaconda3/envs/graph-retina-rcnn/bin/g++
  
  # 之后只要将conda的根目录放在$PATH里面的/usr/bin之前就好
  ```

- 安装pytorch只要匹配CUDA版本就好，CUDNN官方会帮你安



# COCO

- pycocotool的安装

  ```bash
  $ conda install cython
  $ git clone https://github.com/pdollar/coco.git
  $ cd coco/PythonAPI
  $ make
  $ python setup.py install
  ```

- 训练集大小118287；验证集大小5000

- object instance.json

  ```bash
  # json文件总体格式
  # categories代表80个物体类别
  {
      "info": info,
      "licenses": [license],
      "images": [image],
      "annotations": [annotation],
      "categories": [category]
  }
  
  # annotation格式
  # 单个的对象（iscrowd=0)polygon，而iscrowd=1时（将标注一组对象，比如一群人）使用RLE格式。
  # segmentation polygon使用的每个polygon点的两维坐标（精确到小数点后两位）
  # RLE其实就是一个稀疏矩阵向量表示
  annotation{
      "id": int,    
      "image_id": int,
      "category_id": int,
      "segmentation": RLE or [polygon],
      "area": float,
      "bbox": [x,y,width,height],
      "iscrowd": 0 or 1,
  }
  ```

  - images数组元素的数量等同于划入训练集（或者测试集）的图片的数量；

  - annotations数组元素的数量等同于训练集（或者测试集）中bounding box的数量；

  - categories数组元素的数量为80（2017年）；

- Object Keypoint.json

  ```bash
  {
      "info": info,
      "licenses": [license],
      "images": [image],
      "annotations": [annotation],
      "categories": [category]
  }
  
  # keypoints是一个三维数组：[x,y,v]，v={0:没标注, 1:不可见, 2:可见}
  annotation{
      "keypoints": [x1,y1,v1,...],
      "num_keypoints": int,
      "id": int,
      "image_id": int,
      "category_id": int,
      "segmentation": RLE or [polygon],
      "area": float,
      "bbox": [x,y,width,height],
      "iscrowd": 0 or 1,
  }
  
  # categories id只有1个person
  # keypoints：关键点名称，skeleton是骨骼图定义
  categories{
      "id": int,
      "name": str,
      "supercategory": str,
      "keypoints": [str],
      "skeleton": [edge]
  }
  ```

  - images数组元素数量是划入训练集（测试集）的图片的数量；
  - annotations是bounding box的数量，在这里只有人这个类别的bounding box；
  - categories数组元素的数量为1，只有一个：person（2017年）；

- Image Caption.json

  ```bash
  {
      "info": info,
      "licenses": [license],
      "images": [image],
      "annotations": [annotation]
  }
  
  # annotation，一张图片可能有多个annotation
  annotation{
      "id": int,
      "image_id": int,
      "caption": str
  }
  ```

  - images数组的元素数量等于划入训练集（或者测试集）的图片的数量；
  - annotations的数量要多于图片的数量，这是因为一个图片可以有多个场景描述；



# Ubuntu Tips

- sudo apt-get

  ```
  用法：apt-get [选项] 命令  
   apt-get [选项] install|remove pkg1 [pkg2 ...]  
   apt-get [选项] source pkg1 [pkg2 ...]  
    
  apt-get 是一个下载安装软件包的简单命令行接口。  
  最常用的命令是update(更新)  
  和install(安装)。  
    
  命令：  
   update - 重新获取软件包列表  
   upgrade - 进行更新  
   install - 安装新的软件包  
   remove - 移除软件包  
   autoremove - 自动移除全部不使用的软件包  
   purge - 移除软件包和配置文件  
   source - 下载源码档案  
   build-dep - 为源码包配置编译依赖  
   dist-upgrade - 发行版升级, 参见 apt-get(8)  
   dselect-upgrade - 依照 dselect 的选择更新  
   clean - 清除下载的归档文件  
   autoclean - 清除旧的的已下载的归档文件  
   check - 检验是否有损坏的依赖  
    
  选项：  
   -h 本帮助文件。  
   -q 输出到日志 - 无进展指示  
   -qq 不输出信息，错误除外  
   -d 仅下载 - 不安装或解压归档文件  
   -s 不实际安装。模拟执行命令  
   -y 假定对所有的询问选是，不提示  
   -f 尝试修正系统依赖损坏处  
   -m 如果归档无法定位，尝试继续  
   -u 同时显示更新软件包的列表  
   -b 获取源码包后编译  
   -V 显示详细的版本号  
   -c=? 阅读此配置文件  
   -o=? 设置自定的配置选项，如 -o dir::cache=/tmp  
  ```

  - 出错了用sudo apt-get -f可以自动修复

- 安装deb包：

  ```
  sudo dpkg -i wps-office*.deb
  ```

  ```
  sudo apt-get install gdebi
  sudo gdebi <package.deb>
  ```

- grep关键词查找

  ```bash
  squeue | grep -i pixel
  (-i 是不区分大小写的查找)
  ```

- CUDA：

  ```bash
  conda install cudatoolkit=8.0
  conda install cudnn=7.0.5
  ```

- 前后台转换：

  ```bash
  ctrl z
  jobs # 查看作业号
  fg 1
  bg 1
  ```



# VS code

- 关闭git功能：setting中加入以下内容

  ```
  {
  	"git.enabled": false,
  	"git.path": null,
  	"git.autofetch": false
  }
  ```