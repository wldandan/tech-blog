# 【运维智能体】02-Ask Red Hat，提供技术问题解答和故障排除指导的AI助手

原文链接：https://zhuanlan.zhihu.com/p/1976619770745991853

---

​

目录

## 1 概述

[Ask Red Hat](https://zhida.zhihu.com/search?content_id=266819168&content_type=Article&match_order=1&q=Ask+Red+Hat&zhida_source=entity)是Red Hat于2025年5月推出的生成式AI助手，集成于其客户门户之中。该产品通过RAG（检索增强生成）技术，帮助Red Hat订阅客户快速从官方知识库和产品文档中获取技术问题的答案和故障排除指导。目前，Ask Red Hat仅面向红帽订阅客户，在其专属客户门户网站中提供使用。

## 2 技术方案

Ask Red Hat作为红帽用户的智能门户，致力于打破信息孤岛，整合来自红帽各处的洞察与资源，构建统一高效的支持体验。

图1 Ask Red Hat示意图

  * **[Llama Stack](https://zhida.zhihu.com/search?content_id=266819168&content_type=Article&match_order=1&q=Llama+Stack&zhida_source=entity)** ：提供构建和编排生成式AI应用、管理对话流程的核心框架与工具。
  * **[Granite Guardian 3.2 5B](https://zhida.zhihu.com/search?content_id=266819168&content_type=Article&match_order=1&q=Granite+Guardian+3.2+5B&zhida_source=entity)** ：负责对用户输入进行初步安全检查，检测并过滤有害或不适当内容。
  * **[Granite LLM 3.2 8B Instruct](https://zhida.zhihu.com/search?content_id=266819168&content_type=Article&match_order=1&q=Granite+LLM+3.2+8B+Instruct&zhida_source=entity)** ：用于判断用户意图，决定是否调用外部工具（如搜索功能），并基于检索结果生成最终回复。
  * **[Patternfly](https://zhida.zhihu.com/search?content_id=266819168&content_type=Article&match_order=1&q=Patternfly&zhida_source=entity)** ：基于RedHat开源设计系统构建的前端用户界面，提供流畅的对话交互体验。



## 3 关键能力解析

Ask Red Hat能够根据用户查询，从跨站搜索索引中查找并归纳最佳答案。该索引包含[http://access.redhat.com](https://link.zhihu.com/?target=http%3A//access.redhat.com)、[http://docs.redhat.com](https://link.zhihu.com/?target=http%3A//docs.redhat.com)和[http://console.redhat.com](https://link.zhihu.com/?target=http%3A//console.redhat.com)上发布的内容，包括产品文档、知识库文章、解决方案、CVE 及安全公告等。

语言支持：Ask Red Hat支持包括阿拉伯语、中文、捷克语、荷兰语、英语、法语、德语、意大利语、日语、韩语、葡萄牙语和西班牙语等12种语言。

该服务擅长多种实际应用场景：即时检索错误信息的相关知识、清晰解释新发布的安全漏洞、查找配置和设置信息、发起与Red Hat支持团队的连接、故障排查等。

图2 Ask Red Hat演示图

## 4 总结

Ask Red Hat是红帽公司在其客户门户中推出的一个AI助手，主要功能是让用户通过自然对话，快速从官方知识库、产品文档等资源中获取信息，打破信息孤岛，协助用户完成产品的设置、故障排查和学习使用。其极大地解放了运维人员，让他们能从大量的用户常见问题中抽身。本质上，AI问答助手通过将运维知识、经验和操作“对话化”与“自动化”，实现了从“被动救火”到“主动预防”和“智能处置”的效能跃升，是构建下一代高效、智能运维体系的核心支柱。

## 相关链接

  1. [https://www.redhat.com/en/blog/red-hat-customer-portal-introduces-ai-powered-assistant-ask-red-hat-built-open-innovation](https://link.zhihu.com/?target=https%3A//www.redhat.com/en/blog/red-hat-customer-portal-introduces-ai-powered-assistant-ask-red-hat-built-open-innovation)



> 技术全景图：[智能运维&AgenticOps技术栈-全景图](https://zhuanlan.zhihu.com/p/1995167173094695116)  
> 上一篇：[【运维智能体】01-微软AIOpsLab，集成运维Agents设计、开发和评估的整体框架](https://zhuanlan.zhihu.com/p/1976373761038124600)  
> 下一篇：[【运维智能体】03-Gemini Cloud Assist，谷歌云的云原生应用全生命周期助手](https://zhuanlan.zhihu.com/p/1977059222085730460)
