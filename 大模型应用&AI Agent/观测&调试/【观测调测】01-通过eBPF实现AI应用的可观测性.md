# 【观测/调测】01-通过eBPF实现AI应用的可观测性

原文链接：https://zhuanlan.zhihu.com/p/1897702390850885133

---

​

目录

## 1 概述

随着大型语言模型（[LLM](https://zhida.zhihu.com/search?content_id=256733359&content_type=Article&match_order=1&q=LLM&zhida_source=entity)s）的兴起，基于大模型的AI应用，尤其是Agent，迎来了快速的发展，今年被认为是Agent元年。这些Agent通过调用多个LLM、决策以及与外部API的交互，来执行复杂任务。对于这些复杂的动态工作流，传统的观测工具如[APM](https://zhida.zhihu.com/search?content_id=256733359&content_type=Article&match_order=1&q=APM&zhida_source=entity)（应用程序性能监控）、日志记录和[分布式跟踪](https://zhida.zhihu.com/search?content_id=256733359&content_type=Article&match_order=1&q=%E5%88%86%E5%B8%83%E5%BC%8F%E8%B7%9F%E8%B8%AA&zhida_source=entity)等（针对可预测的请求-响应模式而设计），难以满足这些AI应用的可观测性。AI应用给程序运行的可观测性带来了新的挑战：

  * **复杂多步骤工作流** ：Agent执行任务时对模型、工具和数据库的链式调用，导致追踪异常困难。传统APM缺乏对提示词和Agent决策的原生支持。
  * **非确定性行为** ：LLM对相同输入会产生不同输出，这使得静态阈值设定和固定追踪路径难以奏效。可观测性系统必须能动态捕捉决策过程。
  * **海量非结构化数据** ：监控需要捕获完整对话和提示词内容，导致日志量激增，存储与分析成本高。
  * **性能开销问题** ：传统监控Agent可能使CPU利用率增加高达249%，可能导致ML模型资源不足或引发延迟问题。



为了应对这些挑战，[eBPF](https://zhida.zhihu.com/search?content_id=256733359&content_type=Article&match_order=1&q=eBPF&zhida_source=entity) (Extended Berkeley Packet Filter) 由此诞生，其是一种不依赖于代码埋点或预定义请求结构的新方法。

## 2 什么是eBPF

eBPF是**Linux内核中的一种机制，允许小型程序响应内核事件（如系统调用、网络数据包、函数调用等）运行，而无需修改内核源码或加载新的内核模块** 。可以将eBPF视为内核中的轻量级、安全隔离的沙箱虚拟机，它能够安全地执行自定义字节码。其具有如下优势：

  * **全系统可见性** ：eBPF能够自动追踪进程、网络调用和文件访问，自动提供AI活动的全貌。
  * **性能影响最小** ：与传统监控工具不同，eBPF在后台高效运行，CPU/内存消耗极低，适合实时AI应用。
  * **无需代码修改** ：eBPF在系统层面运行，即使是第三方或闭源系统，也无需修改AI模型。
  * **内置安全机制** ：除监控外，eBPF可检测并阻止未授权的操作，在不影响性能的前提下确保AI系统安全。



本质上，eBPF作为高性能、底层的可观测层，完美契合AI系统。它提供了传统监控难以实现的深度洞察和控制能力，满足现代LLM系统的需求。eBPF在AI应用的可观测性中的应用场景如下所示：

  * **LLM API监控** ：拦截外部LLM API（如OpenAI、[Anthropic](https://zhida.zhihu.com/search?content_id=256733359&content_type=Article&match_order=1&q=Anthropic&zhida_source=entity)等）的调用，自动记录提示词、统计Token用量、衡量延迟并监控错误，无需修改应用代码。
  * **资源监控** ：追踪每次模型调用的CPU/GPU消耗，并将性能峰值与特定触发因素（如某个提示词导致的大量内存消耗）相关联。
  * **数据流追踪** ：通过追踪套接字连接或文件访问，监测AI智能体与其他服务（如数据库、向量存储、其他微服务）之间的数据流动，发现瓶颈或配置错误。
  * **异常检测** ：eBPF可以捕获详细的时间和系统事件遥测数据，并将其输入异常检测系统（甚至是AI模型）。
  * **精细日志记录** ：捕获详细日志（如智能体对向量数据库的每次查询，或调用链中的每次工具调用），仅在内存中保留或进行采样存储，避免冗余日志开销。



## 3 使用eBPF构建AI网关，实现LLM的可观测性

通过eBPF，在内核级嵌入监控组件，构建一个AI网关，用于监控所有的LLM查询、响应和内部Agent活动，无需Agent配合。

eBPF驱动的AI网关运行在主机层（或[Kubernetes](https://zhida.zhihu.com/search?content_id=256733359&content_type=Article&match_order=1&q=Kubernetes&zhida_source=entity)节点），由捕获数据的eBPF程序和收集并处理信息的用户空间组件组成。

该架构具有很强的可扩展性，可适用于不同的编程语言。无论AI应用使用Python、Java、容器还是无服务器环境，只要运行于主机即可生效。在Kubernetes中，通过[DaemonSet](https://zhida.zhihu.com/search?content_id=256733359&content_type=Article&match_order=1&q=DaemonSet&zhida_source=entity)在各节点加载eBPF程序，即可构建集群级AI可观测网络。Protect AI已实现此类Agent，可在不改动应用的情况下监控应用与LLM的通信。

## 4 使用eBPF进行实时LLM跟踪

### **4.1 拦截LLM API调用**

多数LLM应用通过HTTP(S)请求访问模型（例如调用OpenAI的API或内部模型服务器）。使用eBPF可通过以下方式捕获这些请求：

  * **套接字层** ：将eBPF附加到connect()/send()系统调用。当应用连接[http://api.openai.com](https://link.zhihu.com/?target=http%3A//api.openai.com)并发送数据时，eBPF可以记录目标地址、负载大小等。同样，对于recv()的响应，可以记录状态码和响应大小。
  * **数据包层** ：在网络接口使用XDP程序或[TC eBPF挂钩](https://zhida.zhihu.com/search?content_id=256733359&content_type=Article&match_order=1&q=TC+eBPF%E6%8C%82%E9%92%A9&zhida_source=entity)。可获取原始数据包，结合流量追踪（如识别发往已知LLM API IP的流量）测量吞吐量和延迟。
  * **用户层函数** ：通过[uprobes](https://zhida.zhihu.com/search?content_id=256733359&content_type=Article&match_order=1&q=uprobes&zhida_source=entity)挂接到高级库函数（如调用OpenAI SDK的Python函数）。尽管需要了解内部符号结构，通用性较低但能获取更丰富信息（如传入该函数的实际提示文本）。



### **4.2 检测提示词注入攻击**

  * 扫描出站网络数据中的恶意模式或异常大的提示词。
  * 若存在加密，eBPF可挂接TLS库在加密前分析数据。
  * eBPF作为入侵检测系统（IDS），标记可疑负载和异常请求规模，实时阻止未授权注入。



### **4.3 记录AI对话和LLM响应**

  * eBPF在网络层捕获LLM请求/响应，完整重建对话。
  * 对于因加密无法直接记录的内容，eBPF仍可记录请求规模、频率等元数据。



### **4.4 追踪多智能体交互**

  * eBPF可通过追踪网络调用、文件访问或进程活动来监控多智能体交互。
  * 对于进程内函数调用（如LangChain），通过最小化的应用程序埋点补充eBPF追踪，从而全面了解AI智能体的交互方式。



## 5 使用eBPF加固Agent的安全性与合规性

**基于eBPF的LLM调用防火墙与速率限制**

使用LLM API的Agent可能面临高昂成本和风险（如无限循环导致费用激增或API密钥被封禁）。eBPF通过内核级速率限制，追踪LLM API请求（如发往[http://api.openai.com](https://link.zhihu.com/?target=http%3A//api.openai.com)的调用），自动拦截超额请求来解决这些问题。对于本地部署的LLM，eBPF还可以限制token用量，例如请求的输出大小超过定义的阈值，eBPF可以中断请求，从而防止系统过载。

**防止未经授权或恶意的Agent行为**

Agent可能会因漏洞或提示注入攻击而执行危险操作，eBPF可以通过以下方式提供保护：

  * **阻止未经授权的命令** ：如果AI系统尝试执行如/bin/sh的命令，eBPF可立即拦截并终止进程。
  * **保护敏感文件** ：禁止Agent访问密码、密钥或关键文件。
  * **网络访问管控** ：仅允许AI Agent和批授权的API通信，拦截未知或高风险的连接。
  * **防止工具的误用** ：当AI应用越权使用工具（如未授权的网络扫描行为），eBPF可以实时检测并阻止该行为。



## 6 总结

AI系统的可观测性正在快速发展，eBPF只是这场变革中的一部分。尽管eBPF在基础设施监控中发挥了重要作用，但未来可能会将eBPF与AI感知的可观测性框架深度融合，从而实现更全面的解决方案。

## 相关连接

  1. [https://blog.stackademic.com/why-ai-observability-needs-a-new-approach-ebpf-13a21f30f63c](https://link.zhihu.com/?target=https%3A//blog.stackademic.com/why-ai-observability-needs-a-new-approach-ebpf-13a21f30f63c)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)
