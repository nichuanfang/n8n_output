群晖（Synology NAS）系统基于 Linux，但它的包管理和系统结构与常见的 Linux 发行版不同，通常不建议直接在群晖上从源码编译安装软件，因为可能会影响系统稳定性和升级。

如果你是在群晖上通过某些第三方途径（比如 Entware、Docker 或者手动编译）安装的 ffmpeg 和 libfdk_aac，卸载方法会有所不同：

1. 如果是通过 Entware 安装的：

Entware 是群晖常用的第三方包管理器，卸载命令类似于：

```bash
opkg remove ffmpeg
opkg remove libfdk-aac
```

2. 如果是通过 Docker 运行的 ffmpeg：

直接删除对应的 Docker 容器和镜像即可。

3. 如果是你手动从源码编译安装的：

请参考我之前给你的手动删除文件的方法，默认安装路径一般是 `/usr/local`，你可以用 SSH 登录群晖，执行：

```bash
sudo rm /usr/local/bin/ffmpeg
sudo rm /usr/local/bin/ffprobe
sudo rm /usr/local/bin/ffplay
sudo rm -rf /usr/local/include/libav*
sudo rm -rf /usr/local/lib/libav*
sudo rm -rf /usr/local/share/ffmpeg
sudo rm -rf /usr/local/lib/pkgconfig/libav*
sudo rm -rf /usr/local/include/fdk-aac
sudo rm -f /usr/local/lib/libfdk-aac.*
sudo rm -f /usr/local/lib/pkgconfig/fdk-aac.pc
```

注意：群晖系统默认没有 sudo，可能需要切换到 root 用户或者使用 `sudo` 命令（如果已配置）。

4. 如果你不确定安装方式，建议先确认：

- 你是否用 Entware 安装过？
- 是否用 Docker 运行过？
- 是否手动编译安装过？

你可以告诉我具体安装方式，我帮你确认卸载步骤。