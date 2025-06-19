要让你的 Linux 服务器上的 ffmpeg 支持 libfdk_aac 编码器，通常需要重新编译 ffmpeg，并在编译时启用 libfdk_aac 支持。因为大多数 Linux 发行版自带的 ffmpeg 版本默认不包含 libfdk_aac 支持，主要是因为许可问题。

以下是一个通用的步骤指南，帮助你从源码编译支持 libfdk_aac 的 ffmpeg：

1. 安装依赖
确保你安装了编译 ffmpeg 所需的依赖，包括编译工具和库：

```bash
sudo apt-get update
sudo apt-get install -y autoconf automake build-essential cmake git libass-dev libfreetype6-dev libgnutls28-dev libmp3lame-dev libnuma-dev libopus-dev libtool libvorbis-dev libvpx-dev pkg-config texinfo wget yasm zlib1g-dev nasm libfdk-aac-dev
```

注意：`libfdk-aac-dev` 是 libfdk_aac 的开发库，如果你的系统仓库没有，可以从源码安装。

2. 下载并编译 libfdk_aac（如果系统没有）

```bash
git clone https://github.com/mstorsjo/fdk-aac.git
cd fdk-aac
autoreconf -fiv
./configure --prefix=/usr/local
make
sudo make install
cd ..
```

3. 下载 ffmpeg 源码

```bash
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
cd ffmpeg
```

4. 配置编译选项，启用 libfdk_aac

```bash
./configure --prefix=/usr/local --enable-gpl --enable-nonfree --enable-libfdk_aac
```

5. 编译并安装

```bash
make -j$(nproc)
sudo make install
```

6. 验证

```bash
ffmpeg -encoders | grep libfdk_aac
```

如果看到 `libfdk_aac` 编码器，说明支持成功。

总结：
- 你需要先安装 libfdk_aac 库。
- 重新编译 ffmpeg，配置时加上 `--enable-libfdk_aac --enable-nonfree`。
- 编译完成后安装使用。

如果你需要，我可以帮你生成更详细的脚本或步骤。你也可以告诉我你使用的 Linux 发行版，我帮你定制安装步骤。