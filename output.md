要在 Ubuntu 20 上静态编译支持 libfdk_aac 编解码器的 ffmpeg，可以按照以下步骤操作。静态编译可以将大部分依赖库打包进可执行文件，减少在目标系统（如群晖）上的依赖问题。

---

## 静态编译 ffmpeg 支持 libfdk_aac 的步骤

### 1. 安装必要的依赖

```bash
sudo apt-get update
sudo apt-get install -y autoconf automake build-essential cmake git libtool pkg-config nasm yasm libx264-dev libx265-dev libnuma-dev libfdk-aac-dev
```

> 注意：如果系统没有 libfdk-aac-dev，可以从源码编译安装 libfdk-aac（见步骤2）。

### 2. 编译并安装 libfdk-aac（如果没有）

```bash
git clone https://github.com/mstorsjo/fdk-aac.git
cd fdk-aac
autoreconf -fiv
./configure --prefix="$HOME/ffmpeg_build" --disable-shared
make
make install
cd ..
```

这里使用 `--disable-shared` 生成静态库，安装到 `$HOME/ffmpeg_build` 目录。

### 3. 下载 ffmpeg 源码

```bash
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
cd ffmpeg
```

### 4. 配置编译参数，启用静态编译和 libfdk_aac

```bash
PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
  --prefix="$HOME/ffmpeg_build" \
  --pkg-config-flags="--static" \
  --extra-cflags="-I$HOME/ffmpeg_build/include" \
  --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
  --extra-libs="-lpthread -lm" \
  --ld="gcc" \
  --enable-gpl \
  --enable-nonfree \
  --enable-libfdk_aac \
  --enable-static \
  --disable-shared \
  --disable-debug \
  --disable-doc \
  --disable-ffplay
```

说明：
- `--enable-static` 和 `--disable-shared` 让 ffmpeg 生成静态可执行文件。
- `--enable-nonfree` 是因为 libfdk_aac 是非自由许可。
- 关闭了 ffplay 和文档以简化编译。

### 5. 编译并安装

```bash
make -j$(nproc)
make install
```

### 6. 验证静态编译结果

```bash
ldd $HOME/ffmpeg_build/bin/ffmpeg
```

输出应显示 `not a dynamic executable`，表示是静态编译。

同时检查编码器：

```bash
$HOME/ffmpeg_build/bin/ffmpeg -encoders | grep libfdk_aac
```

应显示 `libfdk_aac` 编码器。

---

## 编译完成后

- 你可以将 `$HOME/ffmpeg_build/bin/ffmpeg` 上传到群晖使用。
- 由于是静态编译，依赖较少，更容易在不同系统上运行。
- 仍需确认群晖的 CPU 架构与编译架构一致。

---

如果需要，我可以帮你生成完整的脚本，或者指导交叉编译（如果群晖是 ARM 架构）。