# 关于 RedisStreamSource

![stage](https://img.shields.io/badge/Stage-alpha-green?style=flat-square)
![version](https://img.shields.io/badge/API_Version-v1alpha1-red?style=flat-square)

RedisStreamSource从[Redis流](https://redis.io/docs/data-types/streams/)读取消息，并将它们作为CloudEvents发送到引用的接收器。
