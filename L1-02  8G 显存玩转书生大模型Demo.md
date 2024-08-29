### L1 -02  8G 显存玩转书生大模型 Demo
#### 主要任务
通过三种方式在开发机部署internLM2-chat模型
分别是
- Cli Demo
- Streamlit Web Demo
- LMDepoly 

##### 1、环境配置

为 Demo 创建一个可用的环境。

```shell
# 创建环境
conda create -n demo python=3.10 -y
# 激活环境
conda activate demo
# 安装 torch
conda install pytorch==2.1.2 torchvision==0.16.2 torchaudio==2.1.2 pytorch-cuda=12.1 -c pytorch -c nvidia -y
# 安装其他依赖
pip install transformers==4.38
pip install sentencepiece==0.1.99
pip install einops==0.8.0
pip install protobuf==5.27.2
pip install accelerate==0.33.0
pip install streamlit==1.37.0
```

##### 2、Cli Demo部署InternLM2-chat-1.8B模型
创建一个目录，用于存放我们的代码。并创建一个 `cli_demo.py`。

```shell
mkdir -p /root/demo
touch /root/demo/cli_demo.py
```

将下面的代码复制到 `cli_demo.py` 中。

```python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM


model_name_or_path = "/root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b"

tokenizer = AutoTokenizer.from_pretrained(model_name_or_path, trust_remote_code=True, device_map='cuda:0')
model = AutoModelForCausalLM.from_pretrained(model_name_or_path, trust_remote_code=True, torch_dtype=torch.bfloat16, device_map='cuda:0')
model = model.eval()

system_prompt = """You are an AI assistant whose name is InternLM (书生·浦语).
- InternLM (书生·浦语) is a conversational language model that is developed by Shanghai AI Laboratory (上海人工智能实验室). It is designed to be helpful, honest, and harmless.
- InternLM (书生·浦语) can understand and communicate fluently in the language chosen by the user such as English and 中文.
"""

messages = [(system_prompt, '')]

print("=============Welcome to InternLM chatbot, type 'exit' to exit.=============")

while True:
    input_text = input("\nUser  >>> ")
    input_text = input_text.replace(' ', '')
    if input_text == "exit":
        break

    length = 0
    for response, _ in model.stream_chat(tokenizer, input_text, messages):
        if response is not None:
            print(response[length:], flush=True, end="")
            length = len(response)
```

通过 `python /root/demo/cli_demo.py` 来启动我们的 Demo。

![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240829170030.png)

##### 2、Streamlit Web Demo 部署InternLM2-chat-1.8B模型
把本教程仓库 clone 到本地，以执行后续的代码。

```shell
cd /root/demo  
git clone https://github.com/InternLM/Tutorial.git
```
**【踩坑】**
在demo目录下使用git clone 会拉取代码失败，不确定是文件写入权限问题还是什么
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240829173658.png)
在/root 目录下直接git clone可以成功
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240829173747.png)


执行如下代码启动一个 Streamlit 服务。

```shell
streamlit run /root/Tutorial/tools/streamlit_demo.py --server.address 127.0.0.1 --server.port 6006
```

>  **本地**的 PowerShell 中输入以下命令，将端口映射到本地。 
```shell
ssh -CNg -L 6006:127.0.0.1:6006 root@ssh.intern-ai.org.cn -p 你的 ssh 端口号
```
> 然后将 SSH 密码复制并粘贴到 PowerShell 中，回车，即可完成端口映射
   在完成端口映射后，我们便可以通过浏览器访问 `http://localhost:6006` 来启动 Demo

如果是VSCODE 远程SSH的话就不用在本地power shell操作了，会直接有弹窗，点击浏览器打开就行
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240829174004.png)

稍微有点卡，耐心等待即可

![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240829174040.png)

效果图-**进行一些弱智对话**
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240829174343.png)


##### 3、LMDeploy 部署 InternLM-XComposer2-VL-1.8B 模型
InternLM-XComposer2的介绍可以参考闯关手册
总结一下就是一个甚于对视觉语言进行分析的大模型

可以实际部署下看看效果
安装LMDeploy和依赖
```
conda activate demo
pip install lmdeploy[all]==0.5.1
pip install timm==1.0.7
```

使用 LMDeploy 启动一个与 InternLM-XComposer2-VL-1.8B 模型交互的 Gradio 服务。

```
lmdeploy serve gradio /share/new_models/Shanghai_AI_Laboratory/internlm-xcomposer2-vl-1_8b --cache-max-entry-count 0.1
```

用这张照片进行上传，让模型反馈一下图片中的人数，看下效果
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240829175017.png)


**被气笑了**

![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240829180305.png)

而且再次追问也得不到答案

![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240829180356.png)

不过在第一次解析中花费大概半分钟后，后续的问题反应基本都是秒回，效果还行
