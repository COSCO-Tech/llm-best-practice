# Atlas 300I Duo + FastDeploy + vNPU 部署OCR模型

## 环境准备

 - [驱动与固件准备](env.md)
 - [Docker准备](docker.md)


## [编译环境搭建](https://github.com/PaddlePaddle/FastDeploy/blob/develop/docs/cn/build_and_install/huawei_ascend.md#%E4%BA%8C%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA)

 - 使用Docker开发环境
    
    能访问公网并具备Docker-Registry私有镜像仓库的开发机：
    ```bash
    # 下载 Dockerfile
    wget https://bj.bcebos.com/fastdeploy/test/Ascend_ubuntu18.04_aarch64_5.1.rc2.Dockerfile
    # 通过 Dockerfile 生成镜像
    docker build --network=host -f Ascend_ubuntu18.04_aarch64_5.1.rc2.Dockerfile -t paddlelite/ascend_aarch64:arm64_v8_cann_5.1.rc2 .
    # Re-tag镜像
    docker tag paddlelite/ascend_aarch64:cann_5.1.rc2 localhost:{docker_registry_port}/ascend_aarch64:cann_5.1.rc2
    docker push localhost:{docker_registry_port}/ascend_aarch64:cann_5.1.rc2
    ```
    
    300I Duo推理机：
    ```bash
    export username=xxxxxx
    export docker_registry_ip=xxx.xxx.xxx.xxx
    export docker_registry_port=xxxxx

    # 在300I Duo中拉取镜像
    docker login -u ${username} ${docker_registry_ip}:${docker_registry_port}
    docker pull ${docker_registry_ip}:${docker_registry_port}/ascend_aarch64:cann_5.1.rc2
    ```

    ```bash title="vNPU容器启动命令"
    docker run -it \
        --device=/dev/vdavinci213:/dev/davinci213 \
        --device=/dev/davinci_manager \
        --device=/dev/devmm_svm \
        --device=/dev/hisi_hdc \
        -v /usr/local/bin/npu-smi:/usr/local/bin/npu-smi \
        -v /home:/home \
        -v $PWD:/Work \
        -v /usr/local/Ascend/driver/lib64/common:/usr/local/Ascend/driver/lib64/common \
        -v /usr/local/Ascend/driver/lib64/driver:/usr/local/Ascend/driver/lib64/driver \
        -v /etc/ascend_install.info:/etc/ascend_install.info \
        -v /usr/local/Ascend/driver/version.info:/usr/local/Ascend/driver/version.info \
        docker_image_id  /bin/bash
docker run -itd --privileged --name=ascend-aarch64 --net=host -v $PWD:/Work -w /Work --device=/dev/davinci0 --device=/dev/davinci_manager --device=/dev/hisi_hdc --device /dev/devmm_svm -v /usr/local/bin/npu-smi:/usr/local/bin/npu-smi  -v /usr/local/Ascend/driver/:/usr/local/Ascend/driver/ paddlelite/ascend_aarch64:cann_5.1.rc2 /bin/bash

    ```

 - 报错解决
    ```bash
    WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
    ```

    ```bash title="构建镜像时指定平台"
    docker build --platform linux/arm64/v8 --network=host -f Ascend_ubuntu18.04_aarch64_5.1.rc2.Dockerfile -t paddlelite/ascend_aarch64:arm64_v8_cann_5.1.rc2 .
    ```

## 参考资料

https://github.com/PaddlePaddle/FastDeploy/blob/develop/examples/vision/ocr/PP-OCR/ascend/README.md
https://github.com/PaddlePaddle/FastDeploy/blob/develop/docs/cn/build_and_install/huawei_ascend.md#%E4%BA%8C%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA