# Centos7安装docker指定版本

````bash
    # step 1: 安装必要的一些系统工具
    sudo yum install -y yum-utils device-mapper-persistent-data lvm2
    # step 2: 添加软件源信息
    sudo yum-config-manager --add-repo \
        http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    # step 3: 更新并安装 Docker-CE
    sudo yum makecache fast
    # 可选移除旧版本
    sudo yum remove -y docker-ce.x86_64 docker-ce-cli.x86_64 containerd.io
    # 显示可安装版本
    sudo yum list available docker-ce --showduplicates
    # 举例 安装 docker 18.09.9
    sudo yum install -y docker-ce-18.09.9 docker-ce-cli-18.09.9 containerd.io
    # 举例 安装 docker 19.03.15
    sudo yum install -y docker-ce-19.03.15 docker-ce-cli-19.03.15 containerd.io
    # step 4: 开启Docker服务
    sudo systemctl start docker && sudo systemctl enable docker && sudo systemctl status docker
    # step 5: 创建默认网络名
    sudo docker network create app-net
    # docker执行权限分配
    # sudo usermod -aG docker your-user
    # docker执行权限分配（当前用户）
    # sudo usermod -aG docker $(whoami)
    # docker执行权限分配给 dev用户
    sudo usermod -aG docker dev
    # 重新登录以后测试
    docker -v

    # https://github.com/docker/for-linux/issues/264
    # Support host.docker.internal DNS name to host
    # ip -4 route list match 0/0 | awk '{print $3" host.docker.internal"}'
    # 在容器中设定 docker.host.internal
    echo -e "`/sbin/ip route|awk '/default/ { print $3 }'`\tdocker.host.internal" | sudo tee -a /etc/hosts > /dev/null
````