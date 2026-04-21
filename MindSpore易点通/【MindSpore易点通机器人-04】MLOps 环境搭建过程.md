# 【MindSpore易点通机器人-04】MLOps 环境搭建过程

原文链接：https://zhuanlan.zhihu.com/p/514996638

---

​

目录

在上一篇[【MindSpore易点通机器人-03】迭代0的准备工作](https://zhuanlan.zhihu.com/p/509494866)，我们从整体上概述了MindSpore易点通机器人项目开始前需要在迭代0的准备工作，本篇将会为大家讲述迭代0中具体的MLOps 环境搭建过程。整体的MLOps流水线设计如下图，包含持续训练流水线和CI/CD流水线。相关代码请参考[MindSpore易点通机器人代码仓](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig/robot)。

实际的技术选型上，我们基于Jenkins构建CI/CD流水线，基于Argo构建持续训练流水线。同时，我们把Jenkins和Argo都运行在K8S上来保证可用性和弹性。下文将为大家介绍如何在本地使用Minikube完成机器人项目的MLOps流水线的搭建，并在本地运行起来。

具体的过程如下：

  1. 安装配置WSL+Ubuntu+Docker；
  2. 基于Minikube运行K8S；
  3. 基于K8S+Jenkins构建CI/CD流水线；
  4. 基于K8S+Argo持续训练流水线。



## 1\. 安装配置WSL+Ubuntu+Docker

因为团队大部分人的[开发环境](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83&zhida_source=entity)都是Windows，所以需要选择WSL+Linux+Docker的方式。这里我们没有采用Docker Desktop+WSL Backend，而是利用Distord让Docker在Ubuntu上直接运行，相关的安装配置文档可以参考[如何不安装Docker Desktop在WSL下运行Docker](https://zhuanlan.zhihu.com/p/500450853)这篇文章。

## 2\. 基于Minikube运行K8S

[Minikube](https://link.zhihu.com/?target=https%3A//minikube.sigs.k8s.io/docs/)是一个单机安装配置[K8S集群](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=K8S%E9%9B%86%E7%BE%A4&zhida_source=entity)的工具，它支持多平台（Mac/Linux/Windows）。Minikube可以将K8S集群安装配置在单个Docker容器或者VM（hyper-v/VMWare等）中，过程也非常简单。首先在Ubuntu中安装Minikube：
    
    
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube

然后执行`minikube start` ，因为在第一步中我们已经配置好了Docker，Minikube在启动时会默认使用Docker作为VM，然后在容器中启动K8S集群。启动完成后在Ubuntu上使用`docker ps`只能看到一个容器：
    
    
    CONTAINER ID   IMAGE                    COMMAND                  CREATED       STATUS       PORTS                                                                                                                                  NAMES
    699a71fee349   kicbase/stable:v0.0.30   "/usr/local/bin/entr…"   2 weeks ago   Up 2 hours   127.0.0.1:49157->22/tcp, 127.0.0.1:49156->2376/tcp, 127.0.0.1:49155->5000/tcp, 127.0.0.1:49154->8443/tcp, 127.0.0.1:49153->32443/tcp   minikube

如果我们执行`docker exec -it 699a71fee349`进入容器后再执行`[docker ps](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=2&q=docker+ps&zhida_source=entity)`，就能发现K8S集群的服务了：
    
    
    CONTAINER ID   IMAGE                                  COMMAND                    CREATED       STATUS       PORTS     NAMES
    af45f3caa0d9   99a3486be4f2                           "[kube-scheduler](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=kube-scheduler&zhida_source=entity) --au…"    2 hours ago   Up 2 hours             k8s_kube-scheduler_kube-scheduler-minikube_kube-system_be132fe5c6572cb34d93f5e05ce2a540_1
    e648e7d30a7d   Error 404 (Not Found)!!1               "/pause"                   2 hours ago   Up 2 hours             k8s_POD_kube-apiserver-minikube_kube-system_cd6e47233d36a9715b0ab9632f871843_1
    e26d9e92c4e3   k8s.gcr.io/pause:3.6                   "/pause"                   2 hours ago   Up 2 hours             k8s_POD_kube-scheduler-minikube_kube-system_be132fe5c6572cb34d93f5e05ce2a540_1
    e658bf17922d   Error 404 (Not Found)!!1               "/pause"                   2 hours ago   Up 2 hours             k8s_POD_kube-controller-manager-minikube_kube-system_b965983ec05322d0973594a01d5e8245_1
    1f85a9bae877   Error 404 (Not Found)!!1               "/pause"                   2 hours ago   Up 2 hours             k8s_POD_etcd-minikube_kube-system_9d3d310935e5fabe942511eec3e2cd0c_1
    ....

从容器中退出，安装kubectl之后就在Ubuntu上使用Kubectl管理集群了：
    
    
    curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

执行`kubectl get nodes`，查询K8S管理的节点。 
    
    
    ~$ kubectl get nodes
    NAME       STATUS   ROLES                  AGE   VERSION
    minikube   Ready    control-plane,master   19d   v1.23.3

如果关注Kubernetes的界面，使用`minikube dashboard`就可以启动K8S的管理界面，并在Windows上通过浏览器访问 `http://127.0.0.1:44185/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/`。

使用Minikube可以让我们在本地拥有一个和生产环境一样功能的K8S集群，但这种方式同样带来了网络的复杂性，如下图，Ubuntu运行在Hyper-v的虚机中，在K8S部署的服务运行在Ubuntu的[Docker容器](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=2&q=Docker%E5%AE%B9%E5%99%A8&zhida_source=entity)的容器中（Docker in Docker）。所以，如果要在Windows的浏览器上访问K8S中运行的服务，需要先通过`kubectl port-forward`完成Ubuntu VM和Minikube容器的[端口映射](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=%E7%AB%AF%E5%8F%A3%E6%98%A0%E5%B0%84&zhida_source=entity)，然后再使用`minikube tunnel`完成VM到Windows的端口映射。

## 3\. 基于K8S+Jenkins构建CI/CD流水线

[Jenkins](https://link.zhihu.com/?target=https%3A//www.jenkins.io/)是一个经久不衰的[持续集成](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90&zhida_source=entity)工具，它的插件生态比较强大。我们构建基于Jenkins+Kubernetes的CI/CD流水线要达到的目的如下：

  1. 基于K8S实现Jenkins的弹性部署；
  2. 基于Jenkins插件实现CI任务在K8S上的运行；
  3. 基于Pipeline as Code实现Jenkins的流水线[配置管理](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=%E9%85%8D%E7%BD%AE%E7%AE%A1%E7%90%86&zhida_source=entity)（持续集成任务+持续部署任务）。



### 基于K8S实现Jenkins的弹性部署

MindSpore易点通机器人的[代码仓](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig/robot/blob/master/infra/jenkins/)已经给出了在K8S上部署Jenkins的[配置文件](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6&zhida_source=entity)，这里要设置成`LoadBalancer`类型。
    
    
    ---
    apiVersion: v1
    kind: Service
    [metadata](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=metadata&zhida_source=entity):
      name: jenkins
    spec:
      type: LoadBalancer
      selector:
        name: jenkins
      ports:
        -
          name: http
          port: 8080
          targetPort: 8080
          protocol: TCP

再用`kubectl create -n jenkins`创建Jenkins的namespace，通过`kubectl apply -f jenkins.yaml -n jenkins`完成部署，然后用`kubectl apply -f service-account.yaml -n jenkins`完成API调用的授权。

在配置文件中我们指定了对外暴露的端口是8080，所以可以用`kubectl port-forward svc jenkins/jenkins 8080:8080 -n jenkins`完成端口映射，再执行`minikube tunnel`就可以在浏览器中使用`127.0.0.1:8080`打开Jenkins界面了，初始密码可以在pod启动的日志中获得。

### 基于Jenkins插件实现CI任务在K8S上的运行

完成Jenkins在K8S上的部署后，需要在Jenkins中安装`kubernetes`插件，配置节点类型为K8S集群。从插件管理中先安装`[kubernetes](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=4&q=kubernetes&zhida_source=entity)`插件，然后在`节点管理`中选择`配置集群`。  
首先配置K8S集群的信息，因为在一个K8S集群，服务间都可以通过[主机名](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=%E4%B8%BB%E6%9C%BA%E5%90%8D&zhida_source=entity)的方式相互访问，所以“Kubernetes地址”配置只需要输入`https://kubernetes.defaults`，同时补充配置“Kubernetes[命名空间](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4&zhida_source=entity)”为`jenkins`，如下图所示。 

同理，对于“Jenkins地址”，只需要填入`http://jenkins.jenkins:8080`。

另一个要配置是pod模板，既任务运行的[pod](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=3&q=pod&zhida_source=entity)的基础镜像及相关信息配置。要同时运行Java和Python，所以在[dockerhub](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=dockerhub&zhida_source=entity)找了一个Java和Python都有的镜像，如下图所示，保存后即可完成配置。

### 基于Pipeline as Code实现Jenkins的流水线配置管理

我们的最后一步是基于Jenkins的流水线即代码功能完成CI/CD流水线搭建，按照设计，它应该包含如下任务：

  * 持续集成流水线
    * 代码检查：[数据处理](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86&zhida_source=entity)代码、模型代码、推理代码以及脚本代码规范检查任务
    * 单元测试：数据处理逻辑、模型代码逻辑、推理代码逻辑的单元测试任务
    * API测试：推理接口功能测试
    * 训练触发：如果修改了训练代码，触发Argo的训练流水线
  * 部署



使用Jenkins流水线即代码功能，对应的配置如下：
    
    
    pipeline {
        agent {
            kubernetes {
                containerTemplate {
                    name 'python'
                    image 'bitnami/java:1.8'
                    command 'sleep'
                    args 'infinity'
                }
                defaultContainer 'python'
            }
        }
       stages {
            stage('Code Check ') {
                steps("Code Check") {
                    echo 'checking python code.'
                }
            }
            stage('Unit Testing') {
                steps("Unit Testing") {
                    echo "running unit tests"
                }
            }
            stage('API Testing') {
                steps {
                    echo 'running inference API Testing'
                }
            }
            stage('Training Trigger') {
                when {
                    changeset "src/train/*.py"
                }
                steps("trigger training") {
                    echo 'trigger new round of training'
                }
            }
        }
    }

部署流水线的pipeline脚本如下：
    
    
    pipeline {
      agent  any
       stages {
            stage('Packaging') {
                steps("Packaging") {
                    echo 'packaging with model and inference code'
                }
            }
            stage('Continuous Deployment') {
                steps("deploying") {
                    echo 'deploy new version of model'
                }
            }
        }
    }

配置文件中每个step先置空，是为了方便调试。在Jenkins基于上面的配置文件创建一个流水线后的测试结果如下：

## 4\. 基于K8S+Argo持续训练流水线

[Argo](https://link.zhihu.com/?target=https%3A//argoproj.github.io/)是一个基于K8S的开源的工作流管理工具，也支持机器学习的工作流。MindSpore DX Sig已经在先前的社区机器人项目中使用了该工具，所以这里我们复用了工具和配置。Argo Workflow的安装配置如下：

  1. 在K8S上安装Argo；
  2. 基于Argo Workflow 配置机器学习流水线；
  3. 运行机器学习流水线。



### 1\. 在K8S上安装Argo

首先，我们执行`kubectl create -n argo`为Argo创建新的命名空间。然后，基于[配置文件](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig/robot/tree/master/infra/argo)，执行`kubect apply -f install.yml -n argo`完成安装。最后，执行`kubect apply -f manifests/create_serviceaccount.yaml -n argo`完成权限配置。

和前面的Jenkins配置类似，Argo Server需要设置为`LoadBalancer`类型。
    
    
    apiVersion: v1
    kind: Service
    metadata:
      name: argo-server
    spec:
      ports:
      - name: web
        port: 2746
        targetPort: 2746
      type: LoadBalancer
      sessionAffinity: None
      externalTrafficPolicy: Cluster
      selector:
        app: argo-server

执行`kubectl get svc -n argo`可以看到：
    
    
    ~$ kubectl get svc -n argo
    NAME                          TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    argo-server                   LoadBalancer   10.97.145.232   127.0.0.1     2746:30001/TCP   19d
    workflow-controller-metrics   ClusterIP      10.99.196.47    <none>        9090/TCP         19d

如果要在Windows的浏览器上访问Argo的Web界面，则还需要把argo-server的端口暴露出来，同上使用`kubectl port-forward svc/argo-server 2746:2746 -n argo`完成端口映射。然后浏览器上访问`https://127.0.0.1:2746`，通过下面的脚本获得密码，登录后就可以正常使用Argo的Web管理界面了。
    
    
    #!/bin/bash
    SECRET=$(kubectl get sa argo-server -n argo -o=jsonpath='{.secrets[0].name}')
    ARGO_TOKEN="Bearer $(kubectl get secret $SECRET -n argo -o=jsonpath='{.data.token}' | base64 --decode)"
    echo $ARGO_TOKEN

### 2\. 基于Argo Workflow 配置机器学习流水线

我们期望训练的工作流可以完成以下任务：

  1. 数据处理
  2. 训练
  3. 评估
     1. 质量评估：基于测试集数据评估模型，预测性能需要高于[基线值](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=%E5%9F%BA%E7%BA%BF%E5%80%BC&zhida_source=entity)
     2. 可解释性评估
     3. 可靠性评估
  4. 总结



基于工作流的设计以及Argo工作流语法，可以得出基础的配置：
    
    
    apiVersion: Page Not Found
    kind: Workflow
    metadata:
      generateName: robot-train-eval-
    spec:
      serviceAccountName: robot-sa
      entrypoint: robot-controller
      onExit: summary
      templates:
      - name: robot-controller
        steps:
        - - name: data-process
            template: process
        - - name: robot-train
            template: train
        - - name: robot-eval
            template: eval
          - name: robot-interpretability
            template: interpretability
          - name: robot-reliability
            template: reliability
        - - name: summary
            template: summary

而后，展开每个任务需要的配置，如训练的配置代码：
    
    
      - name: train
        container:
          image: ubuntu
          imagePullPolicy: Always
          env:
          - name: IS_TRAIN
            value: "True"
          - name: NUM_STEPS
            value: "10"
          command: ['echo']
          args: ["trainning"]
        ...

把每个阶段的任务汇总在一起就完成了整个的工作流。接下来，我们尝试使用Argo运行下训练流水线。

### 3\. 运行机器学习流水线

可以使用Argo CLI的客户端完成工作流任务的提交，首先安装客户端：
    
    
    #!/bin/bash
    # Download the binary
    curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v3.3.5/argo-linux-amd64.gz
    # Unzip
    gunzip argo-linux-amd64.gz
    # Make binary executable
    chmod +x argo-linux-amd64
    # Move binary to path
    mv ./argo-linux-amd64 /usr/local/bin/argo

然后通过argo cli提交工作流`argo submit -n robot --watch robot-train-eval.yaml`，执行结果如下，和我们期望的流水线步骤一致。

  


  


## 总结

本篇文章总结了如何在本地完成MLOps环境的搭建，基于Minikube、Argo等工具可以让我们很好的在本地展开开发验证工作，不用依赖复杂的基础设施，也不用有额外的开销。在配置文件中和脚本中，我们没有加真实的实现，是为了先打通流程，然后再将调试好内容逐步补充进去，始终都可以从端到端的角度来完成验证。

  


> 上一篇：[【MindSpore易点通机器人-03】迭代0的准备工作](https://zhuanlan.zhihu.com/p/509494866)  
> 下一篇：[【MindSpore易点通机器人-05】问答数据[预处理](https://zhida.zhihu.com/search?content_id=202575960&content_type=Article&match_order=1&q=%E9%A2%84%E5%A4%84%E7%90%86&zhida_source=entity)及编码](https://zhuanlan.zhihu.com/p/561830878)
