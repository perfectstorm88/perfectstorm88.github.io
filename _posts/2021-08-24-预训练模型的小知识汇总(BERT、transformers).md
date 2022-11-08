

# 中文预训练模型(哈工大讯飞实验室)

## 中文BERT-wwm预训练模型
https://github.com/ymcui/Chinese-BERT-wwm  最常用，最早2019年6月发布，目前仍在维护阶段

基于全词遮罩（Whole Word Masking）技术的中文预训练模型BERT-wwm，以及与此技术密切相关的模型：BERT-wwm-ext，RoBERTa-wwm-ext，RoBERTa-wwm-ext-large, RBT3, RBTL3。

## 中文ELECTRA预训练模型
https://github.com/ymcui/Chinese-ELECTRA，2020年3月发布，基于谷歌与斯坦福大学的ELECTRA

特点是小，ELECTRA-small模型可与BERT-base甚至其他同等规模的模型相媲美，而参数量仅为BERT-base的1/10

预先训练文本编码器作为鉴别器而不是生成器，ELECTRA提出了一套新的预训练框架，其中包括两个部分：Generator和Discriminator。

- Generator: 一个小的MLM，在[MASK]的位置预测原来的词。Generator将用来把输入文本做部分词的替换。
- Discriminator: 判断输入句子中的每个词是否被替换，即使用Replaced Token Detection (RTD)预训练任务，取代了BERT原始的Masked Language Model (MLM)。需要注意的是这里并没有使用Next Sentence Prediction (NSP)任务。
- 在预训练阶段结束之后，我们只使用Discriminator作为下游任务精调的基模型。


## 中文MacBERT预训练模型：
https://github.com/ymcui/MacBERT ，2020年9月15

MacBERT是一种改进的BERT，它以新的MLM方式作为校正(MLM as Corretion)预训练任务，缓解了预训练和微调的差异。
在MLM过程中使用相似词作为掩码，而不是[MASK]。相似词的获取使用了Synonyms库

根据官方自己的评测指标看，MacBERT-base比BERT、BERT-wwm、ELECTRA效果要好

## 中文XLNet预训练模型：
https://github.com/ymcui/Chinese-XLNet，于2019年8月发布，基于CMU/谷歌官方的XLNet，已经不在维护


# 模型压缩(知识蒸馏工具)
TextBrewer（文本酿酒师）：https://github.com/airaria/TextBrewer 

TextBrewer是一个基于PyTorch的、为实现NLP中的知识蒸馏任务而设计的工具包， 融合并改进了NLP和CV中的多种知识蒸馏技术，提供便捷快速的知识蒸馏框架， 用于以较低的性能损失压缩神经网络模型的大小，提升模型的推理速度，减少内存占用。

现在的transformer上的预训练比较全，在业务场景上，相对使用的较少



# transformers（NLP界的github）
https://github.com/huggingface/transformers

- Transformers 提供了数以千计的预训练模型，支持 100 多种语言的文本分类、信息抽取、问答、摘要、翻译、文本生成。它的宗旨让最先进的 NLP 技术人人易用。

- Transformers 提供了便于快速下载和使用的API，让你可以把预训练模型用在给定文本、在你的数据集上微调然后通过 model hub 与社区共享。同时，每个定义的 Python 模块均完全独立，方便修改和快速研究实验。
- Transformers 支持三个最热门的深度学习库： Jax, PyTorch and TensorFlow — 并与之无缝整合。你可以直接使用一个框架训练你的模型然后用另一个加载和推理。

transformers的介绍参见：
https://github.com/huggingface/transformers/blob/master/README_zh-hans.md

微调的例子：https://huggingface.co/transformers/training.html



## 手工下载transformers上的库

### 首先安装git lfs 
参见:https://github.com/git-lfs/git-lfs/wiki/Installation

```bash
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
git lfs install
```

### 以bert-base-chinese为例进行下载

```bash
git clone https://huggingface.co/bert-base-chinese
# if you want to clone without large files – just their pointers
# prepend your git clone with the following env var:
GIT_LFS_SKIP_SMUDGE=1  # 是否跳过大文件下载
```