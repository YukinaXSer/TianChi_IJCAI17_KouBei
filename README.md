# 2017天池口碑商家客流量预测
队伍名：卡文尼尔<br>
第一赛季排名：191/4058<br>
第二赛季排名：168/4058<br>
成绩：0.0824<br>
<br>
   这次比赛是抱着学习做特征工程的心态来做的，所以整体思路是模型加少许规则。<br>
<br>
   一路做下来发现对于时间序列的预测，最有效果的特征还是它的历史数据。model1将星期与假期哑变量作为特征，前三周为训练集，分别对每个商家用Lasso来预测，成绩可以到0.0847。但若是对每个商家分别建模，样本量太少，容易过拟合。因此改用整体建模的方式。<br>

## 数据预处理<br>
   对缺失值进行填充。将客流量数据以星期为列，重排为数据框，将有零的行去掉后取后三周的平均作为填充值。也就是说填充值是一个字典{周一均值，周二均值，...}。这样对预测一些近期没营业数据，在预测区间内却正常营业的商家时，有很好的效果。<br>
对浏览数据进行同样处理。

## 特征工程<br>
   取前三周作为训练集，预测一周，剩下的一周与前一周一致。理论上把预测的第一周也作为训练集重新预测第二周，是使残差平方和最小的估计。但实际情况却比不上复制。因此取两周预测相同。<br>
主要提取四方面特征：
* 商家特征。包括自身属性特征与商家客流量、浏览量最近三周的统计特征。
* 时间特征。包括星期与假期特征，假期分为三类:工作日，小假期，大假期。考虑到训练集均在九月以后，就没有考虑寒暑假。
* 天气特征。日最高温，日最低温，日前期天气状况，后期天气状况。共四维。天气状况分为四阶段进行因子化：0-优（如：晴，多云)；１-良（如：阴，雾，霾）；２-差（如：小雨，小雪）；３-恶劣（如：中雨，中雪，大雨，大雪，雷阵雨，雨夹雪）。如：晴转阴->(0,1)
* 商家客流量与浏览量的星期滑动特征及其统计特征。如上周一，上上周一，上上上周一数据。
* 对部分特征进行二次多项式交互。

## 模型选择<br>
   Extremely Randomized Trees　极端随机树。它与随机森林的不同之处在于，随机森林选择最佳分割点时，以香农熵或gini系数为依据；极端随机树是随机选择分割点。这使得森林里每棵树的差异比随机森林更大，集成时方差更小。GBDT和RF在该数据集上表现不如ET。因此选择ET作为模型。<br>

## 规则调整<br>
   有部分商家近三周的销售数据突降，却又不为零，与前段时间相差甚远(如23号，727号，810号...)。因为训练集取的是前三周，这使得预测不能反映出更前的历史状态，需要进行修改。<br>
因为官方说预测区间内商家正常营业,因此我尝试用更前的“正常”数据对预测进行填充修改。23号商家因为最近的几天出现明显的回升，因此这样的修改还是很有把握的，而727号商家近期无明显回升趋势，因此修改后成绩有所下降。<br>
 这说明在没有明显回升趋势的情况下，该规则不适用。<br>

## 其他<br>
* 试过特征标准化，归一化，PCA降维，k-means特征，均对结果有消极影响。<br>
* stacking在这里也不适用，因为每个模型训练出来的答案高度共线性，stacking出来的结果方差很大，线上提交也显示的效果并不好．要做模型融合的话可以做加权融合，但是没找到明确的分类，因此也没有做．日后学习一下高分答案再补充．<br>
* 数据地址链接: https://pan.baidu.com/s/1Wrp3TodeN55sG0HVOcs0Fw 密码: cb8m.<br>

## 其他大神的做法<br>
* 获奖解决方案看这里！包括思路和PPT！[传送门](https://tianchi.aliyun.com/competition/new_articleDetail.html?postsId=2525&from=singlemessage)
* 第十六名的队伍 皮皮虾，我们回去吧 也开源啦，大家快过去围观 [传送门](https://github.com/RogerMonkey/IJCAI_CUP_2017). 预处理方面他们多了一份数据：以四分之一天为单位;模型融合是对几个模型的预测加权回归;规则方面他们对火锅店与双十一进行了单独处理<br>
* 其他大神的方法暂未开源，因此这里待续。热烈欢迎大家补充<br>
### p.s.<br>
   现在还是一名新手，之前未用过Hadoop。这次用MapReduce来对数据进行聚合，速度真的好快啊，内存控制得也很好，以后一定好好学Hadoop。MR的代码也放上去了，其实就是一个简单的WordCount，欢迎指出不足的地方。若能获得大家指点一二，是我之幸事！
（MapReduce对user_pay.txt(2.1G)单机聚合时间约３分钟，使用内存控制在１G以内）

