# XTuner微调个人小助手认知
- 基础任务：使用 XTuner 微调 InternLM2-Chat-1.8B 实现自己的小助手认知，记录复现过程并截图。
- 进阶任务：将自我认知的模型上传到 OpenXLab，并将应用部署到 OpenXLab

### 基础任务
先根据[准备工作](https://github.com/InternLM/Tutorial/blob/camp3/docs/L1/XTuner/readme.md#2-%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)把环境搭建好
比如xtuner的安装和模型准备
【踩坑】，指导中设置的软路由，执行streamlit会报错，model_or_path_error.
将demo.py的路径改成真实路径/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b就可以了

微调前结果
![运行结果](https://imgsur.quicklydating.com/gh/nlospc/imgsur@main/img/20240907201139.png)

<details>
<summary>微调文件</summary>
```python
# 设置用户的名字
name = '胡来'
# 设置需要重复添加的数据次数
n = 8000
```
# 初始化数据
```
data = [
    {"conversation": [{"input": "请介绍一下你自己", "output": "我是{}的小助手，内在是上海AI实验室书生·浦语的1.8B大模型哦".format(name)}]},
    {"conversation": [{"input": "你在实战营做什么", "output": "没什么特别的目的，我被安排在这里只是为了帮助{}完成XTuner微调个人小>
助手的任务".format(name)}]}
]
```
</details>

# 通过循环，将初始化的对话数据重复添加到data列表中
```python
for i in range(n):
    data.append(data[0])
    data.append(data[1])
```

# 将data列表中的数据写入到'datas/assistant.json'文件中
```python
with open('datas/assistant.json', 'w', encoding='utf-8') as f:
    # 使用json.dump方法将数据以JSON格式写入文件
    # ensure_ascii=False 确保中文字符正常显示
    # indent=4 使得文件内容格式化，便于阅读
    json.dump(data, f, ensure_ascii=False, indent=4)
```                                                    

最后跑一遍，微调成功

![成功](https://imgsur.quicklydating.com/gh/nlospc/imgsur@main/img/20240907234509.png)
