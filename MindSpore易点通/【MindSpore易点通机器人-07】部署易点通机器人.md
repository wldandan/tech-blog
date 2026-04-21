# 【MindSpore易点通机器人-07】部署易点通机器人

原文链接：https://zhuanlan.zhihu.com/p/566066575

---

​

目录

在[【MindSpore易点通机器人-06】基于相似度模型实现问答匹配及推荐功能](https://zhuanlan.zhihu.com/p/562778149)一文中，我们已经使用QA数据集验证了相似度模型，接下来就需要考虑如何部署到线上，对外提供服务。在机器人设计之初，我们就考虑将其以[容器化](https://zhida.zhihu.com/search?content_id=213924960&content_type=Article&match_order=1&q=%E5%AE%B9%E5%99%A8%E5%8C%96&zhida_source=entity)的方式部署到云上。而完成整个过程，需要以下几步：

  1. 设计Restful的推理API接口；
  2. 构建机器人API的[容器镜像](https://zhida.zhihu.com/search?content_id=213924960&content_type=Article&match_order=1&q=%E5%AE%B9%E5%99%A8%E9%95%9C%E5%83%8F&zhida_source=entity)；
  3. 将机器人部署上云。



## 1\. 设计Restful的推理API接口

对于推理的API，用户只需要输入问题，响应则需要包含对应的答案，以及问题没有命中后给出的推荐提问。推理的REST API设计如下：
    
    
    /robot/api/:
        get:
          summary: get question/answer result from robot
          description: get question/answer result from robot
          operationId: get_answer
          parameters:
            - name: q
              in: query
              description: put in questions
              required: true
              schema:
                type: string
          responses:
            '200':
              description: successful response
              content:
                application/json:
                  schema:
                    type: object
                        properties:
                            question:
                                type: integer
                            answer:
                                type: string

出于快速实现的考虑，我们使用了Django框架，核心的代码实现包括`qa_api/views.py`（[推理逻辑](https://zhida.zhihu.com/search?content_id=213924960&content_type=Article&match_order=1&q=%E6%8E%A8%E7%90%86%E9%80%BB%E8%BE%91&zhida_source=entity)）以及`qa_api/views.py`（问题编码/匹配）。

`qa_api/views.py`是[api接口](https://zhida.zhihu.com/search?content_id=213924960&content_type=Article&match_order=1&q=api%E6%8E%A5%E5%8F%A3&zhida_source=entity)的入口，其主要实现的逻辑如下：

  * 获取用户输入。
  * 对用户输入使用Bert模型进行编码。
  * 在知识库中搜索获取匹配的问题。
  * 从知识库中获取问题的答案，组装成JSON格式数据返回给用户，如果没有相似度达标的答案，则返回推荐的问题。



代码实现如下：
    
    
    @api_view(['GET'])
    def get_answer(request):
        question = request.query_params.get('q', None)  # 获取输入
        input_encode = encode_sentence(question)        # 编码输入的问题
        match_keys = get_match_question(input_encode)   # 在QA数据编码集合中查找是否有匹配的问题对
        file_path = os.path.join(module_dir, 'data/q_a.json')
        q_a_data = {}
        if not q_a_data:
            q_a_data = load_q_a_data(file_path)
        r = []
        if match_keys:                                 # 组装返回内容 
            for k in match_keys:
                q_a = {"question": k}
                if k in q_a_data.keys():
                    q_a["answer"] = q_a_data[k]
                    r.append(q_a)
        return HttpResponse(json.dumps(r, ensure_ascii=False), content_type='application/json')

`get_match_question`实现了问题编码，问题匹配等逻辑，在文件`qaRobt/qa_api/qa/q_a.py`中，通过相似度计算判断是否返回问题答案或者只返回推荐的问题。
    
    
    def get_match_question(input_encode):
        with open(os.path.join(module_dir, '../data/resource_sentence_encode.json')) as f:
            resource_sentences_encode = json.load(f)
            question_similarity = {}
            for k, v in resource_sentences_encode.items():
                similarity = [cosine_similarity](https://zhida.zhihu.com/search?content_id=213924960&content_type=Article&match_order=1&q=cosine_similarity&zhida_source=entity)(
                    [input_encode[1][0].asnumpy()], [np.asarray(v)])
                question_similarity[k] = similarity
            sorted_similarity = sorted(question_similarity.items(), key=lambda x: x[1], reverse=True)
            if not sorted_similarity or len(sorted_similarity) == 0:
                return None
            elif sorted_similarity[0][1] >= 0.7:
                return [sorted_similarity[0][0]]
            elif len(sorted_similarity) > 1 and sorted_similarity[0][1] < 0.7:
                return [sorted_similarity[0][0], sorted_similarity[1][0]]

除了功能实现外，还需要考虑在生产环境部署时，开启跨站访问，以便于集成验证。 在`settings.py`配置中加入如下内容：
    
    
    # Application definition
    INSTALLED_APPS = [
        'django.contrib.admin',
        ……
        'django.contrib.staticfiles',
        'corsheaders',  #增加cors应用
        'qa_api',
        'rest_framework',
    ]
    
    MIDDLEWARE = [
        'django.middleware.security.SecurityMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        'corsheaders.middleware.CorsMiddleware',   # cors[中间件](https://zhida.zhihu.com/search?content_id=213924960&content_type=Article&match_order=1&q=%E4%B8%AD%E9%97%B4%E4%BB%B6&zhida_source=entity)
        ……,
    ]
    
    CORS_ALLOW_CREDENTIALS = True
    CORS_ORIGIN_ALLOW_ALL = True
    #allow all requests
    CORS_ALLOW_HEADERS = ('*')

此外，在API的依赖文件`requirements.txt`中，增加`[django-cors-headers](https://zhida.zhihu.com/search?content_id=213924960&content_type=Article&match_order=1&q=django-cors-headers&zhida_source=entity)==3.11.0`依赖即可。

## 2\. 构建机器人API的容器镜像

为了方便应用部署，我们将机器人API打包为Docker容器进行部署，Dockerfile内容如下。
    
    
    FROM swr.cn-south-1.myhuaweicloud.com/[mindspore](https://zhida.zhihu.com/search?content_id=213924960&content_type=Article&match_order=1&q=mindspore&zhida_source=entity)/mindspore-cpu:1.8.0
    RUN mkdir /code
    WORKDIR /code
    COPY src/qaRobot/requirements.txt /code/
    RUN pip install -r requirements.txt
    COPY src/qaRobot /code/
    EXPOSE 8000
    CMD ["python", "./manage.py", "runserver", "0.0.0.0:8000"]

我们选择MindSpore[官方镜像](https://zhida.zhihu.com/search?content_id=213924960&content_type=Article&match_order=1&q=%E5%AE%98%E6%96%B9%E9%95%9C%E5%83%8F&zhida_source=entity)作为基础镜像，然后拷贝代码，安装依赖，启动服务，端口为8000。使用`docker build . -t robot:v0.1.0`命令即可完成镜像构建。

## 3\. 将机器人部署上云

容器化后部署的方式比较自由，可以选择[华为云](https://zhida.zhihu.com/search?content_id=213924960&content_type=Article&match_order=1&q=%E5%8D%8E%E4%B8%BA%E4%BA%91&zhida_source=entity)ECS虚机、CCI容器服务或者函数服务进行部署。这个阶段我们选择最简单的方式，在ECS虚机上进行部署，只需要在虚机系统的命令行中输入
    
    
    docker run -d -p 80:8000 robot:v0.1.0

然后在ECS主机的[安全组](https://zhida.zhihu.com/search?content_id=213924960&content_type=Article&match_order=1&q=%E5%AE%89%E5%85%A8%E7%BB%84&zhida_source=entity)开放HTTP协议访问许可，就可以接受来自外部的请求了。

在浏览器中尝试发访问该API，即可看到响应的结果。

至此，我们简单完成易点通机器人的部署上线，下一步要做的事情就是和IDE进行集成验证，以Demo形式开放给用户使用。

### 参考资料

  1. [易点通机器人源码](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig/robot)



  


> 上一篇：[【MindSpore易点通机器人-06】基于相似度模型实现问答匹配及推荐功能](https://zhuanlan.zhihu.com/p/562778149)
