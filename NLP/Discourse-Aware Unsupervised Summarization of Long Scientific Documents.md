- The first is that important information typically oc- curs at the start and end of sections;
  - 边界权重函数

（https://arxiv.org/abs/2005.00513）

- 代码：https://github.com/mirandrom/HipoRank.

论文阅读：Discourse-Aware Unsupervised Summarization of Long Scientific Documents（https://arxiv.org/abs/2005.00513）2021  已读内容：

- 本文提出了一种无监督的基于段落的长篇科技文章的提取方法。

- 通过构建有向层次图的方法，是基于图排序的摘要方法。

  1. 对文档按照sections划分，划分出各个部分。每个部分为*G* = (*V*, *E*),其中*V*为顶点（句子）集合，*E*为边界的集合。

     - Intra-sectional connections：在section内部建立连接图，主要目的是建立句子在该section中的局部重要性模型。 并认为：在统一topci/section下，当一个句子与更多的其中的句子相似时，该句子的重要程度更高。 为每个句子节点建立sentence-sentence 边界。
     - Inter-sectional connections：建模某一个句子相对于其他的topics/sections的重要程度。并认为：该句子若与更多的sections中的内容相似，则该句的重要程度越高。
       - 但是若对所有的句子进行重要性建模会对计算资源造成严重浪费、性能下降，因此需要减少边的数量。解决方法：该作者在sentence节点的上方增加section节点，通过计算section-sentence的边界来减少边的数量，并获得全局的重要度。真么做的理由是：在科学文章中，每一段往往都主题独立。

  2. 边界权重计算

     - 相似度计算方式：*cosine similarity*,实验中分别使用不同的向量来测试。

       - $ sim(v^I_j,v^I_i) $  ----------  sentence-sentence
       - $ sim(v^I,v^I_i) $  ----------  section-sentence

     - 节点与节点之间虽然有对称性，但是某一个节点可能相对于另一节点的重要性跟高。因此在section之间的连接上增加不对称边界来捕获这种不对称性。

       - 不对称边界加权：

         $d_b(v^I_i)=min(x^I_i,α(n^I - x_i^I))$

         $n^I$为sentences在section $I$中的数量，$x_i^I$表示句子$i$的位置 $α $为超参数，用于控制段头段位的重要性。