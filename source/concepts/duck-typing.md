# 鸭子类型

Knative通过使用[鸭子类型](https://zh.wikipedia.org/wiki/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B){target=_blank}支持其组件的[松散耦合](https://zh.wikipedia.org/wiki/%E6%9D%BE%E8%80%A6%E5%90%88){target=_blank}。


鸭子类型意味着在Knative系统中使用的资源的兼容性由用于识别资源控制平面形状和行为的某些属性决定。
这些属性基于一组针对不同类型资源的公共定义，称为鸭子类型。

Knative可以像使用泛型鸭类型一样使用资源，而不需要对资源类型有具体的了解，如果:

* 资源在公共定义指定的相同模式位置中具有相同的字段
* 与公共定义指定的相同的控件或数据平面行为

有些资源可以选择加入多种鸭子类型。

<!-- TODO: 指向发现ClusterDuckType文档。 -->

Knative中鸭子类型的一个基本用途是在资源 _规范_ 中使用对象引用来指向其他资源。

包含引用的对象的定义规定了被引用资源的预期鸭子类型。

## 例子

在下面的例子中，一个名为 `pointer` 的Knative`示例`资源在其规范中引用了一个名为 `pointee` 的 `Dog` 资源:

```yaml
apiVersion: sample.knative.dev/v1
kind: Example
metadata:
  name: pointer
spec:
  size:
    apiVersion: extension.example.com/v1
    kind: Dog
    name: pointee
```

如果一个可观的鸭子类型的期望形状是，在`status`中，模式形状如下:

```yaml
status:
  height: <in centimetres>
  weight: <in kilograms>
```

现在`pointee`的实例看起来像这样:

```yaml
apiVersion: extension.example.com/v1
kind: Dog
metadata:
  name: pointee
spec:
  owner: Smith Family
  etc: more here
status:
  lastFeeding: 2 hours ago
  hungry: true
  age: 2
  height: 60
  weight: 20
```

当`示例`资源起作用时，它只作用于大尺寸鸭子类型形状的信息，而`Dog`实现可以自由地拥有对该资源最有意义的信息。

当我们用一种新类型扩展系统时，鸭子类型的威力是显而易见的，例如，
`Human`, 如果新资源符合大公司设定的合同。

```yaml
apiVersion: sample.knative.dev/v1
kind: Example
metadata:
  name: pointer
spec:
  size:
    apiVersion: people.example.com/v1
    kind: human
    name: pointee
---
apiVersion: people.example.com/v1
kind: Human
metadata:
  name: pointee
spec:
  etc: even more here
status:
  college: true
  hungry: true
  age: 22
  height: 170
  weight: 50
```

`示例`资源能够应用为其配置的逻辑，而无需显式地了解`Human`或`Dog`。

## Knative 鸭子类型

Knative定义了几个在整个项目中使用的鸭子类型的契约:

- [鸭子类型](#鸭子类型)
  - [例子](#例子)
  - [Knative 鸭子类型](#knative-鸭子类型)
    - [Addressable(可寻址)](#addressable可寻址)
    - [Binding(绑定)](#binding绑定)
    - [Source(源)](#source源)

### Addressable(可寻址)

可寻址的形状预期如下:

```yaml
apiVersion: group/version
kind: Kind
status:
  address:
    url: http://host/path?query
```

### Binding(绑定)

有了直接的`subject`, Binding 应该是以下形状:

```yaml
apiVersion: group/version
kind: Kind
spec:
  subject:
    apiVersion: group/version
    kind: SomeKind
    namespace: the-namespace
    name: a-name
```

使用间接`subject`时, Binding 应该是以下形状:

```yaml
apiVersion: group/version
kind: Kind
spec:
  subject:
    apiVersion: group/version
    kind: SomeKind
    namespace: the-namespace
    selector:
      matchLabels:
        key: value
```

### Source(源)

使用`ref`接收器，Source 应该是以下形状:

```yaml
apiVersion: group/version
kind: Kind
spec:
  sink:
    ref:
      apiVersion: group/version
      kind: AnAddressableKind
      name: a-name
  ceOverrides:
    extensions:
      key: value
status:
  observedGeneration: 1
  conditions:
    - type: Ready
      status: "True"
  sinkUri: http://host
```

使用`uri`接收器时，Source 应该是以下形状:

```yaml
apiVersion: group/version
kind: Kind
spec:
  sink:
    uri: http://host/path?query
  ceOverrides:
    extensions:
      key: value
status:
  observedGeneration: 1
  conditions:
    - type: Ready
      status: "True"
  sinkUri: http://host/path?query
```

对于`ref` 和 `uri`接收器，Source 应该是以下形状:

```yaml
apiVersion: group/version
kind: Kind
spec:
  sink:
    ref:
      apiVersion: group/version
      kind: AnAddressableKind
      name: a-name
    uri: /path?query
  ceOverrides:
    extensions:
      key: value
status:
  observedGeneration: 1
  conditions:
    - type: Ready
      status: "True"
  sinkUri: http://host/path?query
```
