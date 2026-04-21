# 【GraphRAG】03-GraphRAG Indexing介绍及源码解读

原文链接：https://zhuanlan.zhihu.com/p/3115108247

---

​

目录

## 介绍

GraphRAG Indexing（索引）是一个数据Pipeline和转换套件，旨在**使用[LLM](https://zhida.zhihu.com/search?content_id=249645438&content_type=Article&match_order=1&q=LLM&zhida_source=entity)从非结构化文本中提取有意义的结构化数据**。Indexing Pipeline是可配置的，由工作流（Workflows）、标准和自定义步骤、提示模板（Prompt template）以及输入/输出适配器（Adapter）组成。

标准Pipeline设计用于：

  1. 从原始文本中提取实体（Entity）、关系（[Relationship](https://zhida.zhihu.com/search?content_id=249645438&content_type=Article&match_order=1&q=Relationship&zhida_source=entity)）和声明（Claim）
  2. 在实体中识别社区（Community）
  3. 在多个粒度级别生成社区报告（Community Report）和社区摘要（[Community Summary](https://zhida.zhihu.com/search?content_id=249645438&content_type=Article&match_order=1&q=Community+Summary&zhida_source=entity)）
  4. 将实体嵌入到图向量（[Graph vector](https://zhida.zhihu.com/search?content_id=249645438&content_type=Article&match_order=1&q=Graph+vector&zhida_source=entity)）空间中
  5. 将文本块（Chunk）嵌入到文本向量空间中



pipeline的输出可以以多种格式存储，包括JSON和Parquet，或者可以通过Python API手动处理。

## 工作流

### GraphRAG知识模型

**知识模型** 是符合此项目数据模型定义的数据输出的规范，可以在GraphRAG存储库中的python/graphrag/graphrag/model文件夹中找到这些定义。提供以下实体类型：

  * **Document** ：系统中的输入文档。这些文档要么代表 CSV 中的单独行，要么代表单独的 .txt 文件。
  * **[TextUnit](https://zhida.zhihu.com/search?content_id=249645438&content_type=Article&match_order=1&q=TextUnit&zhida_source=entity)** ：要分析的文本块chunk。一个常见的用例是设置CHUNK_BY_COLUMNS为id，以便文档和 TextUnits 之间存在一对多关系，而不是多对多关系。
  * **Entity** ：从 TextUnit 中提取的实体。这些实体代表人物、地点、事件或您提供的其他实体模型。
  * **Relationship** ：两个实体之间的关系。这些关系由协变量（Covariate）生成。
  * **Covariate** ：提取的声明信息，其中包含有关可能受时间限制的实体的陈述。
  * **Community Report** ：一旦生成实体，此项目就对它们执行分层社区检测，并为该层次结构中的每个社区生成报告。
  * **Node** ：该表包含已嵌入和汇聚的实体和文档。



### 默认配置工作流

如下图所示，后面会详细讲解每个流程阶段

### 第一阶段：编写TextUnit文本单元

将输入文档转换为TextUnit，**TextUnit是用于图形提取技术的文本块** 。它们还被提取的知识项用作源参考，以便找回源文本。

块大小（以tokens计）是用户可配置的。默认情况下，该大小设置为 300 个tokens。较大的块会导致较低的保真度输出和较少有意义的参考文本；但是，使用较大的块可以产生更快的处理时间。

默认情况下，将块与文档边界对齐，这意味着在Documents和TextUnits之间存在严格的一对多关系。

TextUnits中的每一个都是文本嵌入的，并被传递到Pipeline的下一阶段。

### 第二阶段：图提取

这个阶段分析每个文本单元并提取图形基元：**实体、关系和声明** 。实体和关系在entity_extract动词中一次性提取，声明在claim_extract动词中提取。然后将结果组合（Graph表）并传递到Pipeline的后续阶段。

**实体和关系摘要** ：现在有了实体和关系图，每个图都有一串描述，可以将这些列表汇总为每个实体和关系的单个描述。这可以通过要求 LLM 提供简短的摘要来完成，该摘要捕获每个描述中的所有不同信息。这使得所有的实体和关系都有一个简洁的描述。

### 第三阶段：图扩充/图增强

现在已经有了一个可用的实体和关系图，下面需要了解它们的社区结构，并用额外的信息来扩充图。这分为两步：社区检测和图嵌入。提供了显式（社区）和隐式（嵌入）的方式来理解图的拓扑结构。

**社区检测** ：在这一步中将使用分层Leiden算法来生成实体社区的层次结构。该方法将对图应用递归的社区聚类，直到达到社区大小阈值。这将能够理解图的社区结构，并提供一种在不同粒度级别导航和汇总图的方法。

**图向量** ：在这一步中使用[Node2Vec](https://zhida.zhihu.com/search?content_id=249645438&content_type=Article&match_order=1&q=Node2Vec&zhida_source=entity)算法生成图的向量表示。这将能够理解图的隐式结构，并提供一个额外的向量空间，在查询阶段搜索相关概念。

**扩充图-表发出** ：一旦图扩充步骤完成，最终的实体和关系表将在其文本字段嵌入文本后发出。

### 第四阶段：社区总结

此时，已有一个实体和关系的功能图、实体的社区层次结构以及 node2vec 嵌入。现在需要在社区数据的基础上构建，并为每个社区生成报告。这使得在图粒度的几个点上对图有了一个高层次的理解。

例如，如果社区A是顶级社区将得到关于整个图的报告。如果社区级别较低，也将得到一个关于本地集群的报告。

**生成社区报告** ：在这一步中使用LLM生成每个社区的摘要。这将能够理解每个社区中包含的不同信息，并从高层或低层角度提供对图的范围理解。

**汇总社区报告** ：在此步骤中，然后通过LLM汇总每个社区报告，以便速记使用。

**社区向量** ：在这一步中通过生成社区报告、社区报告摘要和社区报告标题的文本嵌入来生成社区的向量表示。

**社区-表发出** ：此时，将执行一些记录工作，并发出社区-表。

### 第五阶段：文件处理

在工作流的此阶段为知识模型创建文档表。

**链接到TextUnits** ：在这一步中将每个文档链接到第一阶段创建的文本单元。这将能够理解哪些文档与哪些文本单元相关，反之亦然。

**文档向量** ：在这一步中使用文档切片的平均embedding生成文档的向量表示。重新分块文档，不重叠分块，然后为每个分块生成一个embedding。创建按token-count加权的这些块的平均值，并将其用作文档嵌入。这将能够理解文档之间的隐含关系，并将帮助生成文档的网络表示。

**文档-表发出** ：可以将文档表发布到知识模型中。

### 第六阶段：网络可视化

在工作流程的这个阶段，将执行一些步骤来支持现有图中高维向量空间的网络可视化。在这一点上，有两个逻辑图在起作用：**实体关系图和文档图** 。

对于每个逻辑图，执行[UMAP](https://zhida.zhihu.com/search?content_id=249645438&content_type=Article&match_order=1&q=UMAP&zhida_source=entity)降维以生成图的2D表示。这将能够在2D空间中可视化图，并理解图中节点之间的关系。然后，UMAP嵌入作为Nodes表发出。

这个表的行包含一个鉴别器，指示该节点是文档还是实体，以及UMAP坐标。

## Prompt Tuning

默认Prompts是开始使用GraphRAG系统的最简单方法。它被设计为以最少的配置开箱即用。

初始化之后会生成Prompts文件夹，LLM通过Prompt提取实体、社区报告、总结描述等等

**如果效果不满意，可以进行Prompt Tuning进行调整，优先是Auto Prompt Tuning** 。

### Auto Prompt Tuning

GraphRAG提供了创建领域自适应模板的能力，用于生成知识图谱。此步骤是可选的，但强烈建议运行它，因为在执行索引运行时它将产生更好的结果。

模板的生成是通过加载输入，将它们分成块（文本单元），然后运行一系列LLM调用和模板替换来生成最终Prompt。前提条件：已初始化。每个默认值的详细信息见官网（[https://microsoft.github.io/graphrag/posts/prompt_tuning/auto_prompt_tuning/](https://link.zhihu.com/?target=https%3A//microsoft.github.io/graphrag/posts/prompt_tuning/auto_prompt_tuning/)）

样例如下：

python -m graphrag.prompt_tune --root /path/to/project --config /path/to/settings.yaml --domain "environmental news" \\--method random --limit 10 --language English --max-tokens 2048 --chunk-size 256 --min-examples-required 3 \\--no-entity-types --output /path/to/output  
---  
  
推荐使用最少配置：

python -m graphrag.prompt_tune --root /path/to/project --config /path/to/settings.yaml --no-entity-types  
---  
  
**根据开发者的经验，Auto Prompt Tuning之后是有效果的，但效果不一定是好的，可以手动配置Prompt Tuning。**

### Manual Configuration

默认情况下，GraphRAG 索引器会使用一些旨在在广泛的知识发现环境中运行良好的Prompt。然而，通常希望调整Prompt以更好地适应你的特定用例。

此项目提供了一种方法，允许你指定自定义Prompt文件，该文件将在内部使用一系列的Tokens替换。可以通过编写纯文本的自定义提示文件来覆盖这些提示。

在实际样例中，开发者推荐从文档中选取一些**内容丰富** 的段落，使用较好的大模型（如GPT4等）抽取出对应的**实体关系** ，对entity_extraction.txt文件中的任务说明部分、Example中的entity_types，以及Real Data部分的entity_types进行修改，同样也让大模型生成新的Example输出，并复制到entity_extraction.txt文件的相应位置，这样得到手动微调版的entity_extraction.txt，之后再进行Prompt Tuning。

## Indexing源码解析

索引阶段应该算是整个项目的核心，执行indexing如下代码：

python -m graphrag.index --init --root ./ragtestpython -m graphrag.index --root ./ragtest  
---  
  
实际调用的是graphrag/index/__main__.py文件中的主函数，使用argparse解析输入参数，实际调用的是graphrag/index/cli.py中的index_cli函数。

先简单看下相关函数的调用链路，如上图所示。

### index_cli()函数

首先根据用户输入参数，如--init，确定是否需要初始化当前文件夹，这部分具体由index_cli()执行，其具体实现逻辑比较简单，检查目录下的一些配置、prompt、.env等文件是否存在，没有则创建，具体文件内容，就是上一节目录中展示的 settings.yaml、prompts等。

真正执行indexing操作时，实际上会执行一个内部函数index_cli()._run_workflow_async()，主要会涉及到create_default_config()与run_pipeline_with_config()两个函数。

### 默认配置生成逻辑create_default_config()

首先检查根目录以及配置文件settings.yaml，然后执行read_config_parameters()读取系统配置，如llm、chunks、cache、storage等，接下来的操作比较关键，后续会根据当前参数创建一个pipeline的配置，其核心逻辑如下：

result = PipelineConfig( root_dir=settings.root_dir, input=_get_pipeline_input_config(settings), reporting=_get_reporting_config(settings), storage=_get_storage_config(settings), cache=_get_cache_config(settings), workflows=[ *_document_workflows(settings，embedded_fields), *_text_unit_workflows(settings，covariates_enabled，embedded_fields), *_graph_workflows(settings，embedded_fields), *_community_workflows(settings，covariates_enabled，embedded_fields), *(_covariate_workflows(settings) if covariates_enabled else []), ],)  
---  
  
这段代码的基本逻辑就是根据不同的功能生成完整的workflow序列，此处需要注意的是这里并不考虑workflow之间的依赖关系，单纯基于workflows/v1目录下的各workflow的模板生成一系列的workflow。

### Pipeline执行逻辑run_pipeline_with_config()

首先根据load_pipeline_config()加载现有pipeline的配置（由上一步中create_default_config()生成），然后创建目标文件中的其他子目录，如cache，storage，input和output等(详见上一节目录结构树)，然后再利用run_pipeline()依次执行每一个工作流，并返回相应结果。

### run_pipeline()函数

该函数用于真正的执行所有pipeline，其核心逻辑包含以下两部分：

  1. 加载workflows：load_workflows()，除了常规工作流的创建外，还会涉及到拓扑排序问题。


  * create_workflow()：利用已有模板，创建不同的工作流。
  * topological_sort()：根据workflow之间的依赖关系，计算DAG的拓扑排序。


  1. 进一步执行inject_workflow_data_dependencies()、write_workflow_stats()、emit_workflow_output等操作，分别用于依赖注入，数据写入以及保存，真正的workflow.run操作会在write_workflow_stats()之前执行，此处的核心逻辑可参考以下代码。

await dump_stats()for workflow_to_run in workflows_to_run: # Try to flush out any intermediate dataframes gc.collect() workflow = workflow_to_run.workflow workflow_name: str = workflow.name last_workflow = workflow_name [http://log.info](https://link.zhihu.com/?target=http%3A//log.info)("Running workflow: %s..."，workflow_name) if is_resume_run and await storage.has( f"{workflow_to_run.workflow.name}.parquet" ): [http://log.info](https://link.zhihu.com/?target=http%3A//log.info)("Skipping %s because it already exists"，workflow_name) continue stats.workflows[workflow_name] = {"overall": 0.0} await inject_workflow_data_dependencies(workflow) workflow_start_time = time.time() result = await workflow.run(context，callbacks) await write_workflow_stats(workflow，result，workflow_start_time) # Save the output from the workflow output = await emit_workflow_output(workflow) yield PipelineRunResult(workflow_name，output，None) output = None workflow.dispose() workflow = Nonestats.total_runtime = time.time() - start_timeawait dump_stats()  
---  
  
根据以上信息，可以大致梳理出Indexing环节的完整工作流。

  1. 初始化：生成必要的配置文件、input/output目录等。
  2. Indexing：根据配置文件，利用workflow模板创建一系列的pipeline，并依据依赖关系，调整实际执行顺序，再依次执行。



## 相关链接

  1. 官网：[https://microsoft.github.io/graphrag/posts/index/overview/](https://link.zhihu.com/?target=https%3A//microsoft.github.io/graphrag/posts/index/overview/)
  2. 博客：[https://www.cnblogs.com/fanzhidongyzby/p/18294348/ms-graphrag](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/fanzhidongyzby/p/18294348/ms-graphrag)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【GraphRAG技术洞察】02-微软&蚂蚁GraphRAG方案解析](https://zhuanlan.zhihu.com/p/1854007770)  
> 下一篇：[【GraphRAG技术洞察】04-GraphRAG Query介绍及源码解读](https://zhuanlan.zhihu.com/p/3990498959)
