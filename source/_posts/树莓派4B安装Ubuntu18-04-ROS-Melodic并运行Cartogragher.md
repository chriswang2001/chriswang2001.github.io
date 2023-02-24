---
title: 树莓派4B安装Ubuntu18.04+ROS_Melodic并运行Cartogragher
date: 2023-02-24 21:31:55
tags: SLAM
---

## 安装Ubuntu18.04

1. 下载ubuntu18.04镜像
    
    目前官方系统已经升级到ubuntu22.04LTS，官网页面没有提供历史版本的下载。所以本文从清华的开源镜像网站下载[ubuntu18.04镜像](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cdimage/releases/bionic/release/)。目前只有server版，需要后面手动安装桌面。树莓派4B选择arm64+raspi4。
    ![20230224213943](https://image-1305582579.cos.ap-chengdu.myqcloud.com/20230224213943.png)

1. 烧录镜像

    使用树莓派官方工具Raspberry Pi Imager烧录镜像。[官网下载地址](https://www.raspberrypi.com/software/)
    ![20230224214326](https://image-1305582579.cos.ap-chengdu.myqcloud.com/20230224214326.png)

    选择自定义镜像
    ![20230224214509](https://image-1305582579.cos.ap-chengdu.myqcloud.com/20230224214509.png)

    高级设置里面配置用户名、网络等信息
    ![20230224214648](https://image-1305582579.cos.ap-chengdu.myqcloud.com/20230224214648.png)
    点击烧录后等待烧录完成

1. 将SD卡插入树莓派，连接HDMI显示器，启动树莓派。

    如果显示屏无信号，修改SD卡下的config.txt文件, 增加如下语句：

    ``` bash
    hdmi_force_hotplug=1 #强制使用HDMI输出（强行认为HDMI口已经插入了设备）
    config_hdmi_boost=4 #HDMI信号增强。（1~9）
    ```

1. 连接网络
    如果在烧录前正确配置了网路信息，此时应该可以成功连接网络。
    没有的话，可修改SD卡目录下的network-config文件。（注意：务必使用空格缩进）

    ```bash
    wifis:
        wlan0:
            access-points:
            "SSID":
                password: "Password"
            dhcp4: true
            optional: true
    ```

    如果上述操作仍无法连接网络，可开机后修改/etc/netplan/50-cloud-init.yaml文件。

    ```bash
    network:
        version: 2
        renderer: networkd
        ethernets:
            eth0:
                dhcp4: false
                optional: true
        wifis:
            wlan0:
            access-points:
                SSID:
                password: Password
            dhcp4: false
            optional: true
    ```

    修改完成后执行下述命令

    ```bash
    sudo netplan generate
    sudo netplan apply
    ```

1. 修改软件源

    修改/etc/apt/sources.list，为如下内容（清华源，arm架构使用ubuntu-ports)

    ``` bash
    # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main restricted universe multiverse

    # 预发布软件源，不建议启用
    # deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-proposed main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-proposed main restricted universe multiverse
    ```

    刷新软件源

    ```bash
    sudo apt update
    ```

1. 安装桌面

    后期使用ROS的rviz, gazebo等仿真工具，需要桌面图形环境。本文选择ubuntu系统默认的desktop。

    ```bash
    sudo apt install ubuntu-desktop
    ```

    安装完成后重启，就能看见图形界面了

    需要注意ubuntu图形界面采用NetworkManager进行网络管理，所以如需使用图形界面连接网络，需要将/etc/netplan/50-cloud-init.yaml中的 renderer: networkd 改为 renderer: NetworkManager

## 安装ROS_Melodic 和 Cartograher

1. 参照[ROS官网教程](http://wiki.ros.org/melodic/Installation/Ubuntu) 安装

    其中sudo rosdep init 需要科学上网，并更改dns（或修改hosts）

1. 参照[Cartograher官网教程](https://google-cartographer-ros.readthedocs.io/en/latest/compilation.html) 安装

    ```bash
    rosdep install --from-paths src --ignore-src --rosdistro=${ROS_DISTRO} -y
    ```

    执行会出错。你需要删除或者注释在cartographer包下(不是cartographer_ros) package.xml文件里的(<depend>libabsl-dev</depend>)。

    ```bash
    catkin_make_isolated --install --use-ninja
    ```

    执行到中间会卡死。原因是没有设置swap分区，导致内存不足。根据[修改树莓派交换分区 SWAP 的正确姿势](https://shumeipai.nxez.com/2017/12/18/how-to-modify-raspberry-pi-swap-partition.html)这篇博客设置即可

## 远程桌面

1. 打开ubuntu自带的远程桌面
    ![20230224222809](https://image-1305582579.cos.ap-chengdu.myqcloud.com/20230224222809.png)

1. 在本地电脑上启动 Vnc Client，输入IP地址连接

1. 为了后面能够无需显示器，开机后就可以连接远程桌面。还需要设置[自动登录](https://blog.csdn.net/mbdong/article/details/114069882)和[虚拟显示器](https://askubuntu.com/questions/1033436/how-to-use-ubuntu-18-04-on-vnc-without-display-attached)
