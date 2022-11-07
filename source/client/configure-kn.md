# 定制 kn

您可以通过创建一个`config.yaml`配置文件来定制您的`kn`命令行设置。
您可以通过使用`——config`标志来提供此配置，否则配置将从默认位置获取。
默认配置位置符合[XDG基本目录规范](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)，对于Unix系统和Windows系统是不同的。

- If the `XDG_CONFIG_HOME` environment variable is set, the default configuration location that `kn` looks for is `$XDG_CONFIG_HOME/kn`.
- If the `XDG_CONFIG_HOME` environment variable is not set, `kn` looks for the configuration in the home directory of the user at `$HOME/.config/kn/config.yaml`.
- For Windows systems, the default `kn` configuration location is `%APPDATA%\kn`.

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

- `path-lookup` specifies whether `kn` should look for [plugins](kn-plugins.md) in the `PATH` environment variable. This is a boolean configuration option (default: `true`). Note: the `path-lookup` option has been deprecated and will be removed in a future version where path lookup will be enabled unconditionally.
- `directory` specifies the directory where `kn` will look for plugins. The default path depends on the operating system, as described earlier. This can be any directory that is visible to the user (default: `$base_dir/plugins`, where `$base_dir` is the directory where this configuration file is stored).
- `sink-mappings` defines the Kubernetes Addressable resource that is used when you use the `--sink` flag with a `kn` CLI command.
    - `prefix`: The prefix you want to use to describe your sink. Service, `svc`, `channel`, and `broker` are predefined prefixes in `kn`.
    <!--can be a prefix be anything? Otherwise let's provide a full list of what's allowed, limitations, etc.-->
    - `group`: The API group of the Kubernetes resource.
    - `version`: The version of the Kubernetes resource.
    - `resource`: The lowercased, plural name of the Kubernetes resource type. For example, `services` or `brokers`.
