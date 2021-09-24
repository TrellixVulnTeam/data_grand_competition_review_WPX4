### 第五届“达观杯” 基于大规模预训练模型的风险事件标签识别赛后总结

##### 比赛链接

https://www.datafountain.cn/competitions/512/ranking?isRedance=0&sch=1804&stage=A

本项目是该比赛的代码和思路整理，许多代码可以多次复用，因为整理出来

#### 数据处理

```
# train dataset and test dataset
/datasets

# train dataset label1 and label2 freq count
/datasets/label_vocab_process.py

# train dataset sentences length count
/datasets/sample_length_stats.py

# split train dataset
/datasets/split_datasets.py
```

#### 词频对应

```
# get word freq count from corpus
src/bert_models/vocak_process.py

# transform train dataset to plaintext according to word freq count
src/bert_models/get_vocab_mapping.py
```

#### 思路整理

比赛过程中，幸得老师的指导，开拓了许多思路，然而因为比赛的时间比较紧，没有来得及所有思路都尝试一遍，下面仅仅列出思路以及paper及相关的代码片段。

- pooling层变化

  **pool** 其实就是bert模型概念中的cls，即对于一个句子他由多个word组成的，然后每个word具有其对应的embedding，那么cls之前的一层的对应的维度就应该为：

  **batch_size * sentence_length * embedding_length**

  那个如果将一个句子用一个向量来表示了（cls），那么就需要使用pool层，将embedding的dim压缩到1

  ```
  # 根据无监督数据训练word2vec
  /src/classic_models/word2vec/train_word2vec.py
  
  # max-pool
  /src/classic_models/modules/max_pool.py
  
  # self_attn_pool
  # reference paper: https://www.cs.cmu.edu/~./hovy/papers/16HLT-hierarchical-attention-networks.pdf
  # 论文中提出了一种计算word embedding到sentence embedding的思想：
  # 其实就是随机初始化一个向量，把该向量做为一个标杆同word embedding做点积得到这个字同这个标杆的相关性，然后利用这个相关性做
  # softmax得到每个word embedding所对应的系数，然后利用每个word embedding的系数得到sentence embedding
  /src/classic_model/modules/self_attn_pool.py
  
  # dynamic_routing_pool
  # reference paper: https://arxiv.org/pdf/1806.01501.pdf
  # 论文中提出了一种胶囊网络，这里主要借鉴的是胶囊网络的pooling，即为dynamic_pooling
  # dynamic_pooling中的主要思想是将word embedding通过多个全联接层，得到对应的多个映射的之后的表示，然后通过特定系数的计算
  # 方式得到这几个映射的系数，最终得到sentence embedding
  /src/classic_model/modules/chid_dynamic_routing.py
  ```

- Bert改进模型使用

  Bert的原始模型中是先切词，使用MLM来mask句子中的一个单词，但是这样会有一个问题就是单纯mask句子中的一个单词，当模型训练的时候周围的单词，会给mask的单词很多提示，这样就会弱化MLM语言模型的锻炼

  **WWM**是Whole Word Masking Models，即mask all of the tokens correspond to a word，剩下的没有什么变化

  比赛中尝试了很多WWM模型，其中效果最好的模型是**nezha-large-bert**

- 样本不均衡问题解决

  统计了train dataset中的标签的频率，可以明显看出标签存在的明显的不均衡的情况

  ​	**todo**

  - Class weights/ sample weights
  - Focal loss
  - dice loss

- Muliti-sample dropout

- Contrastive learning

  - Info NCE
  - Supervised CL

- Adversarial training

- Multi-exit