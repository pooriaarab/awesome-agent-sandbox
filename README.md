# Awesome Agent Sandbox [![Awesome](https://awesome.re/badge-flat2.svg)](https://awesome.re)

> Sandboxes for running untrusted, AI-agent-generated code.

LLM agents write and execute code nobody reviewed. A sandbox is what stands between that code and your kernel, your network, and your secrets.

Every entry is one line: what it is, then its dimensions as tags — `who backs it`, `how it isolates`, `how you run it`. Star badges are live. Full side-by-side tables are in [COMPARISON.md](COMPARISON.md).

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

- [runc](https://github.com/opencontainers/runc) - The default, and the baseline this list measures against. `OCI` `RuntimeClass` ![Stars](https://img.shields.io/github/stars/opencontainers/runc?style=flat-square&label=%20)
- [Sysbox](https://github.com/nestybox/sysbox) - Hardened runc; the strongest option that still shares a kernel. `Docker` `RuntimeClass` ![Stars](https://img.shields.io/github/stars/nestybox/sysbox?style=flat-square&label=%20)
- [bubblewrap](https://github.com/containers/bubblewrap) - Unprivileged process sandboxing, the engine under Flatpak. `Red Hat` `local` ![Stars](https://img.shields.io/github/stars/containers/bubblewrap?style=flat-square&label=%20)
- [nsjail](https://github.com/google/nsjail) - Namespaces, cgroups, rlimits, and seccomp-bpf in one small tool. `Google` `local` ![Stars](https://img.shields.io/github/stars/google/nsjail?style=flat-square&label=%20)

### Intercepted kernel

> Syscalls are reimplemented in user space and never reach the host kernel. No virtualization required, which is why it wins on managed Kubernetes.

- [gVisor](https://github.com/google/gvisor) - The only production option here, and the default on GKE. `Google` `RuntimeClass` ![Stars](https://img.shields.io/github/stars/google/gvisor?style=flat-square&label=%20)

### Separate kernel

> Its own Linux kernel in a virtual machine. Nothing shared, so an exploit stays in the box. The baseline for code you did not write.

- [Firecracker](https://github.com/firecracker-microvm/firecracker) - The microVM that created this category. `AWS` `~125ms boot` ![Stars](https://img.shields.io/github/stars/firecracker-microvm/firecracker?style=flat-square&label=%20)
- [Kata Containers](https://github.com/kata-containers/kata-containers) - The most mature Kubernetes integration, with swappable VMM backends. `OpenInfra` `RuntimeClass` `+ TEE` ![Stars](https://img.shields.io/github/stars/kata-containers/kata-containers?style=flat-square&label=%20)
- [Cloud Hypervisor](https://github.com/cloud-hypervisor/cloud-hypervisor) - Modern Rust VMM, commonly used as a Kata backend. `Linux Foundation` `via Kata` ![Stars](https://img.shields.io/github/stars/cloud-hypervisor/cloud-hypervisor?style=flat-square&label=%20)
- [Edera Protect](https://edera.dev/) - Type-1 hypervisor per pod, with no hardware virtualization required. `Edera` `RuntimeClass` `core closed` ![Stars](https://img.shields.io/github/stars/edera-dev/edera-agent?style=flat-square&label=%20)

WebAssembly is a fourth option on a different axis: no kernel to escape, but no arbitrary Linux binaries either. See [runwasi](https://github.com/containerd/runwasi) and [WasmEdge](https://github.com/WasmEdge/WasmEdge).

## Kubernetes-Native Sandbox Orchestrators

Projects that make a **sandbox** a first-class Kubernetes resource. They do not isolate anything themselves — they pick a runtime from above and manage lifecycle, identity, and storage around it.

Frameworks that orchestrate *agents* rather than sandboxes are under Related Projects.

- [agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) - The emerging standard: four CRDs, warm pools, stable identity. `Google + Red Hat` `any runtime` `v0.x` ![Stars](https://img.shields.io/github/stars/kubernetes-sigs/agent-sandbox?style=flat-square&label=%20)
- [OpenSandbox](https://github.com/opensandbox-group/OpenSandbox) - Complete platform: five language SDKs, credential vault, per-sandbox egress control. `Alibaba` `any runtime` `CNCF Landscape` ![Stars](https://img.shields.io/github/stars/opensandbox-group/OpenSandbox?style=flat-square&label=%20)
- [CubeSandbox](https://github.com/TencentCloud/CubeSandbox) - Production-proven stack that also deploys without a cluster at all. `Tencent Cloud` `intercepted or separate` `native CRDs` ![Stars](https://img.shields.io/github/stars/TencentCloud/CubeSandbox?style=flat-square&label=%20)
- [OpenKruise Agents](https://github.com/openkruise/agents) - Superset of the SIG API that also speaks the E2B API, and checkpoints GPU memory. `Alibaba Cloud` `any runtime` `10 CRDs` ![Stars](https://img.shields.io/github/stars/openkruise/agents?style=flat-square&label=%20)
- [KubeVirt](https://github.com/kubevirt/kubevirt) - Full VMs as CRDs; the answer when a sandbox needs a GUI or Windows. `Red Hat` `separate kernel` `heavy` ![Stars](https://img.shields.io/github/stars/kubevirt/kubevirt?style=flat-square&label=%20)
- [Confidential Containers](https://github.com/confidential-containers/operator) - For threat models where the cloud provider itself is untrusted. `Intel + IBM + Red Hat` `separate kernel + TEE` ![Stars](https://img.shields.io/github/stars/confidential-containers/operator?style=flat-square&label=%20)
- [vCluster](https://github.com/loft-sh/vcluster) - A virtual cluster per tenant; isolates the control plane, not the workload. `Loft Labs` `stack with a real runtime` ![Stars](https://img.shields.io/github/stars/loft-sh/vcluster?style=flat-square&label=%20)
- [mitos](https://github.com/mitos-run/mitos) - Millisecond microVM forking from memory snapshots. `mitos-run` `separate kernel` `native CRDs` ![Stars](https://img.shields.io/github/stars/mitos-run/mitos?style=flat-square&label=%20)

agent-sandbox and OpenKruise Agents are complementary, not competing — the latter states it stays compatible with the SIG API and layers higher-level features on top.

## Self-Hosted Sandboxes

Open source, and you can genuinely run them yourself.

- [E2B](https://github.com/e2b-dev/E2B) - The most widely adopted agent sandbox SDK. `E2B` `separate kernel` `Terraform, not K8s` ![Stars](https://img.shields.io/github/stars/e2b-dev/E2B?style=flat-square&label=%20)
- [Daytona](https://github.com/daytonaio/daytona) - Fastest cold starts in the category; star count predates the sandbox pivot. `Daytona` `shared kernel, upgradeable` `Helm` ![Stars](https://img.shields.io/github/stars/daytonaio/daytona?style=flat-square&label=%20)
- [microsandbox](https://github.com/superradcompany/microsandbox) - libkrun microVMs with sub-200ms starts. `Superrad` `separate kernel` `local-first` ![Stars](https://img.shields.io/github/stars/superradcompany/microsandbox?style=flat-square&label=%20)
- [AIO Sandbox](https://github.com/agent-infra/sandbox) - Browser, shell, files, MCP, and VS Code Server in one container. `ByteDance` `shared kernel` `Helm` ![Stars](https://img.shields.io/github/stars/agent-infra/sandbox?style=flat-square&label=%20)
- [forkd](https://github.com/deeplethe/forkd) - Fork semantics for microVMs: 100 children from a warm parent in ~100ms. `deeplethe` `separate kernel` ![Stars](https://img.shields.io/github/stars/deeplethe/forkd?style=flat-square&label=%20)
- [gondolin](https://github.com/earendil-works/gondolin) - Linux microVMs driven by a TypeScript control plane. `earendil.works` `separate kernel` ![Stars](https://img.shields.io/github/stars/earendil-works/gondolin?style=flat-square&label=%20)
- [sandbox-runtime](https://github.com/anthropic-experimental/sandbox-runtime) - Filesystem and network limits for a coding agent on your own laptop. `Anthropic` `shared kernel, by design` `local` ![Stars](https://img.shields.io/github/stars/anthropic-experimental/sandbox-runtime?style=flat-square&label=%20)
- [shuru](https://github.com/superhq-ai/shuru) - Local-first microVM sandbox for macOS and Linux. `SuperHQ` `separate kernel` `local` ![Stars](https://img.shields.io/github/stars/superhq-ai/shuru?style=flat-square&label=%20)
- [clawk](https://github.com/clawkwork/clawk) - Hands a coding agent a disposable Linux VM instead of your laptop. `Clawkwork` `separate kernel` `local` ![Stars](https://img.shields.io/github/stars/clawkwork/clawk?style=flat-square&label=%20)
- [arrakis](https://github.com/abshkbh/arrakis) - Self-hosted sandboxing with computer-use support. `Community` `separate kernel` `AGPL-3.0` ![Stars](https://img.shields.io/github/stars/abshkbh/arrakis?style=flat-square&label=%20)
- [brood-box](https://github.com/stacklok/brood-box) - CLI for running coding agents inside hardware-isolated microVMs. `Stacklok` `separate kernel` `local` ![Stars](https://img.shields.io/github/stars/stacklok/brood-box?style=flat-square&label=%20)
- [pullrun](https://github.com/pullrun/pullrun) - Boots any OCI image as a microVM, a container, or an Apple VM. `pullrun` `separate or shared` ![Stars](https://img.shields.io/github/stars/pullrun/pullrun?style=flat-square&label=%20)
- [arcbox](https://github.com/arcboxlabs/arcbox) - Own kernel, filesystem, and network per agent, in under 200ms. `Arcbox Labs` `separate kernel` ![Stars](https://img.shields.io/github/stars/arcboxlabs/arcbox?style=flat-square&label=%20)
- [llm-sandbox](https://github.com/vndee/llm-sandbox) - Small Python library for a quick integration. `Community` `shared kernel` `Docker, Podman, K8s` ![Stars](https://img.shields.io/github/stars/vndee/llm-sandbox?style=flat-square&label=%20)
- [SWE-ReX](https://github.com/SWE-agent/SWE-ReX) - Massively parallel execution built for agent benchmarking. `Princeton NLP` `shared kernel` ![Stars](https://img.shields.io/github/stars/SWE-agent/SWE-ReX?style=flat-square&label=%20)
- [SandboxFusion](https://github.com/bytedance/SandboxFusion) - Multi-language execution aimed at evaluation and RL. `ByteDance` `shared kernel` ![Stars](https://img.shields.io/github/stars/bytedance/SandboxFusion?style=flat-square&label=%20)
- [cua](https://github.com/trycua/cua) - Computer-use drivers and cross-OS fleets, for agents that need a screen. `Cua` `computer-use` ![Stars](https://img.shields.io/github/stars/trycua/cua?style=flat-square&label=%20)
- [steel-browser](https://github.com/steel-dev/steel-browser) - Open-source browser API for AI agents. `Steel` `browser` ![Stars](https://img.shields.io/github/stars/steel-dev/steel-browser?style=flat-square&label=%20)
- [Lima](https://github.com/lima-vm/lima) - Linux VMs on macOS; the substrate under several entries above. `CNCF` `separate kernel` `local` ![Stars](https://img.shields.io/github/stars/lima-vm/lima?style=flat-square&label=%20)
- [container](https://github.com/apple/container) - Linux containers as lightweight VMs on Apple silicon. `Apple` `separate kernel` `local` ![Stars](https://img.shields.io/github/stars/apple/container?style=flat-square&label=%20)

## Managed Platforms

Hosted services. Note how few offer a self-hosting path — that is the defining tradeoff here.

- [Modal](https://modal.com/) - The only platform where a sandbox can hold a GPU; idle costs nothing. `Modal Labs` `intercepted kernel` `no self-host` ![Stars](https://img.shields.io/github/stars/modal-labs/modal-client?style=flat-square&label=%20)
- [E2B Cloud](https://e2b.dev/) - Managed counterpart to the open-source SDK. `E2B` `separate kernel` `BYOC on enterprise` ![Stars](https://img.shields.io/github/stars/e2b-dev/code-interpreter?style=flat-square&label=%20)
- [Vercel Sandbox](https://github.com/vercel/sandbox) - Ephemeral compute with tight Vercel integration. `Vercel` `separate kernel` `no self-host` ![Stars](https://img.shields.io/github/stars/vercel/sandbox?style=flat-square&label=%20)
- [Cloudflare Sandbox SDK](https://github.com/cloudflare/sandbox-sdk) - Lowest latency, most constrained environment. `Cloudflare` `shared kernel or isolate` `edge only` ![Stars](https://img.shields.io/github/stars/cloudflare/sandbox-sdk?style=flat-square&label=%20)
- [Northflank](https://northflank.com/) - The one managed platform that also runs on your own Kubernetes. `Northflank` `separate kernel` `self-host + BYOC`.
- [Blaxel](https://github.com/blaxel-ai/sandbox) - Persistent state with ~25ms resume; standby costs nothing. `Blaxel` `separate kernel` `no self-host` ![Stars](https://img.shields.io/github/stars/blaxel-ai/sandbox?style=flat-square&label=%20)
- [Runloop](https://runloop.ai/) - Devboxes aimed specifically at coding agents. `Runloop` `separate kernel` `no self-host` ![Stars](https://img.shields.io/github/stars/runloopai/api-client-ts?style=flat-square&label=%20)
- [Azure Container Apps Dynamic Sessions](https://learn.microsoft.com/en-us/azure/container-apps/sessions) - Warm pool allocating in under 100ms at P90. `Microsoft` `separate kernel, Hyper-V` `no self-host`.
- [AWS Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/) - Managed Code Interpreter and Browser Tool, priced per session. `AWS` `separate kernel, Firecracker` `no self-host`.
- [GKE Agent Sandbox](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/agent-sandbox) - The rare managed product whose upstream is the open standard. `Google Cloud` `intercepted kernel` `upstream is open`.
- [Alibaba Cloud AgentBay](https://www.aliyun.com/product/agentbay) - Four form factors: browser, cloud desktop, code, cloud phone. `Alibaba Cloud` `intercepted or separate` `via ACS` ![Stars](https://img.shields.io/github/stars/agentbay-ai/wuying-agentbay-sdk?style=flat-square&label=%20)
- [Tencent Cloud AGSX](https://cloud.tencent.com/product/agsx) - Managed counterpart to CubeSandbox. `Tencent Cloud` `intercepted or separate` `via CubeSandbox`.
- [Browserbase](https://www.browserbase.com/) - Managed headless browser infrastructure. `Browserbase` `browser` `no self-host`.

## Related Projects

Adjacent tooling routinely mistaken for a sandbox, plus the frameworks that bundle one.

- [kagent](https://github.com/kagent-dev/kagent) - Kubernetes-native, but **not a sandbox** — it orchestrates agents and leaves isolation to something else. `Solo.io` `no isolation` `pairs with agent-sandbox` ![Stars](https://img.shields.io/github/stars/kagent-dev/kagent?style=flat-square&label=%20)
- [kmcp](https://github.com/kagent-dev/kmcp) - Kubernetes controller and CLI for deploying MCP servers. `Solo.io` `not a sandbox` ![Stars](https://img.shields.io/github/stars/kagent-dev/kmcp?style=flat-square&label=%20)
- [OpenHands](https://github.com/OpenHands/OpenHands) - Agent platform with a pluggable Docker or remote runtime. `All Hands AI` `bundles a sandbox` ![Stars](https://img.shields.io/github/stars/OpenHands/OpenHands?style=flat-square&label=%20)
- [DeerFlow](https://github.com/bytedance/deer-flow) - Agent harness with local, Docker, and Kubernetes sandbox modes. `ByteDance` `bundles a sandbox` ![Stars](https://img.shields.io/github/stars/bytedance/deer-flow?style=flat-square&label=%20)
- [Coder](https://github.com/coder/coder) - Kubernetes-native environments for developers and their agents. `Coder` `bundles a sandbox` ![Stars](https://img.shields.io/github/stars/coder/coder?style=flat-square&label=%20)
- [The Agent Sandbox Taxonomy](https://github.com/kajogo777/the-agent-sandbox-taxonomy) - Seven defense layers scored across 27 products; the most rigorous framework in the space. `Community` `evaluation` ![Stars](https://img.shields.io/github/stars/kajogo777/the-agent-sandbox-taxonomy?style=flat-square&label=%20)

## Articles

- [Comparing AI agent sandbox platforms](https://blog.logrocket.com/comparing-ai-agent-sandbox-platforms-e2b-modal-daytona-and-more/) - Measured cold-start and resume timings rather than vendor claims.
- [Bringing you Agent Sandbox on GKE and Agent Substrate](https://cloud.google.com/blog/products/containers-kubernetes/bringing-you-agent-sandbox-on-gke-and-agent-substrate) - Google Cloud on the origin and trajectory of the SIG project.
- [Red Hat build of Agent Sandbox](https://developers.redhat.com/articles/2026/07/15/red-hat-build-agent-sandbox-isolated-workload-management-kubernetes) - What a supported downstream distribution of the standard looks like.
- [Sandboxes on Kubernetes: isolation options](https://northflank.com/blog/sandboxes-on-kubernetes) - Walks runc, gVisor, Kata, and Firecracker with concrete tradeoffs.
- [How to sandbox AI agents in 2026](https://northflank.com/blog/how-to-sandbox-ai-agents) - Practical isolation strategies for agent workloads.
- [Kata vs Firecracker vs gVisor](https://edera.dev/stories/kata-vs-firecracker-vs-gvisor-isolation-compared) - Runtime comparison, written by a vendor with a horse in the race.
- [Bringing Agentic AI to Kubernetes](https://www.solo.io/blog/bringing-agentic-ai-to-kubernetes-contributing-kagent-to-cncf) - Solo.io on contributing kagent to CNCF.

## Contributing

Contributions are welcome. Read the [contribution guidelines](CONTRIBUTING.md) first — entries are one line, and need isolation and backer tags rather than just a link.
