# Using clustering on breast-cancer dataset

文件说明：

- model_selection.ipynb - 模型筛选代码 & 结果
- cluster.ipynb - 聚类代码 & 结果
- breast.csv - 数据集

## 数据预处理

- 标签类别：良性编码为 2，恶性编码为 4，分别有 458、241 个
- 分离最后一列为 labels_true
- **数据标准化效果不理想，于是用 minmax 缩放到 [ 0, 1 ]**

## 模型筛选

 ***model_selection.ipynb*** | ***model_selection.html***

- **试验了如下模型**
  * k - means
  * Spectral 谱聚类
  * t - sne 非线性嵌入 + Spectral 谱聚类
  * t - sne 非线性嵌入 + 高斯混合模型
- **此外还对以下模型进行了初步筛选（基于 NMI ）**
  - MeanShift
  - DBSCAN
  - Ward 层次聚类
  - AffinityPropagation
  - AgglomerativeClustering
  - OPTICS
  - Birch
  - MiniBatchKMeans
  - SpectralClustering

- **选取最佳的四个模型进一步实验**
  - k - means
  - Spectral 谱聚类
  - Ward 层次聚类
  - Average 层次聚类
  
## 聚类过程

***cluster.ipynb | cluster.html***

- **对先验的反思**

初步尝试的实验结果表明大多数聚类器的结果并不是很好，NMI 只有 0.7 左右。由于聚类个数的选取是人为预先设置的，最初的想法考虑到真实标签只有两个类别，所以聚成两类。可能是这种先验知识带来的影响，于是尝试不同的聚类数。

- **聚类数寻优**

以聚类个数 2 ~ 5 个，在**平均距离层次聚类**模型上进行实验，使用 R 包 Nbclust 中提供的 23 个聚类评判准则得到的推荐个数，如下图所示：

| 聚类数 |  2   |  3   |  4   |  5   |
| :------: | :-----: | :-----: | :-----: | :-----: |
| 投票数 |  3   |  15  |  1   |  4   |

- **根据最佳聚类数 k = 3 进行划分**

1. 每个类中的样本数目：

|    聚类   |  1   |  2   |  3   |
| :------: | :-----: | :-----: | :-----: |
|  样本数目  | 455  | 207  |  37  |

2. 每个类中的 9 个特征的中位数：

|      |  x1  |  x2  |  x3  |  x4  |  x5  |  x6  |  x7  |  x8  | x9   |
| :-----: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
|  1   |  3   |  1   |  1   |  1   |  2   |  1   |  2   |  1   | 1    |
|  2   |  8   |  6   |  6   |  5   |  5   |  10  |  6   |  6   | 1    |
|  3   |  8   |  10  |  10  |  8   |  7   |  10  |  7   |  8   | 8    |

考虑到 breast 数据集中的特征为细胞大小、肿块厚度等信息，综合上表中一二三类的样本中位数逐渐变大，可以得出：**第一个类为良性样本、第二个类为中性样本、第三个类为恶性样本。**

- **合并聚类结果**

考虑到第三类样本数据量过少（只有 37 个），于是将第二类和第三类合并，即把中性样本合并到恶性样本类中，这与日常生活经验是不矛盾的。

## 实验结果总结 - 创新点

***cluster.ipynb | cluster.html***

本文在 breast - cancer 数据集上，通过聚类数寻优后发现最佳聚类数为 3 类，于是采用两种模式对比实验：

1. **直接聚成两类**
2. **先聚成三类，再将后两类合并**

通过 ***model_selection.ipynb*** | ***model_selection.html*** 筛选出四个性能较好的模型（见上文），分别对两种模式进行实验，将所有的聚类结果进行对比分析：

|             | Aver-2 | Aver-3 | Kmeans-2 | Kmeans-3 | Spectral-2 | Spectral-3 | Ward-2 | Ward-3 |
| :---------: | :----: | :----: | :------: | :------: | :--------: | :--------: | :----: | :----: |
|     NMI     | 0.1392 | 0.7501 |  0.7361  |  0.7789  |   0.8203   |   0.2200   | 0.7308 | 0.7308 |
|  Accuracy   | 70.82% | 96.14% |  95.85%  |  96.71%  |   97.28%   |   60.66%   | 95.71% | 95.71% |
| Wrong Count |  204   |   27   |    29    |    23    |     19     |    275     |   30   |   30   |

**Insights:**

- 对于平均距离层次聚类 NMI 指标提升显著（0.1392 –> 0.7501），准确率提升明显，错误数大幅减少，采用第二种模式的聚类优势明显。
- 对于 K - means 聚类，第二种模式对 NMI 指标有一定提升（0.7361 –> 0.7789），错误实例减少到 23  个。
- Spectral 谱聚类直接聚为两类就有很好的性能，对比第二种模式性能反而降低。
- ward 层次聚类则比较稳健，在两种模式下性能表现一致。

**其它尝试：*cluster.ipynb | cluster.html***

- 模型调参对聚类效果提升不太显著。
- 多个模型 voting 对聚类效果提升不太显著。
- 多个模型 stacking 对聚类效果提升不太显著。
- 提取聚类与标签不相符的样本，发现聚类器普遍出错，模型提升空间有待探索。

