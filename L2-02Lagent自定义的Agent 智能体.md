## Lagent自定义的Agent 智能体
## **环境配置**

---

开发机选择 30% A100，镜像选择为 Cuda12.2-conda。

首先来为 Lagent 配置一个可用的环境。

```Bash
# 创建环境
conda create -n agent_camp3 python=3.10 -y
# 激活环境
conda activate agent_camp3
# 安装 torch
conda install pytorch==2.1.2 torchvision==0.16.2 torchaudio==2.1.2 pytorch-cuda=12.1 -c pytorch -c nvidia -y
# 安装其他依赖包
pip install termcolor==2.4.0
pip install lmdeploy==0.5.2
```

接下来，我们通过源码安装的方式安装 lagent。

```Bash
# 创建目录以存放代码
mkdir -p /root/agent_camp3
cd /root/agent_camp3
git clone https://github.com/InternLM/lagent.git
cd lagent
git checkout 81e7ace
pip install -r requirements.txt #一定要先安装所有依赖库，再安装e.

pip install -e . 
cd ..
pip install griffe==0.48.0
```

**此处跟教程稍有不同，按教程无法正常运行**

```Bash
在执行pip install -e . 之前需要先执行
pip install -r requirements.txt
```

**base_action.py需要修改**

```Bash
# 16行
from griffe import DocstringSectionKind
```

## **Lagent Web Demo 使用**

接下来，我们将使用 Lagent 的 Web Demo 来体验 InternLM2.5-7B-Chat 的智能体能力。

首先，我们先使用 LMDeploy 部署 InternLM2.5-7B-Chat，并启动一个 API Server。

```Bash
conda activate agent_camp3
lmdeploy serve api_server /share/new_models/Shanghai_AI_Laboratory/internlm2_5-7b-chat --model-name internlm2_5-7b-chat
```

  

然后，我们在另一个窗口中启动 Lagent 的 Web Demo。

```Bash
cd /root/agent_camp3/lagent
conda activate agent_camp3
streamlit run examples/internlm2_agent_web_demo.py
```

效果如下
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240916151803.png)



## **基于 Lagent 自定义智能体**

---

在本节中，我们将带大家基于 Lagent 自定义自己的智能体。

Lagent 中关于工具部分的介绍文档位于 [https://lagent.readthedocs.io/zh-cn/latest/tutorials/action.html](https://lagent.readthedocs.io/zh-cn/latest/tutorials/action.html) 。

使用 Lagent 自定义工具主要分为以下几步：

1. 继承 `BaseAction` 类
    
2. 实现简单工具的 `run` 方法；或者实现工具包内每个子工具的功能
    
3. 简单工具的 `run` 方法可选被 `tool_api` 装饰；工具包内每个子工具的功能都需要被 `tool_api` 装饰
    

下面我们将实现一个调用 MagicMaker API 以完成文生图的功能。

首先，我们先来创建工具文件：

```Bash
cd /root/agent_camp3/lagent
touch lagent/actions/magicmaker.py
```

然后，我们将下面的代码复制进入 `/root/agent_camp3/lagent/lagent/actions/magicmaker.py`

```Bash
import json
import requests

from lagent.actions.base_action import BaseAction, tool_api
from lagent.actions.parser import BaseParser, JsonParser
from lagent.schema import ActionReturn, ActionStatusCode


class MagicMaker(BaseAction):
    styles_option = [
        'dongman',  # 动漫
        'guofeng',  # 国风
        'xieshi',   # 写实
        'youhua',   # 油画
        'manghe',   # 盲盒
    ]
    aspect_ratio_options = [
        '16:9', '4:3', '3:2', '1:1',
        '2:3', '3:4', '9:16'
    ]

    def __init__(self,
                 style='guofeng',
                 aspect_ratio='4:3'):
        super().__init__()
        if style in self.styles_option:
            self.style = style
        else:
            raise ValueError(f'The style must be one of {self.styles_option}')
        
        if aspect_ratio in self.aspect_ratio_options:
            self.aspect_ratio = aspect_ratio
        else:
            raise ValueError(f'The aspect ratio must be one of {aspect_ratio}')
    
    @tool_api
    def generate_image(self, keywords: str) -> dict:
        """Run magicmaker and get the generated image according to the keywords.

        Args:
            keywords (:class:`str`): the keywords to generate image

        Returns:
            :class:`dict`: the generated image
                * image (str): path to the generated image
        """
        try:
            response = requests.post(
                url='https://magicmaker.openxlab.org.cn/gw/edit-anything/api/v1/bff/sd/generate',
                data=json.dumps({
                    "official": True,
                    "prompt": keywords,
                    "style": self.style,
                    "poseT": False,
                    "aspectRatio": self.aspect_ratio
                }),
                headers={'content-type': 'application/json'}
            )
        except Exception as exc:
            return ActionReturn(
                errmsg=f'MagicMaker exception: {exc}',
                state=ActionStatusCode.HTTP_ERROR)
        image_url = response.json()['data']['imgUrl']
        return {'image': image_url}
```

最后，我们修改 `/root/agent_camp3/lagent/examples/internlm2_agent_web_demo.py` 来适配我们的自定义工具。

1. 在 `from lagent.actions import ActionExecutor, ArxivSearch, IPythonInterpreter` 的下一行添加 `from lagent.actions.magicmaker import MagicMaker`
    
2. 在第27行添加 `MagicMaker()`。
    

```Bash
from lagent.actions import ActionExecutor, ArxivSearch, IPythonInterpreter
+ from lagent.actions.magicmaker import MagicMaker
from lagent.agents.internlm2_agent import INTERPRETER_CN, META_CN, PLUGIN_CN, Internlm2Agent, Internlm2Protocol

...
        action_list = [
            ArxivSearch(),
+             MagicMaker(),
        ]
```

接下来，启动 Web Demo 来体验一下吧！我们同时启用两个工具，然后输入“请帮我生成一幅建筑画稿”。


![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240916151848.png)

