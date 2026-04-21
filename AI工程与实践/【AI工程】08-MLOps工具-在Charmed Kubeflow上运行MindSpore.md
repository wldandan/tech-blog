# 【AI工程】08-MLOps工具-在Charmed Kubeflow上运行MindSpore

原文链接：https://zhuanlan.zhihu.com/p/568398333

---

​

目录

在[【AI工程】02-AI工程（AI Engineering）面面观](https://zhuanlan.zhihu.com/p/484046440)中，提到Gartner把AI工程化作为未来重要战略技术趋势，Gartner认为AI工程主要由DataOps、MLOps和DevOps三部分核心技术组成，其目标是通过跨职能协作、自动化、快速反馈等方法，来缩短数据分析、机器学习和应用部署上线的周期，从而让AI模型快速、持续地提供业务价值。开发者基于传统的工具平台很难实现MLOps等AI工程领域的实践，需要新的工具来完成对MLOps等技术实践的支持。

[Kubeflow](https://link.zhihu.com/?target=https%3A//www.kubeflow.org/)是一个基于K8S的机器学习平台，为开发者提供了从实验（Notebook）、训练（MLOps流水线）、调优以及部署、监控的端到端能力，也是当前排名第一的开源MlOps工具。

Kubeflow本身也是由一系列的开源工具组成，从它的架构图中不难看出，Kubeflow主要提供三部分能力：

  1. **ML工具** ：主流[开源框架](https://zhida.zhihu.com/search?content_id=214443154&content_type=Article&match_order=1&q=%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6&zhida_source=entity)支持，如Tensorflow，PyTorch等。
  2. **Kubeflow应用及脚手架工具** ：



(1). jupyter Notebook：开箱即用的Notebook，支持多AI框架。

(2). 分布式训练：支持Tensorflow、PyTorch等多框架的分布式训练（参数服务器形式）。

(3). 流水线管理：基于Argo的工作流管理，提供训练流水线管理能力。

(4). 镜像构建：将训练、notebook代码打包，以支持训练及部署任务。

(5). Serving部署：支持多AI框架的部署。

3.**周边配套** ：支持三方部署、监控等能力。

Canonical公司（Ubuntude发行商）在Kubeflow的基础上，包装了[Charmed Kubeflow](https://link.zhihu.com/?target=https%3A//charmed-kubeflow.io/docsyo)项目，提供构成KubeFlow最新版本的30多个应用程序和服务，并且让Kubeflow的部署更快，更简单。

在最新的1.6版本中，Charmed Kubeflow的Notebook原生支持了MindSpore，下面我们来看下如何基于Charmed Kubeflow 快速启动支持MindSpore的Notebook。

## 安装Charmed Kubeflow

要安装Kubeflow，首先得准备好K8S集群，然后通过`juju`这个运维管理工具安装Charmed Kubeflow。

### 通过MicroK8S工具部署K8S集群

Canonical提供了一个和Minikube类似的工具MicroK8S，通过Snap工具可以快速完成其安装。

在Ubuntu 20.04系统上执行如下命令：
    
    
    sudo snap install microk8s --classic --channel=1.22/stable

安装完成后，为了方便使用，可以将当前的用户加入到microk8s的用户组中。
    
    
    sudo usermod -a -G microk8s $USER
    newgrp microk8s

确认用户可以访问kubectl的[配置文件](https://zhida.zhihu.com/search?content_id=214443154&content_type=Article&match_order=1&q=%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6&zhida_source=entity)。
    
    
    sudo chown -f -R $USER ~/.kube

MicroK8s在安装的时候就会启动，为了运行kubeflow，我们还需要一些额外的能力，比如DNS（[服务发现](https://zhida.zhihu.com/search?content_id=214443154&content_type=Article&match_order=1&q=%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0&zhida_source=entity)）、存储、ingress（[负载均衡](https://zhida.zhihu.com/search?content_id=214443154&content_type=Article&match_order=1&q=%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1&zhida_source=entity)），MicroK8S可以以插件的形式快速的添加这些能力
    
    
    microk8s enable dns storage ingress metallb:10.64.140.43-10.64.140.49

整个安装需要花费一点时间，通过`microk8s status --wait-ready`可以确认MicroK8S是否安装成功。
    
    
    microk8s is running
    high-availability: no
      datastore master nodes: 127.0.0.1:19001
      datastore standby nodes: none
    ……

MicroK8S提供了`kubectl`命令，但是每次都需要在命令行输入`microk8s kubectl`，可以考虑增加一个别名`alias kubectl='microk8s kubectl'`方便使用，其次，如果集群的配置信息没有写到`~/.kube/`中，可以通过 `microk8s config > ~/.kube/config`完成覆写。

### 通过`juju`安装Charmed Kubeflow

Charmed Operator Lifecycle Manager (OLM)是一个应用（以特殊格式封装，称为Charm Operator）编排的平台，它可以方便的管理混合云中部署在虚机、K8S集群、裸机上的应用，对应用进行安装、配置、维护及更新。Canonical提供了这样的OLM框架，名为Juju，同时也提供了`juju`这个同名的命令行工具。

首先，我们使用`sudo snap install juju --classic`命令安装`juju`。其次，通过`juju bootstrap microk8s`在MicroK8S部署好的集群上安装juju的controller，作为juju在集群中的代理，管理Kubeflow应用。最后为juju在集群上添加kubeflow的[命名空间](https://zhida.zhihu.com/search?content_id=214443154&content_type=Article&match_order=1&q=%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4&zhida_source=entity)`juju add-model kubeflow`。

接下来，通过`juju deploy kubeflow-lite --trust`命令即可完成kubeflow的安装，通过`watch -c juju status --color`可以看到kubeflow组件准备状态。

最后，通过`juju refresh jupyter-ui --channel=latest/edge`命令确保`jupyter-ui`更新到最新的版本，包含MindSpore的Notebook镜像。

### 配置kubeflow

Kubeflow安装完成后需要做下简单的配置才能访问。首先配置访问的地址：
    
    
    juju config dex-auth public-url=http://10.64.140.43.nip.io
    juju config oidc-gatekeeper public-url=http://10.64.140.43.nip.io

然后配置访问的用户名和密码：
    
    
    juju config dex-auth static-username=admin
    juju config dex-auth static-password=ucantseeme

接下来在浏览器中输入`http://10.64.140.43.nip.io`，以及刚设置的用户密码，就可以看到Kubeflow完整的Dashboard了。

## 运行支持MindSpore的Notebook

在Kubeflow上运行Notebook非常简单。在Notebook tab选择创建notebook，输入notebook名称MindSpore，镜像选择jupyterlab，在列表中使用`mindspore/jupyter-mindspore`这个镜像，然后分配合适的CPU和内容资源，点击创建即可。

在Notebook界面很快就可以看到创建完成的提示。

点击`Connect`，我们就可以在另一个浏览器tab页打开notebook了。这里我们可以使用MindSpore官网现成的[notebook](https://link.zhihu.com/?target=https%3A//obs.dualstack.cn-north-4.myhuaweicloud.com/mindspore-website/notebook/r1.8/tutorials/zh_cn/beginner/mindspore_quick_start.py)，通过notebook页面将这个手写数字识别的notebook上传上去。

点击执行，就可以看到这个notebook直接运行起来了，不用额外的去安装MindSpore以及Vision套件。 

## 总结

这是MindSpore和Charmed Kubeflow集成的第一步，后续我们还将持续的把MindSpore更多能力集成到Charmed Kubeflow中，方便开发者能在MlOps平台上更方便的使用MindSpore。

## 参考资料

  1. [Charmed Kubeflow Quick Start](https://link.zhihu.com/?target=https%3A//charmed-kubeflow.io/docs/quickstart)



  


> 上一篇：[07-CD4ML-机器学习的持续交付（下）](https://zhuanlan.zhihu.com/p/554459107)
