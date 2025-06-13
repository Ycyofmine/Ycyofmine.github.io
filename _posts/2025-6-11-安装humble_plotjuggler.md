---
title: 自行编译ros2-humble-plotjuggler
date: 2025-6-11 10:22:30 +0800
categories: [机器人]
tags: [robot] # TAG names should always be lowercase
media_subpath: /assets/img/robot
---
不知道什么原因 `ros2-humble` 的环境下，`sudo apt install ros-$ROS_DISTRO-plotjuggler-ros` 失效了，库里没有这个包。

只能通过自行编译的方式来安装 plotjuggler

```shell
mkdir -p ~/ws_plotjuggler/src
cd ~/ws_plotjuggler/src
git clone https://github.com/PlotJuggler/plotjuggler_msgs.git
git clone https://github.com/facontidavide/PlotJuggler.git
git clone https://github.com/PlotJuggler/plotjuggler-ros-plugins.git
```

然后编译

```shell
cd ~/ws_plotjuggler
rosdep install --from-paths src --ignore-src -y
colcon build
```
在 `build` 的时候你可能会遇到以下的报错。

```shell
ERROR: your rosdep installation has not been initialized yet.

/usr/bin/ld: ... undefined reference to `lua_remove'
/usr/bin/ld: ... undefined reference to `lua_insert'
/usr/bin/ld: ... undefined reference to `luaL_loadbuffer'
/usr/bin/ld: ... undefined reference to `lua_pcall'
/usr/bin/ld: ... undefined reference to `lua_newuserdata'
...
collect2: error: ld returned 1 exit status
```

这是rosdep未初始化和lua库缺失导致的

1. 我们先解决rosdep的问题
    
    ```shell
    sudo rosdep init
    rosdep update
    rosdep install --from-paths src --ignore-src -y -r
    ```

2. 安装lua库

    ```shell
    sudo apt-get update
    sudo apt-get install liblua5.3-dev
    ```

3. 清理并编译

    ```shell
    cd ~/ws_plotjuggler
    rm -rf build/ install/ log/
    colcon build --symlink-install
    ```

最后 `ros2 run plotjuggler plotjuggler` 即可