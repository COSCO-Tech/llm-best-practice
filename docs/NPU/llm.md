# Atlas 300I Duo MindIE模型推理

Atlas 300I Duo是华为推理卡，支持MindIE模型推理。本文介绍如何在Atlas 300I Duo上进行MindIE模型推理。

## 环境准备

 - 物理机已安装Docker，且Docker网络可用

    ```bash
    docker --version
    ```

 - 验证Docker网络

    ```bash
    # 创建测试用Docker容器
    docker run -it --rm alpine /bin/sh

    # 在容器中执行网络连通性测试
    ping -c 3 www.baidu.com

    # 检查网络连通性测试结果 ping 命令的返回值为0表示网络连通，非0表示网络不通。
    echo $?
    ```



