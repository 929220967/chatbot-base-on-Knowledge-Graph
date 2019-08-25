It is indeed a luxury to keep human reason forever. by  Moss, a robot of the film The Wandering Earth

“让人类永远保持理智，确实是一种奢求” ，机器人莫斯，《流浪地球》

<p align="center">
	<img src=./pictures/moss.jpg alt="Sample"  width="600">
	<p align="center">
		<em> </em>
	</p>
</p>


### 项目概况

本项目为一个使用深度学习方法解析问题，知识图谱存储、查询知识点，基于医疗垂直领域的对话系统的后台程序

+ 运行效果：



+ 项目的搭建大致分为四个模块：
  + 基础数据爬取
  + 知识图谱构建
  + 问句分析
  + 回答生成

+ 项目运行环境：

python   :  python 3.6.8

运行系统：ubuntu 16.04 

知识图谱：neo4j 3.2.2 图形数据库

```
neo4j对应的python驱动
py2neo          3.1.1  
```

深度学习：

```
jieba           0.39   
numpy           1.17.0 
pandas          0.25.0 
tensorflow      1.10.0 
```

文本匹配：ahocorasick （安装方法 pip install pyahocorasick）

必要说明：

深度学习模块深度网络的训练使用tensorflow的gpu版本，在应用阶段由于要部署要服务器上使用的对应的tensorflow的cpu版本

若要clone项目，尽量保持扩展包的版本一致

+ 项目运行方式

1. 搭建知识图谱：python build_grapy.py。大概几个小时，耐心等待。 
2. 启动问答系统：python chatbot_graph.py  

+ 项目主要文件目录结构

```shell
chatbot
├── answer_search.py                        # 问题查询及返回
├── BiLSTM_CRF.py                           # 实体识别的双向LSTM-CRF网络
├── build_medicalgraph.py                   # 将结构化json数据导入neo4j
├── chatbot_graph.py                        # 问答程序脚本
├── classifyApp.py                          # 问句分类应用脚本
├── classifyUtils.py                        # 工具函数集合
├── data
│   └── medical.json                        # 全科知识数据
├── data_ai
│   ├── cbowData                            # 词向量文件
│   │   ├── classifyDocument.txt.ebd.npy    # 词向量查找表
│   │   ├── classifyDocument.txt.vab        # 词向量中词与索引对照表
│   │   ├── document.txt.ebd.npy            
│   │   └── document.txt.vab
│   ├── classifyData                        # 问句分类训练数据
│   │   ├── test_data.txt
│   │   └── train_data.txt
│   ├── classifyModel                       # 问句分类模型
│   │   ├── checkpoint
│   │   ├── model-3500.data-00000-of-00001
│   │   ├── model-3500.index
│   │   └── model-3500.meta
│   ├── nerData                          
│   └── nerModel                            # 命名实体识别模型
├── dict                                    # 实体数据文件
├── nerApp.py                               # 命名实体识别应用脚本
├── nerUtils.py                             # 工具函数集合
├── prepare_data                           
│   ├── build_data.py                       # 数据库操作脚本
│   ├── data_spider.py                      # 数据采集脚本
│   └── max_cut.py                          # 基于词典的最大前向/后向匹配
├── question_analysis.py                    # 问句类型分类脚本
├── question_parser.py                      # 回答生成脚本
└── text_cnn.py                             # 文本分类的cnn网络
```



### 基础数据爬取

基础数据爬取于[寻医问药](<http://www.xywy.com/>)网站，一家医疗信息提供平台，上面的数据做了较好的分类处理，爬下来后可以较为方便的保存为json格式的结构化文件，格式展示如下：

<p align="center">
	<img src=./pictures/json_show.gif alt="Sample"  width="700">
	<p align="center">
		<em> 爬取的数据保存为json格式文件 </em>
	</p>
</p>


### 知识图谱搭建


知识图谱可以用若干三元组来表示，三元组的基本形式：

+ 实体1-关系-实体2
+ 实体-属性-属性值

将爬取的数据调用`build_medicalgraph.py    `脚本将结构化json数据导入neo4j图数据库，部分数据库展示如下：

<p align="center">
	<img src=./pictures/graph.svg alt="Sample"  width="800">
	<p align="center">
		<em> 图形数据库部分展示 </em>
	</p>
</p>

知识图谱实体类型

| 实体类型   |   中文含义   | 实体数量 | 举例                                   |
| :--------- | :----------: | :------: | :------------------------------------- |
| Check      | 诊断检查项目 |  3,353   | 支气管造影;关节镜检查                  |
| Department |   医疗科目   |    54    | 整形美容科;烧伤科                      |
| Disease    |     疾病     |  8,807   | 血栓闭塞性脉管炎;胸降主动脉动脉瘤      |
| Drug       |     药品     |  3,828   | 京万红痔疮膏;布林佐胺滴眼液            |
| Food       |     食物     |  4,870   | 番茄冲菜牛肉丸汤;竹笋炖羊肉            |
| Producer   |   在售药品   |  17,201  | 通药制药青霉素V钾片;青阳醋酸地塞米松片 |
| Symptom    |   疾病症状   |  5,998   | 乳腺组织肥厚;脑实质深部出血            |
| Total      |     总计     |  44,111  | 约4.4万实体量级                        |

知识图谱实体关系类型

| 实体关系类型   |   中文含义   | 关系数量 | 举例                                                 |
| :------------- | :----------: | :------: | :--------------------------------------------------- |
| belongs_to     |     属于     |  8,844   | <妇科,属于,妇产科>                                   |
| common_drug    | 疾病常用药品 |  14,649  | <阳强,常用,甲磺酸酚妥拉明分散片>                     |
| do_eat         | 疾病宜吃食物 |  22,238  | <胸椎骨折,宜吃,黑鱼>                                 |
| drugs_of       | 药品在售药品 |  17,315  | <青霉素V钾片,在售,通药制药青霉素V钾片>               |
| need_check     | 疾病所需检查 |  39,422  | <单侧肺气肿,所需检查,支气管造影>                     |
| no_eat         | 疾病忌吃食物 |  22,247  | <唇病,忌吃,杏仁>                                     |
| recommand_drug | 疾病推荐药品 |  59,467  | <混合痔,推荐用药,京万红痔疮膏>                       |
| recommand_eat  | 疾病推荐食谱 |  40,221  | <鞘膜积液,推荐食谱,番茄冲菜牛肉丸汤>                 |
| has_symptom    |   疾病症状   |  5,998   | <早期乳腺癌,疾病症状,乳腺组织肥厚>                   |
| acompany_with  | 疾病并发疾病 |  12,029  | <下肢交通静脉瓣膜关闭不全,并发疾病,血栓闭塞性脉管炎> |
| Total          |     总计     | 294,149  | 约30万关系量级                                       |

知识图谱属性类型

| 属性类型      |   中文含义   |            举例             |
| :------------ | :----------: | :-------------------------: |
| name          |   疾病名称   |       喘息样支气管炎        |
| desc          |   疾病简介   |    又称哮喘性支气管炎...    |
| cause         |   疾病病因   |    常见的有合胞病毒等...    |
| prevent       |   预防措施   | 注意家族与患儿自身过敏史... |
| cure_lasttime |   治疗周期   |          6-12个月           |
| cure_way      |   治疗方式   |   "药物治疗","支持性治疗"   |
| cured_prob    |   治愈概率   |             95%             |
| easy_get      | 疾病易感人群 |        无特定的人群         |



### 自动问答实现

自动问答采用深度学习的方法，由于缺少问句训练语料，训练数据来源于自制的问句生成器，然后对问句分词，问句中的每个词进行嵌入，即由词向量组成的问句代替自然语言的问句输入，再进行命名实体识别及实体/问句关系抽取（问句分类），实现对问句的语义解析。

<p align="center">
	<img src=./pictures/082501.png alt="Sample"  width="600">
	<p align="center">
		<em> 自动问答实现流程图 </em>
	</p>
</p>

本仓库为了代码结构清晰，只放了深度学习的模型应用的脚本，词向量及模型训练的脚本会放在另一个代码仓库中。

#### 数据准备

+ 数据冷启动

问句解析部分是用深度学习的方法实现的，那自然需要数据来训练模型。在通常的垂直领域内，由于缺乏系统性地数据积累或合作项目，本项目所用地问句语义解析必须依赖大规模地问句语料，因此设计了一个问句生成器（专业点地叫法为数据冷启动？），就是根据设定好的问句模板将上文爬取到的实体填充到模板的槽当中，同时对问句进行逐词的命名实体标注（BIOES标注法）及问句类别标注，用于后面的实体抽取及实体/问句关系抽取（问句分类）

命名实体标注标签：

| 实体  | 序号 | 含义          |
| ----- | ---- | ------------- |
| O     | 0    | 其它          |
| B-dis | 1    | 疾病实体开头  |
| I-dis | 2    | 疾病实体中间  |
| E-dis | 3    | 疾病实体末尾  |
| B-sym | 4    | 症状          |
| I-sym | 5    |               |
| E-sym | 6    |               |
| B-dru | 7    | 药品          |
| I-dru | 8    |               |
| E-dru | 9    |               |
| S-dis | 10   | 单个-疾病实体 |
| S-sym | 11   |               |
| S-dru | 12   |               |

问句类别标注标签

| 类别            | 序号 | 含义               |
| --------------- | ---- | ------------------ |
| disease_symptom | 0    | 疾病有啥症状       |
| symptom_curway  | 1    | 症状有啥治疗方法   |
| symptom_disease | 2    | 症状对应啥疾病     |
| disease_drug    | 3    | 疾病要吃啥药品     |
| drug_disease    | 4    | 药品治疗啥疾病     |
| disease_check   | 5    | 疾病要做啥检查检查 |
| disease_prevent | 6    | 疾病有啥预防方式   |

+ 数据增强

经过人工造问句后，再针对问句结构类型单一不够多样进行了数据增强，比如采取了如下措施：句子结构倒装，同义词替换，随机插入标点加入噪音

+ 数据类别平衡及shuffle

在未做类别平衡及数据shuffle时，模型会严重过拟合，有时只能预测出一种结果，在测试集上正确率很低，做了类别平衡及输入数据打乱之后预测结果大幅改观

#### 词向量训练

采用word2vec模型中的连续词袋模型cbow进行词向量的训练

一些参数：

| 参数名                        | 参数值 |
| ----------------------------- | ------ |
| 学习率                        | 0.0001 |
| 词向量长度（中间/隐藏层维度） | 200    |
| 上下文window_size             | 10     |
| batch_size                    | 300    |
| 最小词频min_frq               | 2      |

值得注意的是：

1. 词向量的训练也会有loss，但是在训练词向量的过程中没有太必要关注其loss，因为训练词向量一般只是我们想要的中间结果，与我们的目的相去甚远，经验是等loss稳定之后将词向量先用于后面的任务，看后面任务的实际效果怎么样，若效果不佳再调整参数甚至更换其它词向量模型。

2. 最开始是用的字向量，但是用于之后的任务效果不佳，会出现ner标注偏执及正确率低的现象。然后使用了词向量，效果提升较大。究其原因可能是词包含的信息更多，对模型的辅助效果更明显
3. 网上说词向量一般在`200~300`维度表示效果较好，字向量在`100~200`维度就够了，当语料很小时候，词向量维度应调小。实际测试在10多M的语料大小情况下，词向量维度50都能达到可用效果。

#### 医疗命名实体识别

+ 模型训练描述

在知识库问答系统处理过程中，解析问句意图首先需要进行命名实体识别（NER），正确提取出问句中询问的医疗实体。当前NER模型大多采用LSTM-CRF模型。基于词的中文NER，则需要预先对句子进行分词。

0.词的嵌入

最开始我是用的字向量，但是效果不好，换成了词向量。句子分词后，将每个词查找词向量的lookup tabel获得对应词向量，用词向量替换原句子中的词，形成新的句子作为输入，为保证训练效果，当句子太长时候要截断，句子太短时要填充，本项目使用的0填充。

1.句子输入到BiLSTM模型

然后将规整后的句子输入到BiLSTM网络中，就是句子正向导入LSTM网络一次，再把句子反向导入LSTM网络一次，经过多次迭代输出LSTM网络的两个预测结果（正向，反向），然后将两个预测结果拼接成一个长向量作为下一层的CRF层的输入

具体是怎么拼接的：做ner时，前向时候得到的LSTM的的中间状态输出向量和后向时中间状态输出向量中对应的单词的中间状态拼接，如下图：

<p align="center">
	<img src=./pictures/082502.png alt="Sample"  width="700">
	<p align="center">
		<em> lstm中间状态向量拼接作为输出用于ner </em>
	</p>
</p>

若用于句子的情感分类则作以下拼接：

<p align="center">
	<img src=./pictures/082503.png alt="Sample"  width="700">
	<p align="center">
		<em> lstm中间状态向量拼接作为输出用于情感分类 </em>
	</p>
</p>
tensorflow中tf.nn.dynamic_rnn函数

```
outputs, state = tf.nn.dynamic_rnn(
    cell,
    inputs,
    sequence_length=None,
    initial_state=None,
    dtype=None,
    parallel_iterations=None,
    swap_memory=False,
    time_major=False,
    scope=None
)
outputs: The RNN output Tensor. this will be a Tensor shaped: [batch_size, max_time, cell.output_size].

state: The final state. If cell.state_size is an int, this will be shaped [batch_size, cell.state_size]. 
```

第一个输出`outputs`就是一批数据的中间状态输出的集合（张量）。第二个输出`state`就是LSTM最后一个状态，

它含了一个方向的所有信息

2.中间状态输入到CRF层

CRF的`转移矩阵A`由神经网络的CRF学习得到，而`发射概率P矩阵` 就是由Bi-LSTM的输出来作近似模拟。	



<p align="center">
	<img src=./pictures/082504.png alt="Sample"  width="900">
	<p align="center">
		<em> Bi-LSMT+CRF </em>
	</p>
</p>
3.最小化对数损失，反向传播更新参数

损失函数用的对数损失，反向传播更新参数，进行下一批数据前向传播训练

+ 参数

a）使用句子分词后词的词向量作为输入，

b）dropout的值调到0.5，

c）句子的最大长度sentence_length调到30以下（我使用的20），

d）句子填充那里使用的0填充，

e）语料中实体种类数目做平衡（不出现某个种类严重偏执，否者就回导致预测偏执严重和过拟合），

f）语料标注使用的BIOES标注（之前用的BIO标注）

训练出来模型的F1值可以达到0.98，

| 参数名                | 参数值  |
| --------------------- | ------- |
| lstm 隐藏层神经元个数 | 600     |
| 学习速率              | 0.00075 |
| batch_size            | 100     |
| 句子截断长度          | 25      |
| 梯度截断              | [-5,5]  |
| 标签数目              | 13      |
| 训练时dropout         | 0.5     |
| 句子填充              | 0值填充 |
| 句子标注方式          | BIOES   |

#### 医疗问句分类

通过命名实体识别模型正确提取出问句中询问的医疗实体之后，还需要理解用户问句的意图，其意图的具体表现就是医疗实体的关系或属性，即需要进行问句意图和知识库关系的映射。考虑医疗问诊场景的用户问题通常是短文本，因此本项目将关系／属性映射设定为短文本分类任务，

