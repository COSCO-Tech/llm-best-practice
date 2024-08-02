# Atlas 300I Duo NPU驱动和固件安装

## [准备](https://support.huawei.com/enterprise/zh/doc/EDOC1100288556/58f4d9a2#ZH-CN_TOPIC_0000001453871513)

1. 确认操作系统

    查询服务器及配套PCIe卡支持的操作系统，详细请参考[计算产品兼容性查询助手](https://info.support.huawei.com/computing/ftca/zh/product/atlas)。

    ``` title="bash input"
    uname -m && cat /etc/*release
    ```

    ``` title="bash output example"  hl_lines="2"
    aarch64
    Kylin Linux Advanced Server release V10 (Sword)
    DISTRIB_ID=Kylin
    DISTRIB_RELEASE=V10
    DISTRIB_CODENAME=juniper
    DISTRIB_DESCRIPTION="Kylin V10"
    DISTRIB_KYLIN_RELEASE=V10
    DISTRIB_VERSION_TYPE=enterprise
    DISTRIB_VERSION_MODE=normal
    NAME="Kylin Linux Advanced Server"
    VERSION="V10 (Sword)"
    ID="kylin"
    VERSION_ID="V10"
    PRETTY_NAME="Kylin Linux Advanced Server V10 (Sword)"
    ANSI_COLOR="0;31"

    Kylin Linux Advanced Server release V10 (Sword)
    ```

2. 获取软件包和配套表


    - [固件与驱动安装包](https://www.hiascend.com/hardware/firmware-drivers/community?product=2&model=17&cann=8.0.RC3.alpha001&driver=1.0.22.alpha)

        - 产品型号: Atlas 300I Duo 推理卡
        - CANN版本: 8.0.RC3.alpha001
        - 固件与驱动: 1.0.22.alpha
        - 软件包格式: run
        - 组件: NPU
        - CPU架构: AArch64


    - [固件与驱动安装指导](https://www.hiascend.com/document/detail/zh/canncommercial/80RC2/softwareinst/instg/instg_0003.html)


3. 校验安装包：

    ```bash
    chmod a+x Ascend-hdk-310p-npu-driver_23.0.1_linux-aarch64.run
    chmod a+x Ascend-hdk-310p-npu-firmware_7.1.0.4.220.run
    ./Ascend-hdk-310p-npu-driver_23.0.1_linux-aarch64.run --check  # 校验run安装包的一致性和完整性。
    ./Ascend-hdk-310p-npu-firmware_7.1.0.4.220.run --check
    ```

    出现如下回显信息，表示软件包校验成功。

    ```bash
    Verifying archive integrity...  100%   SHA256 checksums are OK. All good.
    ```

4. 创建用户

    ```bash
    groupadd HwHiAiUser
    useradd -g HwHiAiUser -d /home/HwHiAiUser -m HwHiAiUser -s /bin/bash
    password HwHiAiUser  # 设置密码
    ```

5. 安装

    ```bash

    ./Ascend-hdk-310p-npu-driver_23.0.1_linux-aarch64.run --full --install-for-all
    ./Ascend-hdk-310p-npu-firmware_7.1.0.4.220.run --full
    ```

6. 结论

    确认安装成功

    ```bash
    npu-smi info
    ```

    返回信息重写后如下
    
    Atlas 300I Duo 每张卡内有2张芯片 每张芯片的显存为48GB 实际显存大小约为44GB左右
    
    | NPU | Name     | Health | Power(W) | Temp(C) | Hugepages-Usage(page) | Chip | Device | Bus-Id        | AICore(%) | Memory-Usage(MB) |
    |-----|----------|--------|----------|---------|-----------------------|------|--------|---------------|-----------|------------------|
    | 1   | 310P3    | OK     | NA       | 54      | 0 / 0                 | 0    | 0      | 0000:01:00.0  | 0         | 1611 / 44216     |
    | 1   | 310P3    | OK     | NA       | 56      | 0 / 0                 | 1    | 1      | 0000:01:00.0  | 0         | 1225 / 43757     |
    | 2   | 310P3    | OK     | NA       | 58      | 0 / 0                 | 0    | 2      | 0000:02:00.0  | 0         | 1616 / 44216     |
    | 2   | 310P3    | OK     | NA       | 57      | 0 / 0                 | 1    | 3      | 0000:02:00.0  | 0         | 1229 / 43757     |
    | 4   | 310P3    | OK     | NA       | 57      | 0 / 0                 | 0    | 4      | 0000:81:00.0  | 0         | 1849 / 44216     |
    | 4   | 310P3    | OK     | NA       | 57      | 0 / 0                 | 1    | 5      | 0000:81:00.0  | 0         | 997 / 43757      |
    | 5   | 310P3    | OK     | NA       | 58      | 0 / 0                 | 0    | 6      | 0000:82:00.0  | 0         | 1423 / 44216     |
    | 5   | 310P3    | OK     | NA       | 57      | 0 / 0                 | 1    | 7      | 0000:82:00.0  | 0         | 1422 / 43757     |

