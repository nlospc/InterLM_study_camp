本文将进行使用 OpenCompass 来评测 InternLM2 1.8B实践

# 概览

在 OpenCompass 中评估一个模型通常包括以下几个阶段：配置 -> 推理 -> 评估 -> 可视化。

- 配置：这是整个工作流的起点。您需要配置整个评估过程，选择要评估的模型和数据集。此外，还可以选择评估策略、计算后端等，并定义显示结果的方式。
- 推理与评估：在这个阶段，OpenCompass 将会开始对模型和数据集进行并行推理和评估。推理阶段主要是让模型从数据集产生输出，而评估阶段则是衡量这些输出与标准答案的匹配程度。这两个过程会被拆分为多个同时运行的“任务”以提高效率。
- 可视化：评估完成后，OpenCompass 将结果整理成易读的表格，并将其保存为 CSV 和 TXT 文件。

接下来，我们将展示 OpenCompass 的基础用法，分别用命令行方式和配置文件的方式评测InternLM2-Chat-1.8B，展示书生浦语在 `C-Eval` 基准任务上的评估。更多评测技巧请查看 [https://opencompass.readthedocs.io/zh-cn/latest/get_started/quick_start.html](https://opencompass.readthedocs.io/zh-cn/latest/get_started/quick_start.html) 文档。

# 环境配置

## 创建开发机和 conda 环境

在创建开发机界面选择镜像为 Cuda11.7-conda，并选择 GPU 为10% A100。

## 安装——面向GPU的环境安装


```
conda create -n opencompass python=3.10
conda activate opencompass
conda install pytorch==2.1.2 torchvision==0.16.2 torchaudio==2.1.2 pytorch-cuda=12.1 -c pytorch -c nvidia -y

# 注意：一定要先 cd /root
cd /root
git clone -b 0.2.4 https://github.com/open-compass/opencompass
cd opencompass
pip install -e .


apt-get update
apt-get install cmake
pip install -r requirements.txt
pip install protobuf
```
【踩坑】报错
```
Collecting pyext (from -r requirements/runtime.txt (line 24))
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/b0/be/9b6005ac644aaef022527ce49617263379e49dbdbd433d1d3dd66d71f570/pyext-0.7.tar.gz (7.8 kB)
  Preparing metadata (setup.py) ... error
  error: subprocess-exited-with-error
  
  × python setup.py egg_info did not run successfully.
  │ exit code: 1
  ╰─> [9 lines of output]
      Traceback (most recent call last):
        File "<string>", line 2, in <module>
        File "<pip-setuptools-caller>", line 34, in <module>
        File "/tmp/pip-install-wz6_qau4/pyext_91c42e423d904ff8b0ff00fa4d4a53b4/setup.py", line 6, in <module>
          import pyext
        File "/tmp/pip-install-wz6_qau4/pyext_91c42e423d904ff8b0ff00fa4d4a53b4/pyext.py", line 117, in <module>
          oargspec = inspect.getargspec
                     ^^^^^^^^^^^^^^^^^^
      AttributeError: module 'inspect' has no attribute 'getargspec'. Did you mean: 'getargs'?
      [end of output]
  
  note: This error originates from a subprocess, and is likely not a problem with pip.
error: metadata-generation-failed

× Encountered error while generating package metadata.
╰─> See above for output.

```
定位到是因为安装pyext这个包出错
查了一圈似乎是因为python环境的原因，无奈重新准备环境
```
cd ~ 
studio-conda -o internlm-base -t opencompass 
source activate opencompass 
git clone -b 0.2.4 https://github.com/open-compass/opencompass 
cd opencompass 
pip install -r requirements.txt #一定要先安装所有依赖库，再安装e. 
pip install -e . 
pip install protobuf 
export MKL_SERVICE_FORCE_INTEL=1 
cp /share/temp/datasets/OpenCompassData-core-20231110.zip /root/opencompass/ 
unzip OpenCompassData-core-20231110.zip 
```

# 启动评测 (10% A100 8GB 资源)


## 使用命令行配置参数法进行评测


打开 opencompass文件夹下configs/models/hf_internlm/的`hf_internlm2_chat_1_8b.py` ,贴入以下代码

```
from opencompass.models import HuggingFaceCausalLM


models = [
    dict(
        type=HuggingFaceCausalLM,
        abbr='internlm2-1.8b-hf',
        path="/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b",
        tokenizer_path='/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b',
        model_kwargs=dict(
            trust_remote_code=True,
            device_map='auto',
        ),
        tokenizer_kwargs=dict(
            padding_side='left',
            truncation_side='left',
            use_fast=False,
            trust_remote_code=True,
        ),
        max_out_len=100,
        min_out_len=1,
        max_seq_len=2048,
        batch_size=8,
        run_cfg=dict(num_gpus=1, num_procs=1),
    )
]
```

```
python run.py --datasets ceval_gen --models hf_internlm2_chat_1_8b --debug
```

命令解析

```
python run.py
--datasets ceval_gen \ # 数据集准备
--models hf_internlm2_chat_1_8b \  # 模型准备
--debug
```

如果一切正常，您应该看到屏幕上显示：

```
[2024-08-09 16:48:07,016] [opencompass.openicl.icl_inferencer.icl_gen_inferencer] [INFO] Starting inference process...
```

测评过程及其漫长
![执行过程](https://imgsur.quicklydating.com/gh/nlospc/imgsur@main/img/20240906192942.png)


评测完成后，将会看到：
![执行结果](https://imgsur.quicklydating.com/gh/nlospc/imgsur@main/img/20240906193042.png)

## 调用API评测（优秀学员必做

case：以月之暗面为例
1. 在[控制台](https://platform.moonshot.cn/console/info)申请api token
2. 接下来将申请的api_key替换/root/opencompass/configs/api_examples/eval_api_moonshot.py文件中的key··
```
from openai import OpenAI
 
client = OpenAI(
    api_key = "$MOONSHOT_API_KEY",
    base_url = "https://api.moonshot.cn/v1",
)
 
completion = client.chat.completions.create(
    model = "moonshot-v1-8k",
    messages = [
        {"role": "system", "content": "你是 Kimi，由 Moonshot AI 提供的人工智能助手，你更擅长中文和英文的对话。你会为用户提供安全，有帮助，准确的回答。同时，你会拒绝一切涉及恐怖主义，种族歧视，黄色暴力等问题的回答。Moonshot AI 为专有名词，不可翻译成其他语言。"},
        {"role": "user", "content": "你好，我叫李雷，1+1等于多少？"}
    ],
    temperature = 0.3,
)
 
print(completion.choices[0].message.content)
```
4. 执行评测命令
```shell 
## 评测
 python tools/test_api_model.py configs/api_examples/eval_api_moonshot.py 
```

![结果](https://imgsur.quicklydating.com/gh/nlospc/imgsur@main/img/20240906225715.png)
