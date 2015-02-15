###Kaggle Notes
####Algorithmic Trading Challenge



####Kaggle Criteo Notes

训练数据特征包括13维的整数值特征和26维的类别特征

步骤：

1.处理测试集，添加lable，便于后面得出预测值直接替换

处理前

	vagrant@vagrant-VirtualBox:~/software/kaggle-2014-criteo$ cat test.csv |head -10
	Id,I1,I2,I3,I4,I5,I6,I7,I8,I9,I10,I11,I12,I13,C1,C2,C3,C4,C5,C6,C7,C8,C9,C10,C11,C12,C13,C14,C15,C16,C17,C18,C19,C20,C21,C22,C23,C24,C25,C26
	60000000,,29,50,5,7260,437,1,4,14,,1,0,6,5a9ed9b0,a0e12995,a1e14474,08a40877,25c83c98,,964d1fdd,5b392875,a73ee510,de89c3d2,59cd5ae7,8d98db20,8b216f7b,1adce6ef,78c64a1d,3ecdadf7,3486227d,1616f155,21ddcdc9,5840adea,2c277e62,,423fab69,54c91918,9b3e8820,e75c9ae9
	60000001,27,17,45,28,2,28,27,29,28,1,1,,23,68fd1e64,960c983b,9fbfbfd5,38c11726,25c83c98,7e0ccccf,fe06fd10,062b5529,a73ee510,ca53fc84,67360210,895d8bbb,4f8e2224,f862f261,b4cc2435,4c0041e5,e5ba7672,b4abdd09,21ddcdc9,5840adea,36a7ab86,,32c7478e,85e4d73f,010f6491,ee63dd9b

处理后

	vagrant@vagrant-VirtualBox:~/software/kaggle-2014-criteo$ cat te.csv |head -10
	Id,Label,I1,I2,I3,I4,I5,I6,I7,I8,I9,I10,I11,I12,I13,C1,C2,C3,C4,C5,C6,C7,C8,C9,C10,C11,C12,C13,C14,C15,C16,C17,C18,C19,C20,C21,C22,C23,C24,C25,C26
	60000000,0,,29,50,5,7260,437,1,4,14,,1,0,6,5a9ed9b0,a0e12995,a1e14474,08a40877,25c83c98,,964d1fdd,5b392875,a73ee510,de89c3d2,59cd5ae7,8d98db20,8b216f7b,1adce6ef,78c64a1d,3ecdadf7,3486227d,1616f155,21ddcdc9,5840adea,2c277e62,,423fab69,54c91918,9b3e8820,e75c9ae9
	60000001,0,27,17,45,28,2,28,27,29,28,1,1,,23,68fd1e64,960c983b,9fbfbfd5,38c11726,25c83c98,7e0ccccf,fe06fd10,062b5529,a73ee510,ca53fc84,67360210,895d8bbb,4f8e2224,f862f261,b4cc2435,4c0041e5,e5ba7672,b4abdd09,21ddcdc9,5840adea,36a7ab86,,32c7478e,85e4d73f,010f6491,ee63dd9b


2. 处理train.csv数据集，count类别特征(count.py)，得到`field,value,neg,pos,total`,按total升序排序得到`Field,Value,Neg,Pos,Total,Ratio`,例如：

	Field,Value,Neg,Pos,Total,Ratio
	C15,943169c2,9,1,10,0.1
	C4,41274cd7,8,2,10,0.2
	C13,7301027a,7,3,10,0.3
	C13,00e20e7b,8,2,10,0.2
	C24,ded4aac9,7,3,10,0.3
	C7,fe4dce68,6,4,10,0.4

	有些`Field`的`Value`可能是空的;出现低于4,000,000次的特征被剔除,存到tr.gbdt.sparse.__tmp__.0文件中;数值特征全部保留存到tr.gbdt.dense.__tmp__.0文件 pre-a.py


		vagrant@vagrant-VirtualBox:~/software/kaggle-2014-criteo$ cat tr.gbdt.dense.__tmp__.0|head -10
		0 1 1 5 0 1382 4 15 2 181 1 2 -10 2
		0 2 0 44 1 102 8 2 2 4 1 1 -10 4
		0 2 0 1 14 767 89 4 2 245 1 3 3 45
		0 -10 893 -10 -10 4392 -10 0 0 0 -10 0 -10 -10
		0 3 -1 -10 0 2 0 3 0 0 1 1 -10 0
		0 -10 -1 -10 -10 12824 -10 0 0 6 -10 0 -10 -10
		0 -10 1 2 -10 3168 -10 0 1 2 -10 0 -10 -10
		1 1 4 2 0 0 0 1 0 0 1 1 -10 0
		0 -10 44 4 8 19010 249 28 31 141 -10 1 -10 8
		0 -10 35 -10 1 33737 21 1 2 3 -10 1 -10 1

格式Label 13维数值特征

		vagrant@vagrant-VirtualBox:~/software/kaggle-2014-criteo$ cat  tr.gbdt.sparse.__tmp__.0|head -10
		0 1 2 3 6 8 12 13 17 25
		0 1 2 7 8 12 14 15 20 25
		0 1 4 6 9 10 12 19
		0 1 2 4 9 12 15
		0 1 2 4 5 17
		0 1 4 9 10
		0 1 2 4 9 10 15
		1 1 2 3 4 5 9
		0 1 2 3 4 5 6 7
		0 1 2 4 5 9 21 23

格式Label 超过4百万的特征的编号(降序？编号越小出现越多？)

同样逻辑处理测试集合test.csv


2. 用GBDT生成特征

输入为dense和sparse数据，训练30棵树，深度为7

	vagrant@vagrant-VirtualBox:~/software/kaggle-2014-criteo$ cat tr.gbdt.out |head -2
	-1 147 232 226 142 227 228 194 202 196 248 193 147 159 248 213 135 136 150 143 132 152 129 184 216 136 224 128 236 233 247
	-1 128 209 192 131 208 131 243 128 196 161 200 147 128 196 128 128 134 128 133 128 154 151 173 202 136 224 128 228 165 132


3. Pre-b.py


共有三组特征
数值特征，类别特征，GBDT特征

Numerical features:log(v)^2
Categorical features:greater >10
GBDT:directly includ

13+26+30=69维度

4. 生成Factorization Machine的特征
4. Hash trick
5. 预测




一起读论文吧，title：Practical Lessons from Predicting Clicks on Ads at Facebook
本问给出了一种模型融合方法的模型，即：GBDT+LR，进行CTR预估。并在此基础上研究了一些基本参数（例如：SGD的学习率、树的棵树与树的深度等）是如何影响最终模型的性能的，并最终表明一旦获取正确的特征和正确的模型（GBDT+LR）其他的因素对最终的预测起到非常小的作用。
 Practical_Lessons_from_Predicting_Clicks_on_Ads_at_Facebook.pdf
 2  分享 2015-01-28
July  mathdong
11 个评论
July
July
@mathdong
2015-01-28 13:00
mathdong
mathdong
这个我已经看过，研究了，是利用GBDT提取出数据的特征，然后利用GBDT提取出的特征与原始特征结合起来作为新的特征，然后利用logistic regression来预测ctr，据别人实验，效果不错，目前自己还在实验
2015-01-28 14:16
bamboo
bamboo 回复 mathdong
前些日子，按着这篇论文的思路实现一个GBDT+FM效果相当好，所以打算仔细的研究下这篇论文
2015-01-28 14:39
mathdong
mathdong 回复 bamboo
你用GBDT提取出数据的特征，然后再利用FM进行分类或回归？我前段时间单独跑了下FM，不过没有用GBDT提取特征，请问一下，你用GBDT提取特征是怎么来提取的啊？
2015-01-28 14:54
mathdong
mathdong 回复 bamboo
能不能好好聊聊，这个，我加你
2015-01-28 14:55
lucasyang
lucasyang
一直有个问题，什么时候用rf，什么时候GBDT？
2015-01-28 15:21
bamboo
bamboo 回复 mathdong
其实说白了，就是在用GBDT做特征离散化，变成0、1形式的布尔特征，再利用FM进行特征组合，这样就避免了很多人工的操作
2015-01-28 15:24
bamboo
bamboo 回复 lucasyang
我也不是太清楚什么时候选这两个中的哪一个，感觉差不多，但是rf可以并行创建，GBDT只能串行的创建，rf创建速度肯定比GBDT快，而且，rf不容易过拟合。rf采用投票的形式，GBDT则在残差的基础上继续学习，感觉GBDT更具有针对性。
2015-01-28 17:24
jiqihuman
jiqihuman 回复 mathdong
刚看了下abstract, cool!, 
请教一下，这个实验对机器有要求不？数据量怎么样？单机能跑不？
2015-01-29 16:11
mathdong
mathdong 回复 jiqihuman
数据量这个太大的话，肯定还是对内存有要求啊，这个我是在服务器上跑的，内存大点。
2015-01-29 18:45
jiqihuman
jiqihuman 回复 mathdong
非常感谢