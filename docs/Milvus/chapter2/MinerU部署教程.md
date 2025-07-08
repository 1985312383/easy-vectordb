# MinerU部署教程

> 教程参考：https://mp.weixin.qq.com/s/ylVXT0dB_tcDG6zFPmwo8A

## 什么是MinerU？

MinerU是一款由上海人工智能实验室 OpenDataLab 团队开发的开源 PDF 转 Markdown 工具，可以高质量地提取 PDF 文档内容，生成结构化的 Markdown 格式文本，可用于RAG、LLM语料准备等场景。

> 笔者记，笔者根据自己的实际使用体验来看，MinerU在文档解析上准确度很高，同时开源部署，非常适合学习使用

**官方客户端**：

*https://mineru.net/*

**MinerU Readme 地址（中文）：**

*https://github.com/opendatalab/MinerU/blob/master/README_zh-CN.md*

MinerU 仓库文档说明

```
MinerU/
├── demo/                # 用于运行转换演示的脚本
├── docker/              # 用于容器化的 Dockerfile 配置文件
├── docs/                # 存储各类说明文档
├── projects/            # 存放由 MinerU 衍生或相关的项目
│   ├── gradio_app/        # MinerU Gradio 界面的源代码
│   ├── multi_gpu/         # 为 MinerU 提供多 GPU 支持的解决方案
│   ├── web_api/           # 提供本地 Web API 接口的服务端代码
```

## MinerU 功能特性

MinerU具有以下核心功能：

**1. 文档处理**

● 删除页眉、页脚、脚注、页码等元素，确保语义连贯

● 保留原文档的结构，包括标题、段落、列表等

● 提取图像、图片描述、表格、表格标题及脚注

**2. 格式转换**

● 自动识别并转换文档中的公式为LaTeX格式

● 自动识别并转换文档中的表格为HTML格式

**3. 运行环境**

● 支持纯 CPU 环境运行

● 支持 GPU 加速，提升处理效率

## 本地部署系统要求

在开始安装前，请确保您的系统满足以下要求：

### **基础环境**

● Python 3.10～3.13

● Conda（包管理器）

### GPU加速要求

● NVIDIA显卡（显存≥6GB）

基础环境配置推荐：

# 安装教程

## 1.环境配置

### **1.1 创建Conda环境**

```python
conda create -n mineru 'python=3.12' -y
conda activate mineru
pip install -U "magic-pdf[full]" -i https://mirrors.aliyun.com/pypi/simple 
#-i 是指定国内的加速源，可选清华源或阿里云源，此处用阿里云源示例
```

### **1.2 下载模型文件**

```
方法一：从Hugging Face下载模型（国际用户推荐）

pip install huggingface_hub
curl -o download_models_hf.py https://gcore.jsdelivr.net/gh/opendatalab/MinerU@master/scripts/download_models_hf.py
python download_models_hf.py

方法二：从ModelScope下载模型（国内用户推荐）

pip install modelscope 
curl -o download_models.py https://gcore.jsdelivr.net/gh/opendatalab/MinerU@master/scripts/download_models.py
python download_models.py
```

模型默认存储路径示例：

```
model_dir: C:\Users\用户名\.cache\modelscope\hub\models\opendatalab\PDF-Extract-Kit-1___0/models
layoutreader_model_dir: C:\Users\用户名\.cache\modelscope\hub\models\ppaanngggg\layoutreader
```

**💡提示**：

下载完成后，系统会自动在用户目录下生成 `magic-pdf.json `配置文件，你可以在这个配置文件中修改部分配置，实现不同功能的开关，如表格识别、公式识别关闭或开启（默认二者都是开启的，关闭只需将对应的值改 'true' 为 'false' ）。

用户目录位置：

● Windows：`C:\Users\用户名`

● Linux：`/home/用户名`

● macOS：`/Users/用户名`

## **2.** GPU加速配置

### **2.1** CUDA加速设置

这里以 Windows（NVIDIA 显卡） 为例。如果您的 NVIDIA 显卡显存 ≥ 6GB，可配置 CUDA 加速。这里我们以 CUDA 12.8 安装为例：

```
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu128
```

修改用户目录中配置文件 `magic-pdf.json `：

```

{
    // other config
    "device-mode": "cuda"
}
```

##  3. 功能测试

### **3.1 单文件测试**

执行以下命令自动测试功能：

```
cd demo
magic-pdf -p demo1.pdf -o ./output
```

### 3.2 批量PDF转换

**步骤 1：获取批量转换脚本**

下载名为 `batch_demo.py` 的 Python 文件。你可以将此文件保存在你希望执行转换的任何目录下。

```
下载地址：https://github.com/opendatalab/MinerU/blob/master/demo/batch_demo.py
```

**步骤 2：准备 PDF 文件**

在 `batch_demo.py` 文件的目录下新建如下文件夹：

```
pdfs  # batch_demo.py 相对于脚本的路径 
```

**步骤 3：执行批量转换：**

打开你的终端或命令提示符，导航到你保存 `batch_demo.py` 文件的目录。例如，如果你的 `batch_demo.py` 文件保存在 `demo` 文件夹中，你可以执行以下命令：

```
cd demo
python batch_demo.py
```

**步骤 4：查看转换结果：**

转换后的结果将默认输出到与 `batch_demo.py` 文件同级目录下的一个名为 `output` 的文件夹中。

```
output  # 相对于脚本的路径
```

# 搭建本地 **API** 服务

> 笔者记，推荐使用docker部署，方便运维，因此这里也只复制docker相关部署方法

### **Docker安装方式**

####  **步骤 1：构建方式**

```python
docker build -t mineru-api .

或者使用代理：

docker build --build-arg http_proxy=http://127.0.0.1:7890 --build-arg https_proxy=http://127.0.0.1:7890 -t mineru-api .
```

#### **步骤 2：启动命令**

```
docker run --rm -it --gpus=all -p 8000:8000 mineru-api
```

上述任意一种方式安装完成后，可以通过如下地址访问（测试）

```
http://localhost:8000/docs
http://127.0.0.1:8000/docs
```

