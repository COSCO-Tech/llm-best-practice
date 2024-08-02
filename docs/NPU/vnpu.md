# Atlas 300I Duo 昇腾虚拟化实例（算力切分）

前提：

 - 已经完成昇腾NPU驱动固件安装，驱动安装后，用户就能使用虚拟化实例功能

## 创建vNPU

npu-smi set -t create-vnpu -i id -c chip_id -f vnpu_config [-v vnpu_id] [-g vgroup_id]

- id: 设备id。通过npu-smi info -l命令查出的NPU ID即为设备id。
- chip_id: 芯片id。通过npu-smi info -m命令查出的Chip ID即为芯片id。
- vnpu_config: 算力切分模板名称，可参见[表2](https://www.hiascend.com/document/detail/zh/computepoweralloca/30rc3/cpaug/avi_00005.html#ZH-CN_TOPIC_0000001310658286__table140421911260)
- vnpu_id: 指定需要创建的vNPU的id。
    - 首次创建可以不指定该参数，由系统默认分配。若重启后业务需要使用重启前的vnpu_id，可以使用-v参数指定重启前的vnpu_id进行恢复。
    - 取值范围。
        - Ascend 310P的vnpu_id的取值范围为[phy_id*16 + 100, phy_id * 16+107]。
        - Ascend 910的vnpu_id的取值范围为[phy_id*16 + 100, phy_id * 16+115]。
            - Tips: phy_id表示芯片物理ID，可通过执行ls /dev/davinci*命令获取芯片的物理ID。例如/dev/davinci0，表示芯片的物理ID为0。
        - vnpu_id传入4294967295时表示不指定虚拟设备号。
        - 同一设备内不可重复创建相同vnpu_id的vNPU。
- vgroup_id: 虚拟资源组vGroup的id，取值范围0~3。vGroup的概念可以参见[虚拟化模式](https://www.hiascend.com/document/detail/zh/computepoweralloca/30rc3/cpaug/avi_00007.html)，仅Ascend 310P支持本参数。

## 参数查看

 - 查找`id`
    ```bash title='bash input'
    npu-smi info -l
    ```
    
    > 此处我选择NPU ID为5的设备进行操作

 - 查找`chip_id`
    ```bash title='bash input'
    npu-smi info -m
    ```

    > 此处我选择Chip ID为1的芯片进行操作

 - 算力切分模板 `vnpu_config`

    > 此处我选择 vir04 虚拟化实例模板


## 使用示例

 - 在设备5中编号为1的芯片上根据模板vir04创建vNPU。

    ```bash title='bash input'
    npu-smi set -t create-vnpu -i 5 -c 1 -f vir04
    ```

    ```bash title='bash output example'
        Status                         : OK
        Message                        : Create vnpu success
    ```

 - 配置vNPU恢复状态。该参数用于设备重启时，设备能够保存vNPU配置信息，重启后，vNPU配置依然有效。

    命令格式：`npu-smi set -t vnpu-cfg-recover -d mode`

    mode表示vNPU的配置恢复使能状态，“1”表示开启状态，“0”表示关闭状态，默认为使能状态。

    执行如下命令设置vNPU的配置恢复状态，以下命令表示将vNPU的配置恢复状态设置为使能状态。

    ```bash title='bash input'
    npu-smi set -t vnpu-cfg-recover -d 1
    ```

    ```bash title='bash output example'
        Status                         : OK
        Message                        : The VNPU config recover mode Enable is set successfully.
    ```

 - 查询vNPU的配置恢复状态。

    以下命令表示查询当前环境中vNPU的配置恢复使能状态。
    
    ```bash title='bash input'
    npu-smi info -t vnpu-cfg-recover
    ```

 - 查询vNPU信息

    命令格式：npu-smi info -t info-vnpu -i id -c chip_id

    - id: 设备id。通过npu-smi info -l命令查出的NPU ID即为设备id。
    - chip_id: 芯片id。通过npu-smi info -m命令查出的Chip ID即为芯片id。

    ```bash title='bash input'
    npu-smi info -t info-vnpu -i 5 -c 1
    ```

    ```bash title='bash output example'
    Format:Free/Total                   NA: Currently, query is not supported.    |
     AICORE    Memory    AICPU    VPC    VENC    VDEC    JPEGD    JPEGE    PNGD    |
                GB
     4/8       21/42     3/7      6/12   1/3     6/12    8/16     4/8      NA/NA
    Total number of vnpu: 1 
    Vnpu ID  |  Vgroup ID     |  Container ID  |  Status  |  Template Name
    212      |  0             |  000000000000  |  0       |  vir04
    ```

    > 仍有4AICORE空闲 21GB内存空闲 3AICPU空闲 6核心VPC空闲 1核心VENC空闲 6核心VDEC空闲 8核心JPEGD空闲 4核心JPEGE空闲
    > 故可继续切分

## 继续切分
    
 - [虚拟化实例组合](https://www.hiascend.com/document/detail/zh/computepoweralloca/30rc3/cpaug/avi_00006.html?sub_id=%2Fzh%2Fcomputepoweralloca%2F30rc3%2Fcpaug%2Favi_00005.html)

 - 选择 vir04 + vir02_1c + vir01 + vir01 的虚拟化组合 继续进行切分

    ```bash title='bash input'
    npu-smi set -t create-vnpu -i 5 -c 1 -f vir02_1c
    npu-smi set -t create-vnpu -i 5 -c 1 -f vir01
    npu-smi set -t create-vnpu -i 5 -c 1 -f vir01
    ```
 
 - 查询vNPU信息

    ```bash title='bash input'
    npu-smi info -t info-vnpu -i 5 -c 1
    ```

 - 整理后的vNPU信息

    | 资源类型 | 空闲/总数 | 说明                  |
    |----------|----------|----------------------|
    | AICORE   | 0/8      | AI核心资源            |
    | Memory   | 0/42     | 内存资源（GB）        |
    | AICPU    | 0/7      | AI CPU资源            |
    | VPC      | 1/12     | 视频处理单元          |
    | VENC     | 1/3      | 视频编码单元          |
    | VDEC     | 1/12     | 视频解码单元          |
    | JPEGD    | 0/16     | JPEG解码单元          |
    | JPEGE    | 0/8      | JPEG编码单元          |
    | PNGD     | NA/NA    | PNG解码单元（不支持查询） |

    | Vnpu ID  | Vgroup ID | Container ID | Status | Template Name |
    |----------|----------|--------------|--------|---------------|
    | 212      | 0        | 000000000000 | 0      | vir04         |
    | 213      | 1        | 000000000000 | 0      | vir02_1c      |
    | 214      | 2        | 000000000000 | 0      | vir01         |
    | 215      | 2        | 000000000000 | 0      | vir01         |

 - 销毁指定vNPU

    命令格式：npu-smi set -t destroy-vnpu -i id -c chip_id -v vnpu_id

    ```bash title='bash input'
    npu-smi set -t destroy-vnpu -i 5 -c 1 -v 213
    ```

    显示如下信息表示销毁成功
    ```bash title='bash output example'
        Status                         : OK
        Message                        : Destroy vnpu 213 success
    ```

## [原生Docker使用vNPU](https://www.hiascend.com/document/detail/zh/computepoweralloca/30rc3/cpaug/avi_00012.html?sub_id=%2Fzh%2Fcomputepoweralloca%2F30rc3%2Fcpaug%2Favi_00005.html)


 - 使用方法
    
    - 查看vNPU设备名称

        ```bash title='bash input'
        ls /dev/ | grep davinci*
        ```

        > 此处我选择 `vdavinci213` 进行操作
    
    - **拉起容器时执行以下命令将vNPU挂载至容器中**

        ```bash title='bash input'
        docker run -it \
        --device=/dev/vdavinci213:/dev/davinci213 \
        --device=/dev/davinci_manager \
        --device=/dev/devmm_svm \
        --device=/dev/hisi_hdc \
        -v /usr/local/bin/npu-smi:/usr/local/bin/npu-smi \
        -v /home:/home \
        -v /usr/local/Ascend/driver/lib64/common:/usr/local/Ascend/driver/lib64/common \
        -v /usr/local/Ascend/driver/lib64/driver:/usr/local/Ascend/driver/lib64/driver \
        -v /etc/ascend_install.info:/etc/ascend_install.info \
        -v /usr/local/Ascend/driver/version.info:/usr/local/Ascend/driver/version.info \
        docker_image_id  /bin/bash
        ```

    - 容器启动之后，执行以下命令查看当前Docker容器中可以使用的davinci设备，如果有davinci设备则表示虚拟设备成功映射到容器中。

        ```bash title='bash input'
        ls /dev/ | grep davinci*
        ```

        > 如果有 `davinci213` 表明虚拟设备映射成功