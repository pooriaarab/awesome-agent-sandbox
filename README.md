# Awesome Agent Sandbox [![Awesome](https://awesome.re/badge-flat2.svg)](https://awesome.re)

> Sandboxes for running untrusted, AI-agent-generated code.

LLM agents write and execute code nobody reviewed. A sandbox is what stands between that code and your kernel, your network, and your secrets.

Everything here isolates in one of three ways, and that choice matters more than any other. Start with the Isolation Runtimes section below — the rest of the list builds on top of it. Star badges are live. Side-by-side tables live in [COMPARISON.md](COMPARISON.md).

## Contents

- [Isolation Runtimes](#isolation-runtimes)
- [Kubernetes-Native Sandbox Orchestrators](#kubernetes-native-sandbox-orchestrators)
- [Self-Hosted Sandboxes](#self-hosted-sandboxes)
- [Managed Platforms](#managed-platforms)
- [Related Projects](#related-projects)
- [Articles](#articles)

## Isolation Runtimes

Three ways to isolate, weakest to strongest. Everything else in this list picks one, so these names are used throughout.

| Tier                   | How                                  | Overhead             | Run what               |
| ---------------------- | ------------------------------------ | -------------------- | ---------------------- |
| **Shared kernel**      | Namespaces, cgroups, syscall filters | None                 | Code you trust         |
| **Intercepted kernel** | Syscalls served in user space        | Moderate             | Code you mostly trust  |
| **Separate kernel**    | Own kernel in a virtual machine      | Boot time and memory | Code you did not write |

**Your own code needs a shared kernel at most. A stranger's needs a separate one.** The middle is a business decision, not a technical one.

### Shared kernel

> Same kernel as the host, fenced off with namespaces, cgroups, and syscall filters. Instant and cheap, but a kernel bug is a host compromise.

- [runc](https://github.com/opencontainers/runc) - The default, and the baseline this list measures against. Backed by the OCI. ![Stars](https://img.shields.io/github/stars/opencontainers/runc?style=flat-square&label=%20)
- [Sysbox](https://github.com/nestybox/sysbox) - Hardened runc that runs Docker-in-Docker and systemd safely. The strongest option that still shares a kernel. Backed by Docker. ![Stars](https://img.shields.io/github/stars/nestybox/sysbox?style=flat-square&label=%20)
- [bubblewrap](https://github.com/containers/bubblewrap) - Unprivileged process sandboxing, the engine under Flatpak. Backed by Red Hat. ![Stars](https://img.shields.io/github/stars/containers/bubblewrap?style=flat-square&label=%20)
- [nsjail](https://github.com/google/nsjail) - Namespaces, cgroups, rlimits, and seccomp-bpf in one small tool. Backed by Google. ![Stars](https://img.shields.io/github/stars/google/nsjail?style=flat-square&label=%20)

### Intercepted kernel

> Syscalls are reimplemented in user space and never reach the host kernel. No virtualization required, which is why it wins on managed Kubernetes.

- [gVisor](https://github.com/google/gvisor) - The only production option here, and the default on Google Kubernetes Engine. Backed by Google. ![Stars](https://img.shields.io/github/stars/google/gvisor?style=flat-square&label=%20)

### Separate kernel

> Its own Linux kernel in a virtual machine. Nothing shared, so an exploit stays in the box. The baseline for code you did not write.

- [Firecracker](https://github.com/firecracker-microvm/firecracker) - The microVM that created this category, booting in roughly 125ms. Backed by AWS. ![Stars](https://img.shields.io/github/stars/firecracker-microvm/firecracker?style=flat-square&label=%20)
- [Kata Containers](https://github.com/kata-containers/kata-containers) - The most mature Kubernetes integration, with swappable VMM backends. Add Intel TDX or AMD SEV-SNP when the host operator is also in your threat model. Backed by the OpenInfra Foundation. ![Stars](https://img.shields.io/github/stars/kata-containers/kata-containers?style=flat-square&label=%20)
- [Cloud Hypervisor](https://github.com/cloud-hypervisor/cloud-hypervisor) - Modern Rust VMM, commonly used as a Kata backend. Backed by the Linux Foundation. ![Stars](https://img.shields.io/github/stars/cloud-hypervisor/cloud-hypervisor?style=flat-square&label=%20)
- [Edera Protect](https://edera.dev/) - Type-1 hypervisor giving each pod its own kernel with no hardware virtualization required, unlocking instance types Kata cannot use. Core is closed; the [agent-sandbox integration](https://github.com/edera-dev/edera-agent) is public. ![Stars](https://img.shields.io/github/stars/edera-dev/edera-agent?style=flat-square&label=%20)

WebAssembly is a fourth option on a different axis: no kernel to escape, but no arbitrary Linux binaries either. See [runwasi](https://github.com/containerd/runwasi) and [WasmEdge](https://github.com/WasmEdge/WasmEdge).

## Kubernetes-Native Sandbox Orchestrators

Projects that make a **sandbox** a first-class Kubernetes resource. They do not implement isolation themselves — they pick a runtime from above via RuntimeClass and manage lifecycle, identity, and storage around it. This is where the ecosystem is standardizing.

Frameworks that orchestrate *agents* rather than sandboxes are under Related Projects, near the end.

- [agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) - The emerging standard, under Kubernetes SIG Apps. Four CRDs: Sandbox, SandboxTemplate, SandboxWarmPool, SandboxClaim. Any runtime, from shared to separate kernel. Still v0.x and not declared production-ready for all workloads. Backed by Google and Red Hat. ![Stars](https://img.shields.io/github/stars/kubernetes-sigs/agent-sandbox?style=flat-square&label=%20)
- [OpenSandbox](https://github.com/opensandbox-group/OpenSandbox) - Full platform rather than a bare API: five language SDKs, a unified sandbox protocol, MCP server, credential vault, and per-sandbox egress control. Any runtime. Backed by Alibaba, in the CNCF Landscape. ![Stars](https://img.shields.io/github/stars/opensandbox-group/OpenSandbox?style=flat-square&label=%20)
- [CubeSandbox](https://github.com/TencentCloud/CubeSandbox) - Production-proven stack open-sourced in 2026, deployable with or without Kubernetes, which makes it unusually easy to start small. Intercepted or separate kernel. Backed by Tencent Cloud. ![Stars](https://img.shields.io/github/stars/TencentCloud/CubeSandbox?style=flat-square&label=%20)
- [OpenKruise Agents](https://github.com/openkruise/agents) - A superset of the SIG API with ten CRDs, adding hibernation and checkpointing of memory, the read-write layer, and GPU memory. Speaks the E2B API, so existing E2B clients can target your own cluster. Any runtime. Backed by Alibaba Cloud under CNCF OpenKruise. ![Stars](https://img.shields.io/github/stars/openkruise/agents?style=flat-square&label=%20)
- [KubeVirt](https://github.com/kubevirt/kubevirt) - Full virtual machines as CRDs. Heavy, but the answer when a sandbox needs a GUI, a desktop, or Windows. Separate kernel. Backed by Red Hat under CNCF. ![Stars](https://img.shields.io/github/stars/kubevirt/kubevirt?style=flat-square&label=%20)
- [Confidential Containers](https://github.com/confidential-containers/operator) - Separate kernel plus a hardware TEE, for threat models where the cloud provider itself is untrusted. Backed by Intel, IBM, Red Hat, and Ant Group under CNCF. ![Stars](https://img.shields.io/github/stars/confidential-containers/operator?style=flat-square&label=%20)
- [vCluster](https://github.com/loft-sh/vcluster) - A virtual cluster per tenant. This isolates the control plane, not the workload, so stack it with a real runtime rather than relying on it alone. Backed by Loft Labs. ![Stars](https://img.shields.io/github/stars/loft-sh/vcluster?style=flat-square&label=%20)
- [mitos](https://github.com/mitos-run/mitos) - Millisecond microVM forking on Kubernetes, restoring from memory snapshots. Separate kernel. ![Stars](https://img.shields.io/github/stars/mitos-run/mitos?style=flat-square&label=%20)

Note that agent-sandbox and OpenKruise Agents are complementary rather than competing. OpenKruise Agents states it maintains compatibility with SIG Agent-Sandbox and layers higher-level APIs on top.

## Self-Hosted Sandboxes

Open source, and you can genuinely run them yourself — on a cluster, a server, or a laptop.

- [E2B](https://github.com/e2b-dev/E2B) - The most widely adopted agent sandbox SDK. Separate kernel via Firecracker. Self-hosting goes through [e2b-dev/infra](https://github.com/e2b-dev/infra), which is Terraform rather than Kubernetes. ![Stars](https://img.shields.io/github/stars/e2b-dev/E2B?style=flat-square&label=%20)
- [Daytona](https://github.com/daytonaio/daytona) - Fastest cold starts in the category, with official Helm charts. Shared kernel by default, upgradeable via Sysbox or Kata. The star count reflects an earlier life as a development environment manager, so read it with care. ![Stars](https://img.shields.io/github/stars/daytonaio/daytona?style=flat-square&label=%20)
- [microsandbox](https://github.com/superradcompany/microsandbox) - libkrun microVMs with sub-200ms starts, local-first. Separate kernel. ![Stars](https://img.shields.io/github/stars/superradcompany/microsandbox?style=flat-square&label=%20)
- [AIO Sandbox](https://github.com/agent-infra/sandbox) - Browser, shell, filesystem, MCP, and VS Code Server packaged in a single container. Shared kernel. Backed by ByteDance. ![Stars](https://img.shields.io/github/stars/agent-infra/sandbox?style=flat-square&label=%20)
- [forkd](https://github.com/deeplethe/forkd) - Brings fork semantics to microVMs, spawning a hundred children from a warm parent in roughly 100ms and branching a live VM. Separate kernel. ![Stars](https://img.shields.io/github/stars/deeplethe/forkd?style=flat-square&label=%20)
- [gondolin](https://github.com/earendil-works/gondolin) - Linux microVMs driven by a TypeScript control plane. Separate kernel. ![Stars](https://img.shields.io/github/stars/earendil-works/gondolin?style=flat-square&label=%20)
- [sandbox-runtime](https://github.com/anthropic-experimental/sandbox-runtime) - Filesystem and network restrictions via Seatbelt and bubblewrap, used by Claude Code. Deliberately a shared kernel, and a good reminder that protecting your laptop from your own agent does not warrant a microVM. Backed by Anthropic. ![Stars](https://img.shields.io/github/stars/anthropic-experimental/sandbox-runtime?style=flat-square&label=%20)
- [shuru](https://github.com/superhq-ai/shuru) - Local-first microVM sandbox for macOS and Linux. Separate kernel. ![Stars](https://img.shields.io/github/stars/superhq-ai/shuru?style=flat-square&label=%20)
- [clawk](https://github.com/clawkwork/clawk) - Hands a coding agent a disposable Linux VM instead of your laptop. Separate kernel. ![Stars](https://img.shields.io/github/stars/clawkwork/clawk?style=flat-square&label=%20)
- [arrakis](https://github.com/abshkbh/arrakis) - Self-hosted sandboxing with computer-use support, under AGPL-3.0. Separate kernel. ![Stars](https://img.shields.io/github/stars/abshkbh/arrakis?style=flat-square&label=%20)
- [brood-box](https://github.com/stacklok/brood-box) - CLI for running coding agents inside hardware-isolated microVMs. Separate kernel. Backed by Stacklok. ![Stars](https://img.shields.io/github/stars/stacklok/brood-box?style=flat-square&label=%20)
- [pullrun](https://github.com/pullrun/pullrun) - Boots any OCI image as a Firecracker microVM, a Linux container, or an Apple virtualization VM. ![Stars](https://img.shields.io/github/stars/pullrun/pullrun?style=flat-square&label=%20)
- [arcbox](https://github.com/arcboxlabs/arcbox) - Gives each agent its own kernel, filesystem, and network with sub-200ms boot. Separate kernel. ![Stars](https://img.shields.io/github/stars/arcboxlabs/arcbox?style=flat-square&label=%20)
- [llm-sandbox](https://github.com/vndee/llm-sandbox) - Small Python library with Docker, Podman, and Kubernetes backends, good for a quick integration. Shared kernel. ![Stars](https://img.shields.io/github/stars/vndee/llm-sandbox?style=flat-square&label=%20)
- [SWE-ReX](https://github.com/SWE-agent/SWE-ReX) - Massively parallel sandboxed execution built for agent benchmarking. Shared kernel. Backed by Princeton NLP. ![Stars](https://img.shields.io/github/stars/SWE-agent/SWE-ReX?style=flat-square&label=%20)
- [SandboxFusion](https://github.com/bytedance/SandboxFusion) - Multi-language execution aimed at evaluation and reinforcement learning. Shared kernel. Backed by ByteDance. ![Stars](https://img.shields.io/github/stars/bytedance/SandboxFusion?style=flat-square&label=%20)
- [cua](https://github.com/trycua/cua) - Computer-use drivers, cross-OS fleets, and benchmarks, for agents that need a screen rather than a shell. ![Stars](https://img.shields.io/github/stars/trycua/cua?style=flat-square&label=%20)
- [steel-browser](https://github.com/steel-dev/steel-browser) - Open-source browser API for AI agents and apps. ![Stars](https://img.shields.io/github/stars/steel-dev/steel-browser?style=flat-square&label=%20)
- [Lima](https://github.com/lima-vm/lima) - Linux virtual machines on macOS. Not agent-specific, but the substrate under several entries above. Backed by CNCF. ![Stars](https://img.shields.io/github/stars/lima-vm/lima?style=flat-square&label=%20)
- [container](https://github.com/apple/container) - Runs Linux containers as lightweight virtual machines on Apple silicon. Backed by Apple. ![Stars](https://img.shields.io/github/stars/apple/container?style=flat-square&label=%20)

## Managed Platforms

Hosted services. Note how few offer a self-hosting path, since that is the main tradeoff in this section.

- [Modal](https://modal.com/) - The only platform where a sandbox can hold a GPU, and it drops compute charges while idle. Intercepted kernel via gVisor. ![Stars](https://img.shields.io/github/stars/modal-labs/modal-client?style=flat-square&label=%20)
- [E2B Cloud](https://e2b.dev/) - Managed counterpart to the open-source SDK, with bring-your-own-cloud on enterprise plans. Separate kernel. ![Stars](https://img.shields.io/github/stars/e2b-dev/code-interpreter?style=flat-square&label=%20)
- [Vercel Sandbox](https://github.com/vercel/sandbox) - Ephemeral compute primitive with tight Vercel integration. Separate kernel. ![Stars](https://img.shields.io/github/stars/vercel/sandbox?style=flat-square&label=%20)
- [Cloudflare Sandbox SDK](https://github.com/cloudflare/sandbox-sdk) - Sandboxed code on the edge network, with the lowest latency and the most constrained environment. Shared kernel or WebAssembly isolate. ![Stars](https://img.shields.io/github/stars/cloudflare/sandbox-sdk?style=flat-square&label=%20)
- [Northflank](https://northflank.com/) - The one managed platform that also runs on your own Kubernetes, via self-hosting or bring-your-own-cloud. Separate kernel via Kata, plus gVisor.
- [Blaxel](https://github.com/blaxel-ai/sandbox) - Persistent environment state with roughly 25ms resume and no compute charge in standby. Separate kernel. ![Stars](https://img.shields.io/github/stars/blaxel-ai/sandbox?style=flat-square&label=%20)
- [Runloop](https://runloop.ai/) - Devboxes aimed specifically at coding agents. Separate kernel. ![Stars](https://img.shields.io/github/stars/runloopai/api-client-ts?style=flat-square&label=%20)
- [Azure Container Apps Dynamic Sessions](https://learn.microsoft.com/en-us/azure/container-apps/sessions) - Hyper-V isolation fronted by a warm pool, allocating in under 100ms at the 90th percentile. Separate kernel. Backed by Microsoft.
- [AWS Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/) - Managed Code Interpreter and Browser Tool with per-session pricing, joined by Lambda MicroVMs in 2026. Separate kernel.
- [GKE Agent Sandbox](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/agent-sandbox) - The rare managed product whose upstream is the open standard itself. Intercepted kernel by default. Backed by Google Cloud.
- [Alibaba Cloud AgentBay](https://www.aliyun.com/product/agentbay) - Four sandbox form factors: browser, cloud desktop, code, and cloud phone. ![Stars](https://img.shields.io/github/stars/agentbay-ai/wuying-agentbay-sdk?style=flat-square&label=%20)
- [Tencent Cloud AGSX](https://cloud.tencent.com/product/agsx) - Managed counterpart to CubeSandbox, covering browser, code, and custom sandboxes.
- [Browserbase](https://www.browserbase.com/) - Managed headless browser infrastructure for agents.

## Related Projects

Adjacent tooling that is routinely mistaken for a sandbox, plus the frameworks that bundle one.

- [kagent](https://github.com/kagent-dev/kagent) - Kubernetes-native, but **not a sandbox**. It orchestrates *agents* — Agent, ModelConfig, and ToolServer CRDs — and provides no isolation for untrusted code; that gets bolted on externally. It pairs with agent-sandbox rather than competing with it: kagent decides what runs, agent-sandbox decides what it runs inside. Backed by Solo.io under CNCF. ![Stars](https://img.shields.io/github/stars/kagent-dev/kagent?style=flat-square&label=%20)
- [kmcp](https://github.com/kagent-dev/kmcp) - Kubernetes controller and CLI for building, testing, and deploying MCP servers. Backed by Solo.io. ![Stars](https://img.shields.io/github/stars/kagent-dev/kmcp?style=flat-square&label=%20)
- [OpenHands](https://github.com/OpenHands/OpenHands) - Agent platform with a pluggable Docker or remote runtime. ![Stars](https://img.shields.io/github/stars/OpenHands/OpenHands?style=flat-square&label=%20)
- [DeerFlow](https://github.com/bytedance/deer-flow) - Agent harness with a three-tier sandbox covering local, Docker, and Kubernetes modes. Backed by ByteDance. ![Stars](https://img.shields.io/github/stars/bytedance/deer-flow?style=flat-square&label=%20)
- [Coder](https://github.com/coder/coder) - Kubernetes-native environments for developers and their agents. ![Stars](https://img.shields.io/github/stars/coder/coder?style=flat-square&label=%20)
- [The Agent Sandbox Taxonomy](https://github.com/kajogo777/the-agent-sandbox-taxonomy) - Seven defense layers, from compute isolation through credentials to audit logging, scored for strength and granularity across 27 products. The most rigorous evaluation framework in the space. ![Stars](https://img.shields.io/github/stars/kajogo777/the-agent-sandbox-taxonomy?style=flat-square&label=%20)

## Articles

- [Comparing AI agent sandbox platforms](https://blog.logrocket.com/comparing-ai-agent-sandbox-platforms-e2b-modal-daytona-and-more/) - Measured cold-start and resume timings rather than vendor claims.
- [Bringing you Agent Sandbox on GKE and Agent Substrate](https://cloud.google.com/blog/products/containers-kubernetes/bringing-you-agent-sandbox-on-gke-and-agent-substrate) - Google Cloud on the origin and trajectory of the SIG project.
- [Red Hat build of Agent Sandbox](https://developers.redhat.com/articles/2026/07/15/red-hat-build-agent-sandbox-isolated-workload-management-kubernetes) - What a supported downstream distribution of the standard looks like.
- [Sandboxes on Kubernetes: isolation options](https://northflank.com/blog/sandboxes-on-kubernetes) - Walks runc, gVisor, Kata, and Firecracker with concrete tradeoffs.
- [How to sandbox AI agents in 2026](https://northflank.com/blog/how-to-sandbox-ai-agents) - Practical isolation strategies for agent workloads.
- [Kata vs Firecracker vs gVisor](https://edera.dev/stories/kata-vs-firecracker-vs-gvisor-isolation-compared) - Runtime comparison, written by a vendor with a horse in the race.
- [Bringing Agentic AI to Kubernetes](https://www.solo.io/blog/bringing-agentic-ai-to-kubernetes-contributing-kagent-to-cncf) - Solo.io on contributing kagent to CNCF.

## Contributing

Contributions are welcome. Read the [contribution guidelines](CONTRIBUTING.md) first, since entries need a stated isolation model and a stated backer rather than just a link.
