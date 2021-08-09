# Centos7内核通过yum方式升级

````bash
    # 首先更新仓库(仅新系统安装使用一次，后续更新会导致一些特定版本应用被动升级)
    sudo yum -y update
    # 启用 ELRepo 仓库
    sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    sudo rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
    # 查看可用的系统内核包:
    sudo yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
    # 可安装的软件包
    # elrepo-release.noarch                  7.0-5.el7.elrepo        elrepo-kernel
    # kernel-lt.x86_64                       5.4.138-1.el7.elrepo    elrepo-kernel
    # kernel-lt-devel.x86_64                 5.4.138-1.el7.elrepo    elrepo-kernel
    # kernel-lt-doc.noarch                   5.4.138-1.el7.elrepo    elrepo-kernel
    # kernel-lt-headers.x86_64               5.4.138-1.el7.elrepo    elrepo-kernel
    # kernel-lt-tools.x86_64                 5.4.138-1.el7.elrepo    elrepo-kernel
    # kernel-lt-tools-libs.x86_64            5.4.138-1.el7.elrepo    elrepo-kernel
    # kernel-lt-tools-libs-devel.x86_64      5.4.138-1.el7.elrepo    elrepo-kernel
    # kernel-ml.x86_64                       5.13.8-1.el7.elrepo     elrepo-kernel
    # kernel-ml-devel.x86_64                 5.13.8-1.el7.elrepo     elrepo-kernel
    # kernel-ml-doc.noarch                   5.13.8-1.el7.elrepo     elrepo-kernel
    # kernel-ml-headers.x86_64               5.13.8-1.el7.elrepo     elrepo-kernel
    # kernel-ml-tools.x86_64                 5.13.8-1.el7.elrepo     elrepo-kernel
    # kernel-ml-tools-libs.x86_64            5.13.8-1.el7.elrepo     elrepo-kernel
    # kernel-ml-tools-libs-devel.x86_64      5.13.8-1.el7.elrepo     elrepo-kernel
    # perf.x86_64                            5.13.8-1.el7.elrepo     elrepo-kernel
    # python-perf.x86_64                     5.13.8-1.el7.elrepo     elrepo-kernel
    
    # 简单解释下几个名词
    # ELRepo 国外Linux的第三方免费软件资源库, an RPM repository for Enterprise Linux packages.
    # elrepo-kernel, ELRepo内核相关的库, the ELRepo kernel repository is called elrepo-kernel.
    # kernel-lt基于长期支持分支, the Long Term Support version of the latest Linux kernel.
    # kernel-ml基于主线稳定分支, Mainline Stable version of the latest Linux kernel.

    # 选择安装最新内核 kernel-ml  --enablerepo 选项开启 CentOS 系统上的指定仓库。默认开启的是 elrepo，这里用 elrepo-kernel 替换
    sudo yum --enablerepo=elrepo-kernel install kernel-ml
    # 已安装:
    #   kernel-ml.x86_64 0:5.13.8-1.el7.elrepo

    # 设置 grub2
    # 内核安装好后，需要设置为默认启动选项并重启后才会生效

    # 查看系统上的所有可以内核
    sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
    # 0 : CentOS Linux (5.13.8-1.el7.elrepo.x86_64) 7 (Core)
    # 1 : CentOS Linux (3.10.0-957.21.3.el7.x86_64) 7 (Core)
    # 2 : CentOS Linux (3.10.0-957.el7.x86_64) 7 (Core)
    # 3 : CentOS Linux (0-rescue-20190711105006363114529432776998) 7 (Core)

    # 我们可以选择 序号 0 的内核
    sudo grub2-set-default 0

    # 生成启动配置
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg

    # 准备重启下机器生效
    sudo reboot
````