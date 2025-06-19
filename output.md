目前没有官方直接提供的Ubuntu 20预编译且带libfdk_aac支持的FFmpeg二进制包，因为libfdk_aac属于专有编码器，很多官方仓库默认不包含。

不过，你可以参考以下几种方式快速获得带libfdk_aac的FFmpeg：

1. **第三方静态编译版本**  
   有一些社区或个人会发布静态编译的FFmpeg版本，带有libfdk_aac支持。你可以在GitHub或相关论坛搜索“ffmpeg static build libfdk_aac ubuntu”，找到合适版本下载。

2. **使用PPA仓库**  
   某些PPA仓库可能提供带libfdk_aac的FFmpeg版本，例如`jonathonf/ffmpeg-4`等，但需要确认是否包含libfdk_aac支持。你可以尝试：
   ```bash
   sudo add-apt-repository ppa:jonathonf/ffmpeg-4
   sudo apt update
   sudo apt install ffmpeg
   ffmpeg -encoders | grep libfdk_aac
   ```
   如果显示有libfdk_aac编码器，则说明支持。

3. **自行编译（推荐）**  
   虽然麻烦，但自行编译能确保最新版本和完整功能，且安全可靠。

---

如果你希望我帮你找具体的第三方静态编译版本下载链接，或者帮你写详细的编译脚本，请告诉我。