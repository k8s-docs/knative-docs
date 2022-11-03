# 自动缩放样本应用程序-Go

Knative服务修订的自动缩放功能的演示。

## 先决条件

1. 安装了[Knative服务](../../../install/yaml-install/serving/install-serving-with-yaml.md)的Kubernetes集群
   .
1. 安装了`hey` 负载生成器 (`go get -u github.com/rakyll/hey`).
1. 克隆这个存储库，并移动到示例目录:

     ```bash
     git clone -b "{{ branch }}" https://github.com/knative/docs knative-docs
     cd knative-docs
     ```

## 部署服务

1. 部署[样例](service.yaml)Knative服务:

     ```bash
     kubectl apply -f docs/serving/autoscaling/autoscale-go/service.yaml
     ```

1. 获取服务的URL(一旦为`Ready`):

     ```{ .bash .no-copy }
     $ kubectl get ksvc autoscale-go
     NAME            URL                                                LATESTCREATED         LATESTREADY           READY   REASON
     autoscale-go    http://autoscale-go.default.1.2.3.4.sslip.io    autoscale-go-96dtk    autoscale-go-96dtk    True
     ```

## 加载服务

1. 向自动伸缩应用程序发出一个请求，以查看它消耗一些资源。

     ```bash
     curl "http://autoscale-go.default.1.2.3.4.sslip.io?sleep=100&prime=10000&bloat=5"
     ```

     ```{ .bash .no-copy }
     Allocated 5 Mb of memory.
     The largest prime less than 10000 is 9973.
     Slept for 100.13 milliseconds.
     ```

1. 发送30秒的流量，维持50个飞行请求。

     ```bash
     hey -z 30s -c 50 \
       "http://autoscale-go.default.1.2.3.4.sslip.io?sleep=100&prime=10000&bloat=5" \
       && kubectl get pods
     ```

     ```{ .bash .no-copy }
     Summary:
       Total:        30.3379 secs
       Slowest:      0.7433 secs
       Fastest:      0.1672 secs
       Average:      0.2778 secs
       Requests/sec: 178.7861

       Total data:   542038 bytes
       Size/request: 99 bytes

     Response time histogram:
       0.167 [1]     |
       0.225 [1462]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
       0.282 [1303]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■
       0.340 [1894]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
       0.398 [471]   |■■■■■■■■■■
       0.455 [159]   |■■■
       0.513 [68]    |■
       0.570 [18]    |
       0.628 [14]    |
       0.686 [21]    |
       0.743 [13]    |

     Latency distribution:
       10% in 0.1805 secs
       25% in 0.2197 secs
       50% in 0.2801 secs
       75% in 0.3129 secs
       90% in 0.3596 secs
       95% in 0.4020 secs
       99% in 0.5457 secs

     Details (average, fastest, slowest):
       DNS+dialup:   0.0007 secs, 0.1672 secs, 0.7433 secs
       DNS-lookup:   0.0000 secs, 0.0000 secs, 0.0000 secs
       req write:    0.0001 secs, 0.0000 secs, 0.0045 secs
       resp wait:    0.2766 secs, 0.1669 secs, 0.6633 secs
       resp read:    0.0002 secs, 0.0000 secs, 0.0065 secs

     Status code distribution:
       [200] 5424 responses
     ```

     ```{ .bash .no-copy }
     NAME                                             READY   STATUS    RESTARTS   AGE
     autoscale-go-00001-deployment-78cdc67bf4-2w4sk   3/3     Running   0          26s
     autoscale-go-00001-deployment-78cdc67bf4-dd2zb   3/3     Running   0          24s
     autoscale-go-00001-deployment-78cdc67bf4-pg55p   3/3     Running   0          18s
     autoscale-go-00001-deployment-78cdc67bf4-q8bf9   3/3     Running   0          1m
     autoscale-go-00001-deployment-78cdc67bf4-thjbq   3/3     Running   0          26s
     ```

## 分析

### 算法

Knative服务自动伸缩是基于每个Pod的飞行请求的平均数量(并发性)。
系统默认[目标并发数为100](https://github.com/knative/serving/blob/main/config/core/configmaps/autoscaler.yaml)(搜索 container-concurrency-target-default),但我们的服务[用了10](service.yaml#L25-L26)。
我们加载了50个并发请求的服务，因此自动缩放器创建了5个Pod(`50 concurrent requests / target of 10 = 5 pods`)

#### 恐慌

自动计算器在60秒的窗口内计算平均并发性，因此系统需要一分钟时间才能稳定在所需的并发性水平上。
然而，自动缩放器也会计算一个6秒的“恐慌”窗口，如果该窗口达到目标并发数的2倍，就会进入恐慌模式。
在恐慌模式下，自动缩放器在更短、更敏感的恐慌窗口上操作。
一旦恐慌条件在60秒内不再满足，自动缩放器将返回到最初的60秒“稳定”窗口。

```{ .bash .no-copy }
                                                       |
                                  Panic Target--->  +--| 20
                                                    |  |
                                                    | <------Panic Window
                                                    |  |
       Stable Target--->  +-------------------------|--| 10   CONCURRENCY
                          |                         |  |
                          |                      <-----------Stable Window
                          |                         |  |
--------------------------+-------------------------+--+ 0
120                       60                           0
                     TIME
```

#### 定制

自动缩放器支持通过注释进行定制。Knative中内置了两个自动缩放类:

1. `kpa.autoscaling.knative.dev` 它是前面描述的基于并发的自动缩放器(默认值), 和
2. `hpa.autoscaling.knative.dev` 它委托给Kubernetes HPA，它会根据CPU使用自动伸缩.

   在CPU上扩展服务的示例:

   ```yaml
   apiVersion: serving.knative.dev/v1
   kind: Service
   metadata:
     name: autoscale-go
     namespace: default
   spec:
     template:
       metadata:
         annotations:
           # Standard Kubernetes CPU-based autoscaling.
           autoscaling.knative.dev/class: hpa.autoscaling.knative.dev
           autoscaling.knative.dev/metric: cpu
       spec:
         containers:
           - image: gcr.io/knative-samples/autoscale-go:0.1
   ```

   此外，自动缩放器目标和缩放边界可以在注释中指定。具有自定义目标和规模边界的服务示例:

   ```yaml
   apiVersion: serving.knative.dev/v1
   kind: Service
   metadata:
     name: autoscale-go
     namespace: default
   spec:
     template:
       metadata:
         annotations:
           # Knative concurrency-based autoscaling (default).
           autoscaling.knative.dev/class: kpa.autoscaling.knative.dev
           autoscaling.knative.dev/metric: concurrency
           # Target 10 requests in-flight per pod.
           autoscaling.knative.dev/target: "10"
           # Disable scale to zero with a min scale of 1.
           autoscaling.knative.dev/min-scale: "1"
           # Limit scaling to 100 pods.
           autoscaling.knative.dev/max-scale: "100"
       spec:
         containers:
           - image: gcr.io/knative-samples/autoscale-go:0.1
   ```

!!! note
    对于 `hpa.autoscaling.knative.dev` 类服务， `autoscaling.knative.dev/target` 指定CPU百分比目标(默认为`"80"`)。

#### 演示

查看Knative自动缩放自定义的[Kubecon演示](https://youtu.be/OPSIPr-Cybs)(32分钟)。

### 其他的实验

1. 发送60秒的流量，维持100个并发请求。

     ```bash
     hey -z 60s -c 100 \
       "http://autoscale-go.default.1.2.3.4.sslip.io?sleep=100&prime=10000&bloat=5"
     ```

1. 发送60秒的流量，保持100 qps的短请求(10毫秒)。

     ```bash
     hey -z 60s -q 100 \
       "http://autoscale-go.default.1.2.3.4.sslip.io?sleep=10"
     ```

1. 发送60秒的流量，保持100 qps的长请求(1秒)。

     ```bash
     hey -z 60s -q 100 \
       "http://autoscale-go.default.1.2.3.4.sslip.io?sleep=1000"
     ```

1. 发送60秒的高CPU使用率的流量(~1 cpu/sec/request，总共100个CPU)。

     ```bash
     hey -z 60s -q 100 \
       "http://autoscale-go.default.1.2.3.4.sslip.io?prime=40000000"
     ```

2. 发送60秒的高内存使用流量(每请求1gb，共5gb)。

     ```bash
     hey -z 60s -c 5 \
       "http://autoscale-go.default.1.2.3.4.sslip.io?bloat=1000"
     ```

## 清理

```bash
kubectl delete -f docs/serving/autoscaling/autoscale-go/service.yaml
```

## 进一步的阅读

[自动定量开发人员文档](https://github.com/knative/serving/blob/main/docs/scaling/SYSTEM.md)
