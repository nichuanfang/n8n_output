docker-compose 配置 DNS 主要是在 docker-compose.yml 文件中通过 dns 或 dns_search 参数来设置。具体用法如下：

示例配置：

```yaml
version: '3'
services:
  web:
    image: nginx
    dns:
      - 8.8.8.8
      - 8.8.4.4
    dns_search:
      - example.com
```

说明：
- dns：指定容器使用的 DNS 服务器地址，可以是多个。
- dns_search：指定 DNS 搜索域。

默认情况下，Docker Compose 会使用宿主机的 DNS 配置。如果需要自定义 DNS 服务器，可以通过上述方式配置。

参考链接：
- [51CTO博客 docker-compose 设置dns](https://blog.51cto.com/u_16213400/7443051)
- [云原生实践 Docker Compose 配置容器网络的DNS解析](https://www.oryoy.com/news/jie-mi-docker-compose-ru-he-qing-song-pei-zhi-rong-qi-wang-luo-de-dns-jie-xi.html)

如果需要更详细的配置示例或遇到具体问题，可以告诉我，我帮你进一步解答。