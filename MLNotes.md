

1. 有监督学习


这种情况下，数据是带有标注的（点此进入scikit-learn有监督学习页面）。有监督学习可分为两类：

1.1 分类

样本属于两个或者更多的类，我们希望学习已经标注的数据，从而将未标注的数据分类。例如，手写数字识别，我们将每个输入向量分到某个类别。从另外一个角度，也可以把分类问题看作是一种离散式（相对于连续）的有监督学习。在此，分类的数量是有限的，对于每一个样本，我们都试着给他们标注正确的类别。

1.2 回归

如果我们想要的结果是由一个或者多个连续变量组成时，我们则称其为回归。例如，通过三文鱼的年龄和重量来推测它的长度。

2. 无监督学习

这种情况下，训练数据是一个集合，该集合由未标注目标值的向量X组成。此类问题的目标有几种。要么是要将数据划分为相似的几组，这种叫做聚类；要么是确定数据的分布，这种叫做密度估计；要么是将数据从高维度空间投射到二、三维空间里，这种叫做可视化。


####目标函数与优化


误差函数表达式

#####Regularization

Regularization make model simple and prediction stable


存在大量噪声特征时，L1正则化的LogisticRegression只需要与特征维度成对数比的训练样本，L2及其他旋转不变的模型需要线性比例的训练样本。

L1与L2的区别

考虑到拉普拉斯分布的特性，在对大数据进行LR分析的时候，很多参数会被直接优化成0。这就说明这些特征在整个学习过程中没有影响，降低了运算复杂度，同时也会提高模型效果。从这一点上来说，L1正则要比PCA降维效果要好。因为PCA的过程中会构造新的特征，这其实不利于我们反过来理解模型。而L1正则没有加入新特征，而是降低无关特征的权重，更加容易理解。


Practically, I think the biggest reasons for regularization are 1) to `avoid overfitting` by not generating high coefficients for predictors that are sparse.   2) to `stabilize the estimates` especially when there's collinearity共线性 in the data.  

1) is inherent in the regularization framework.  Since there are two forces pulling each other in the objective function, if there's no meaningful loss reduction, the increased penalty from the regularization term wouldn't improve the overall objective function. This is a great property since  a lot of noise would be automatically filtered out from the model.  


To give you an example for 2),  if you have two predictors that have same values, if you just run a regression algorithm on it since the data matrix is singular, your beta coefficients will be Inf if you try to do a straight matrix inversion. But if you  add a very small regularization lambda to it, you will get stable beta coefficients with the coefficient values evenly divided between the equivalent two variables. 


For the difference between L1 and L2, the following graph demonstrates why people bother to have L1 since L2 has such an elegant analytical solution and is so computationally straightforward. Regularized regression can also be represented as a constrained regression problem (since they are Lagrangian equivalent). In Graph (a), the black square represents the feasible region of of the L1 regularization while graph (b) represents the feasible region for L2 regularization. The contours in the plots represent different loss values (for the unconstrained regression model ). The feasible point that minimizes the loss is more likely to happen on the coordinates on graph (a) than on graph (b) since graph (a) is more angular.  This effect amplifies when your number of coefficients increases, i.e. from 2 to 200. 




The implication of this is that the L1 regularization gives you sparse estimates. Namely, in a high dimensional space, you got mostly zeros and a small number of non-zero coefficients. This is huge since it incorporates variable selection to the modeling problem. In addition, if you have to score a large sample with your model, you can have a lot of computational savings since you don't have to compute features(predictors) whose coefficient is 0. I personally think L1 regularization is one of the most beautiful things in machine learning and convex optimization. It is indeed widely used in bioinformatics and large scale machine learning for companies like Facebook, Yahoo, Google and Microsoft.

As per Andrew Ng's Feature selection, l1 vs l2 regularization, and rotational invariance paper, expect l1 regularization be better than l2 regularization if you have a lot less examples than features.

Conversely, if your features are generated from something like PCA, SVD, or any other model that assumes rotational invariance, or you have enough examples, l2 regularization is expected to do better because it is directly related to minimizing the VC dimension of the learned classifier, while l1 regularization doesn't have this property.

OS course, you can always use the elastic net regularization (which is a sum of the l1 and l2 regularizers) and tune the parameters to get the best of both worlds. Scikits.learn has a good and efficient implementation of elastic net and grid search to tune the hyperparameters.

**Reference**

[Sparsity and Some Basics of L1 Regularization](http://freemind.pluskid.org/machine-learning/sparsity-and-some-basics-of-l1-regularization/)

####生成模型和判别模型的区别

**Reference**

[Discriminative Modeling vs Generative Modeling](http://freemind.pluskid.org/machine-learning/discriminative-modeling-vs-generative-modeling/)

[判别式模型与生成式模型](http://www.leexiang.com/discriminative-model-and-generative-model)

判别模型包括

	Logistic Regression
	SVM	
	Traditional Neural Networks	
	Nearest Neighbor	
	CRF	
	Linear Discriminant Analysis	
	Boosting	
	Linear Regression


生成模型包括

	Gaussians, Naive Bayes, Mixtures of Multinomials
	Mixtures of Gaussians, Mixtures of Experts, HMMs
	Sigmoidal Belief Networks, Bayesian Networks
	Markov Random Fields
	Latent Dirichlet Allocation

成功应用情况
Generative Model的应用
 NLP
– Traditional rule-based or Boolean logic systems
Dialog and Lexis-Nexis) are giving way to statistical
approaches (Markov models and stochastic context
grammars)
 Medical Diagnosis
– QMR knowledge base, initially a heuristic expert
systems for reasoning about diseases and symptoms
been augmented with decision theoretic formulation
 Genomics and Bioinformatics
– Sequences represented as generative HMMs

Discriminative Model的应用：
 Image and document classification
 Biosequence analysis
 Time series prediction

Discriminative Model缺点：
? Lack elegance of generative
– Priors, structure, uncertainty
? Alternative notions of penalty functions,
regularization, kernel functions
? Feel like black-boxes
– Relationships between variables are not explicit
and visualizable

Bridging Generative and Discriminative：
? Can performance of SVMs be combined
elegantly with flexible Bayesian statistics?
? Maximum Entropy Discrimination marries
both methods
– Solve over a distribution of parameters (a
distribution over solutions)

####分类器

#####Logistic Regression

######核函数

sigmoid函数:

	g(x)=e^x/(1+e^x)
	值域(0,1)

hyperbolic tangent函数 

	tanh(x)=(e^x-e^(-x))/(e^x+e^(-x))
	值域(-1,1)，因此常用于神经网络

* a rescaling of the logistic sigmoid
* Often performs better than the logistic function because of its symmetry. Ideal for customization of multilayer perceptrons, particularly the hidden layers.

The reason behind using the sigmoid function is that it is derived from probability and maximum likelihood. While the other functions may work very similarly, they will lack this probabilistic theory background. For details see for example http://luna.cas.usf.edu/~mbrannic/files/regression/Logistic.html or http://www.cs.cmu.edu/~tom/mlbook/NBayesLogReg.pdf

关于为何选择sigmoid函数而不选择tanh函数是因为是源于概率论和最大似然估计，而其他函数虽然效果类似，但是没有这个背景。按我的理解，事件发生的几率是由logit函数定义的，而logit函数推导可得到事件发生的几率的对数正好等于w*x。细节参考：
[http://luna.cas.usf.edu/~mbrannic/files/regression/Logistic.html](http://luna.cas.usf.edu/~mbrannic/files/regression/Logistic.html)
[http://www.cs.cmu.edu/~tom/mlbook/NBayesLogReg.pdf](http://www.cs.cmu.edu/~tom/mlbook/NBayesLogReg.pdf)

#####损失函数

* 0-1 损失函数
* 平方损失函数
* 绝对损失函数
* 对数损失函数

逻辑回归的对数损失函数正好是最大似然函数的负值，求损失函数的最小值即求最大似然函数的最大值

#####多分类问题

one vs. all框架
对每一类训练一个二分类器，取N个分类器输出概率最大的那个作为预测值

######如何处理缺失值

1. 使用均值代替
2. 使用特殊值如-1
3. 忽略该数据
4. 使用相似item的平均值
5. 使用机器学习算法来预测此值

C4.5决策树中缺失值的处理

在数据挖掘， 机器学习问题中， 很多时候会存在数据缺失，例如user profile构建数据中，用户性别，或是年龄，性别的数据缺失（比如用户没有填），一种选择是丢弃该类数据。但很多时候这类数据很多， 直接丢弃不能接受，所以模型需要有处理缺失值的能力， 这样才能发现更多模式。一般考虑缺失值的时候，需要考虑下边几个问题【2】：

2个缺失值数量不一样的特征，选择哪个？
选定拆分特征后，训练数据中有缺失值的instance归到哪个拆分分支？
在分类的时候，instance的未知值如何处理？
我们从information的定义出发， 在计算某维特征A的info时， 仅考虑A有值的instance。在A特征上， 对值x进行测试，计算info及info_x时仅考虑有值的情况；计算split ratio的时候， 需要考虑未知值。以此计算出gain 或是 gain ratio决定选择哪个特征进行拆分，计算方式为：gain(x) = A有值的概率 * （info(T) – info_x(T)） + A没有值的概率 * 0

此处T为待测试的集合， A为待测试的属性， x为A上的测试点

在Quinlan c4.5实现版本中，gain的计算方式如下

屏幕快照 2014-09-21 下午1.33.13

其中128行ThisInfo为仅考虑known值的info， 134行对其进行更新，其中 1-UnknFrac 表示在本拆分节点中known节点占比， BaseInfo为待拆分节点的info， 之所以134行ThisInfo要除以Totaltems，和TotalInfo的实现有关

屏幕快照 2014-09-21 下午1.34.31

TotalInfo实现： 就是Info乘上TotalItems倍，具体推导就是将log(a/b)分解为log(a)-log(b)就可以。使用此方式就能够处理未知特征选择的问题（缺失值问题1）

在c4.5实现的过程中， 在处理缺失值问题2时，使用了一个诀窍解决按照分布将一个instance归到各子节点。c4.5实现时， 每个instance都是按照概率出现在某个分支中， 如果该instance没有特征缺失， 则在进行节点拆分的时候， 该节点只能落到唯一确定的分支中， 但如果instance 在待拆分的特征上值缺失， 则该instance会按照每个类别在known值在该节点出现的概率（weight），加入子分支。此处的问题是： 难道需要为每个有未知值的instance在每个分支均设置一个weight？ 那空间复杂度是无法忍受的。 c4.5实现版本巧妙地使用递归解决了该问题：

屏幕快照 2014-09-21 下午1.55.09

注意此处的Weight[i]表示第i条instance在当前分支的概率，在进入下一个分支时，先将weight[i]计算为下一个分支需要的值，并进入递归。待该分支子树都建立后， 再于361行， 恢复weight[i]的值

在分类的时候 ，碰到未知特征分支时， 则需要计算所有子树特征，从weight累加计算出分布进行计算，时间复杂度和空间复杂度会急剧增加。所以最好的方式还是从特征的角度进行处理。


[Logistic Regression垃圾邮件分类](http://guangchun.wordpress.com/2012/06/19/logistic-regression/)

[http://blog.xlvector.net/2014-02/different-logistic-regression/](http://blog.xlvector.net/2014-02/different-logistic-regression/)
#####Random Forest(随机森林)

对多元公线性不敏感，结果对缺失数据和非平衡的数据比较稳健，可以很好地预测多达几千个解释变量的作用

######随机森林优点

随机森林是一个最近比较火的算法，它有很多的优点：

* 在数据集上表现良好，两个随机性的引入，使得随机森林不容易陷入过拟合

* 在当前的很多数据集上，相对其他算法有着很大的优势，两个随机性的引入，使得随机森林具有很好的抗噪声能力

* 它能够处理很高维度（feature很多）的数据，并且不用做特征选择，对数据集的适应能力强：既能处理离散型数据，也能处理连续型数据，数据集无需规范化

* 可生成一个Proximities=（pij）矩阵，用于度量样本之间的相似性： pij=aij/N, aij表示样本i和j出现在随机森林中同一个叶子结点的次数，N随机森林中树的颗数

* 在创建随机森林的时候，对generlization error使用的是无偏估计

* 训练速度快，可以得到变量重要性排序（两种：基于OOB误分率的增加量和基于分裂时的GINI下降量

* 在训练过程中，能够检测到feature间的互相影响

* 容易做成并行化方法

* 实现比较简单

######训练集的采样

**行采样**：即样本的采样

* 放回抽样(Sampling with replacement,Bootstrap Sampling)
* 不放回抽样(Sampling without replacement)

随机森林采用的是放回抽样，保证有重复样本被不同决策树分类；每一棵树的输入样本都不是全部的样本，使得相对不容易出现over-fitting。

**列采样**：即特征的采样，从M个feature中，选择m个（m << M）

######完全分裂

对采样之后的数据使用完全分裂的方式建立出决策树，这样决策树的某一个叶子节点要么是无法继续分裂的，要么里面的所有样本的都是指向的同一个分类。一般很多的决策树算法都一个重要的步骤——剪枝，但是这里不这样干，由于之前的两个随机采样的过程保证了随机性，所以就算不剪枝，也不会出现over-fitting。

######决策树的建立

* 信息增益

假如我们拥有M个类别标签

	C={C1,C2,C3....Cn}

并且拥有N个特征：

	T={T1,T2,T3....Tn}

那么对于某一个特征来说，加入特征项Ti是离散的

IG(C|Ti)=H(C)-H(C|Ti)

**熵**：

	E(s1,s2,......,sm)=sum(Pilog2(Pi))(i=1...m)

其中数据集为S,m为S的分类数目,Pi≈|Si/|S|，Ci为某分类标号，Pi为任意样本属于Ci的概率，Si为分类Ci上的样本数

熵E(s1,s2,……,sm)越小，s1,s2,……,sm就越有序（越纯），分类效果就越好。

由属性`A`划分为子集的熵：`A`为属性，具有`V`个不同的取值,S被A划分为V个子集s1,s2,……,sv，sij是子集sj中类Ci的样本数。E(A)= ∑(s1j+ ……+smj)/s * I(s1j,……,smj)

**条件熵**

**信息增益**：
	
	IG=E(S)-E(S|T)

选择划分属性

* 几种不同决策树

ID3使用信息增益，c4.5使用信息增益比，我在网上找到的说法是信息增益的缺点是比较偏向选择取值多的属性

一个属性的信息增益越大，表明属性对样本的熵减少的能力更强，这个属性使得数据由不确定性变成确定性的能力越强。
所以如果是取值更多的属性，更容易使得数据更“纯”（尤其是连续型数值），其信息增益更大，决策树会首先挑选这个属性作为树的顶点。结果训练出来的形状是一棵庞大且深度很浅的树，这样的划分是极为不合理的。

C4.5使用了信息增益率，在信息增益的基础上除了一项split information,来惩罚值更多的属性。

####GBDT(Gradient Boosting Decision Tree)

为什么facebook用GBDT来做feature变换

http://www.quora.com/Why-do-people-use-gradient-boosted-decision-trees-to-do-feature-transform

#####随机森林实例

[http://www.stat.berkeley.edu/~breiman/RandomForests/cc_home.htm#micro3](http://www.stat.berkeley.edu/~breiman/RandomForests/cc_home.htm#micro3)



####几种采样方式的对比

**Reference**

[http://blog.sina.com.cn/s/blog_49ea41a201018ctt.html](http://blog.sina.com.cn/s/blog_49ea41a201018ctt.html)
 
####如何选择分类器

你知道如何为你的分类问题选择合适的机器学习算法吗？当然，如果你真正关心准确率，那么最佳方法是测试各种不同的算法（同时还要确保对每个算法测试不同参数），然后通过交叉验证选择最好的一个。但是，如果你只是为你的问题寻找一个**“足够好”**的算法，或者一个起点，这里有一些我这些年发现的还不错的一般准则。

#####你的训练集有多大？

如果训练集很小，那么高偏差/低方差分类器（如朴素贝叶斯分类器）要优于低偏差/高方差分类器（如k近邻分类器），因为后者容易过拟合。然而，随着训练集的增大，低偏差/高方差分类器将开始胜出（它们具有较低的渐近误差），因为高偏差分类器不足以提供准确的模型。
你也可以认为这是生成模型与判别模型的区别。

#####一些特定算法的优点

1. 朴素贝叶斯的优点：

超级简单，你只是在做一串计算。如果朴素贝叶斯（NB）条件独立性假设成立，相比于逻辑回归这类的判别模型，朴素贝叶斯分类器将收敛得更快，所以你只需要较小的训练集。而且，即使NB假设不成立，朴素贝叶斯分类器在实践方面仍然表现很好。如果想得到简单快捷的执行效果，这将是个好的选择。它的主要缺点是，不能学习特征之间的相互作用（比如，它不能学习出：虽然你喜欢布拉德·皮特和汤姆·克鲁斯的电影，但却不喜欢他们一起合作的电影）。

2. 逻辑回归的优点：

有许多正则化模型的方法，你不需要像在朴素贝叶斯分类器中那样担心特征间的相互关联性。与决策树和支撑向量机不同，你还可以有一个很好的概率解释，并能容易地更新模型来吸收新数据（使用一个在线梯度下降方法）。如果你想要一个概率框架（比如，简单地调整分类阈值，说出什么时候是不太确定的，或者获得置信区间），或你期望未来接收更多想要快速并入模型中的训练数据，就选择逻辑回归。

3. 决策树的优点：

易于说明和解释（对某些人来说—我不确定自己是否属于这个阵营）。它们可以很容易地处理特征间的相互作用，并且是非参数化的，所以你不用担心异常值或者数据是否线性可分（比如，决策树可以很容易地某特征x的低端是类A，中间是类B，然后高端又是类A的情况）。一个缺点是，不支持在线学习，所以当有新样本时，你将不得不重建决策树。另一个缺点是，容易过拟合，但这也正是诸如随机森林（或提高树）之类的集成方法的切入点。另外，随机森林往往是很多分类问题的赢家（我相信通常略优于支持向量机），它们快速并且可扩展，同时你不须担心要像支持向量机那样调一堆参数，所以它们最近似乎相当受欢迎。

4. SVMs的优点：

高准确率，为过拟合提供了好的理论保证，并且即使你的数据在基础特征空间线性不可分，只要选定一个恰当的核函数，它们仍然能够取得很好的分类效果。它们在超高维空间是常态的文本分类问题中尤其受欢迎。然而，它们内存消耗大，难于解释，运行和调参也有些烦人，因此，我认为随机森林正渐渐开始偷走它的“王冠”。

#####然而…尽管如此

回忆一下，**更好的数据往往打败更好的算法**，设计好的特征大有裨益。并且，如果你有一个庞大数据集，这时你使用哪种分类算法在分类性能方面可能并不要紧（所以，要基于速度和易用性选择算法）。
重申我上面说的,如果你真的关心准确率,一定要尝试各种各样的分类器,并通过交叉验证选择最好的一个。或者，从Netflix Prize(和Middle Earth)中吸取教训,只使用了一个集成方法进行选择。

列表对比

|分类器|优点|缺点|并行|大数据|在线学习|
|:--:|:--:|:--:|:--:|:--:||
|Logistic Regression|||||Y|
|Decission Tree||||||||N|


####Scikit-learn的common practice

各种分类器的数据加载，模型建立,训练,交叉验证流程

[scikit-learn使用笔记与sign prediction简单小结](http://chentingpc.me/article/?id=1399)


####Ensemble methods

Bagging works well for unstable base models and can reduce variance in predictions. Boosting can be used with any type of model and can reduce variance and bias in predictions.

Tree Ensemble methods

* Very widely used, look for GBM, random forest…
Almost half of data mining competition are won by using some variants of tree ensemble methods

* Invariant to scaling of inputs, so you do not need to do careful features normalization.

* Learn higher order interaction between features.

* Can be scalable, and are used in Industry

#####Averaging methods

Examples: Bagging methods, Forests of randomized trees...

#####Boosting methods

Examples: AdaBoost, Gradient Tree Boosting, ..

######GBDT

http://homes.cs.washington.edu/~tqchen/pdf/BoostedTree.pdf

#####Boosting

给训练样本以不同的权重

• Multiple models developed in sequence by assigning 
higher weights (boosting) for those training cases that 
are difficult to classify
流程：

生成第一个模型

循环:
	对错分样本更高的权重
	生成新的模型

根据模型精确度给各个模型一个权重,线性加权各个模型的分类结果得到最终结果

Generate the first model
Repeat
Weight the training data such that the misclassified cases get 
higher weights
Generate the next model
Combine predictions from individual models (weighted by 
accuracy of the models) 

Adaboost

#####Bagging

各模型的训练数据是抽样得到的，抽样策略

得到结果的两种方法:Average predictions和majority voting

适用于不同的抽样数据集训练得到的分类器的分类结果有极大差别的情况

• Combining predictions from multiple models
• Different models obtained from bootstrap samples of 
training data
• `Average predictions` or `majority voting` from multiple 
models
• If **different training datasets cause significant differences in learned model**, then bagging can improve accuracy.

优点:
• Advantage – improved accuracy

缺点：
模型不好解释
可以改进不稳定学习算法的性能，但有可能降低稳定算法的性能,双面刃
• Disadvantage – no simple, interpretable model
• Bagging can improve performance for unstable learning 
algorithms 
– Performance of stable learning methods can deteriorate
• Bagging good models can make them optimal
– Bagging poor models can make them worse




#####Ensemble

Stacking


adaboost decition tree random forest fpgrowth


####分布式机器学习算法实现

方便程度：Spark > Hadoop > MPI， 代码简洁度：Spark > Hadoop > MPI，限制程度：MapReduce > Spark > MPI，MPI是最自由的，写起来也是最麻烦的，所以任何分布式计算框架，都似乎是在抽象和性能之间进行折中。

######常见面试问题

如何对待标称型(类别、类目)特征?[stackoverflow](http://stats.stackexchange.com/questions/95212/improve-classification-with-many-categorical-variables)

* 使用dummy variables,例子：color:blue,purple,red,如果直接编码为1,2,3并建模y=a+bx.如果blue卖得比purple多，blue卖的比red多，由purple-blue销量得1b<2b，由blue-red销量得3b<2b,矛盾，正确的做法是使用3个布尔值color#purple, color#blue and color#red 见scikit-learn文档[http://www.astroml.org/sklearn_tutorial/general_concepts.html#handling-categorical-features](http://www.astroml.org/sklearn_tutorial/general_concepts.html#handling-categorical-features)


* 随机森林天然的可以用于categorical feature(不过scikit-learn目前不支持)，而不必为每个类编码导致内存不够，另外高维正交的特征容易over fitting，数据足够的话可能不会很显著。建议使用随机森林做特征选择，两种方法[http://www.stat.berkeley.edu/~breiman/RandomForests/cc_home.htm#micro3](http://www.stat.berkeley.edu/~breiman/RandomForests/cc_home.htm#micro3)



逻辑回归和决策树区别？适用范围？
logistic regression可以在线学习,决策树在有新数据输入时需要重新训练整个模型

spark和mapreduce区别？

1，Spark的中间数据放到内存中，对于迭代运算效率比较高。

2，Spark比Hadoop更通用。

Spark提供的数据集操作类型有很多种，不像Hadoop只提供了Map和Reduce两种操作。比如map, filter, flatMap,sample, groupByKey, reduceByKey, union, join, cogroup, mapValues, sort,partionBy等多种操作类型，他们把这些操作称为Transformations。同时还提供Count, collect, reduce, lookup, save等多种actions。

3，容错性。

从Spark的论文《Resilient Distributed Datasets: AFault-Tolerant Abstraction for In-Memory Cluster Computing》中没看出容错性做的有多好。倒是提到了分布式数据集计算，做checkpoint的两种方式，一个是checkpoint data，一个是logging the updates。貌似Spark采用了后者。但是文中后来又提到，虽然后者看似节省存储空间。但是由于数据处理模型是类似DAG的操作过程，由于图中的某个节点出错，由于lineage chains的依赖复杂性，可能会引起全部计算节点的重新计算，这样成本也不低。他们后来说，是存数据，还是存更新日志，做checkpoint还是由用户说了算吧。相当于什么都没说，又把这个皮球踢给了用户。所以我看就是由用户根据业务类型，衡量是存储数据IO和磁盘空间的代价和重新计算的代价，选择代价较小的一种策略。


adaboost原理？
java设计模式？
层次聚类？
预测薪水的特征提取？
svm简述 适合于什么样的数据？
关联规则挖掘 frequent pattern mining
广告和


决策树或随机森林和逻辑回归、SVM等方法的本质区别?

自变量和因变量的关系，决策树的自变量可以使不连续的


**CTR预估**
Logistic Regression

Stochastic Gradient Descent

训练

Batch

所有

Online Learning

小部分

Learning Rate

一般学习率一开始设置得比较高,然后随迭代逐渐减小,Batch中是下一轮迭代减小,Online 是每一个点都减小

One way to adjust the learning rate is to have a constant divide by the square root of N (where N is the number of data point seen so far).

 ɳ = ɳ_initial / (t ^ 0.5).

The learning rate can be adjusted as well to achieve a better stability in convergence.  In general, the learning rate is higher initially and decrease over the iteration of training (in batch learning it decreases in next round, in online learning it decreases at every data point).  


[http://cs229.stanford.edu/notes/cs229-notes1.pdf](http://cs229.stanford.edu/notes/cs229-notes1.pdf)

[sgd推导文档(cmu)](http://www.cs.cmu.edu/~wcohen/10-605/assignments/sgd.pdf)

**点击反作弊**

simjoin

广告id1: ip1 ip2 ip3 
广告id2: ipx ipx ipx ...

众多IP访问大量id的作弊情况,访问量大且相似度高的item对多的话很可疑，可算某种形式的聚类

**神经网络模型**

**决策树中如何减枝**

决策树为什么(WHY)要剪枝？原因是避免决策树过拟合(Overfitting)样本。

PrePrune：预剪枝，及早的停止树增长，方法可以参考见上面树停止增长的方法。
PostPrune：后剪枝，在已生成过拟合决策树上进行剪枝，可以得到简化版的剪枝决策树。
其实剪枝的准则是如何确定决策树的规模，可以参考的剪枝思路有以下几个：
1：使用训练集合(Training Set）和验证集合(Validation Set)，来评估剪枝方法在修剪结点上的效用
2：使用所有的训练集合进行训练，但是用统计测试来估计修剪特定结点是否会改善训练集合外的数据的评估性能，如使用Chi-Square（Quinlan，1986）测试来进一步扩展结点是否能改善整个分类数据的性能，还是仅仅改善了当前训练集合数据上的性能。
3：使用明确的标准来衡量训练样例和决策树的复杂度，当编码长度最小时，停止树增长，如MDL(Minimum Description Length)准则。

后剪枝

Reduced-Error Pruning(REP,错误率降低剪枝）

Pessimistic Error Pruning(PEP，悲观剪枝）

canopy clustering


######Hierarchical tag visualization and application for tag recommendations citation 7

关联分析可以解决一词多义的问题,挖掘到的关系更符合视频网站的特点,而非知识百科

父子关系的预测:

Co(ti, tj ) 标签ti,tj打同一文档的次数
D(ti|¬tj )  标签ti出现在某一文档而tj没有出现的次数
对任意tag,有D(ti) = Co(ti, tj) + D(ti|¬tj ).

Co(programming, java) = 200

D(java|¬programming)= 29

D(programming|¬java) = 239

文氏图

programming比java更抽象,java应该属于其子类

逻辑回归预测父子关系

for each i, j 2 T , i < j
(xk, yk) = 

({ti − tj},+1) if ti >r tj
({ti − tj},−1) if ti <r tj
; otherwise
>r is true when D(ti|¬tj )/D(tj |¬ti) > ,
<r is true when D(ti|¬tj )/D(tj |¬ti) < 1/.

每个标签的feature是

	H(t)=熵 H(t) = -Sum_from1toN(p(ti)*log p(ti)), where the summation is over all N topics that the tag t belongs to, and p(ti) the probability that t appears in the ith topic
	C(t)=总出现数
	D(t)=distinct出现数

D(ti|¬tj )/D(tj |¬ti)的经验阈值是2,记得用spark写预测程序的时候阈值和其paper上的不一样,比这个大很多,因为数据量大很多

tf-idf过滤

#####Bag-of-words model

常用于自然语言处理和信息检索，也用于计算机视觉中

对所有出现的词编码，生成一个词典，词->编号,然后每句话就编码为一个词典内词个数维度的向量，每一维度为词在文档中出现的次数(也可以用tf-idf)

应用：反垃圾信息

[Bag-of-words model](http://en.wikipedia.org/wiki/Bag-of-words_model)

#####聚类

[http://www.cnblogs.com/joyeecheung/p/3442553.html](http://www.cnblogs.com/joyeecheung/p/3442553.html)

DBSCAN

[http://www.tuicool.com/articles/AjQbaa](http://www.tuicool.com/articles/AjQbaa)


#####机器学习博客列表

[http://fastml.com/](http://fastml.com/)

####

一些可以做的最基本的数据分析工作：

1. 特征的分布：按特征的取值分段，每一段包含的样本数量，特征均值，方差。
2. 目标分布同上
3. 特征目标关系：特征分段，每段中包含的样本的目标取值。
4. 目标特征关系：目标分段，每段中包含的样本的特征取值

模型在训练数据上效果不错，但做Cross-validation效果不佳主要原因有两个：

1. 选取的样本数据太少，覆盖度不够，考虑增加训练样本
2. 样本特征过多，可以考虑减少一些特征，只留下重要的特征

模型在类似Cross-validation这样的封闭测试上效果不错，但在开放测试上效果不佳

1. 选取的训练数据覆盖度不够，不具备代表性，不能体现真实数据的分布。
2. 模型迁移（Model drift），随着时间变化，特征数据也随之变化。比如3个月前做的模型对现在的特征可能不会有好的效果。


###评估方法

离线评估为何更倾向采用logloss，而不是采用AUC值。Facebook在他们发布的论文【1】中提到现实环境中更加关注预测的准确性，而不是相对的排序。而AUC值更侧重相对排序，比如把整体的预测概率提升1倍，AUC值保持不变，但是logloss是有变化的。


#####实际问题解决

给定以下格式数据，6,7月几千万条:
userId timestamp 对A品牌的点击数 对A品牌的收藏数目

预测user会不会购买

主要方法:

* 时间序列，购买是时间的模型，进行数学建模，预测概率，按阈值判断
* 逻辑回归，决策树
* 对商品聚类
* 建立用户的价格偏好

####什么是模式识别


###特征选择

去除冗余，无关
redundant,irrelevant

优点:

* efficiency 

更简单的模型和更短的预测时间

* generalization

去除了噪声

* interpretability

可解释性

缺点：

* computation

combinatorial optimization in training

* overfit

combinatorial selection

* mis-interpretability

####Feature Selection Algorithms

There are three general classes of feature selection algorithms: filter methods, wrapper methods and embedded methods.

**Filter Methods**

Filter feature selection methods apply a statistical measure to assign a scoring to each feature. The features are ranked by the score and either selected to be kept or removed from the dataset. The methods are often univariate and consider the feature independently, or with regard to the dependent variable.

Example of some filter methods include the Chi squared test, information gain and correlation coefficient scores.

**Wrapper Methods**

Wrapper methods consider the selection of a set of features as a search problem, where different combinations are prepared, evaluated and compared to other combinations. A predictive model us used to evaluate a combination of features and assign a score based on model accuracy.

The search process may be methodical such as a best-first search, it may stochastic such as a random hill-climbing algorithm, or it may use heuristics, like forward and backward passes to add and remove features.

An example if a wrapper method is the recursive feature elimination algorithm.

**Embedded Methods**

Embedded methods learn which features best contribute to the accuracy of the model while the model is being created. The most common type of embedded feature selection methods are regularization methods.

Regularization methods are also called penalization methods that introduce additional constraints into the optimization of a predictive algorithm (such as a regression algorithm) that bias the model toward lower complexity (less coefficients).

Examples of regularization algorithms are the LASSO, Elastic Net and Ridge Regression.

特征选择的方法大致可分为如下几类：

* 投影法：

求出最优的投影向量w，绝对值较大的分量对应的特征即所选特征。求解w的方法很多，像LDA，linear SVM，Lasso regression，Sparse coding等都是适用的方法。

* Wrapper：

使得特征子集上的分类错误率最小。

* Filter：

对单个特征根据特定准则进行排序（如熵增益，分类错误率等），再选出其中排序较高、且、且互补性较强的构成特征子集。

* 混合型

先用filter滤除候选特征，再用其它方法从中进行筛选。
针对2，3，4方法，可以用顺序前向/后向方法（Sequential Forward/Backward Search）进行特征子集挑选。

再补充一篇参考文献：I. Guyon et al., An introduction to variable and feature selection, JMLR, 2003



- 数据降维 (Data Reduction)

比如PCA这种降维方法最后的主成分（一般两个）很难具体的去解释，因为杂糅了诸多个自变量的主成分没有办法直接获得直观意义的诠释。相比之下，如果可以通过一系列尽管媲美黑魔法但的确有逻辑解释的计算方法，即特征选择，撇去某一些无关紧要的自变量，那么恭喜你，你又迈出了刷算法当屌丝的一步。关于特征选择，这里有几个我读过的相关资料：
    10. [Data Mining Algorithms In R/Dimensionality Reduction/Feature Selection](http://en.wikibooks.org/wiki/Data_Mining_Algorithms_In_R/Dimensionality_Reduction/Feature_Selection)有代码有实战
    20. An Introduction to Feature Selection [不错的介绍](http://machinelearningmastery.com/an-introduction-to-feature-selection/)
    30. Statistical Learning, Chapter 6. 系统介绍了 Subset Selection, Shrinkage (或叫做regularization，比如常用的LASSO), Dimension Reduction. 
    40. Feature selection: Using the caret package 一篇2010年的[老文](http://www.r-bloggers.com/feature-selection-using-the-caret-package/)

- 混杂（confounding）。大白话就是自变量彼此之间有一定关联，比如miRNA彼此。这种相爱相杀的关联真的让你分析时醉生梦死。文中提到Surrogate Variable Analysis可以解决。我也看过一些相关的读物：
    10. Bioconductor上的SVA包，Jef Leek（就是写博客开R公开课的三剑客之一）领衔编写
    20. [Identifying Correlated Predictors using Caret](http://topepo.github.io/caret/preprocess.html)
    30. [Feature Selection with the Caret R Package](http://machinelearningmastery.com/feature-selection-with-the-caret-r-package/)