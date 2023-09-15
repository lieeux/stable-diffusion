# ubuntu 安装 stable-diffusion-webui 

## 安装步骤
- 使用root创建用户xuelei（stable-diffusion-webui安装不允许使用root）
```
adduser  xuelei
```

- 赋予用户xuelei全操作权限
```
export EDITOR=vim
visudo
  .....add.....
  ## Allow root to run any commands anywhere
  root    ALL=(ALL)       ALL
  xuelei  ALL=(ALL)       ALL
```

- 切换到用户xuelei

- 安装 wget 和 git
```
sudo apt install -y wget git
```

- 安装 python 3.10（stable-diffusion-webui是用python 3.10写的，高版本可能会出问题）
```
sudo apt update && sudo apt upgrade -y
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.10 -y
sudo apt-get -y install python3.10-venv
cd /usr/bin/
sudo rm -Rf python3
sudo ln -s python3.10 python3
python3 -V
```

- 安装其它依赖
```
sudo apt install libgl1 libglib2.0-0
```

- 进入准备安装webui的目录，然后执行：
```
wget -q https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui/master/webui.sh
```

- 如果上面的指令执行不成功，可以clone到本地，再用Xftp上传

- 后续安装及安装成功后运行都执行：
```
bash webui.sh
```

## 错误处理

- RuntimeError: Torch is not able to use GPU; add --skip-torch-cuda-test to COMMANDLINE_ARGS variable to disable this check
- 不能按错误提示执行如下操作，会导致sd运行后无法出图
```
export COMMANDLINE_ARGS="--skip-torch-cuda-test"
```
- 正确的处理方式如下：
```
sudo  python -c "import torch; print(torch.version.cuda)" 
#导入PyTorch库，并打印出当前PyTorch版本所对应的CUDA版本号
返回：18

sudo nvidia-smi
#nvidia-smi是NVIDIA提供的一个命令行工具，可以用于监控GPU的使用情况、查看GPU的硬件信息
#使用sudo权限运行nvidia-smi命令可以获取详细的GPU信息，包括当前正在运行的进程、GPU占用率、显存使用情况
返回：NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver.
Make sure that the latest NVIDIA driver is installed and running.

ls /usr/src/ | grep nvidia
#检查驱动
返回：470.161.03

sudo apt-get install dkms
#安装dkms软件包。dkms是一个动态内核模块管理器，可以用于安装和管理Linux内核模块

sudo dkms install -m nvidia -v 470.161.03
#安装名为nvidia、版本号为470.161.03的内核模块

sudo nvidia-smi
#再次检查
Fri Sep 15 17:15:20 2023       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.161.03   Driver Version: 470.161.03   CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla V100-SXM2...  Off  | 00000000:00:07.0 Off |                    0 |
| N/A   37C    P0    55W / 300W |   9133MiB / 32510MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A     40988      C   python3                          9131MiB |
+-----------------------------------------------------------------------------+
```

- pip出问题，重新下载并安装
```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
sudo python3.10 get-pip.py
```

- torch出问题，或下载卡住，可以在本地下载再用Xftp上传后安装
```
pip install /path/to/torch-2.0.1.whl
#/path/to/torch-2.0.1.whl是已经下载的torch 2.0.1的安装包路径
```

- 每次运行webui都重复下载依赖环境，把把webui.sh的第一个变量use_venv改为0可以解决。
但不建议这么做，我不知道原理，可能带来了新的问题。

- 在安装CodeFormer的依赖项时卡住，可以尝试使用 pip 的 --no-cache-dir 选项重新安装依赖项
```
pip install --no-cache-dir -r requirements.txt
#--no-cache-dir     禁用pip的缓存机制
```

- To create a public link, set share=True in launch()
```
bash webui.sh --share
```

- Could not create share link. Missing file: /home/xuelei/.local/lib/python3.10/site-packages/gradio/frpc_linux_amd64_v0.2.
  - 1.下载这个文件：https://cdn-media.huggingface.co/frpc-gradio-0.2/frpc_linux_amd64
  - 2.把下载的文件重命名为：frpc_linux_amd64_v0.2
  - 3.把frpc_linux_amd64_v0.2放入这个目录：/home/xuelei/.local/lib/python3.10/site-packages/gradio
  - 4.执行下面的代码：
```
chmod +x /home/xuelei/.local/lib/python3.10/site-packages/gradio/frpc_linux_amd64_v0.2
```

- 缺少openai/clip-vit-large-patch14所必须的一些内容，从huggingface.co下载失败
  - 1.打开https://huggingface.co/openai/clip-vit-large-patch14/tree/main，下载所有文件。注意.gitattributes在windows上下载时会自动转为txt，需要重命名
  - 2.用Xftp上传到目标位置，我放在了/home/xuelei/.cache/huggingface/transformers
  - 3./home/xuelei/sd/stable-diffusion-webui/repositories/stable-diffusion-stability-ai/ldm/modules/encoders/modules.py和
    /home/xuelei/sd/stable-diffusion-webui/repositories/generative-models/sgm/modules/encoders/modules.py
  里各有2处openai/clip-vit-large-patch14，替换为/home/xuelei/.cache/huggingface/transformers

## 参数设置
- webui-user.sh
```
# Commandline arguments for webui.py, for example: export COMMANDLINE_ARGS="--medvram --opt-split-attention"
#原文件无参数，如下：
#export COMMANDLINE_ARGS=""
export COMMANDLINE_ARGS="--disable-nan-check --no-half-vae --api --xformers --listen --share --enable-insecure-extension-access --medvram --opt-split-attention"
#参数含义如下：
#--disable-nan-check：禁用NaN检查，即不检查是否存在NaN值。
#--no-half-vae：不使用半精度变分自编码器。
#--api：启用API模式，即允许其他应用程序通过API访问此代码。
#--xformers：启用Xformers，即使用Transformer模型进行处理。
#--listen：启用监听模式，即等待其他应用程序的连接请求。
#--share：启用共享模式，即允许多个应用程序共享此代码的实例。
#--enable-insecure-extension-access：启用不安全的扩展访问，即允许扩展程序访问此代码的内部数据结构。
#--medvram：使用MedVRAM，即使用MedVRAM作为内存管理器。
#--opt-split-attention：启用优化的分割注意力机制，即使用优化算法对分割注意力机制进行处理。
```