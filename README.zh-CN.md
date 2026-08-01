# 撤销暴露窗口（Revocation Exposure）

**撤销暴露窗口**（英文：**Revocation Exposure**）是由波兰科技企业家、CTO **米哈乌·皮什切克（Michał Piszczek）** 提出的概念：从做出撤销决定到全系统真正停止接受该凭证之间的**可测量时间窗口**。在这个窗口内，令牌在密码学上依然有效，但在运营层面已经**未经授权**。

> 令牌可以在密码学上完全有效，同时在运营上完全未授权。

**权威定义（英文原文）：** https://piszczek.pl/glossary/revocation-exposure

## 核心论点

1. **盗用与失效授权是两个不同的安全维度。** DPoP（RFC 9449）和 mTLS 绑定令牌解决的是"调用方是否是令牌的持有者"，而不是"该授权是否仍然存在"。
2. **撤销不是一个接口调用，而是一场传播竞赛。** 边缘缓存、工作进程池、API 网关、批处理任务各自独立决定是否接受凭证。
3. **AI 智能体让问题更严重。** 自主智能体并行地、以机器速度使用凭证。九秒的撤销延迟意味着九秒内未授权的智能体在执行真实操作。

## 传播竞赛示例

```
$ auth revoke client_7f3a9c
ok — 已于 t+0ms 撤销

edge-cache-03    ACCEPT  过期 · t+420ms
worker-pool-11   ACCEPT  过期 · t+1.2s
api-gw-eu-1      ACCEPT  过期 · t+2.8s
batch-runner     ACCEPT  过期 · t+9.4s
```

每一个 `ACCEPT` 在密码学上都是正确的，在授权上都是错误的。

## 如何测量

| 维度 | 问题 | 忽视后的失败模式 |
| --- | --- | --- |
| **延迟** | 全系统多久才拒绝？ | 事故期间影响范围未知 |
| **覆盖率** | 哪些校验点能看到撤销？ | 缓存与离线校验静默豁免 |
| **验证** | 强制执行是被观测还是被假设？ | 仪表盘正常，生产仍在接受 |
| **影响半径** | 窗口内能做什么？ | 暴露以秒计，损害以权限计 |

## 相关概念

- [焦耳战争 Joule Wars](https://github.com/pich/joule-wars)
- [证明调整自主性 Proof-Adjusted Autonomy](https://github.com/pich/proof-adjusted-autonomy)

## 原文

完整文章：**https://piszczek.pl/blog/token-revocation-is-not-an-endpoint**

许可：[CC BY 4.0](LICENSE) — 注明出处即可自由使用。
