# Atlas 300I Duo 离线安装Miniforge Python环境

由于众所周知的原因，避免使用miniconda，故选择[miniforge](https://github.com/conda-forge/miniforge)作为开源替代方案

 - 检查本机架构

    ```bash title='bash input'
    uname -m
    ```

    ```bash title='bash output example'
    aarch64
    ```

 - 下载miniforge

    [Miniforge3-Linux-aarch64](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-aarch64.sh)
    
    通过内网传输至服务器内

 - 安装miniforge 

    ```bash title='bash input'
    bash Miniforge3-Linux-aarch64.sh

    Enter > yes > Enter > Enter
    ```

 - 配置miniforge环境变量

    ```bash title='bash input'
    echo 'export PATH="/root/miniforge3/bin:$PATH"' >> ~/.bashrc
    source ~/.bashrc
    conda init
    source ~/.bashrc
    ```

 - 验证miniforge安装

    ```bash title='bash input'
    conda --version
    ```

    ```bash title='bash output example'
    conda 24.3.0
    ```

 - 测试pip

    ```bash title='bash input'
    pip --version
    ```

    ```bash title='bash output example'
    pip 24.0 from /root/miniforge3/lib/python3.10/site-packages/pip (python 3.10)
    ```

 - 测试python

    ```bash title='bash input'
    python
    ```

    ```bash title='bash output example'
    Python 3.10.14 | packaged by conda-forge | (main, Mar 20 2024, 21:44:20) [GCC 12.3.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> 
    ```

 - 克隆base环境以创建新的虚拟环境
    
        ```bash title='bash input'
        conda create -n infer --clone base
        ```
