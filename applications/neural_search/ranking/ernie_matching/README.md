
 **目录**

* [背景介绍](#背景介绍)
* [ERNIE-Gram](#ERNIE-Gram)
    * [1. 技术方案和评估指标](#技术方案)
    * [2. 环境依赖](#环境依赖)
    * [3. 代码结构](#代码结构)
    * [4. 数据准备](#数据准备)
    * [5. 模型训练](#模型训练)
    * [6. 评估](#开始评估)
    * [7. 预测](#预测)
    * [8. 部署](#部署)

<a name="背景介绍"></a>

# 背景介绍

基于ERNIE-Gram训练Pair-wise模型。Pair-wise 匹配模型适合将文本对相似度作为特征之一输入到上层排序模块进行排序的应用场景。


<a name="ERNIE-Gram"></a>

# ERNIE-Gram

<a name="技术方案"></a>

## 1. 技术方案和评估指标

### 技术方案

双塔模型，使用ERNIE-Gram预训练模型，使用margin_ranking_loss训练模型。


### 评估指标

（1）采用 AUC 指标来评估排序模型的排序效果。

**效果评估**

|  模型 |  AUC |
| ------------ | ------------ |
|  ERNIE-Gram |  0.801 |

<a name="环境依赖"></a>

## 2. 环境依赖和安装说明

**环境依赖**

* python >= 3.x
* paddlepaddle >= 2.1.3
* paddlenlp >= 2.2
* pandas >= 0.25.1
* scipy >= 1.3.1

<a name="代码结构"></a>

## 3. 代码结构

以下是本项目主要代码结构及说明：

```
ernie_matching/
├── deply # 部署
    └── python
        ├── deploy.sh # 预测部署bash脚本
        └── predict.py # python 预测部署示例
|—— scripts
    ├── export_model.sh # 动态图参数导出静态图参数的bash文件
    ├── train_pairwise.sh # Pair-wise 单塔匹配模型训练的bash文件
    ├── evaluate.sh # 评估验证文件bash脚本
    ├── predict_pairwise.sh # Pair-wise 单塔匹配模型预测脚本的bash文件
├── export_model.py # 动态图参数导出静态图参数脚本
├── model.py #  Pair-wise 匹配模型组网
├── data.py #  Pair-wise 训练样本的转换逻辑 、Pair-wise 生成随机负例的逻辑
├── train_pairwise.py # Pair-wise 单塔匹配模型训练脚本
├── evaluate.py # 评估验证文件
├── predict_pairwise.py # Pair-wise 单塔匹配模型预测脚本，输出文本对是相似度

```

<a name="数据准备"></a>

## 4. 数据准备

### 数据集说明

样例数据如下:
```
个人所得税税务筹划      基于新个税视角下的个人所得税纳税筹划分析新个税;个人所得税;纳税筹划      个人所得税工资薪金税务筹划研究个人所得税,工资薪金,税务筹划
液压支架底座受力分析    ZY4000/09/19D型液压支架的有限元分析液压支架,有限元分析,两端加载,偏载,扭转       基于ANSYS的液压支架多工况受力分析液压支架,四种工况,仿真分析,ANSYS,应力集中,优化
迟发性血管痉挛  西洛他唑治疗动脉瘤性蛛网膜下腔出血后脑血管痉挛的Meta分析西洛他唑,蛛网膜下腔出血,脑血管痉挛,Meta分析     西洛他唑治疗动脉瘤性蛛网膜下腔出血后脑血管痉挛的Meta分析西洛他唑,蛛网膜下腔出血,脑血管痉挛,Meta分析
氧化亚硅        复合溶胶-凝胶一锅法制备锂离子电池氧化亚硅/碳复合负极材料氧化亚硅,溶胶-凝胶法,纳米颗粒,负极,锂离子电池   负载型聚酰亚胺-二氧化硅-银杂化膜的制备和表征聚酰亚胺,二氧化硅,银,杂化膜,促进传输
```


### 数据集下载


- [literature_search_data](https://bj.bcebos.com/v1/paddlenlp/data/literature_search_data.zip)

```
├── milvus # milvus建库数据集
    ├── milvus_data.csv.  # 构建召回库的数据
├── recall  # 召回（语义索引）数据集
    ├── corpus.csv # 用于测试的召回库
    ├── dev.csv  # 召回验证集
    ├── test.csv # 召回测试集
    ├── train.csv  # 召回训练集
    ├── train_unsupervised.csv # 无监督训练集
├── sort # 排序数据集
    ├── test_pairwise.csv   # 排序测试集
    ├── dev_pairwise.csv    # 排序验证集
    └── train_pairwise.csv  # 排序训练集

```

<a name="模型训练"></a>

## 5. 模型训练

**排序模型下载链接：**


|Model|训练参数配置|硬件|MD5|
| ------------ | ------------ | ------------ |-----------|
|[ERNIE-Gram-Sort](https://bj.bcebos.com/v1/paddlenlp/models/ernie_gram_sort.zip)|<div style="width: 150pt">epoch:3 lr:5E-5 bs:64 max_len:64 </div>|<div style="width: 100pt">4卡 v100-16g</div>|d24ece68b7c3626ce6a24baa58dd297d|


### 训练环境说明


- NVIDIA Driver Version: 440.64.00
- Ubuntu 16.04.6 LTS (Docker)
- Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz


### 单机单卡训练/单机多卡训练

这里采用单机多卡方式进行训练，通过如下命令，指定 GPU 0,1,2,3 卡, 基于ERNIE-Gram训练模型，数据量比较大，需要20小时10分钟左右。如果采用单机单卡训练，只需要把`--gpu`参数设置成单卡的卡号即可

训练的命令如下：

```
python -u -m paddle.distributed.launch --gpus "0,2,3,4" train_pairwise.py \
        --device gpu \
        --save_dir ./checkpoints \
        --batch_size 32 \
        --learning_rate 2E-5 \
        --margin 0.1 \
        --eval_step 100 \
        --train_file data/train_pairwise.csv \
        --test_file data/dev_pairwise.csv
```
也可以运行bash脚本：

```
sh scripts/train_pairwise.sh
```

<a name="评估"></a>

## 6. 评估


```
unset CUDA_VISIBLE_DEVICES
python -u -m paddle.distributed.launch --gpus "0" evaluate.py \
        --device gpu \
        --batch_size 32 \
        --learning_rate 2E-5 \
        --init_from_ckpt "./checkpoints/model_30000/model_state.pdparams" \
        --test_file data/dev_pairwise.csv
```
也可以运行bash脚本：

```
sh scripts/evaluate.sh
```


成功运行后会输出下面的指标：

```
eval_dev auc:0.796
```

<a name="预测"></a>

## 7. 预测

### 准备预测数据

待预测数据为 tab 分隔的 tsv 文件，每一行为 1 个文本 Pair，和文本pair的语义索引相似度，部分示例如下:

```
中西方语言与文化的差异  第二语言习得的一大障碍就是文化差异。    0.5160342454910278
中西方语言与文化的差异  跨文化视角下中国文化对外传播路径琐谈跨文化,中国文化,传播,翻译   0.5145505666732788
中西方语言与文化的差异  从中西方民族文化心理的差异看英汉翻译语言,文化,民族文化心理,思维方式,翻译        0.5141439437866211
中西方语言与文化的差异  中英文化差异对翻译的影响中英文化,差异,翻译的影响        0.5138794183731079
中西方语言与文化的差异  浅谈文化与语言习得文化,语言,文化与语言的关系,文化与语言习得意识,跨文化交际      0.5131710171699524
```



### 开始预测

以上述 demo 数据为例，运行如下命令基于我们开源的 ERNIE-Gram模型开始计算文本 Pair 的语义相似度:

```shell
python -u -m paddle.distributed.launch --gpus "0" \
        predict_pairwise.py \
        --device gpu \
        --params_path "./checkpoints/model_30000/model_state.pdparams"\
        --batch_size 128 \
        --max_seq_length 64 \
        --input_file 'sort/test_pairwise.csv'
```
也可以直接执行下面的命令：

```
sh scripts/predict_pairwise.sh
```
得到下面的输出，分别是query，title和对应的预测概率：

```
{'query': '中西方语言与文化的差异', 'title': '第二语言习得的一大障碍就是文化差异。', 'pred_prob': 0.85112214}
{'query': '中西方语言与文化的差异', 'title': '跨文化视角下中国文化对外传播路径琐谈跨文化,中国文化,传播,翻译', 'pred_prob': 0.78629625}
{'query': '中西方语言与文化的差异', 'title': '从中西方民族文化心理的差异看英汉翻译语言,文化,民族文化心理,思维方式,翻译', 'pred_prob': 0.91767526}
{'query': '中西方语言与文化的差异', 'title': '中英文化差异对翻译的影响中英文化,差异,翻译的影响', 'pred_prob': 0.8601749}
{'query': '中西方语言与文化的差异', 'title': '浅谈文化与语言习得文化,语言,文化与语言的关系,文化与语言习得意识,跨文化交际', 'pred_prob': 0.8944413}
```

<a name="部署"></a>

## 8. 部署

### 动转静导出

首先把动态图模型转换为静态图：

```
python export_model.py --params_path checkpoints/model_30000/model_state.pdparams --output_path=./output
```
也可以运行下面的bash脚本：

```
sh scripts/export_model.sh
```

### Paddle Inference

修改预测文件路径：

```
input_file='../../sort/test_pairwise.csv'
```

然后使用PaddleInference

```
python predict.py --model_dir=../../output
```
也可以运行下面的bash脚本：

```
sh deploy.sh
```
得到下面的输出，输出的是样本的query，title以及对应的概率：

```
Data: {'query': '中西方语言与文化的差异', 'title': '第二语言习得的一大障碍就是文化差异。'}       prob: [0.8511221]
Data: {'query': '中西方语言与文化的差异', 'title': '跨文化视角下中国文化对外传播路径琐谈跨文化,中国文化,传播,翻译'}      prob: [0.7862964]
Data: {'query': '中西方语言与文化的差异', 'title': '从中西方民族文化心理的差异看英汉翻译语言,文化,民族文化心理,思维方式,翻译'}   prob: [0.91767514]
Data: {'query': '中西方语言与文化的差异', 'title': '中英文化差异对翻译的影响中英文化,差异,翻译的影响'}   prob: [0.8601747]
Data: {'query': '中西方语言与文化的差异', 'title': '浅谈文化与语言习得文化,语言,文化与语言的关系,文化与语言习得意识,跨文化交际'}     prob: [0.8944413]
```

## Reference

[1] Xiao, Dongling, Yu-Kun Li, Han Zhang, Yu Sun, Hao Tian, Hua Wu, and Haifeng Wang. “ERNIE-Gram: Pre-Training with Explicitly N-Gram Masked Language Modeling for Natural Language Understanding.” ArXiv:2010.12148 [Cs].
