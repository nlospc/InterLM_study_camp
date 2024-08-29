### L0python实现wordcount&远程debug
####  **作业内容**

- 闯关任务Python实现wordcount
- 闯关任务Vscode连接InternStudio debug笔记
####  **整体思路**
本地实现wordcount，再连接开发机进行debug
#### 1.  wordcount.py实现
1. 数据格式化  
方法如下
```
import re

def formatter(text):

    # 将文本转换为小写

    newtext=text.lower()

    # 去除标点符号和特殊字符，只保留字母、数字和空格

    newtext=re.sub(r'[^\w\s]', '', newtext)

    return newtext
```

2. 统计各个单词的数量

```
def wordcount(text):

    try:

        # 使用空格分割单词

        words = formatter(text).split()

        # 创建一个字典来存储单词计数

        word_count = {}

        # 统计每个单词的出现次数

        for word in words:

            word_count[word] = word_count.get(word, 0) + 1

        # 将字典转换为格式化的字符串

        result = ""

        for word, count in word_count.items():

            result += f"{word}: {count}\n"

        return result.strip()  # 使用strip()去除最后的换行符

    except AttributeError:

        return "错误：输入必须是字符串。"

    except Exception as e:

        return f"发生错误：{str(e)}"
```


3. 运行查看
运行代码
```
input="Got this panda plush toy for my daughter's birthday,
who loves it and takes it everywhere. It's soft and
super cute, and its face has a friendly look. It's
a bit small for what I paid though. I think there
might be other options that are bigger for the
same price. It arrived a day earlier than expected,
so I got to play with it myself before I gave it
to her."

print(f"计算各个单词的输出\n{wordcount2(input)}")
```

4. 查看输出
```
计算各个单词的输出
{'got': 2, 'this': 1, 'panda': 1, 'plush': 1, 'toy': 1, 'for': 3, 'my': 1, 'daughters': 1, 'birthday': 1, 'who': 1, 'loves': 1, 'it': 5, 'and': 3, 'takes': 1, 'everywhere': 1, 'its': 3, 'soft': 1, 'super': 1, 'cute': 1, 'face': 1, 'has': 1, 'a': 3, 'friendly': 1, 'look': 1, 'bit': 1, 'small': 1, 'what': 1, 'i': 4, 'paid': 1, 'though': 1, 'think': 1, 'there': 1, 'might': 1, 'be': 1, 'other': 1, 'options': 1, 'that': 1, 'are': 1, 'bigger': 1, 
'the': 1, 'same': 1, 'price': 1, 'arrived': 1, 'day': 1, 'earlier': 1, 'than': 1, 'expected': 1, 'so': 1, 'to': 2, 'play': 1, 'with': 1, 'myself': 1, 'before': 1, 'gave': 1, 'her': 1}
```


##### 任务2 通过VSCODE在开发机进行debug
将本地的py文件复制进开发机
开发机没有debugpy的话安装一下
```
pip install debugpy
```
点击VSCode侧边栏的“Run and Debug”（运行和调试)，单击"create a lauch.json file"

选择debugger时选择python debuger。选择debug config时选择remote attach就行，随后会让我们选择debug server的地址，因为我们是在本地debug，所以全都保持默认直接回车就可以了，也就是我们的server地址为localhost:5678。
会生成一个launch.json的文件
![imgae][Pasted image 20240827210424.png]
![imgae](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240827210424.png)
打好断点，开始▶️debug
![imgae](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240827211202.png)
可以清楚看到各个字段在数值变化
![imgae](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240827210110.png)
