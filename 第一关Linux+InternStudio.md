####  **作业内容**
- 必要任务：完成SSH连接与端口映射并运行`hello_world.py
- 可选任务：使用 VSCODE 远程连接开发机并创建一个conda环境
####  **整体思路**
直接使用VSCODE 远程连接到开发机创建conda环境，配置端口映射后运行hello_world.py
#### 1.任务1 完成SSH连接与端口映射并运行`hello_world.py
1. 设置开发机
创建开发机并启动
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%2020240823111736.png)
点击SSH连接

复制命令，通过ssh工具连接服务器


配置SSH密钥进行SSH远程连接
执行命令
`ssh-keygen -t rsa `
得到rsa公钥
点击配置SSH Key

![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823114230.png)
执行命令
`cat `~/.ssh/id_rsa.pub`
将内容复制到key中添加

2. 设置VSCODE
可以在点击左侧的扩展页面，在搜索框中输入“SSH”，第一个就是我们要安装的插件，点开它“Install”就可以了。
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823114632.png)
安装完成插件以后，点击侧边栏的远程连接图标，在SSH中点击“+”按钮，添加开发机SSH连接的登录命令
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823114848.png)
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823114916.png)
【踩坑】如果出现报错Bad owner or permissions on C:\\Users\\Administrator/.ssh/config
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823115141.png)
是因为插件创建的config文件 vscode和terminal都没有执行权限
则需要把默认的C:/User/User/.ssh/config文件内容复制出来，把.ssh文件删了
手动创建.ssh文件和config文件，内容粘贴到config文件里，重启VSCODE，就可以正常连接了。

后面就是进入开发机设置远程工作目录了，这里没什么好说的


3. 设置端口映射

找到我们的开发机，点击**自定义服务**，复制第一条命令

![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823163437.png)

并在开发机中运行
然后运行

4. helloworld.py
先运行
`pip install gradio==4.29.0`
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823134701.png)
创建helloworld.py文件
复制代码
执行
```
python helloworld.py
```

5. 访问接口
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823142026.png)


##### 任务2 创建一个conda环境
开发机已经配置好了conda环境
我们可以使用`conda --version` 查看当前版本
当我们要使用`conda`安装包的时候会非常慢，我们可以设置国内镜像提升安装速度，示例如下：

```shell
#设置清华镜像
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
```

使用`conda create -n homework python=3.10` 创建虚拟环境。
创建后，可以在`.conda`目录下的`envs`目录下找到。
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823142435.png)



![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823142625.png)


![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823142939.png)

也可以通过下面命令查看有哪些虚拟环境
```
conda env list
```
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823163955.png)


#####  任务3  执行test.sh文件
目的：写一个简单的Shell脚本来实现这个操作：我们在根目录下创建`test.sh`文件，写入以下内容：
```
#!/bin/bash

# 定义导出环境的函数
export_env() {
    local env_name=$1
    echo "正在导出环境: $env_name"
    # 导出环境到当前目录下的env_name.yml文件
    conda env export -n "$env_name" > "$env_name.yml"
    echo "环境导出完成。"
}

# 定义还原环境的函数
restore_env() {
    local env_name=$1
    echo "正在还原环境: $env_name"
    # 从当前目录下的env_name.yml文件还原环境
    conda env create -n "$env_name" -f "$env_name.yml"
    echo "环境还原完成。"
}

# 检查是否有足够的参数
if [ $# -ne 2 ]; then
    echo "使用方法: $0 <操作> <环境名>"
    echo "操作可以是 'export' 或 'restore'"
    exit 1
fi

# 根据参数执行操作
case "$1" in
    export)
        export_env "$2"
        ;;
    restore)
        restore_env "$2"
        ;;
    *)
        echo "未知操作: $1"
        exit 1
        ;;
esac
```
更改权限 `chmod +x test.sh` ，然后输入`./test.sh restore xtuner0.1.17`并按下回车就可以还原虚拟环境了。
![image](https://github.com/nlospc/InterLM_study_camp/blob/main/IMG/Pasted%20image%20240823164802.png)
