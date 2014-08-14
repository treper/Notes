sklearn.feature_selection模块的作用是feature selection，而不是feature extraction。

Univariate feature selection：单变量的特征选择
单变量特征选择的原理是分别单独的计算每个变量的某个统计指标，根据该指标来判断哪些指标重要。剔除那些不重要的指标。

sklearn.feature_selection模块中主要有以下几个方法：
SelectKBest和SelectPercentile比较相似，前者选择排名排在前n个的变量，后者选择排名排在前n%的变量。而他们通过什么指标来给变量排名呢？这需要二外的指定。
对于regression问题，可以使用f_regression指标。对于classification问题，可以使用chi2或者f_classif变量。
使用的例子：
from sklearn.feature_selection import SelectPercentile, f_classif
selector = SelectPercentile(f_classif, percentile=10)

还有其他的几个方法，似乎是使用其他的统计指标来选择变量：using common univariate statistical tests for each feature: false positive rate SelectFpr, false discovery rate SelectFdr, or family wise error SelectFwe.

文档中说，如果是使用稀疏矩阵，只有chi2指标可用，其他的都必须转变成dense matrix。但是我实际使用中发现f_classif也是可以使用稀疏矩阵的。

Recursive feature elimination：循环特征选择
不单独的检验某个变量的价值，而是将其聚集在一起检验。它的基本思想是，对于一个数量为d的feature的集合，他的所有的子集的个数是2的d次方减1（包含空集）。指定一个外部的学习算法，比如SVM之类的。通过该算法计算所有子集的validation error。选择error最小的那个子集作为所挑选的特征。

这个算法相当的暴力啊。由以下两个方法实现：sklearn.feature_selection.RFE，sklearn.feature_selection.RFECV

L1-based feature selection：
该思路的原理是：在linear regression模型中，有的时候会得到sparse solution。意思是说很多变量前面的系数都等于0或者接近于0。这说明这些变量不重要，那么可以将这些变量去除。

Tree-based feature selection：决策树特征选择
基于决策树算法做出特征选择