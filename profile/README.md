<!-- markdownlint-disable -->
<h1 align="center">LingShu AI Infra · 灵枢</h1>
<p align="center">
  <strong>Pool Engine for GPU Workloads</strong><br>
  Open-source distributed GPU resource pool scheduling service.<br>
  Built on SpringBoot + Netty + Protobuf. Replaces K8s static GPU binding.
</p>

<p align="center">
  <a href="https://lingshu-ai-infra.github.io/lingshu-website/"><img src="https://img.shields.io/badge/website-lingshu--ai--infra.github.io-00d4aa?style=flat-square" alt="Website"></a>
  <a href="./lingshu-design/blob/main/linshu-ai-infra.md"><img src="https://img.shields.io/badge/design-V2.1-blue?style=flat-square" alt="Design"></a>
  <a href="./lingshu-design/blob/main/mvp-stories.md"><img src="https://img.shields.io/badge/mvp-6%20weeks-orange?style=flat-square" alt="MVP"></a>
  <a href="./lingshu-website/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-Apache%202.0-green?style=flat-square" alt="License"></a>
  <a href="./lingshu-design/issues"><img src="https://img.shields.io/badge/status-MVP%20in%20progress-yellow?style=flat-square" alt="Status"></a>
</p>

---

## 灵枢 AI Infra 是什么?

**LingShu AI Infra** 是一个开源的 **GPU 资源池分布式调度服务**,核心解决 Kubernetes 静态绑定 GPU 模式下的资源碎片化问题:

- **资源池化** — 业务 Pod 不再独占 GPU 卡,所有 GPU 进入统一调度池,目标整体利用率 ≥ 70%
- **业务轻量化** — Pod 镜像无需 CUDA/Torch/TensorRT,瘦身后从 8GB+ 降到 &lt; 200MB
- **多维度负载均衡** — 综合 `mem×0.5 + util×0.3 + queue×0.2` 评分,基于实时负载动态分配
- **三档 GPU 隔离** — MIG(硬切)/ MPS(进程组)/ Soft(单进程),按业务等级自动选档
- **四档大张量传输** — 同节点 SHM → 跨节点 RDMA → NAS(CephFS/JuiceFS/Lustre)→ 对象存储
- **无 Etcd 选主** — 复用自研 `ControllerManager` 多轮投票 + force 票反脑裂

> **架构链路**:业务 Pod(Client) → 自研 Netty-RPC 网关 → 全局调度器 → GPU Worker 节点 → 物理 GPU

## 📦 仓库矩阵(12 repo)

### 核心组件

| Repo | 描述 | 状态 |
|---|---|---|
| [**lingshu-design**](https://github.com/lingshu-ai-infra/lingshu-design) | 设计文档源 · V2.1 技术方案 + MVP Story 拆分 | ✅ 已发布 |
| [**lingshu-gpu-proto**](https://github.com/lingshu-ai-infra/lingshu-gpu-proto) | Protobuf 协议定义(11 个消息体) | 🚧 Sprint 1 |
| [**lingshu-gpu-client**](https://github.com/lingshu-ai-infra/lingshu-gpu-client) | Java Client SDK | 🚧 Sprint 1 |
| [**lingshu-gpu-gateway**](https://github.com/lingshu-ai-infra/lingshu-gpu-gateway) | Netty RPC 网关 | 🚧 Sprint 2 |
| [**lingshu-gpu-scheduler**](https://github.com/lingshu-ai-infra/lingshu-gpu-scheduler) | 分布式调度核心 | 🚧 Sprint 2 |
| [**lingshu-gpu-worker**](https://github.com/lingshu-ai-infra/lingshu-gpu-worker) | GPU Worker 执行引擎 | 🚧 Sprint 2 |
| [**lingshu-gpu-demo**](https://github.com/lingshu-ai-infra/lingshu-gpu-demo) | E2E Demo(text_classification) | 🚧 Sprint 2 |

### 元信息 / 文档 / 运维

| Repo | 描述 | 状态 |
|---|---|---|
| [**lingshu-website**](https://github.com/lingshu-ai-infra/lingshu-website) | 组织官网源码 | ✅ 已上线 |
| [**lingshu-docs**](https://github.com/lingshu-ai-infra/lingshu-docs) | Docusaurus 详细技术文档站 | 📝 规划 |
| [**lingshu-deploy**](https://github.com/lingshu-ai-infra/lingshu-deploy) | K8s manifests · Helm · 部署脚本 | 📝 规划 |
| [**lingshu-bench**](https://github.com/lingshu-ai-infra/lingshu-bench) | 性能基准 + 负载画像 | 📝 规划 |
| [**lingshu-examples**](https://github.com/lingshu-ai-infra/lingshu-examples) | 示例算子(PyTorch/ONNX/TensorRT) | 📝 规划 |

## 🚀 快速开始

业务侧一行代码提交任务:

```java
try (GpuClient client = new GpuClient("gateway.lingshu-infra:2381")) {
    String taskId = client.submit(SubmitTaskRequest.newBuilder()
        .setOpId("text_classification")
        .setInput("灵枢调度真的很强")
        .build());
    System.out.println(client.query(taskId).getOutput());
}
```

完整文档: <https://lingshu-ai-infra.github.io/lingshu-website/>

## 📐 关键设计决策(Q1-Q7,已确认)

| # | 问题 | 决策 |
|---|---|---|
| Q1 | GPU 共享模式 | 中期 MIG,长期按业务分级 |
| Q2 | Python Op 默认 | Sidecar(gRPC),性能瓶颈 Op 改 PyO3 |
| Q3 | 大张量默认传输 | 同节点共享内存 → NAS → RDMA → 对象存储 |
| Q4 | 调度器选主 | 复用 `ControllerManager` 自研投票(无 Etcd) |
| Q5 | 优先级抢占 | 默认关闭,关键业务按需开启 |
| Q6 | 同节点亲和 | 默认 Soft Hint,无可用时回退 |
| Q7 | 任务最长超时 | 推理 180s,训练 24h,可在请求中覆盖 |

## 🤝 参与贡献

- 📖 阅读 [CONTRIBUTING.md](./lingshu-design/blob/main/CONTRIBUTING.md)
- 🐛 提交 [Issue](https://github.com/lingshu-ai-infra/lingshu-design/issues)
- 💬 加入 [Discord](https://discord.gg/lingshu-ai-infra)
- 📋 遵守 [Code of Conduct](./lingshu-design/blob/main/CODE_OF_CONDUCT.md)
- 🔒 安全问题请见 [SECURITY.md](./lingshu-design/blob/main/SECURITY.md)

## 📜 License

Apache License 2.0 · [LICENSE](./lingshu-website/blob/main/LICENSE)

---

<p align="center">
  Built with ❤ by the LingShu community · 灵枢开源 · © 2026
</p>