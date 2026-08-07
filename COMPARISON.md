# Side-by-Side Comparison

Four dimensions per table: stars (live), who backs it, how it runs on Kubernetes, and how it isolates.

**Isolation** uses the three tiers from [the README](README.md#isolation-tiers) — *shared kernel*, *intercepted kernel*, *separate kernel* — plus two variants: `+ TEE` for hardware confidential computing, and `WebAssembly` where there is no kernel at all. "Any" means the project delegates to a RuntimeClass and supports all three.

**Kubernetes** is stated plainly: *Native CRDs* (sandboxes are first-class resources), *Helm* (official manifests, no CRDs), *RuntimeClass* (it is a runtime, it plugs in), *Unofficial* (works, but not the documented path), or *None* (no self-hosting at all).

## Isolation Runtimes

| Runtime | Stars | Backed by | Kubernetes | Isolation |
| --- | --- | --- | --- | --- |
| [Firecracker](https://github.com/firecracker-microvm/firecracker) | ![](https://img.shields.io/github/stars/firecracker-microvm/firecracker?style=flat-square&label=%20) | AWS | RuntimeClass | Separate kernel |
| [gVisor](https://github.com/google/gvisor) | ![](https://img.shields.io/github/stars/google/gvisor?style=flat-square&label=%20) | Google | RuntimeClass, native on GKE | Intercepted kernel |
| [Kata Containers](https://github.com/kata-containers/kata-containers) | ![](https://img.shields.io/github/stars/kata-containers/kata-containers?style=flat-square&label=%20) | OpenInfra Foundation | RuntimeClass, most mature | Separate kernel, `+ TEE` available |
| [Cloud Hypervisor](https://github.com/cloud-hypervisor/cloud-hypervisor) | ![](https://img.shields.io/github/stars/cloud-hypervisor/cloud-hypervisor?style=flat-square&label=%20) | Linux Foundation | RuntimeClass, via Kata | Separate kernel |
| [KubeVirt](https://github.com/kubevirt/kubevirt) | ![](https://img.shields.io/github/stars/kubevirt/kubevirt?style=flat-square&label=%20) | Red Hat (CNCF) | Native CRDs | Separate kernel, full VM |
| [Confidential Containers](https://github.com/confidential-containers/operator) | ![](https://img.shields.io/github/stars/confidential-containers/operator?style=flat-square&label=%20) | Intel, IBM, Red Hat, Ant (CNCF) | Native CRDs | Separate kernel `+ TEE` |
| [Sysbox](https://github.com/nestybox/sysbox) | ![](https://img.shields.io/github/stars/nestybox/sysbox?style=flat-square&label=%20) | Docker | RuntimeClass | Shared kernel, hardened |
| [runc](https://github.com/opencontainers/runc) | ![](https://img.shields.io/github/stars/opencontainers/runc?style=flat-square&label=%20) | OCI | RuntimeClass, default | Shared kernel |
| [bubblewrap](https://github.com/containers/bubblewrap) | ![](https://img.shields.io/github/stars/containers/bubblewrap?style=flat-square&label=%20) | Red Hat / Flatpak | Unofficial | Shared kernel |
| [nsjail](https://github.com/google/nsjail) | ![](https://img.shields.io/github/stars/google/nsjail?style=flat-square&label=%20) | Google | Unofficial | Shared kernel |
| [runwasi](https://github.com/containerd/runwasi) | ![](https://img.shields.io/github/stars/containerd/runwasi?style=flat-square&label=%20) | CNCF | RuntimeClass | WebAssembly |
| [WasmEdge](https://github.com/WasmEdge/WasmEdge) | ![](https://img.shields.io/github/stars/WasmEdge/WasmEdge?style=flat-square&label=%20) | CNCF | RuntimeClass | WebAssembly |

## On Kubernetes — sandbox orchestration

| Project | Stars | Backed by | Kubernetes | Isolation |
| --- | --- | --- | --- | --- |
| [agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) | ![](https://img.shields.io/github/stars/kubernetes-sigs/agent-sandbox?style=flat-square&label=%20) | Kubernetes SIG Apps (Google, Red Hat) | Native CRDs | Any |
| [OpenSandbox](https://github.com/opensandbox-group/OpenSandbox) | ![](https://img.shields.io/github/stars/opensandbox-group/OpenSandbox?style=flat-square&label=%20) | Alibaba | Native CRDs | Any |
| [CubeSandbox](https://github.com/TencentCloud/CubeSandbox) | ![](https://img.shields.io/github/stars/TencentCloud/CubeSandbox?style=flat-square&label=%20) | Tencent Cloud | Native CRDs, or no cluster at all | Intercepted or separate kernel |
| [OpenKruise Agents](https://github.com/openkruise/agents) | ![](https://img.shields.io/github/stars/openkruise/agents?style=flat-square&label=%20) | Alibaba Cloud (CNCF OpenKruise) | Native CRDs | Any, but unproven — see below |
| [mitos](https://github.com/mitos-run/mitos) | ![](https://img.shields.io/github/stars/mitos-run/mitos?style=flat-square&label=%20) | mitos-run | Native CRDs | Separate kernel |

### The three CRD approaches, in detail

Not competitors. OpenKruise Agents states it maintains compatibility with SIG Agent-Sandbox and layers higher-level APIs on top.

| | agent-sandbox | OpenKruise Agents | OpenSandbox / CubeSandbox |
| --- | --- | --- | --- |
| Owner | Kubernetes SIG Apps | CNCF OpenKruise (Alibaba Cloud) | Alibaba / Tencent Cloud |
| CRD group | `agents.x-k8s.io` | `agents.kruise.io` | Proprietary |
| CRDs | 4: `Sandbox`, `SandboxTemplate`, `SandboxWarmPool`, `SandboxClaim` | 10: the above plus `SandboxSet`, `Checkpoint`, `Commit`, `SandboxUpdateOps`, `SecurityProfile`, `TrafficPolicy`, `GlobalTrafficPolicy` | CRD-second; CubeSandbox is migrating from Helm to an operator |
| Goal | Define the standard — minimal and vendor-neutral | Add production capabilities on top of the standard | Ship a complete platform with SDKs |
| Differentiators | Warm pools, stable identity, persistent storage | Pause and resume, resource pooling and dynamic resize, session identity with traffic routing, multi-replica `SandboxSet` | Browser, code, and desktop form factors; credential vault; unified ingress with per-sandbox egress control |
| Pause semantics | No pause API | **Deletes the Pod** and rebuilds it on resume from `spec.template` plus a checkpointed pod-template delta. `PauseStrategy: Stop` only — `Hibernate`, `persistentContents`, and GPU-memory checkpointing are declared in the API but [return "not yet supported"](https://github.com/openkruise/agents/blob/master/pkg/controller/sandbox/core/common_control.go) as of August 2026 | Not CRD-driven |
| External API compatibility | None | **E2B SDK compatible**, plus a [Kubernetes SDK](https://github.com/openkruise/agents-api). Caveat: `/pause` returns 204 like E2B but does not preserve memory, and the E2B endpoint offers no way to ask for it | Five first-party language SDKs and an MCP server |
| Strong isolation | Documented use cases and examples for gVisor, Kata, and Firecracker | `spec.template` is a plain `PodTemplateSpec`, so `runtimeClassName` passes through and gVisor or Kata will apply. But `runtimeClassName` appears nowhere in the codebase — no docs, examples, or tests — and [roadmap.md](https://github.com/openkruise/agents/blob/master/roadmap.md) still lists "Kata/gvisor/kuasar best practice" as pending. Pause, checkpoint deltas, and sidecar injection are all written against ordinary containers | README documents gVisor, Kata, and Firecracker support |
| Maturity | v0.x, explicitly not production-ready for all workloads | v0.x, earlier still | Validated at scale inside Alibaba and Tencent |

## Self-Hosted Sandboxes

| Project | Stars | Backed by | Kubernetes | Isolation |
| --- | --- | --- | --- | --- |
| [E2B](https://github.com/e2b-dev/E2B) | ![](https://img.shields.io/github/stars/e2b-dev/E2B?style=flat-square&label=%20) | E2B | None — Terraform via [infra](https://github.com/e2b-dev/infra) | Separate kernel |
| [Daytona](https://github.com/daytonaio/daytona) | ![](https://img.shields.io/github/stars/daytonaio/daytona?style=flat-square&label=%20) | Daytona | Helm | Shared kernel default, upgradeable |
| [microsandbox](https://github.com/superradcompany/microsandbox) | ![](https://img.shields.io/github/stars/superradcompany/microsandbox?style=flat-square&label=%20) | Superrad | Unofficial | Separate kernel |
| [AIO Sandbox](https://github.com/agent-infra/sandbox) | ![](https://img.shields.io/github/stars/agent-infra/sandbox?style=flat-square&label=%20) | ByteDance | Helm | Shared kernel |
| [forkd](https://github.com/deeplethe/forkd) | ![](https://img.shields.io/github/stars/deeplethe/forkd?style=flat-square&label=%20) | deeplethe | Unofficial | Separate kernel |
| [gondolin](https://github.com/earendil-works/gondolin) | ![](https://img.shields.io/github/stars/earendil-works/gondolin?style=flat-square&label=%20) | earendil.works | Unofficial | Separate kernel |
| [sandbox-runtime](https://github.com/anthropic-experimental/sandbox-runtime) | ![](https://img.shields.io/github/stars/anthropic-experimental/sandbox-runtime?style=flat-square&label=%20) | Anthropic | None — local tool | Shared kernel, by design |
| [shuru](https://github.com/superhq-ai/shuru) | ![](https://img.shields.io/github/stars/superhq-ai/shuru?style=flat-square&label=%20) | SuperHQ | None | Separate kernel |
| [clawk](https://github.com/clawkwork/clawk) | ![](https://img.shields.io/github/stars/clawkwork/clawk?style=flat-square&label=%20) | Clawkwork | None | Separate kernel |
| [arrakis](https://github.com/abshkbh/arrakis) | ![](https://img.shields.io/github/stars/abshkbh/arrakis?style=flat-square&label=%20) | Community | None | Separate kernel |
| [brood-box](https://github.com/stacklok/brood-box) | ![](https://img.shields.io/github/stars/stacklok/brood-box?style=flat-square&label=%20) | Stacklok | None | Separate kernel |
| [pullrun](https://github.com/pullrun/pullrun) | ![](https://img.shields.io/github/stars/pullrun/pullrun?style=flat-square&label=%20) | pullrun | Unofficial | Separate or shared kernel |
| [arcbox](https://github.com/arcboxlabs/arcbox) | ![](https://img.shields.io/github/stars/arcboxlabs/arcbox?style=flat-square&label=%20) | Arcbox Labs | Unofficial | Separate kernel |
| [llm-sandbox](https://github.com/vndee/llm-sandbox) | ![](https://img.shields.io/github/stars/vndee/llm-sandbox?style=flat-square&label=%20) | Community | Helm | Shared kernel |
| [SWE-ReX](https://github.com/SWE-agent/SWE-ReX) | ![](https://img.shields.io/github/stars/SWE-agent/SWE-ReX?style=flat-square&label=%20) | Princeton NLP | Unofficial | Shared kernel |
| [SandboxFusion](https://github.com/bytedance/SandboxFusion) | ![](https://img.shields.io/github/stars/bytedance/SandboxFusion?style=flat-square&label=%20) | ByteDance | Unofficial | Shared kernel |

## Managed Platforms

Note how uniformly the Kubernetes column reads *None*. That is the defining tradeoff of this category.

| Platform | SDK stars | Company | Self-host | Isolation |
| --- | --- | --- | --- | --- |
| [Modal](https://modal.com/) | ![](https://img.shields.io/github/stars/modal-labs/modal-client?style=flat-square&label=%20) | Modal Labs | None | Intercepted kernel — **GPU-capable** |
| [E2B Cloud](https://e2b.dev/) | ![](https://img.shields.io/github/stars/e2b-dev/code-interpreter?style=flat-square&label=%20) | E2B | Bring-your-own-cloud | Separate kernel |
| [Vercel Sandbox](https://github.com/vercel/sandbox) | ![](https://img.shields.io/github/stars/vercel/sandbox?style=flat-square&label=%20) | Vercel | None | Separate kernel |
| [Cloudflare Sandbox SDK](https://github.com/cloudflare/sandbox-sdk) | ![](https://img.shields.io/github/stars/cloudflare/sandbox-sdk?style=flat-square&label=%20) | Cloudflare | None | Shared kernel or WebAssembly |
| [Northflank](https://northflank.com/) | — | Northflank | **Yes**, self-host and BYOC | Separate kernel, plus gVisor |
| [Blaxel](https://github.com/blaxel-ai/sandbox) | ![](https://img.shields.io/github/stars/blaxel-ai/sandbox?style=flat-square&label=%20) | Blaxel | None | Separate kernel |
| [Runloop](https://runloop.ai/) | ![](https://img.shields.io/github/stars/runloopai/api-client-ts?style=flat-square&label=%20) | Runloop | None | Separate kernel |
| [Azure ACA Dynamic Sessions](https://learn.microsoft.com/en-us/azure/container-apps/sessions) | — | Microsoft | None | Separate kernel, Hyper-V |
| [AWS Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/) | — | AWS | None | Separate kernel, Firecracker |
| [GKE Agent Sandbox](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/agent-sandbox) | see upstream | Google Cloud | **Yes**, upstream is open | Intercepted kernel by default |
| [Alibaba Cloud AgentBay](https://www.aliyun.com/product/agentbay) | ![](https://img.shields.io/github/stars/agentbay-ai/wuying-agentbay-sdk?style=flat-square&label=%20) | Alibaba Cloud | Via ACS | Intercepted or separate kernel |
| [Tencent Cloud AGSX](https://cloud.tencent.com/product/agsx) | see CubeSandbox | Tencent Cloud | Via CubeSandbox | Intercepted or separate kernel |

## A note on reading star counts

Star counts measure attention, not fitness. Three specific traps in this space:

- **Daytona** and **OpenHands** accumulated most of their stars under earlier or broader positioning. Neither number reflects sandbox-specific adoption.
- **agent-sandbox** has far fewer stars than several products built on top of it, because standards attract contributors rather than stargazers.
- **kagent** is not a sandbox at all. Its star count regularly gets cited in this category by mistake.

## Selection guide

| If you need | Choose |
| --- | --- |
| Multi-tenant agent platform on your own Kubernetes | `agent-sandbox` with gVisor (balanced) or Kata (strong) |
| Pause/resume to stop paying while agents idle | `OpenKruise Agents` — but pause deletes the Pod, so in-memory state does not survive |
| E2B SDK ergonomics with data staying in your network | `OpenKruise Agents` — it speaks the E2B API |
| A complete platform with minimal integration work | `OpenSandbox` or `CubeSandbox` |
| Fastest path to shipping, no infrastructure work | `E2B` or `Daytona` |
| GPU inside the sandbox | `Modal` — currently the only option |
| To protect your laptop from your own coding agent | `sandbox-runtime` — do not over-build this |
| Protection from the cloud provider itself | `Confidential Containers`, or Kata with Intel TDX / AMD SEV-SNP |
| To orchestrate agents, not isolate code | `kagent` — different problem, stack it with a sandbox |
