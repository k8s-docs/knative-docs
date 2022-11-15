# 关于代理器

--8<-- "about-brokers.md"

## 事件交付

事件传递机制是依赖于配置的代理类的实现细节。
使用代理和触发器可以从事件生产者和事件使用者抽象事件路由的细节。

## 高级用例

对于大多数用例来说，每个名称空间一个代理就足够了，但是在一些用例中，多个代理可以简化体系结构。
例如，针对包含个人身份信息(PII)和非PII事件的独立代理可以简化审计和访问控制规则。

## 下一个步骤

- 创建一个[基于MT通道的代理](create-mtbroker.md).
- 配置[默认代理ConfigMap设置](broker-admin-config-options.md).

## 额外的资源

- [代理概念文档](../../concepts/eventing-resources/brokers.md)
- [代理的规范](https://github.com/knative/specs/blob/main/specs/eventing/overview.md#broker){target=_blank}
