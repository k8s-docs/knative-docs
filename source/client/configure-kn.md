# 定制 kn

您可以通过创建一个`config.yaml`配置文件来定制您的`kn`命令行设置。
您可以通过使用`——config`标志来提供此配置，否则配置将从默认位置获取。
默认配置位置符合[XDG基本目录规范](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)，对于Unix系统和Windows系统是不同的。

- 如果设置了`XDG_CONFIG_HOME`环境变量，`kn`寻找的默认配置位置是`$XDG_CONFIG_HOME/kn`。
- 如果没有设置`XDG_CONFIG_HOME`环境变量，`kn`在用户的主目录`$HOME/.config/kn/config.yaml`中查找配置。
- 对于Windows系统，默认的`kn`配置位置是`%APPDATA%\kn`。

## 示例配置文件

```yaml
plugins:
  path-lookup: true
  directory: ~/.config/kn/plugins
eventing:
  sink-mappings:
  - prefix: svc
    group: core
    version: v1
    resource: services
```

哪里

- `path-lookup` 指定`kn`是否应该在`PATH`环境变量中查找[plugins](kn-plugins.md)。
  这是一个布尔配置选项(默认值:`true`)。
  注意: `path-lookup` 选项已弃用，将在将来无条件启用路径查找的版本中删除。
- `directory` 指定`kn`查找插件的目录。
  如前所述，默认路径取决于操作系统。
  这可以是用户可见的任何目录(默认:`$base_dir/plugins`，其中`$base_dir`是存储配置文件的目录)。
- `sink-mappings`定义了Kubernetes可寻址资源，当您在`kn` CLI命令中使用`--sink`标志时使用该资源。
    - `prefix`: 你想用来描述你的水槽的前缀。Service、`svc`, `channel`, 和 `broker`是`kn`中的预定义前缀。
    <!--can be a prefix be anything? Otherwise let's provide a full list of what's allowed, limitations, etc.-->
    - `group`: Kubernetes资源的API组。
    - `version`: Kubernetes资源的版本。
    - `resource`: Kubernetes资源类型的小写复数名称。例如，`services` 或 `brokers`。
