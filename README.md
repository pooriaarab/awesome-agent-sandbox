# Awesome Agent Sandbox [![Awesome](https://awesome.re/badge-flat2.svg)](https://awesome.re)

> Sandboxes for running untrusted, AI-agent-generated code.

LLM agents write and execute code nobody reviewed. A sandbox is what stands between that code and your kernel, your network, and your secrets. This list covers the whole stack: the isolation runtimes underneath, the Kubernetes-native orchestrators forming around them, the self-hostable products, and the managed platforms.

Every star badge is live and updates itself. Each entry states its **isolation level** and who backs it, because those are the two facts that actually decide a selection. Side-by-side tables live in [COMPARISON.md](COMPARISON.md).

## Contents

- [Isolation Levels](#isolation-levels)
- [Kubernetes-Native Orchestrators](#kubernetes-native-orchestrators)
- [Isolation Runtimes](#isolation-runtimes)
- [Self-Hostable Sandboxes](#self-hostable-sandboxes)
- [Local and Desktop Sandboxes](#local-and-desktop-sandboxes)
- [Managed Platforms](#managed-platforms)
- [Computer-Use and Browser Sandboxes](#computer-use-and-browser-sandboxes)
- [Frameworks That Ship a Sandbox](#frameworks-that-ship-a-sandbox)
- [Taxonomies and Benchmarks](#taxonomies-and-benchmarks)
- [Articles](#articles)

## Isolation Levels

One scale, used throughout the list. Pick the weakest level that covers your threat model, because every step up costs startup latency, memory, or hardware requirements.

| Level  | Name                   | Mechanism                              | Kernel shared      | Stops a kernel 0-day       |
| ------ | ---------------------- | -------------------------------------- | ------------------ | -------------------------- |
| **L0** | Process sandbox        | seccomp-bpf, Landlock, macOS Seatbelt  | Yes                | No                         |
| **L1** | Standard container     | namespaces and cgroups (runc)          | Yes                | No                         |
| **L2** | Hardened container     | user namespaces, syscall shimming      | Yes                | Partially                  |
| **L3** | User-space kernel      | syscalls reimplemented in user space   | Logically no       | Mostly                     |
| **L4** | microVM                | KVM or Hyper-V, dedicated guest kernel | **No**             | Yes                        |
| **L5** | Confidential computing | L4 plus a TEE (Intel TDX, AMD SEV-SNP) | No, opaque to host | Yes, and the host operator |
| **Lw** | WebAssembly            | language VM, capability allowlist      | Not applicable     | Yes, but no Linux binaries |

Rule of thumb: your own agent on your own laptop is L0. A customer's generated code is L4. Everything in between is a business decision, not a technical one.

## Kubernetes-Native Orchestrators

Projects that make a sandbox a first-class Kubernetes resource. This is where the ecosystem is standardizing.

- [agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) - The emerging standard, under Kubernetes SIG Apps. Four CRDs: Sandbox, SandboxTemplate, SandboxWarmPool, SandboxClaim. Isolation is delegated to a RuntimeClass, so it spans L1 to L5. Still v0.x and not declared production-ready for all workloads. Backed by Google and Red Hat. ![Stars](https://img.shields.io/github/stars/kubernetes-sigs/agent-sandbox?style=flat-square&label=%20)
- [OpenSandbox](https://github.com/opensandbox-group/OpenSandbox) - Full platform rather than a bare API: five language SDKs, a unified sandbox protocol, MCP server, credential vault, and per-sandbox egress control. Pluggable L1 to L4. Backed by Alibaba, in the CNCF Landscape. ![Stars](https://img.shields.io/github/stars/opensandbox-group/OpenSandbox?style=flat-square&label=%20)
- [CubeSandbox](https://github.com/TencentCloud/CubeSandbox) - Production-proven stack open-sourced in 2026, deployable with or without Kubernetes, which makes it unusually easy to start small. L3 and L4. Backed by Tencent Cloud. ![Stars](https://img.shields.io/github/stars/TencentCloud/CubeSandbox?style=flat-square&label=%20)
- [OpenKruise Agents](https://github.com/openkruise/agents) - A superset of the SIG API with ten CRDs, adding hibernation and checkpointing of memory, the read-write layer, and GPU memory. Speaks the E2B API, so existing E2B clients can target your own cluster. Pluggable L1 to L4. Backed by Alibaba Cloud under CNCF OpenKruise. ![Stars](https://img.shields.io/github/stars/openkruise/agents?style=flat-square&label=%20)
- [KubeVirt](https://github.com/kubevirt/kubevirt) - Full virtual machines as CRDs. Heavy, but the answer when a sandbox needs a GUI, a desktop, or Windows. L4. Backed by Red Hat under CNCF. ![Stars](https://img.shields.io/github/stars/kubevirt/kubevirt?style=flat-square&label=%20)
- [vCluster](https://github.com/loft-sh/vcluster) - A virtual cluster per tenant. This is control-plane isolation, not workload isolation, so stack it with an L3 or L4 runtime rather than relying on it alone. Backed by Loft Labs. ![Stars](https://img.shields.io/github/stars/loft-sh/vcluster?style=flat-square&label=%20)
- [Confidential Containers](https://github.com/confidential-containers/operator) - For threat models where the cloud provider itself is untrusted. L5. Backed by Intel, IBM, Red Hat, and Ant Group under CNCF. ![Stars](https://img.shields.io/github/stars/confidential-containers/operator?style=flat-square&label=%20)
- [mitos](https://github.com/mitos-run/mitos) - Millisecond microVM forking on Kubernetes, restoring from memory snapshots. L4. ![Stars](https://img.shields.io/github/stars/mitos-run/mitos?style=flat-square&label=%20)

Note that agent-sandbox and OpenKruise Agents are complementary rather than competing. OpenKruise Agents states it maintains compatibility with SIG Agent-Sandbox and layers higher-level APIs on top.

## Isolation Runtimes

The layer everything above delegates to. Wire these in with a RuntimeClass.

- [Firecracker](https://github.com/firecracker-microvm/firecracker) - The microVM that created this category, booting in roughly 125ms. L4. Backed by AWS. ![Stars](https://img.shields.io/github/stars/firecracker-microvm/firecracker?style=flat-square&label=%20)
- [gVisor](https://github.com/google/gvisor) - User-space kernel that intercepts syscalls before they reach the host. Needs no nested virtualization, which is why it wins on managed Kubernetes. L3. Backed by Google. ![Stars](https://img.shields.io/github/stars/google/gvisor?style=flat-square&label=%20)
- [Kata Containers](https://github.com/kata-containers/kata-containers) - The most mature Kubernetes integration of the microVM runtimes, with swappable VMM backends. Reaches L5 with TDX or SEV. Backed by the OpenInfra Foundation. ![Stars](https://img.shields.io/github/stars/kata-containers/kata-containers?style=flat-square&label=%20)
- [Cloud Hypervisor](https://github.com/cloud-hypervisor/cloud-hypervisor) - Modern Rust VMM, frequently used as a Kata backend. L4. Backed by the Linux Foundation. ![Stars](https://img.shields.io/github/stars/cloud-hypervisor/cloud-hypervisor?style=flat-square&label=%20)
- [Sysbox](https://github.com/nestybox/sysbox) - Hardened runc that runs Docker-in-Docker and systemd safely without a virtual machine. L2. Backed by Docker. ![Stars](https://img.shields.io/github/stars/nestybox/sysbox?style=flat-square&label=%20)
- [Edera Protect](https://edera.dev/) - Container-native Type-1 hypervisor giving each pod its own kernel without requiring hardware virtualization extensions. Core is closed source; the [agent-sandbox integration](https://github.com/edera-dev/edera-agent) is public. L4. ![Stars](https://img.shields.io/github/stars/edera-dev/edera-agent?style=flat-square&label=%20)
- [runc](https://github.com/opencontainers/runc) - The baseline, listed so the tradeoff is explicit. L1. Backed by the OCI. ![Stars](https://img.shields.io/github/stars/opencontainers/runc?style=flat-square&label=%20)
- [youki](https://github.com/youki-dev/youki) - OCI container runtime written in Rust. L1. ![Stars](https://img.shields.io/github/stars/youki-dev/youki?style=flat-square&label=%20)
- [bubblewrap](https://github.com/containers/bubblewrap) - Unprivileged process sandboxing, the engine underneath Flatpak. L0 to L1. Backed by Red Hat. ![Stars](https://img.shields.io/github/stars/containers/bubblewrap?style=flat-square&label=%20)
- [nsjail](https://github.com/google/nsjail) - Namespaces, cgroups, rlimits, and seccomp-bpf in one small tool. L0 to L1. Backed by Google. ![Stars](https://img.shields.io/github/stars/google/nsjail?style=flat-square&label=%20)
- [runwasi](https://github.com/containerd/runwasi) - Runs WebAssembly and WASI workloads under containerd. Lw. Backed by CNCF. ![Stars](https://img.shields.io/github/stars/containerd/runwasi?style=flat-square&label=%20)
- [WasmEdge](https://github.com/WasmEdge/WasmEdge) - WebAssembly runtime for cloud-native and edge workloads. Lw. Backed by CNCF. ![Stars](https://img.shields.io/github/stars/WasmEdge/WasmEdge?style=flat-square&label=%20)

## Self-Hostable Sandboxes

Open source, and you can genuinely run them yourself.

- [E2B](https://github.com/e2b-dev/E2B) - The most widely adopted agent sandbox SDK, built on Firecracker. Self-hosting goes through [e2b-dev/infra](https://github.com/e2b-dev/infra), which is Terraform rather than Kubernetes. L4. ![Stars](https://img.shields.io/github/stars/e2b-dev/E2B?style=flat-square&label=%20)
- [Daytona](https://github.com/daytonaio/daytona) - Fastest cold starts in the category, with official Helm charts. L1 by default, optionally L2 or L4 through Sysbox or Kata. The star count reflects an earlier life as a development environment manager, so read it with care. ![Stars](https://img.shields.io/github/stars/daytonaio/daytona?style=flat-square&label=%20)
- [microsandbox](https://github.com/superradcompany/microsandbox) - libkrun microVMs with sub-200ms starts, local-first. L4. ![Stars](https://img.shields.io/github/stars/superradcompany/microsandbox?style=flat-square&label=%20)
- [AIO Sandbox](https://github.com/agent-infra/sandbox) - Browser, shell, filesystem, MCP, and VS Code Server packaged in a single container. L1. Backed by ByteDance. ![Stars](https://img.shields.io/github/stars/agent-infra/sandbox?style=flat-square&label=%20)
- [forkd](https://github.com/deeplethe/forkd) - Brings fork semantics to microVMs, spawning a hundred children from a warm parent in roughly 100ms and branching a live VM. L4. ![Stars](https://img.shields.io/github/stars/deeplethe/forkd?style=flat-square&label=%20)
- [gondolin](https://github.com/earendil-works/gondolin) - Linux microVMs driven by a TypeScript control plane. L4. ![Stars](https://img.shields.io/github/stars/earendil-works/gondolin?style=flat-square&label=%20)
- [SandboxFusion](https://github.com/bytedance/SandboxFusion) - Multi-language execution aimed at evaluation and reinforcement learning. L1. Backed by ByteDance. ![Stars](https://img.shields.io/github/stars/bytedance/SandboxFusion?style=flat-square&label=%20)
- [llm-sandbox](https://github.com/vndee/llm-sandbox) - Small Python library with Docker, Podman, and Kubernetes backends, good for a quick integration. L1. ![Stars](https://img.shields.io/github/stars/vndee/llm-sandbox?style=flat-square&label=%20)
- [SWE-ReX](https://github.com/SWE-agent/SWE-ReX) - Massively parallel sandboxed execution built for agent benchmarking. L1. Backed by Princeton NLP. ![Stars](https://img.shields.io/github/stars/SWE-agent/SWE-ReX?style=flat-square&label=%20)
- [pullrun](https://github.com/pullrun/pullrun) - Boots any OCI image as a Firecracker microVM, a Linux container, or an Apple virtualization VM. L4 or L1. ![Stars](https://img.shields.io/github/stars/pullrun/pullrun?style=flat-square&label=%20)
- [arcbox](https://github.com/arcboxlabs/arcbox) - Gives each agent its own kernel, filesystem, and network with sub-200ms boot. L4. ![Stars](https://img.shields.io/github/stars/arcboxlabs/arcbox?style=flat-square&label=%20)

## Local and Desktop Sandboxes

For protecting your own machine from your own coding agent. The threat model here is mistakes, not adversaries, so a microVM is usually more than you need.

- [sandbox-runtime](https://github.com/anthropic-experimental/sandbox-runtime) - Filesystem and network restrictions via Seatbelt and bubblewrap, used by Claude Code. Deliberately L0, and a good reminder that not every agent needs a kernel boundary. Backed by Anthropic. ![Stars](https://img.shields.io/github/stars/anthropic-experimental/sandbox-runtime?style=flat-square&label=%20)
- [shuru](https://github.com/superhq-ai/shuru) - Local-first microVM sandbox for macOS and Linux. L4. ![Stars](https://img.shields.io/github/stars/superhq-ai/shuru?style=flat-square&label=%20)
- [clawk](https://github.com/clawkwork/clawk) - Hands a coding agent a disposable Linux VM instead of your laptop. L4. ![Stars](https://img.shields.io/github/stars/clawkwork/clawk?style=flat-square&label=%20)
- [arrakis](https://github.com/abshkbh/arrakis) - Self-hosted sandboxing with computer-use support, under AGPL-3.0. L4. ![Stars](https://img.shields.io/github/stars/abshkbh/arrakis?style=flat-square&label=%20)
- [brood-box](https://github.com/stacklok/brood-box) - CLI for running coding agents inside hardware-isolated microVMs. L4. Backed by Stacklok. ![Stars](https://img.shields.io/github/stars/stacklok/brood-box?style=flat-square&label=%20)
- [Lima](https://github.com/lima-vm/lima) - Linux virtual machines on macOS. Not agent-specific, but the substrate under several entries above. L4. Backed by CNCF. ![Stars](https://img.shields.io/github/stars/lima-vm/lima?style=flat-square&label=%20)
- [container](https://github.com/apple/container) - Runs Linux containers as lightweight virtual machines on Apple silicon. L4. Backed by Apple. ![Stars](https://img.shields.io/github/stars/apple/container?style=flat-square&label=%20)

## Managed Platforms

Hosted services. Note how few offer a self-hosting path, since that is the main tradeoff in this section.

- [Modal](https://modal.com/) - The only platform where a sandbox can hold a GPU, and it drops compute charges while idle. L3 via gVisor. ![Stars](https://img.shields.io/github/stars/modal-labs/modal-client?style=flat-square&label=%20)
- [E2B Cloud](https://e2b.dev/) - Managed counterpart to the open-source SDK, with bring-your-own-cloud on enterprise plans. L4. ![Stars](https://img.shields.io/github/stars/e2b-dev/code-interpreter?style=flat-square&label=%20)
- [Vercel Sandbox](https://github.com/vercel/sandbox) - Ephemeral compute primitive with tight Vercel integration. L4. ![Stars](https://img.shields.io/github/stars/vercel/sandbox?style=flat-square&label=%20)
- [Cloudflare Sandbox SDK](https://github.com/cloudflare/sandbox-sdk) - Sandboxed code on the edge network, with the lowest latency and the most constrained environment. L1 container or Lw isolate. ![Stars](https://img.shields.io/github/stars/cloudflare/sandbox-sdk?style=flat-square&label=%20)
- [Northflank](https://northflank.com/) - The one managed platform that also runs on your own Kubernetes, via self-hosting or bring-your-own-cloud. L4 Kata plus L3 gVisor.
- [Blaxel](https://github.com/blaxel-ai/sandbox) - Persistent environment state with roughly 25ms resume and no compute charge in standby. L4. ![Stars](https://img.shields.io/github/stars/blaxel-ai/sandbox?style=flat-square&label=%20)
- [Runloop](https://runloop.ai/) - Devboxes aimed specifically at coding agents. L4. ![Stars](https://img.shields.io/github/stars/runloopai/api-client-ts?style=flat-square&label=%20)
- [Azure Container Apps Dynamic Sessions](https://learn.microsoft.com/en-us/azure/container-apps/sessions) - Hyper-V isolation fronted by a warm pool, allocating in under 100ms at the 90th percentile. L4. Backed by Microsoft.
- [AWS Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/) - Managed Code Interpreter and Browser Tool with per-session pricing, joined by Lambda MicroVMs in 2026. L4.
- [GKE Agent Sandbox](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/agent-sandbox) - The rare managed product whose upstream is the open standard itself. L3 natively, L4 optionally. Backed by Google Cloud.
- [Alibaba Cloud AgentBay](https://www.aliyun.com/product/agentbay) - Four sandbox form factors: browser, cloud desktop, code, and cloud phone. L3 to L4. ![Stars](https://img.shields.io/github/stars/agentbay-ai/wuying-agentbay-sdk?style=flat-square&label=%20)
- [Tencent Cloud AGSX](https://cloud.tencent.com/product/agsx) - Managed counterpart to CubeSandbox, covering browser, code, and custom sandboxes. L3 to L4.

## Computer-Use and Browser Sandboxes

For agents that need a screen or a browser rather than a shell.

- [cua](https://github.com/trycua/cua) - Computer-use drivers, cross-OS fleets, and benchmarks for training and evaluation. ![Stars](https://img.shields.io/github/stars/trycua/cua?style=flat-square&label=%20)
- [steel-browser](https://github.com/steel-dev/steel-browser) - Open-source browser API for AI agents and apps. ![Stars](https://img.shields.io/github/stars/steel-dev/steel-browser?style=flat-square&label=%20)
- [Browserbase](https://www.browserbase.com/) - Managed headless browser infrastructure for agents.

## Frameworks That Ship a Sandbox

Not sandboxes themselves, but they bundle one. Listed because they are routinely mistaken for this category.

- [kagent](https://github.com/kagent-dev/kagent) - Not a sandbox, despite the name. A Kubernetes-native framework for running agents, with Agent, ModelConfig, and ToolServer CRDs; isolation is bolted on externally. Complementary to agent-sandbox, which decides what the agent runs inside. Backed by Solo.io under CNCF. ![Stars](https://img.shields.io/github/stars/kagent-dev/kagent?style=flat-square&label=%20)
- [OpenHands](https://github.com/OpenHands/OpenHands) - Agent platform with a pluggable Docker or remote runtime. ![Stars](https://img.shields.io/github/stars/OpenHands/OpenHands?style=flat-square&label=%20)
- [DeerFlow](https://github.com/bytedance/deer-flow) - Agent harness with a three-tier sandbox covering local, Docker, and Kubernetes modes. ![Stars](https://img.shields.io/github/stars/bytedance/deer-flow?style=flat-square&label=%20)
- [kmcp](https://github.com/kagent-dev/kmcp) - Kubernetes controller and CLI for building, testing, and deploying MCP servers. ![Stars](https://img.shields.io/github/stars/kagent-dev/kmcp?style=flat-square&label=%20)
- [Coder](https://github.com/coder/coder) - Kubernetes-native environments for developers and their agents. ![Stars](https://img.shields.io/github/stars/coder/coder?style=flat-square&label=%20)

## Taxonomies and Benchmarks

- [The Agent Sandbox Taxonomy](https://github.com/kajogo777/the-agent-sandbox-taxonomy) - Seven defense layers, from compute isolation through credentials to audit logging, scored for strength and granularity across 27 products. The most rigorous evaluation framework in the space. ![Stars](https://img.shields.io/github/stars/kajogo777/the-agent-sandbox-taxonomy?style=flat-square&label=%20)
- [Comparing AI agent sandbox platforms](https://blog.logrocket.com/comparing-ai-agent-sandbox-platforms-e2b-modal-daytona-and-more/) - Measured cold-start and resume timings rather than vendor claims.

## Articles

- [Bringing you Agent Sandbox on GKE and Agent Substrate](https://cloud.google.com/blog/products/containers-kubernetes/bringing-you-agent-sandbox-on-gke-and-agent-substrate) - Google Cloud on the origin and trajectory of the SIG project.
- [Red Hat build of Agent Sandbox](https://developers.redhat.com/articles/2026/07/15/red-hat-build-agent-sandbox-isolated-workload-management-kubernetes) - What a supported downstream distribution of the standard looks like.
- [Sandboxes on Kubernetes: isolation options](https://northflank.com/blog/sandboxes-on-kubernetes) - Walks runc, gVisor, Kata, and Firecracker with concrete tradeoffs.
- [How to sandbox AI agents in 2026](https://northflank.com/blog/how-to-sandbox-ai-agents) - Practical isolation strategies for agent workloads.
- [Kata vs Firecracker vs gVisor](https://edera.dev/stories/kata-vs-firecracker-vs-gvisor-isolation-compared) - Runtime comparison, written by a vendor with a horse in the race.
- [Bringing Agentic AI to Kubernetes](https://www.solo.io/blog/bringing-agentic-ai-to-kubernetes-contributing-kagent-to-cncf) - Solo.io on contributing kagent to CNCF.

## Contributing

Contributions are welcome. Read the [contribution guidelines](CONTRIBUTING.md) first, since entries need a stated isolation level and a stated backer rather than just a link.
