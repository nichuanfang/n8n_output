你这张截图展示了两条音频转换命令，分别是转换成 mp3 和 aac 格式，默认比特率都是 320 kbps，使用的是 ffmpeg 命令。

如果你想要类似的格式和命令，这里给你补充几个常见音频格式的示例，格式、默认比特率和对应的 ffmpeg 命令：

| 名称         | 目标格式 | 默认比特率 | 命令示例                                                                                  |
|--------------|----------|------------|-------------------------------------------------------------------------------------------|
| mp3 audio    | mp3      | 320        | ffmpeg -i %s -ss %t -map 0:a:0 -b:a 320k -y 0.mp3                                        |
| aac audio    | aac      | 320        | ffmpeg -i %s -ss %t -map 0:a:0 -c:a aac -b:a 320k -vn -f adts 0.aac                       |
| wav audio    | wav      | 无损       | ffmpeg -i %s -ss %t -map 0:a:0 -c:a pcm_s16le -vn 0.wav                                  |
| ogg audio    | ogg      | 192        | ffmpeg -i %s -ss %t -map 0:a:0 -c:a libvorbis -b:a 192k -vn 0.ogg                         |
| flac audio   | flac     | 无损       | ffmpeg -i %s -ss %t -map 0:a:0 -c:a flac -vn 0.flac                                     |

其中 `%s` 是输入文件，`%t` 是开始时间，`-map 0:a:0` 表示选择第一个音频流。

如果你需要我帮你生成某个具体格式的命令，或者想要添加到你的列表里，可以告诉我。