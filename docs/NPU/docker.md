# 国产银河麒麟V10系统 aarch64架构 离线安装Docker

1. 信息确认

    - 操作系统版本

        ```bash title='bash input'
        uname -a
        
        ```

        ```bash title='bash output example'
        Linux localhost.localdomain 4.19.90-24.4.v2101.ky10.aarch64 #1 SMP Mon May 24 14:45:37 CST 2021 aarch64 aarch64 aarch64 GNU/Linux
        ```

    - 操作系统架构

        ```bash title='bash input'
        uname -m
        ```

        ```bash title='bash output example'
        aarch64
        ```

2. 下载Docker 和 Docker-Compose


    2.1 离线下载Docker安装包 下载完成后 通过内网传输至服务器内 

    - [Docker Linux 稳定版本下载地址](https://download.docker.com/linux/static/stable/)

    以该机器为例：可直接点击该链接进行下载aarch64架构的docker27.0.2版本：

    - [aarch64/docker-27.0.2.tgz](https://download.docker.com/linux/static/stable/aarch64/docker-27.0.2.tgz)


    2.2 离线下载Docker-Compose安装包

    - [Docker Compose Github Releases](https://github.com/docker/compose/releases)

    以该机器为例：可直接点击该链接进行下载aarch64架构的docker27.0.2版本：

    - [docker-compose-linux-aarch64](https://github.com/docker/compose/releases/download/v2.29.1/docker-compose-linux-aarch64)

3. 安装Docker 和 Docker-Compose

    - 解压Docker安装包

        ```bash title='bash input'
        tar -xvf docker-27.0.2.tgz
        ```

    - 制作docker.service文件

        ```bash title='bash input'
        cat > docker.service <<EOF
        docker.service
  
        [Unit]
        Description=Docker Application Container Engine
        Documentation=https://docs.docker.com
        After=network-online.target firewalld.service
        Wants=network-online.target

        [Service]
        Type=notify
        # the default is not to use systemd for cgroups because the delegate issues still
        # exists and systemd currently does not support the cgroup feature set required
        # for containers run by docker
        ExecStart=/usr/bin/dockerd
        ExecReload=/bin/kill -s HUP $MAINPID
        # Having non-zero Limit*s causes performance problems due to accounting overhead
        # in the kernel. We recommend using cgroups to do container-local accounting.
        LimitNOFILE=infinity
        LimitNPROC=infinity
        LimitCORE=infinity
        # Uncomment TasksMax if your systemd version supports it.
        # Only systemd 226 and above support this version.
        #TasksMax=infinity
        TimeoutStartSec=0
        # set delegate yes so that systemd does not reset the cgroups of docker containers
        Delegate=yes
        # kill only the docker process, not all processes in the cgroup
        KillMode=process
        # restart the docker process if it exits prematurely
        Restart=on-failure
        StartLimitBurst=3
        StartLimitInterval=60s

        [Install]
        WantedBy=multi-user.target
        EOF
        ```

    - 复制Docker文件夹至/usr/bin目录下

        ```bash title='bash input'
        cp -a docker/* /usr/bin/
        ```

    - 将docker-compose文件复制到/usr/local/bin/目录下
    
        ```bash title='bash input'
        cp docker-compose-linux-aarch64 /usr/local/bin/docker-compose
        ```

    - 添加执行权限

        ```bash title='bash input'
        chmod +x /usr/local/bin/docker-compose
        ```

    - 将 docker.service 移到 /etc/systemd/system/ 目录
    
        ```bash title='bash input'
        cp docker.service /etc/systemd/system/
        ```

    - 设置docker.service文件权限
    
        ```bash title='bash input'
        chmod +x /etc/systemd/system/docker.service
        ```

    - 重载配置文件
    
        ```bash title='bash input'
        systemctl daemon-reload
        ```
    
    - 启动docker服务
    
        ```bash title='bash input'
        systemctl start docker
        ```

        
    - 设置开机自启
    
        ```bash title='bash input'
        systemctl enable docker.service
        ```
    
        ```bash title='bash output'
        Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /etc/systemd/system/docker.service.
        ```

    - 验证docker是否安装成功
    
        ```bash title='bash input'
        docker --version
        ```

        ```bash title='bash output'
        Docker version 27.0.2, build 912c1dd
        ```

    - 验证docker-compose是否安装成功
    
        ```bash title='bash input'
        docker-compose --version
        ```

        ```bash title='bash output'
        Docker Compose version v2.29.1
        ```


4. Docker 镜像加速

    - 编辑 /etc/docker/daemon.json 文件
    
        ```bash title='bash input'
        mkdir -p /etc/docker
        cat > /etc/docker/daemon.json <<EOF
        {
            "registry-mirrors": ["https://dockerhub.icu"]
        }
        EOF
        ```

    - 重启docker服务
    
        ```bash title='bash input'
        systemctl restart docker
        ```

    - 测试镜像加速是否生效
    
        ```bash title='bash input'
        docker info
        ```

        ```bash title='bash output'
        ...
        Registry Mirrors:
         https://dockerhub.icu/
        ...
        ```
