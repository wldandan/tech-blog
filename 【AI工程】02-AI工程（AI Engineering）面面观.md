# 【AI工程】02-AI工程（AI Engineering）面面观

原文链接：https://zhuanlan.zhihu.com/p/484046440

---

在[《01.[AI工程](https://zhida.zhihu.com/search?content_id=195697443&content_type=Article&match_order=1&q=AI%E5%B7%A5%E7%A8%8B&zhida_source=entity)（AI Engineering）初探》](https://zhuanlan.zhihu.com/p/464692752)中，我们介绍了为何[Gartner](https://zhida.zhihu.com/search?content_id=195697443&content_type=Article&match_order=1&q=Gartner&zhida_source=entity)要把AI工程化作为未来重要战略技术趋势之一，这是因为AI软件与传统软件之间**存在根本性差异** ，简单照搬传统[软件工程](https://zhida.zhihu.com/search?content_id=195697443&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E5%B7%A5%E7%A8%8B&zhida_source=entity)方法来解决AI领域的问题是行不通的，因此需要发展新的工程技术来支撑AI大规模、平民化的落地。

本篇中，我们将进一步对AI工程进行解读，主要包括：

  * **AI工程的由来**
  * **业界发展现状**
  * **技术全景图**



* * *

### **一、AI工程的由来**

“软件工程”这一名词诞生于1968年，它的出现是为了克服当时出现的“软件危机”，目标是**通过系统化、规范化、可度量的工程方法，来确保软件项目的进度、成本和质量能够达到预期** 。  
  
AI工程的出现，也有着类似的背景，同样也是为了应对AI软件可能的“危机”而诞生的。

在2019年，由国际系统工程理事会（INCOSE）主办的“系统工程的未来”（FuSE）研讨会上，首次提出了“**SE for AI** ”（又称**[SE4AI](https://zhida.zhihu.com/search?content_id=195697443&content_type=Article&match_order=1&q=SE4AI&zhida_source=entity)** ）这一术语，来描述**应对人工智能实际应用的挑战所需要的工程方法与技术** ，这些挑战主要被概括为三方面：**AI系统本身的不可预测性、AI系统中新的故障模式、以及AI系统在可信和健壮性方面的不足**[1]。

后来，Gartner公司等其他机构也经常用**“AI工程”（AI Engineering）** 这个词来描述**面向AI系统的过程、方法与工具** ，虽然表述不同，但要解决的都是AI领域工程化的问题。这就是AI工程这一概念的由来。

* * *

### **二、业界发展现状**

近年来，学术界研究机构 及 工业界各大公司都已争先在AI工程领域布局，学术界以创新研究、前沿技术探索、人才培养为主；工业界则以实用的技术和工具的发展为主，如下所示：

  * **学术界以创新研究、前沿技术探索、人才培养为主**  
  
据调查统计，2010-2020十年间，与“人工智能”和”软件工程”关键词均匹配的论文有4733篇，其中与SE4AI强相关的有248篇，涉及软件工程知识体系中的全部11个领域。按照研究领域对部分顶尖研究机构所发表论文的统计结果来看，**软件测试、软件质量、软件工程模型和方法是过去十年间AI工程领域Top3的研究热点，如下图所示：**



图2. 2010-2020年AI工程相关论文分布(Software Engineering for AI-Based Systems: A Survey, ACM TOSEM, 2021)

  * **工业界各大公司以AI工程方法、技术和工具的发展为主，主要分两类：**  
  
1）以Google、Meta为代表的科技巨头，围绕TensorFlow、[PyTorch](https://zhida.zhihu.com/search?content_id=195697443&content_type=Article&match_order=1&q=PyTorch&zhida_source=entity)等开源深度学习框架**打造了一系列与AI工程相关的生态技术和工具，如高级API、领域套件、模型可视化工具、调试调优工具等** ，近年来国内的华为、百度等，也以MindSpore、[PaddlePaddle](https://zhida.zhihu.com/search?content_id=195697443&content_type=Article&match_order=1&q=PaddlePaddle&zhida_source=entity)等AI框架为基础，丰富技术和工具生态，发展迅猛。  
  
2）以[ThoughtWorks](https://zhida.zhihu.com/search?content_id=195697443&content_type=Article&match_order=1&q=ThoughtWorks&zhida_source=entity)为代表的业界知名咨询公司，**以发展机器学习相关的流程、方法论和最佳实践为主** ，例如，ThoughtWorks的咨询师Arif Wider等人在2019年提出”机器学习的持续交付“（Continuous Delivery for Machine Learning，CD4ML）理念，认为需要为机器学习系统研究其专有的持续交付原则、方法和实践，以应对机器学习系统更复杂、更难以测试、解释和改进的挑战。



* * *

### **三、多视角看AI工程技术**

在上面的内容中，我们分析了AI工程在业界的发展现状。接下来，我们从不同视角解读AI工程。

  1. **从软件工程视角看AI工程**



回到软件工程的本质，我们会发现**AI工程的本质与软件工程是一致的，即在AI软件开发的整个生命周期过程中系统化、规范化、可度量地使用各种工程方法和工具，来确保所交付的软件能达到预期** 。

而AI工程相对于传统软件工程方法变化的部分，只是在应用对象和要达成的目标上发生了外延，**应用对象变成了基于AI的软件** ，而达成的目标从进度、成本、质量可控进一步外延为**希望克服AI软件固有的问题（不可预测性、特有的故障模式等）** ，从而交付可信、健壮、行为可预期的AI系统。

图3 AI工程与软件工程、人工智能的关系

**2\. 从产业实践视角看AI工程（** Gartner**）**  
  
Gartner公司在关于2022年战略技术趋势的一份报告中提出[2]，如果企业让人工智能提供变革性的价值，就不能只是单点地应用AI技术，而是需要在其商业生态系统中将AI模型工业化，以便快速、持续地提供新的业务价值，而要做到这一点的关键就是应用AI工程技术。  
  
围绕着**数据处理、机器学习模型训练与推理、应用交付与维护这几个关键流程** ，Gartner认为，AI工程主要由**DataOps、[MLOps](https://zhida.zhihu.com/search?content_id=195697443&content_type=Article&match_order=1&q=MLOps&zhida_source=entity)和DevOps**三部分核心技术组成，其目标是通过跨职能协作、自动化、快速反馈等方法，来缩短数据分析、机器学习和应用部署上线的周期，从而让AI模型快速、持续地提供业务价值。

  


图4 Gartner AI Engineering的组成

> **DevOps：是一组软件开发和运维团队之间的文化理念、实践和工具的结合** ，以便提高团队的快速交付能力，在微服务与云原生应用的交付团队中备受推崇；  
> **DataOps：** 是将数据处理和集成过程与自动化和敏捷软件工程方法相结合的技术实践，以提高数据分析的质量、速度和团队协作，并促进持续改进[3]；  
> **MLOps：是** 在生产环境中可靠而高效地部署和维护机器学习模型的实践[4]。

  


**3\. 从生态视角看AI工程([LF AI &amp; Data Landscape](https://zhida.zhihu.com/search?content_id=195697443&content_type=Article&match_order=1&q=LF+AI+%26amp%3B+Data+Landscape&zhida_source=entity))**

> AI软件开发的生态系统，特指围绕AI软件开发的一系列基础设施和工具链，包括制品仓、数据管理工具、模型管理器、持续训练流水线、模型验证与评估工具、模型部署与监控系统等。

Linux基金会旗下的子基金会LF AI & Data Foundation，致力于AI与大数据领域的开源软件发展，目前在维护一个名为[LF AI & Data Landscape](https://link.zhihu.com/?target=https%3A//landscape.lfai.foundation/)的开源软件产品全景图，可以作为**AI软件开发生态系统的一个很好的参考** 。  
该全景图当前已包括300多款开源软件，涉及AI框架、MLOps流水线、特征工程、可视化等多个子领域。

https://landscape.lfai.foundation/ (二维码自动识别)

AI软件开发生态系统中的组件，在功能、用途甚至名称上都与传统软件开发的生态系统存在差异，包括但不限于：

  * **从IDE到数据管理工具** ：除了编写算法之外，AI软件开发中有大量的工作在收集、处理和清洗数据集上，因此一个具有数据可视化能力、方便进行数据标记、甚至能够自动/半自动给出数据标记建议的数据管理工具将成为开发者频繁使用的工具；
  * **从代码仓到制品仓：** AI软件开发过程中除了需要存储少量的代码外，还要对大量的数据和标签进行存储，因此制品仓应能够具备对大量数据的管理能力，并能够追溯对数据和标签的变更；
  * **从包管理器到模型管理器：** 模型是AI软件开发活动的重要产出物，为了更容易地对模型进行共享、导入和组合，需要新的模型管理器；
  * **从持续集成到持续训练：** AI软件开发过程中需要经常地进行调参和精细化调优，数据发生变化、模型发生漂移都会触发这个过程，因此需要自动化的数据流水线和机器学习流水线，帮助AI工程师及时监控到数据和模型的变化，进行持续训练。



* * *

**四、AI工程技术全景图**  
  
综合业界、学术界以及研究机构的主流观点，再结合目前在MindSpore中研究的内容，笔者认为，AI工程从广义上应包括**AI软件开发生命周期** 、**工程实践、生态系统三** 部分：

> 1\. **AI软件开发生命周期** ：即从问题分析与抽象开始，进行设计开发，直到最后部署上线的整个流程，各类软件工程技术也是按照软件开发生命周期进行分类和组织的。   
> 2\. **AI工程实践：** AI软件开发过程中所使用的各种工程化方法和最佳实践，可分为两类，一类是与AI软件开发生命周期中某个步骤对应的专项技术，如数据分析对应于数据准备环节、设计模式对应于模型训练环节；另一类则是与整个软件开发生命周期相关的技术，如机器学习团队中的角色组成、工程师的能力模型，以及对AI系统的治理技术。  
>  **3\. 生态系统：** AI软件开发所需使用的基础设施和软件工具的集合，包括数据分析、编码检查、调试调优、部署监控等各种专项工具。

图6 AI工程技术全景图

  
在本专栏后续的文章中，会围绕MindSpore框架，全方位介绍基于AI工程的技术实践与生态组件。

* * *

**小结**  
  
软件工程是一门为了克服危机而诞生的工程学科，AI工程的出现也有着类似的背景，同样也是为了应对AI软件可能的“危机”而诞生的。作为一门新兴学科，目前学术界和工业界都已在这个领域积极布局。  
  
AI工程，就是在AI软件开发生命周期中应用各种工程方法和工具，来确保所交付的AI系统可信、健壮、行为可预期。AI工程技术包括了对AI软件开发生命周期的定义、AI工程方法实践，以及由各种基础设施和工具链所组成的生态系统。  
  
AI框架作为AI领域的基础软件，在AI软件栈中处于承上启下的位置，并能够在AI软件开发生命周期的主要步骤中提供多项关键工程能力。学好AI框架的使用，能够帮助打下牢固的AI工程技术基础。

  


> 上一篇：[01 - 开篇词：AI工程（AI Engineering）初探](https://zhuanlan.zhihu.com/p/464692752)  
> 下一篇：[03-迭代0：机器学习项目开始前要做哪些准备工作？](https://zhuanlan.zhihu.com/p/500536548)

## 参考

  1. ^<https://www.sebokwiki.org/wiki/Artificial_Intelligence>
  2. ^Top Strategic Technology Trends for 2022: AI Engineering - Gartner <https://www.gartner.com/en/documents/4006919/top-strategic-technology-trends-for-2022-ai-engineering>
  3. ^DataOps - Wikipedia <https://en.wikipedia.org/wiki/DataOps>
  4. ^MLOps - Wikipedia <https://en.wikipedia.org/wiki/MLOps>


