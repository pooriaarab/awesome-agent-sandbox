# Side-by-Side Comparison

Four dimensions, one table per category. Star badges are live. Isolation levels refer to [the scale in the README](README.md#isolation-levels).

Kubernetes support is graded as follows:

| Grade | Meaning |
| --- | --- |
| **K4** | Native CRDs and a controller — sandboxes are first-class Kubernetes resources |
| **K3** | Official Helm chart or manifests, but the sandbox is not a CRD |
| **K2** | It *is* a runtime; plugs in through `RuntimeClass` |
| **K1** | Runs on Kubernetes, but not the documented path |
| **K0** | No self-hosting path at all |

## Kubernetes-Native Orchestrators

| Project | Stars | Backed by | K8s | Isolation |
| --- | --- | --- | --- | --- |
| [agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) | ![](https://img.shields.io/github/stars/kubernetes-sigs/agent-sandbox?style=flat-square&label=%20) | Kubernetes SIG Apps (Google, Red Hat) | **K4** | L1–L5 pluggable |
| [OpenSandbox](https://github.com/opensandbox-group/OpenSandbox) | ![](https://img.shields.io/github/stars/opensandbox-group/OpenSandbox?style=flat-square&label=%20) | Alibaba | **K4** | L1–L4 pluggable |
| [CubeSandbox](https://github.com/TencentCloud/CubeSandbox) | ![](https://img.shields.io/github/stars/TencentCloud/CubeSandbox?style=flat-square&label=%20) | Tencent Cloud | **K4** | L3/L4 |
| [OpenKruise Agents](https://github.com/openkruise/agents) | ![](https://img.shields.io/github/stars/openkruise/agents?style=flat-square&label=%20) | Alibaba Cloud (CNCF OpenKruise) | **K4** | L1–L4 pluggable |
| [KubeVirt](https://github.com/kubevirt/kubevirt) | ![](https://img.shields.io/github/stars/kubevirt/kubevirt?style=flat-square&label=%20) | Red Hat (CNCF) | **K4** | L4 |
| [vCluster](https://github.com/loft-sh/vcluster) | ![](https://img.shields.io/github/stars/loft-sh/vcluster?style=flat-square&label=%20) | Loft Labs | **K4** | L1 (control plane only) |
| [Confidential Containers](https://github.com/confidential-containers/operator) | ![](https://img.shields.io/github/stars/confidential-containers/operator?style=flat-square&label=%20) | CNCF (Intel, IBM, Red Hat, Ant) | **K4** | L5 |
| [mitos](https://github.com/mitos-run/mitos) | ![](https://img.shields.io/github/stars/mitos-run/mitos?style=flat-square&label=%20) | mitos-run | **K4** | L4 |

### The three CRD approaches, in detail

These are not competitors. OpenKruise Agents states it maintains compatibility with SIG Agent-Sandbox and layers higher-level APIs on top.

| | agent-sandbox | OpenKruise Agents | OpenSandbox / CubeSandbox |
| --- | --- | --- | --- |
| Owner | Kubernetes SIG Apps | CNCF OpenKruise (Alibaba Cloud) | Alibaba / Tencent Cloud |
| CRD group | `agents.x-k8s.io` | `agents.kruise.io` | Proprietary |
| CRDs | 4: `Sandbox`, `SandboxTemplate`, `SandboxWarmPool`, `SandboxClaim` | 10: the above plus `SandboxSet`, `Checkpoint`, `Commit`, `SandboxUpdateOps`, `SecurityProfile`, `TrafficPolicy`, `GlobalTrafficPolicy` | CRD-second (CubeSandbox is migrating from Helm to an operator) |
| Goal | Define the standard — minimal and vendor-neutral | Add production capabilities on top of the standard | Ship a complete platform with SDKs |
| Differentiators | Warm pools, stable identity, persistent storage | Hibernation and checkpointing (memory, R/W layer, **GPU memory**), `PauseStrategy: Stop\|Hibernate`, resource pooling and dynamic resize, session identity with traffic routing, multi-replica `SandboxSet` | Browser/code/desktop form factors, credential vault, unified ingress with per-sandbox egress control |
| External API compatibility | None | **E2B SDK compatible**, plus a [Kubernetes SDK](https://github.com/openkruise/agents-api) | Five first-party language SDKs and an MCP server |
| Maturity | v0.x, explicitly not production-ready for all workloads | v0.x, earlier still | Validated at scale inside Alibaba and Tencent |

**Choosing between them.** Betting on the long-term standard and willing to build your own operational tooling: `agent-sandbox`. Need pause/resume to cut costs while agents idle, GPU checkpointing, or E2B-compatible APIs without data leaving your network: `OpenKruise Agents`. Want a complete platform with minimal code: `OpenSandbox` or `CubeSandbox`.

## Isolation Runtimes

| Runtime | Stars | Backed by | K8s | Isolation |
| --- | --- | --- | --- | --- |
| [Firecracker](https://github.com/firecracker-microvm/firecracker) | ![](https://img.shields.io/github/stars/firecracker-microvm/firecracker?style=flat-square&label=%20) | AWS | K2 | L4 |
| [gVisor](https://github.com/google/gvisor) | ![](https://img.shields.io/github/stars/google/gvisor?style=flat-square&label=%20) | Google | K2 (native on GKE) | L3 |
| [Kata Containers](https://github.com/kata-containers/kata-containers) | ![](https://img.shields.io/github/stars/kata-containers/kata-containers?style=flat-square&label=%20) | OpenInfra Foundation | K2 (most mature) | L4, L5 with TEE |
| [Cloud Hypervisor](https://github.com/cloud-hypervisor/cloud-hypervisor) | ![](https://img.shields.io/github/stars/cloud-hypervisor/cloud-hypervisor?style=flat-square&label=%20) | Linux Foundation | K2 (via Kata) | L4 |
| [Sysbox](https://github.com/nestybox/sysbox) | ![](https://img.shields.io/github/stars/nestybox/sysbox?style=flat-square&label=%20) | Docker | K2 | L2 |
| [Edera Protect](https://edera.dev/) | — (core closed) | Edera | K2 | L4, no hardware virtualization required |
| [runc](https://github.com/opencontainers/runc) | ![](https://img.shields.io/github/stars/opencontainers/runc?style=flat-square&label=%20) | OCI | K2 (default) | L1 |
| [youki](https://github.com/youki-dev/youki) | ![](https://img.shields.io/github/stars/youki-dev/youki?style=flat-square&label=%20) | Community | K2 | L1 |
| [bubblewrap](https://github.com/containers/bubblewrap) | ![](https://img.shields.io/github/stars/containers/bubblewrap?style=flat-square&label=%20) | Red Hat / Flatpak | K1 | L0/L1 |
| [nsjail](https://github.com/google/nsjail) | ![](https://img.shields.io/github/stars/google/nsjail?style=flat-square&label=%20) | Google | K1 | L0/L1 |
| [runwasi](https://github.com/containerd/runwasi) | ![](https://img.shields.io/github/stars/containerd/runwasi?style=flat-square&label=%20) | CNCF | K2 | Lw |
| [WasmEdge](https://github.com/WasmEdge/WasmEdge) | ![](https://img.shields.io/github/stars/WasmEdge/WasmEdge?style=flat-square&label=%20) | CNCF | K2 | Lw |

## Self-Hostable Sandboxes

| Project | Stars | Backed by | K8s | Isolation |
| --- | --- | --- | --- | --- |
| [E2B](https://github.com/e2b-dev/E2B) | ![](https://img.shields.io/github/stars/e2b-dev/E2B?style=flat-square&label=%20) | E2B | K0 (Terraform via [infra](https://github.com/e2b-dev/infra)) | L4 |
| [Daytona](https://github.com/daytonaio/daytona) | ![](https://img.shields.io/github/stars/daytonaio/daytona?style=flat-square&label=%20) | Daytona | K3 | L1 default, L2/L4 optional |
| [microsandbox](https://github.com/superradcompany/microsandbox) | ![](https://img.shields.io/github/stars/superradcompany/microsandbox?style=flat-square&label=%20) | Superrad | K0/K1 | L4 |
| [AIO Sandbox](https://github.com/agent-infra/sandbox) | ![](https://img.shields.io/github/stars/agent-infra/sandbox?style=flat-square&label=%20) | ByteDance | K3 | L1 |
| [forkd](https://github.com/deeplethe/forkd) | ![](https://img.shields.io/github/stars/deeplethe/forkd?style=flat-square&label=%20) | deeplethe | K1 | L4 |
| [gondolin](https://github.com/earendil-works/gondolin) | ![](https://img.shields.io/github/stars/earendil-works/gondolin?style=flat-square&label=%20) | earendil.works | K1 | L4 |
| [SandboxFusion](https://github.com/bytedance/SandboxFusion) | ![](https://img.shields.io/github/stars/bytedance/SandboxFusion?style=flat-square&label=%20) | ByteDance | K1 | L1 |
| [llm-sandbox](https://github.com/vndee/llm-sandbox) | ![](https://img.shields.io/github/stars/vndee/llm-sandbox?style=flat-square&label=%20) | Community | K3 | L1 |
| [SWE-ReX](https://github.com/SWE-agent/SWE-ReX) | ![](https://img.shields.io/github/stars/SWE-agent/SWE-ReX?style=flat-square&label=%20) | Princeton NLP | K1 | L1 |
| [pullrun](https://github.com/pullrun/pullrun) | ![](https://img.shields.io/github/stars/pullrun/pullrun?style=flat-square&label=%20) | pullrun | K1 | L4/L1 |
| [arcbox](https://github.com/arcboxlabs/arcbox) | ![](https://img.shields.io/github/stars/arcboxlabs/arcbox?style=flat-square&label=%20) | Arcbox Labs | K1 | L4 |

## Local and Desktop Sandboxes

| Project | Stars | Backed by | Isolation |
| --- | --- | --- | --- |
| [sandbox-runtime](https://github.com/anthropic-experimental/sandbox-runtime) | ![](https://img.shields.io/github/stars/anthropic-experimental/sandbox-runtime?style=flat-square&label=%20) | Anthropic | L0 |
| [shuru](https://github.com/superhq-ai/shuru) | ![](https://img.shields.io/github/stars/superhq-ai/shuru?style=flat-square&label=%20) | SuperHQ | L4 |
| [clawk](https://github.com/clawkwork/clawk) | ![](https://img.shields.io/github/stars/clawkwork/clawk?style=flat-square&label=%20) | Clawkwork | L4 |
| [arrakis](https://github.com/abshkbh/arrakis) | ![](https://img.shields.io/github/stars/abshkbh/arrakis?style=flat-square&label=%20) | Community | L4 |
| [brood-box](https://github.com/stacklok/brood-box) | ![](https://img.shields.io/github/stars/stacklok/brood-box?style=flat-square&label=%20) | Stacklok | L4 |
| [Lima](https://github.com/lima-vm/lima) | ![](https://img.shields.io/github/stars/lima-vm/lima?style=flat-square&label=%20) | CNCF Sandbox | L4 |
| [container](https://github.com/apple/container) | ![](https://img.shields.io/github/stars/apple/container?style=flat-square&label=%20) | Apple | L4 |

## Managed Platforms

Note the near-uniform `K0`. That is the defining tradeoff of this category.

| Platform | SDK stars | Company | Self-host | Isolation |
| --- | --- | --- | --- | --- |
| [Modal](https://modal.com/) | ![](https://img.shields.io/github/stars/modal-labs/modal-client?style=flat-square&label=%20) | Modal Labs | K0 | L3 (gVisor), **GPU-capable** |
| [E2B Cloud](https://e2b.dev/) | ![](https://img.shields.io/github/stars/e2b-dev/code-interpreter?style=flat-square&label=%20) | E2B | BYOC (enterprise) | L4 |
| [Vercel Sandbox](https://github.com/vercel/sandbox) | ![](https://img.shields.io/github/stars/vercel/sandbox?style=flat-square&label=%20) | Vercel | K0 | L4 |
| [Cloudflare Sandbox SDK](https://github.com/cloudflare/sandbox-sdk) | ![](https://img.shields.io/github/stars/cloudflare/sandbox-sdk?style=flat-square&label=%20) | Cloudflare | K0 | L1 container / Lw isolate |
| [Northflank](https://northflank.com/) | — | Northflank | **Yes**, self-host and BYOC | L4 Kata + L3 gVisor |
| [Blaxel](https://github.com/blaxel-ai/sandbox) | ![](https://img.shields.io/github/stars/blaxel-ai/sandbox?style=flat-square&label=%20) | Blaxel | K0 | L4 |
| [Runloop](https://runloop.ai/) | ![](https://img.shields.io/github/stars/runloopai/api-client-ts?style=flat-square&label=%20) | Runloop | K0 | L4 |
| [Azure ACA Dynamic Sessions](https://learn.microsoft.com/en-us/azure/container-apps/sessions) | — | Microsoft | K0 | L4 (Hyper-V) |
| [AWS Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/) | — | AWS | K0 | L4 (Firecracker) |
| [GKE Agent Sandbox](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/agent-sandbox) | see upstream | Google Cloud | **Yes**, upstream is open | L3 native, L4 optional |
| [Alibaba Cloud AgentBay](https://www.aliyun.com/product/agentbay) | ![](https://img.shields.io/github/stars/agentbay-ai/wuying-agentbay-sdk?style=flat-square&label=%20) | Alibaba Cloud | Via ACS | L3/L4 |
| [Tencent Cloud AGSX](https://cloud.tencent.com/product/agsx) | see CubeSandbox | Tencent Cloud | Via CubeSandbox | L3/L4 |

## A note on reading star counts

Star counts measure attention, not fitness. Three specific traps in this space:

- **Daytona** and **OpenHands** accumulated most of their stars under earlier or broader positioning. Neither number reflects sandbox-specific adoption.
- **agent-sandbox** has far fewer stars than several products built on top of it, because standards attract contributors rather than stargazers.
- **kagent** is not a sandbox at all. Its star count regularly gets cited in this category by mistake.

## Selection guide

| If you need | Choose |
| --- | --- |
| Multi-tenant agent platform on your own Kubernetes | `agent-sandbox` with `RuntimeClass: gvisor` (balanced) or `kata` (strong) |
| Pause/resume to stop paying while agents idle, or GPU checkpointing | `OpenKruise Agents` |
| E2B SDK ergonomics with data staying in your network | `OpenKruise Agents` (E2B API compatible) |
| A complete platform with minimal integration work | `OpenSandbox` or `CubeSandbox` |
| Fastest path to shipping, no infrastructure work | `E2B` or `Daytona` |
| GPU inside the sandbox | `Modal` (currently the only option) |
| To protect your laptop from your own coding agent | `sandbox-runtime` — do not over-build this |
| Protection from the cloud provider itself | `Confidential Containers`, or Kata with Intel TDX / AMD SEV-SNP |
| To orchestrate agents, not isolate code | `kagent` — different problem, stack it with a sandbox |
