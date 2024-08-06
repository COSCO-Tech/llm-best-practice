# Atlas 300I Duo MindIE部署Qwen1.5-7B-Chat

Atlas 300I Duo是华为推理卡，支持MindIE模型推理。本文介绍如何在Atlas 300I Duo上进行MindIE模型推理。

## 环境准备

 - 物理机已安装Docker，且Docker网络可用

    ```bash
    docker --version
    ```

 - 内网环境中存在docker-registry私有镜像仓库

## Docker配置

- 配置非 https 仓库地址 daemon.json

    ```bash
    sudo vim /etc/docker/daemon.json  

    {
        "insecure-registries": ["{ip}:{docker-registry-port}"]
    }

    systemctl restart docker
    ```

 - 登录至内网docker-registry私有镜像仓库

    ```bash title="bash input"
    docker login -u {username} {ip}:{docker-registry-port}
    ```

    ```bash title="bash output"
    WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/engine/reference/commandline/login/#credential-stores

    Login Succeeded
    ```

 - 尝试拉取任意镜像 测试内网docker-registry私有镜像仓库连通性 

    ```bash
    docker pull {ip}:{docker-registry-port}/python:3.9
    ```
 
 - 查看已拉取镜像

    ```bash title="bash input"
    docker images
    ```

    ```bash title="bash output"
    REPOSITORY                  TAG       IMAGE ID       CREATED        SIZE
    {ip}:{docker-registry-port}/python          3.9       185523026cbb   4 months ago   996MB
    ```

## 内网拉取MindIE镜像

- MindIE官方镜像：https://www.hiascend.com/developer/ascendhub/detail/af85b724a7e5469ebd7ea13c3439d48f
- 版本：1.0.RC2-300I-Duo-aarch64
- CANN版本：8.0.RC2
- MindIE版本：	1.0.RC2
- FrameworkPTAdapter版本：6.0.RC2
- HDK版本：24.1.RC2
- 支持的推理芯片：Atlas 300I Duo
- 架构：aarch64
- 更新时间：2024/07/30
- 需要联系华为对接人，申请权限，获取镜像拉取权限

---
 - **在一台能够访问公网的机器上拉取镜像**

    ```bash
    sudo docker login -u cn-south-1@LQD01JL1KE4MVUULGERV swr.cn-south-1.myhuaweicloud.com
    8b2046193e1420c8376503d0231d0dbaa766a2e21f973f7d0dd4c9a3e5a992d8
    sudo docker pull swr.cn-south-1.myhuaweicloud.com/ascendhub/mindie:1.0.RC2-300I-Duo-aarch64
    ```

 - re-tag镜像

    ```bash
    sudo docker tag swr.cn-south-1.myhuaweicloud.com/ascendhub/mindie:1.0.RC2-300I-Duo-aarch64 localhost:{docker-registry-port}/mindie:1.0.RC2-300I-Duo-aarch64
    ```

 - push镜像至docker-registry私有镜像仓库

    ```bash
    sudo docker push localhost:{docker-registry-port}/mindie:1.0.RC2-300I-Duo-aarch64
    ```

- 切换至Atlas 300I Duo，拉取docker-registry私有镜像仓库中的MindIE镜像

    ```bash
    docker pull {ip}:{docker-registry-port}/mindie:1.0.RC2-300I-Duo-aarch64
    ```

 - 上传模型至Atlas 300I Duo

    模型示例选择：`Qwen1.5/Qwen1.5-7B-Chat` 存放于 `/home/models/Qwen1.5/Qwen1.5-7B-Chat`


## 测试运行

 - 编写shell脚本 用于启动MindIE镜像 shell脚本存放于`/root/start-docker.sh`

    ```shell title="start-docker.sh"
    IMAGES_ID=$1
    NAME=$2
    if [ $# -ne 2 ]; then
        echo "error: need one argument describing your container name."
        exit 1
    fi
    docker run --name ${NAME} -it -d --net=host --shm-size=500g \
        --privileged=true \
        -w /home \
        --device=/dev/davinci0 \
        --device=/dev/davinci1 \
        --device=/dev/davinci2 \
        --device=/dev/davinci3 \
        --device=/dev/davinci4 \
        --device=/dev/davinci5 \
        --device=/dev/davinci6 \
        --device=/dev/davinci7 \
        --device=/dev/davinci_manager \
        --device=/dev/hisi_hdc \
        --device=/dev/devmm_svm \
        --entrypoint=bash \
        -v /usr/local/Ascend/driver:/usr/local/Ascend/driver \
        -v /usr/local/dcmi:/usr/local/dcmi \
        -v /usr/local/bin/npu-smi:/usr/local/bin/npu-smi \
        -v /usr/local/sbin:/usr/local/sbin \
        -v /home:/home \
        -v /tmp:/tmp \
        -v /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime \
        -e http_proxy=$http_proxy \
        -e https_proxy=$https_proxy \
        ${IMAGES_ID}
    ```

      参数说明：

    - IMAGES_ID 为镜像版本号。（docker images 命令回显中的 IMAGES ID）
    - NAME 为启动容器名，可自定义设置。

 - 获取镜像ID

    ```bash title="bash input"
    docker images
    ```

 - 执行命令并进入容器

    ```bash
    cd /root/
    # 设置模型权重路径
    CHECKPOINT=/home/models/Qwen1.5/Qwen1.5-7B-Chat
    # 用户可以设置 docker images 命令回显中的 IMAGES ID
    image_id=50f78f73681d
    # 用户可以自定义设置镜像名
    custom_image_name=Qwen1.5-7B-Chat-MindIE
    # 启动容器(确保启动容器前，本机可访问外网)
    bash start-docker.sh ${image_id} ${custom_image_name}
    # 进入容器
    docker exec -itu root ${custom_image_name} bash
    ```


 - 使能昇腾CANN软件栈

    ```bash
    cd /opt/package
    # 安装CANN包
    source install_and_enable_cann.sh
    # 若退出后重新进入容器，则需要重新加载 CANN 环境变量，执行以下三行命令
    source /usr/local/Ascend/ascend-toolkit/set_env.sh
    source /usr/local/Ascend/mindie/set_env.sh
    source /usr/local/Ascend/llm_model/set_env.sh
    ```


 - 修改 Qwen1.5-7B-Chat 模型配置

    > Qwen + MindIE适配详细README容器内路径：`/usr/local/Ascend/llm_model/examples/models/qwen/README.md`

    ```bash
    cd /usr/local/Ascend/llm_model
    ```

 - **修改`config.json`：Qwen1.5模型权重所在路径中的config.json文件需将字段`torch_dtype`的值修改为"float16"**


    ```bash
    CHECKPOINT=/home/models/Qwen1.5/Qwen1.5-7B-Chat
    vim ${CHECKPOINT}/config.json

    "torch_dtype": "float16"
    ```

 - **修改`examples/models/qwen/run_pa.sh`中的临时环境变量, Qwen1.5-7B-Chat仅需单卡即可运行**

    ```bash
    vim /usr/local/Ascend/llm_model/examples/models/qwen/run_pa.sh

    export ASCEND_RT_VISIBLE_DEVICES=0
    ```


 - 执行推理测试脚本
    - `-m`: 指定模型权重文件夹路径
    - `-c`: 指定是否是Chat

    ```
    bash examples/models/qwen/run_pa.sh -m ${CHECKPOINT} -c true
    ```

 - 再次进入容器

    ```bash
    custom_image_name=Qwen1.5-7B-Chat-MindIE
    docker exec -itu root ${custom_image_name} bash
    source /usr/local/Ascend/ascend-toolkit/set_env.sh
    source /usr/local/Ascend/mindie/set_env.sh
    source /usr/local/Ascend/llm_model/set_env.sh
    cd /usr/local/Ascend/llm_model
    CHECKPOINT=/home/models/Qwen1.5/Qwen1.5-7B-Chat
    bash examples/models/qwen/run_pa.sh -m ${CHECKPOINT} -c true
    ```

## 部署推理服务

 - 修改配置 `/usr/local/Ascend/mindie/latest/mindie-service/conf/config.json` 
    ```bash
    cp /usr/local/Ascend/mindie/latest/mindie-service/conf/config.json /usr/local/Ascend/mindie/latest/mindie-service/conf/config-backup.json
    rm -rf /usr/local/Ascend/mindie/latest/mindie-service/conf/config.json
    touch /usr/local/Ascend/mindie/latest/mindie-service/conf/config.json
    ```

 - 配置显卡可见性

    ```bash
    export ASCEND_RT_VISIBLE_DEVICES=0
    ```

 - 配置如下：
    ```json
    cat > /usr/local/Ascend/mindie/latest/mindie-service/conf/config.json <<EOF
    {
        "OtherParam" :
        {
            "ResourceParam" :
            {
                "cacheBlockSize" : 128
            },
            "LogParam" :
            {
                "logLevel" : "Info",
                "logPath" : "logs/mindservice.log"
            },
            "ServeParam" :
            {
                "ipAddress" : "127.0.0.1",
                "managementIpAddress" : "127.0.0.2",
                "port" : 1025,
                "managementPort" : 1026,
                "maxLinkNum" : 1000,
                "httpsEnabled" : false,
                "tlsCaPath" : "security/ca/",
                "tlsCaFile" : ["ca.pem"],
                "tlsCert" : "security/certs/server.pem",
                "tlsPk" : "security/keys/server.key.pem",
                "tlsPkPwd" : "security/pass/mindie_server_key_pwd.txt",
                "tlsCrl" : "security/certs/server_crl.pem",
                "managementTlsCaFile" : ["management_ca.pem"],
                "managementTlsCert" : "security/certs/management_server.pem",
                "managementTlsPk" : "security/keys/management_server.key.pem",
                "managementTlsPkPwd" : "security/pass/management_mindie_server_key_pwd.txt",
                "managementTlsCrl" : "security/certs/management_server_crl.pem",
                "kmcKsfMaster" : "tools/pmt/master/ksfa",
                "kmcKsfStandby" : "tools/pmt/standby/ksfb",
                "multiNodesInferPort" : 1120,
                "interNodeTLSEnabled" : false,
                "interNodeTlsCaFile" : "security/ca/ca.pem",
                "interNodeTlsCert" : "security/certs/server.pem",
                "interNodeTlsPk" : "security/keys/server.key.pem",
                "interNodeTlsPkPwd" : "security/pass/mindie_server_key_pwd.txt",
                "interNodeKmcKsfMaster" : "tools/pmt/master/ksfa",
                "interNodeKmcKsfStandby" : "tools/pmt/standby/ksfb"
            }
        },
        "WorkFlowParam" :
        {
            "TemplateParam" :
            {
                "templateType" : "Standard",
                "templateName" : "Standard_llama"
            }
        },
        "ModelDeployParam" :
        {
            "engineName" : "mindieservice_llm_engine",
            "modelInstanceNumber" : 1,
            "tokenizerProcessNumber" : 8,
            "maxSeqLen" : 2560,
            "npuDeviceIds" : [[0]],
            "multiNodesInferEnabled" : false,
            "ModelParam" : [
                {
                    "modelInstanceType" : "Standard",
                    "modelName" : "Qwen1_5_7B_Chat",
                    "modelWeightPath" : "/home/models/Qwen1.5/Qwen1.5-7B-Chat",
                    "worldSize" : 1,
                    "cpuMemSize" : 5,
                    "npuMemSize" : 8,
                    "backendType" : "atb",
                    "pluginParams" : ""
                }
            ]
        },
        "ScheduleParam" :
        {
            "maxPrefillBatchSize" : 50,
            "maxPrefillTokens" : 8192,
            "prefillTimeMsPerReq" : 150,
            "prefillPolicyType" : 0,

            "decodeTimeMsPerReq" : 50,
            "decodePolicyType" : 0,

            "maxBatchSize" : 200,
            "maxIterTimes" : 512,
            "maxPreemptCount" : 0,
            "supportSelectBatch" : false,
            "maxQueueDelayMicroseconds" : 5000
        }
    }
    EOF
    ```

    ```
    export PYTHONPATH=/usr/local/Ascend/llm_model:$PYTHONPATH
    cd /usr/local/Ascend/mindie/latest/mindie-service/bin
    ./mindieservice_daemon

    cat ../logs/mindservice.log
    ```


  - 测试服务可用性
    
    在物理机另开终端
    
    [使用兼容OpenAI接口](https://www.hiascend.com/document/detail/zh/mindie/10RC2/mindieservice/servicedev/mindie_service0010.html)

    ```bash
    curl -H "Accept: application/json" -H "Content-type: application/json" -X POST -d '{
      "model": "gpt-3.5-turbo",
      "messages": [{
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {"role": "system",
      "content": "你是谁"}],
      "presence_penalty": 1.03,
      "frequency_penalty": 1.0,
      "seed": null,
      "temperature": 0.5,
      "top_p": 0.95,
      "stream": false
      }' http://127.0.0.1:1025/v1/chat/completions
    ```


<!-- ## 两芯片（单卡300I Duo）部署Hi-Dolphin-7B-Chat推理服务 (TODO)

 - 修改配置 `/usr/local/Ascend/mindie/latest/mindie-service/conf/config.json` 
    ```bash
    cp /usr/local/Ascend/mindie/latest/mindie-service/conf/config.json /usr/local/Ascend/mindie/latest/mindie-service/conf/config-backup.json
    rm -rf /usr/local/Ascend/mindie/latest/mindie-service/conf/config.json
    touch /usr/local/Ascend/mindie/latest/mindie-service/conf/config.json
    vim /usr/local/Ascend/mindie/latest/mindie-service/conf/config.json
    ```

 - 配置显卡可见性

    ```bash
    export ASCEND_RT_VISIBLE_DEVICES=0,1,2,3
    ```

 - 配置如下：
    ```json
    rm -rf /usr/local/Ascend/mindie/latest/mindie-service/conf/config.json
    touch /usr/local/Ascend/mindie/latest/mindie-service/conf/config.json
    cat > /usr/local/Ascend/mindie/latest/mindie-service/conf/config.json <<EOF
    {
        "OtherParam" :
        {
            "ResourceParam" :
            {
                "cacheBlockSize" : 128,
                "preAllocBlocks" : 4
            },
            "LogParam" :
            {
                "logLevel" : "Info",
                "logPath" : "logs/mindservice.log"
            },
            "ServeParam" :
            {
                "ipAddress" : "127.0.0.1",
                "managementIpAddress" : "127.0.0.2",
                "port" : 1025,
                "managementPort" : 1026,
                "maxLinkNum" : 1000,
                "httpsEnabled" : false,
                "tlsCaPath" : "security/ca/",
                "tlsCaFile" : ["ca.pem"],
                "tlsCert" : "security/certs/server.pem",
                "tlsPk" : "security/keys/server.key.pem",
                "tlsPkPwd" : "security/pass/mindie_server_key_pwd.txt",
                "tlsCrl" : "security/certs/server_crl.pem",
                "managementTlsCaFile" : ["management_ca.pem"],
                "managementTlsCert" : "security/certs/management_server.pem",
                "managementTlsPk" : "security/keys/management_server.key.pem",
                "managementTlsPkPwd" : "security/pass/management_mindie_server_key_pwd.txt",
                "managementTlsCrl" : "security/certs/management_server_crl.pem",
                "kmcKsfMaster" : "tools/pmt/master/ksfa",
                "kmcKsfStandby" : "tools/pmt/standby/ksfb",
                "multiNodesInferPort" : 1120,
                "interNodeTLSEnabled" : false,
                "interNodeTlsCaFile" : "security/ca/ca.pem",
                "interNodeTlsCert" : "security/certs/server.pem",
                "interNodeTlsPk" : "security/keys/server.key.pem",
                "interNodeTlsPkPwd" : "security/pass/mindie_server_key_pwd.txt",
                "interNodeKmcKsfMaster" : "tools/pmt/master/ksfa",
                "interNodeKmcKsfStandby" : "tools/pmt/standby/ksfb"
            }
        },
        "WorkFlowParam" :
        {
            "TemplateParam" :
            {
                "templateType" : "Standard",
                "templateName" : "Standard_llama",
                "pipelineNumber" : 1
            }
        },
        "ModelDeployParam" :
        {
            "engineName" : "mindieservice_llm_engine",
            "modelInstanceNumber" : 1,
            "tokenizerProcessNumber" : 8,
            "maxSeqLen" : 2560,
            "npuDeviceIds" : [[0,1,2,3]],
            "multiNodesInferEnabled" : false,
            "ModelParam" : [
                {
                    "modelInstanceType" : "Standard",
                    "modelName" : "Qwen",
                    "modelWeightPath" : "/home/models/Qwen1.5/Qwen1.5-7B-Chat",
                    "worldSize" : 4,
                    "cpuMemSize" : 5,
                    "npuMemSize" : 16,
                    "backendType" : "atb",
                    "pluginParams" : ""
                }
            ]
        },
        "ScheduleParam" :
        {
            "maxPrefillBatchSize" : 50,
            "maxPrefillTokens" : 8192,
            "prefillTimeMsPerReq" : 150,
            "prefillPolicyType" : 0,

            "decodeTimeMsPerReq" : 50,
            "decodePolicyType" : 0,

            "maxBatchSize" : 200,
            "maxIterTimes" : 512,
            "maxPreemptCount" : 0,
            "supportSelectBatch" : false,
            "maxQueueDelayMicroseconds" : 5000
        }
    }
    EOF
    ```

    ```
    export PYTHONPATH=/usr/local/Ascend/llm_model:$PYTHONPATH
    cd /usr/local/Ascend/mindie/latest/mindie-service/bin
    ./mindieservice_daemon

    cat ../logs/mindservice.log
    ```


  - 测试服务可用性
    
    在物理机另开终端

    [使用兼容OpenAI接口](https://www.hiascend.com/document/detail/zh/mindie/10RC2/mindieservice/servicedev/mindie_service0010.html)

    ```bash
    curl -H "Accept: application/json" -H "Content-type: application/json" -X POST -d '{
      "model": "dolphin",
      "messages": [{
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {"role": "system",
      "content": "你是谁"}],
      "presence_penalty": 1.03,
      "frequency_penalty": 1.0,
      "seed": null,
      "temperature": 0.5,
      "top_p": 0.95,
      "stream": false
      }' http://127.0.0.1:1025/v1/chat/completions
    ``` -->