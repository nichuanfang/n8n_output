从你提供的Nacos配置截图看，配置格式是YAML，内容包含了nacos的服务发现配置和数据源配置。要支持配置动态刷新，通常需要注意以下几点：

1. 配置格式和内容是否正确：
   - 你的YAML格式看起来是正确的，缩进和层级关系合理。
   - nacos.discovery部分配置了server-addr、namespace、group等，符合Nacos客户端配置规范。
   - 其他配置如datasource也符合常见格式。

2. 支持动态刷新：
   - 仅配置Nacos客户端是不够的，还需要在Spring Boot项目中引入相关依赖，比如`spring-cloud-starter-alibaba-nacos-config`。
   - 需要在Spring Boot启动类或者配置类上添加`@RefreshScope`注解，标注需要动态刷新的Bean。
   - 需要在配置文件中开启配置自动刷新功能，比如`spring.cloud.nacos.config.refresh-enabled=true`。
   - 确保使用了Spring Cloud Alibaba版本支持动态刷新。

3. 其他建议：
   - 你的namespace配置是一个UUID格式，确认是否正确对应Nacos控制台中的命名空间ID。
   - group配置为DEFAULT_GROUP，确认是否和Nacos控制台配置一致。
   - server-addr配置了多个地址，格式正确。

总结：
- 配置本身看起来没明显错误。
- 需要确认项目中引入了Nacos配置相关依赖，并且开启了动态刷新相关配置。
- 需要在需要刷新的Bean上添加`@RefreshScope`注解。

如果你需要，我可以帮你给出一个支持Nacos配置动态刷新的Spring Boot示例配置。你需要吗？