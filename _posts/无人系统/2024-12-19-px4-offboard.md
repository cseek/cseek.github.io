---
title: px4 的 offboard 模式仿真
category: [无人系统]
tags: [px4, offboard, gazebo]
---

> PX4 提供了两重仿真方式， 一种是采用 sitl, 另一种是采用 gazebo 的方式， 这里主要记录了使用 gazebo 的仿真步骤。
{: .prompt-info }

## 1、安装 QGC
下载地址为[点击下载](https://github.com/mavlink/qgroundcontrol/releases/download/v4.4.4/QGroundControl-installer.exe)

## 2、克隆 PX4 飞控代码并安装编译环境

本文采用的是 WSL/ubuntu20.04 搭建的环境 

```bash
git clone -b v1.15.4 https://github.com/PX4/PX4-Autopilot.git --recursive
git submodule update --init --recursive
sudo apt install gazebo9-common libgazebo9
sudo apt-get install gazebo11 libgazebo11-dev
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 67170598AF249743
sudo apt update
sudo apt upgrade
# 这一步会比较慢，因为要下载交叉编译器
./Tools/setup/ubuntu.sh
```

## 3、运行仿真环境

```bash
cd PX4-Autopilot
make px4_sitl_default gazebo
```

## 4、在 PX4 的命令行环境下启动 mavlink 通信

```bash
mavlink start -p -o 14550
```

## 5、使用 QGC 连接飞机
+ Application Settings -> 通讯连接 -> 添加:
+ Name: 随便写一个
+ Type：UDP
+ Port: 14550
+ Server Address: 启动 PX4 仿真环境的主机 IP

## 6、使用 mavsdk 发送指令控制无人机

+ 安装 mavsdk:

```bash
pip3 install mavsdk
```

+ 编写控制飞机飞行轨迹的代码，这里控制飞机飞行圆形轨迹：

```bash
# test_offborad.py
import time
import asyncio
import numpy as np
from mavsdk import System
from mavsdk.offboard import VelocityBodyYawspeed

class YawPriorityController:
    def __init__(self):
        self.t = 10

    async def run(self):
        # 连接无人机（保持原样）
        drone = System()
        await drone.connect(system_address="udp://:14540")

        print("等待连接...")
        async for state in drone.core.connection_state():
            if state.is_connected:
                break

        print("解锁中...")
        await drone.action.arm()

        print("起飞...")
        await drone.action.takeoff()

        await asyncio.sleep(10)  # 等待到达安全高度
        velocity = VelocityBodyYawspeed(forward_m_s=1.0, right_m_s=0.0, down_m_s=0.0, yawspeed_deg_s=0.0)
        await drone.offboard.set_velocity_body(velocity)

        print("进入Offboard模式")
        await drone.offboard.start()

        while True:
            velocity1 = VelocityBodyYawspeed(forward_m_s=float(7), right_m_s=0.0, down_m_s=float(-1.0), yawspeed_deg_s=float(-10))
            # 发送指令（保持原样）
            await drone.offboard.set_velocity_body(velocity1)
            await asyncio.sleep(2)  # 保持控制频率

        await drone.offboard.stop()
        await drone.action.land()

# 测试运行
controller = YawPriorityController()
asyncio.run(controller.run())
```
在上面的代码中：
`forward_m_s`: 前进的速度
`right_m_s`: 往左平移的速度
`down_m_s`: 飞机下降的速度，为负值则为上升
`yawspeed_deg_s`: 飞机 yaw 轴的偏转角度，如果设置为正值，从上往下看为顺时针方向偏转，反之则反。

+ 运行代码

```bash
python3 test_offborad.py
```

+ 观察现象
可以看到，飞机在地图上按照圆形轨迹飞行
![](assets/img/post/无人系统/offboard.png)

## 7、问题解决

+ setuptools 版本兼容性问题。

```bash
Installing PX4 Python3 dependencies
Collecting argcomplete (from -r /home/cyber/PX4/PX4-Autopilot/Tools/setup/requirements.txt (line 1))
  Using cached argcomplete-3.6.1-py3-none-any.whl.metadata (16 kB)
Collecting argparse>=1.2 (from -r /home/cyber/PX4/PX4-Autopilot/Tools/setup/requirements.txt (line 2))
  Using cached argparse-1.4.0-py2.py3-none-any.whl.metadata (2.8 kB)
Requirement already satisfied: cerberus in /usr/local/lib/python3.8/dist-packages (from -r /home/cyber/PX4/PX4-Autopilot/Tools/setup/requirements.txt (line 3)) (1.3.7)
Collecting coverage (from -r /home/cyber/PX4/PX4-Autopilot/Tools/setup/requirements.txt (line 4))
  Using cached coverage-7.6.1-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (8.3 kB)
Collecting empy==3.3.4 (from -r /home/cyber/PX4/PX4-Autopilot/Tools/setup/requirements.txt (line 5))
  Using cached empy-3.3.4.tar.gz (62 kB)
  Preparing metadata (setup.py) ... error
  error: subprocess-exited-with-error

  × python setup.py egg_info did not run successfully.
  │ exit code: 1
  ╰─> [30 lines of output]
      Traceback (most recent call last):
        File "<string>", line 2, in <module>
        File "<pip-setuptools-caller>", line 34, in <module>
        File "/tmp/pip-install-seqe1h3n/empy_72464e454cd540bea62af97047a73d89/setup.py", line 24, in <module>
          setup(
        File "/usr/local/lib/python3.8/dist-packages/setuptools/_distutils/core.py", line 183, in setup
          return run_commands(dist)
        File "/usr/local/lib/python3.8/dist-packages/setuptools/_distutils/core.py", line 199, in run_commands
          dist.run_commands()
        File "/usr/local/lib/python3.8/dist-packages/setuptools/_distutils/dist.py", line 954, in run_commands
          self.run_command(cmd)
        File "/usr/local/lib/python3.8/dist-packages/setuptools/dist.py", line 999, in run_command
          super().run_command(command)
        File "/usr/local/lib/python3.8/dist-packages/setuptools/_distutils/dist.py", line 973, in run_command
          cmd_obj.run()
        File "/usr/local/lib/python3.8/dist-packages/setuptools/command/egg_info.py", line 312, in run
          self.find_sources()
        File "/usr/local/lib/python3.8/dist-packages/setuptools/command/egg_info.py", line 320, in find_sources
          mm.run()
        File "/usr/local/lib/python3.8/dist-packages/setuptools/command/egg_info.py", line 546, in run
          self.prune_file_list()
        File "/usr/local/lib/python3.8/dist-packages/setuptools/command/sdist.py", line 162, in prune_file_list
          super().prune_file_list()
        File "/usr/local/lib/python3.8/dist-packages/setuptools/_distutils/command/sdist.py", line 380, in prune_file_list
          base_dir = self.distribution.get_fullname()
        File "/usr/local/lib/python3.8/dist-packages/setuptools/_core_metadata.py", line 267, in get_fullname
          return _distribution_fullname(self.get_name(), self.get_version())
        File "/usr/local/lib/python3.8/dist-packages/setuptools/_core_metadata.py", line 285, in _distribution_fullname
          canonicalize_version(version, strip_trailing_zero=False),
      TypeError: canonicalize_version() got an unexpected keyword argument 'strip_trailing_zero'
      [end of output]

  note: This error originates from a subprocess, and is likely not a problem with pip.
error: metadata-generation-failed

× Encountered error while generating package metadata.
╰─> See above for output.

note: This is an issue with the package mentioned above, not pip.
hint: See above for details.
```

解决办法：

```bash
# 强制降级
python3 -m pip install setuptools==58.2.0
```
