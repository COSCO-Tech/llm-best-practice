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
   {ip}:{port}/python          3.9       185523026cbb   4 months ago   996MB
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
   sudo docker tag swr.cn-south-1.myhuaweicloud.com/ascendhub/mindie:1.0.RC2-300I-Duo-aarch64 localhost:30002/mindie:1.0.RC2-300I-Duo-aarch64
   ```

 - push镜像至docker-registry私有镜像仓库

   ```bash
   sudo docker push localhost:30002/mindie:1.0.RC2-300I-Duo-aarch64
   ```

- 切换至Atlas 300I Duo，拉取docker-registry私有镜像仓库中的MindIE镜像

   ```bash
   docker pull 10.18.57.155:30002/mindie:1.0.RC2-300I-Duo-aarch64
   ```

 - 上传模型至Atlas 300I Duo

   模型示例选择：`Qwen2/Qwen1.5-7B-Chat` 存放于 `/home/models/Qwen2/Qwen1.5-7B-Chat`

   ```bash title="bash input"
   ls /home/models/Qwen2/Qwen1.5-7B-Chat
   ```

 - 编写shell脚本 用于启动MindIE镜像
   
   shell脚本存放于`/home/models/Qwen2`

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

   ```bash title="bash output"
   REPOSITORY                  TAG                        IMAGE ID       CREATED        SIZE
   10.18.57.155:30002/mindie   1.0.RC2-300I-Duo-aarch64   50f78f73681d   3 days ago     7.05GB
   ```

 - 执行命令并进入容器

   ```bash
   cd /home/models/Qwen2
   # 设置模型权重路径
   CHECKPOINT=/home/models/Qwen2/Qwen1.5-7B-Chat
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


 - 推理 Qwen1.5-7B-Chat 模型

   Qwen + MindIE适配详细README路径：`/usr/local/Ascend/llm_model/examples/models/qwen/README.md`

   ```bash
   cd /usr/local/Ascend/llm_model
   ```

   修改`config.json`:Qwen1.5模型权重所在路径中的config.json文件需将字段`torch_dtype`的值修改为"float16"，例如`"torch_dtype": "float16"`

   ```bash
   vim ${CHECKPOINT}/config.json

   "torch_dtype": "float16"
   ```


   ```
   # 执行推理脚本
   CHECKPOINT=/home/models/Qwen2/Qwen1.5-7B-Chat
   CHECKPOINT=/home/models/Dolphin/Dolphin-7B-Instruction-0709
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
  ```