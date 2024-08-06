# 常见问题解决


## 1. NPU丢失问题： `npu-smi info`发现缺失卡 `lspci | grep acc`同样缺失卡


- **解决方案**：升级/重装NPU固件驱动时，需按照**固件 > 驱动**的顺序升级。直接覆盖安装

    ```bash
    ./Ascend-hdk-310p-npu-firmware_7.1.0.4.220.run --check
    ```


    ```bash
    ./Ascend-hdk-310p-npu-firmware_7.1.0.4.220.run --full
    ```

    ```bash
    ./Ascend-hdk-310p-npu-driver_23.0.1_linux-aarch64.run --check
    ```

    ```bash
    ./Ascend-hdk-310p-npu-driver_23.0.1_linux-aarch64.run --full --install-for-all
    ```

    报错：

    ```bash
    [ERROR]The davinci nodes are occupied by some processes, please stop processes and install or uninstall again, details in : /var/log/ascend_seclog/ascend_install.log 
    ```

    检查日志：
    
    ```bash
    cat /var/log/ascend_seclog/ascend_install.log
    ```

    ```bash title="日志信息"
    [WARNING]/dev/davinci_manager has used by docker
    [ERROR]The davinci nodes are occupied by some processes, please stop processes and install or uninstall again.
    ```

    ```bash title="stop&rm docker container"
    docker stop $(docker ps -a -q)
    docker rm $(docker ps -a -q)
    ```

    ```bash title="销毁 vNPU"
    # 查看vNPU信息
    npu-smi info -t info-vnpu -i 5 -c 0
    npu-smi info -t info-vnpu -i 5 -c 1
    # 销毁vNPU
    npu-smi set -t destroy-vnpu -i 5 -c 0 -v 196
    npu-smi set -t destroy-vnpu -i 5 -c 0 -v 197
    npu-smi set -t destroy-vnpu -i 5 -c 0 -v 198
    npu-smi set -t destroy-vnpu -i 5 -c 0 -v 199
    npu-smi set -t destroy-vnpu -i 5 -c 1 -v 212
    npu-smi set -t destroy-vnpu -i 5 -c 1 -v 213
    npu-smi set -t destroy-vnpu -i 5 -c 1 -v 214
    npu-smi set -t destroy-vnpu -i 5 -c 1 -v 215
    # 查看销毁结果
    ls /dev/ | grep davinci*
    ```

    ```bash title="再次尝试重装驱动"
    ./Ascend-hdk-310p-npu-driver_23.0.1_linux-aarch64.run --full --install-for-all --force
    ```

    ```bash title="重启"
    reboot
    ```

    重启后再次查看设备：

    ```bash
    npu-smi info
    ```

    回显所有显卡信息 -> 成功！