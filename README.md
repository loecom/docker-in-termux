在 Termux 中安装 Docker
此存储库包含有关如何在 Termux 中安装 Docker 的说明，Termux 是一个强大的 Android 终端仿真器。

先决条件
在继续安装之前，请确保您满足以下先决条件：
安装了 Termux 的 Android 设备。你可以从 F-Droid 应用商店下载 Termux。
稳定的互联网连接。

安装步骤
请按照以下步骤在 Termux 中安装 Docker：
在您的 Android 设备上打开 Termux。
通过运行以下命令更新和升级软件包：
pkg update -y && pkg upgrade -y
通过运行以下命令安装必要的依赖项：
pkg install qemu-utils qemu-common qemu-system-x86_64-headless wget -y
创建单独的目录：
mkdir alpine && cd alpine
下载 Alpine Linux 3.20.2 （virt optimized） ISO：
wget http://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-virt-3.20.2-x86_64.iso
创建磁盘（注意它实际上不会占用 5GB 的空间，更像是 500-600MB）：
qemu-img create -f qcow2 alpine.img 50G
启动它： 这里我们使用 1024MB 内存和 2 个 CPU
qemu-system-x86_64 -machine q35 -m 1024 -smp cpus=2 -cpu qemu64 -drive if=pflash,format=raw,read-only=on,file=$PREFIX/share/qemu/edk2-x86_64-code.fd -netdev user,id=n1,dns=8.8.8.8,hostfwd=tcp::2222-:22 -device virtio-net,netdev=n1 -cdrom alpine-virt-3.20.2-x86_64.iso -nographic alpine.img
您可以使用nprocfree -m | grep -oP '\d+' | head -n 1

使用用户名登录（无密码）root

设置网络（按 Enter 键使用默认值）：

localhost:~# setup-interfaces
 Available interfaces are: eth0.
 Enter '?' for help on bridges, bonding and vlans.
 Which one do you want to initialize? (or '?' or 'done') [eth0]
 Ip address for eth0? (or 'dhcp', 'none', '?') [dhcp]
 Do you want to do any manual network configuration? [no]
本地主机：~#

ifup eth0
创建 answerfile 以加快安装速度：
wget https://raw.githubusercontent.com/cyberkernelofficial/docker-in-termux/main/answerfile
注意：如果您看到如下错误： .然后运行此命令wget: bad address 'gist.githubusercontent.com'

echo -e "nameserver 192.168.1.1\nnameserver 1.1.1.1" > /etc/resolv.conf
用于在引导时启用串行控制台输出的补丁：setup-disk
sed -i -E 's/(local kernel_opts)=.*/\1="console=ttys0"/' /usr/sbin/setup-disk
运行 setup 以安装到磁盘
setup-alpine -f answerfile
安装完成后，关闭 VM 的电源（命令poweroff)

在没有 cdrom 的情况下再次启动：

qemu-system-x86_64 -M q35 -m 1024 -smp cpus=2 -cpu qemu64 -drive if=pflash,format=raw,read-only=on,file=$PREFIX/share/qemu/edk2-x86_64-code.fd -netdev user,id=n1,dns=8.8.8.8,hostfwd=tcp::2222-:22 -device virtio-net,netdev=n1 -nographic alpine.img
A - 在文本编辑器中，编写以下内容：nano run_qemu.sh

#!/bin/bash
qemu-system-x86_64 -M q35 -m 1024 -smp cpus=2 -cpu qemu64 -drive if=pflash,format=raw,read-only=on,file=$PREFIX/share/qemu/edk2-x86_64-code.fd -netdev user,id=n1,dns=8.8.8.8,hostfwd=tcp::2222-:22 -device virtio-net,netdev=n1 -nographic alpine.img
保存并关闭文件。在 nano 中，您可以通过按 Ctrl+X，然后按 Y 确认保存，然后按 Enter 确认文件名来执行此作。

B - chmod 命令：chmod +x run_qemu.sh

C -./run_qemu.sh

更新系统并安装 docker：
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "nameserver 8.8.4.4" >> /etc/resolv.conf

apk update && apk add docker
启动 docker：
service docker start
在启动时启用 docker：
rc-update add docker
检查 docker install 是否成功：
docker run hello-world
一些有用的键
Ctrl+a x：退出仿真
Ctrl+a h： 切换 QEMU 控制台
用法
现在 Docker 已安装在 Termux 中，您可以开始使用它来管理和运行 Android 设备上的容器。有关如何使用 Docker 的更多信息，请参阅 Docker 官方文档。

贡献
如果您在安装过程中遇到任何问题或有改进建议，请随时打开一个 issue 或提交 pull request。

确认
本文灵感来自： https://gist.github.com/oofnikj/e79aef095cd08756f7f26ed244355d62
许可证
本项目根据 MIT 许可证获得许可。
