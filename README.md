群体驻留时空模式挖掘
=============

本项目所采用的数据采集于杭州移动3G网络。针对城市群体人口轨迹的挖掘，已有工作常聚焦于研究移动行为，例如通勤行为等，固然，城市通勤行为能够直接反应出城市动态特性，与城市区域功能、人类行为模式等密切相关，但另一方面，人口驻留行为，例如不同区域在单日或周内不同时间段内驻留的人口密度、停留时长、驻留时间段的起止时刻等都是研究城市区域功能以及人类行为模式的良好特征。城市大尺度下驻留模式研究一方面受限于数据源的限制，为了获取用户的驻留行为，一般需要获取用户完整的移动轨迹，常用的调查问卷、手机通话、车载GPS以及社交网络签到数据不能很好的满足需求，同时，庞大的城市居民数量对数据的存储和分析也提出了更好的要求。本项目针对群体驻留时空模式进行挖掘，首先从群体轨迹中提取出驻留片段，之后基于层级贝叶斯模型使用无监督聚类的方法自动发现城市人口的驻留模式，层级贝叶斯模型相对于已有的时空聚类方法，包括主成分分析、隐含主题模型等均具有一定优势。

数据采集与数据集
----

本项目所采用的数据集包含一周内连续五天工作日的移动用户上网基站定位数据，采用用户识别码（IMSI）来区分不同用户，并将基站的位置区编码（LAC）联合小区标识（CI）同基站位置数据进行关联转换成为经纬度坐标，结合HTTP请求对应的时间戳即得到用户轨迹。经统计，数据集基本情况如表所示。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/t1.png)

如图是群体驻留时空模式挖掘系统框架。主要包含数据准备、数据挖掘与分析等过程，数据准备包括移动网络日志清洗、基站经纬度映射、轨迹点提取、移动轨迹提取，数据挖掘与分析包括从移动轨迹中抽取驻留片段，并对驻留片段起止时刻进行估计，以及对驻留片段进行时空聚类，基于对时空模式的分析，能够进一步对功能区域进行推测与识别，或对用户轨迹进行语义标注，理解用户的出行目的。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/system.png)

驻留轨迹段提取
----

驻留轨迹段的提取分为两个步骤，首先抽取出用户停留的时间和地点（简称为驻留轨迹段提取），之后对用户的到达和离开时刻进行估计。驻留轨迹段的提取方法如算法所示，输入用户轨迹点，输出驻留片段，基于预设的时间间隔以及空间间隔阈值timeThres和distThres，按照时间序列依次计算出每个轨迹点与初始轨迹点之间的时间间隔和距离，直到首次距离超过空间间隔阈值distThres，此时如果时间间隔超过时间间隔阈值timeThres，则检测到一次驻留行为。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/alg1.png)

在检测出驻留行为后，需要对驻留的起止时刻te和tl进行估计，减小时空模式挖掘结果的误差。本项目基于用户固有模式、区域固有模式、全局模式对起止时刻进行估计，一般而言，用户根据自身生活规律，例如固定的上、下班时间，位置迁移具有一定模式，单天观测时，用户轨迹点具有稀疏性，但针对不同天，随着用户上网需求变化，通常可以检测到较为连续的时间段内的用户位置的变化情况，因此，基于多天数据有助于对用户位置变化的时刻进行估计。同样，不同区域也具有其固有模式，例如同一公司员工上、下班时刻趋于一致。在更大的尺度上，即城市整体也具有较为固定的通勤模式，例如早、晚高峰。因此，本项目基于三种模式对用户驻留轨迹段的起止时刻采用加权最小二乘法进行估计。

如图是用户原始驻留轨迹段的示例与统计。如左图所示，是以10分钟为时间粒度的部分用户的轨迹段抽取结果，橙色和蓝色部分分别对应白天和夜间，由此可见，大多数用户都存在其固有模式，不同用户的状态转移也存在一定的全局相似性。如右图所示，是抽取出的驻留轨迹段之间时间间隔对应的累计概率分布函数，由此可见，大多数驻留轨迹段之间时间间隔较长，因此需要进行起止时刻估计。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/01.png)

为了验证方法的有效性，共对比了两种基本方法，第一种是边值估计法，在提取出驻留轨迹后不做任何处理，第二种是中值估计，即假设用户离开地点A和到达地点B的时刻均为(t1+t2)/2，其中，t1、t2分别是最后一次观测到用户在地点A以及首次观测到用户在地点B的时刻。用于实验验证的数据来源于状态转移过程观测时间间隔较短，能够进行准确推测的部分数据。实验结果显示，本项目所提出的基于联合概率的估计方法具有最小的估计误差。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/t2.png)

驻留行为时空聚类
----

基于所提取出的群体驻留轨迹段，可以采用基于无监督的时空聚类的方法，自动发现城市人口的驻留行为模式。首先介绍本项目所提出的基于层级贝叶斯时空联合聚类模型的生成过程。基于之前提取出的群体驻留轨迹段，假设数据集对应的群体数量为N，每个用户包含的驻留轨迹片段数量为P，每个驻留轨迹段对应的隐含状态为s，不同的隐含状态可能携带了不同的语义信息，例如“工作”、“居家”、“娱乐”、“购物”等，每个驻留轨迹段的观测变量包括，驻留轨迹段的起止时刻te、tl以及空间位置l，由于空间位置与城市功能区域属性息息相关，因此临近的空间位置可能表达了同样的区域功能语义，反映了相似的用户出行的目的，因此可以假设实际观测到的空间位置l由该功能区域r的分布采样而来。

如图是本节模型的图表示（Graphical representation）。如图所示，深色节点表示观测变量或先验变量，浅色节点表示隐含变量。观测变量包括驻留轨迹段的起止时刻te、tl以及空间位置l，隐含变量包括轨迹段所对应的隐含状态s，空间位置l对应的功能区域r，并假设驻留轨迹段的起止时刻te、tl服从联合高斯分布，同时假设空间位置l服从混合高斯分布，混合高斯分布由多个高斯分布叠加而成，分量的选择服从多项分布，功能区域r可以视作分量的选择。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/m1.png)

模型的生成过程如算法所示。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/alg2.png)

在模型推理过程中采用折叠吉布斯采样（Collapsed Gibbs sampling）基于迭代过程对模型参数进行优化。吉布斯采样是一种马尔可夫链蒙特卡罗（Markov chain Monte Carlo/MCMC）算法，在基于指定多元概率分布直接进行采样较为困难时，可以通过采用吉布斯采样获得观测序列进行近似，经常用于贝叶斯推断（Bayesian inference）。折叠吉布斯采样在吉布斯采样的基础上，通过积分避开了实际待估计的参数，转而对隐含变量和观测变量进行采样，并通过积分在统计观测变量的取值频次后对实际待估计的参数进行估计。针对本项目模型中所含有的联合高斯分布和混合高斯分布，采用了折叠吉布斯采样和最大期望算法（Expectation Maximization Algorithm/EM）相互结合的方法对模型进行推理。最大期望算法是一种概率模型，能够通过最大似然估计或者最大后验概率估计对模型的参数进行优化，其优势在于能够对无法观测的隐藏变量（Latent variable）进行建模。最大期望算法基于迭代过程进行计算，每轮迭代主要包含两个步骤，即E步和M步，交替进行计算。在E步中，基于对隐藏变量上一轮迭代得到的估计值，计算出最大似然估计值。在M步中，通过求导数或偏导数的方法，求得最大化似然估计值时模型的参数。

模型的推断过程如算法所示。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/alg3.png)

如图是针对提取出的驻留轨迹段（已经过起止时刻估计）的起止时刻的联合分布的统计结果，概率分布主要位于三个区域，即图的对角线以及关于对角线对称的两个区域，靠近对角线的左上角区域对应了较短的状态转移时间，囊括了各种情况下的短暂停留行为，对角线对称的两个区域所对应的起止时间趋近于上午9点至下午6点以及晚上7点至次日8点，分别对应了普遍的上班状态和居家状态，并且居家时间的开始时间更为分散，体现了不同群体行为的差异性。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/02.png)

如图针对上图对角线对称的两个区域，统计出了对应的空间分布。如图所示，上班状态（Diurnal）和居家状态（Nocturnal）对应的驻留轨迹段的空间分布差异明显，上班状态对应的空间区域更倾向于城市的中心区域，而居家状态对应的空间区域在城市范围内较为分散。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/03.png)

为了说明层级贝叶斯时空联合聚类模型的优越性，共对比了另外两种模型，其中一种是层级贝叶斯时空次序聚类模型，另一种是高斯混合时空次序聚类模型。高斯混合时空次序聚类模型首先基于驻留轨迹段的起止时刻联合分布进行聚类，之后针对时间维度聚类结果得到的每个类，再基于其空间分布进行聚类。实验结果显示，层级贝叶斯时空联合聚类模型优于用于对比的另外两种模型，层级贝叶斯时空联合聚类在使用更少参数的同时达到了更小的负对数似然，同时时间和空间分布的重构准确率也均较高，这是因为该模型对驻留轨迹段的时间和空间分布同时进行优化，而另外两种模型均按照时间、空间的次序进行优化，在时间维度上虽然达到了最优，但时间聚类的结果限制了在空间维度上所能进行优化的极限。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/t3.png)

如图是基于层级贝叶斯时空联合聚类模型的聚类结果。其中，第一行对应的是模式在时间上的概率分布，第二行对应的是模式在空间上的概率分布，一共有五列，每一列都对应了一种不同的行为模式，时间模式子图的横、纵坐标分别为驻留轨迹段的起止时刻，空间模式子图的横、纵坐标分别为根据经纬度坐标转换得到的地理空间网格坐标。由图可见，城市人口行为模式主要可以分为五种模式，分别对应了不同的语义。由图从左至右的五种模式中，模式1的起止时刻分布在时间模式子图的对角线附近，空间位置分布在城市区域中心，可能对应了各种情况下的短暂停留；模式2及模式3的起止时刻分布在时间模式子图对角线的对称区域，空间分布呈现出互补的特性，一者分布在城市中心区域，一者在城市区域中分布较为均匀，分别可推测带有“白天工作”以及“夜间在家”的语义；模式3的起止时刻分布在时间模式子图对角线的右上端处，对应时间为晚上7点至11点，空间分布偏向于城市中心特定区域，可推测带有“夜间休闲娱乐”的语义；模式5对应的行为发生概率最少，起止时刻分布在时间模式子图的对角线的右上端处，但与模式4不同，模式5的起止时刻近似相同，空间分布较为弥散，可以推测为用户位置在天内固定，这可能是由于用户工作地点与居住地点非常临近。如图是在预设挖掘出五种模式的基础上计算得到的结果，如果进一步增加待挖掘出的模式的数量，还可以进一步获得更加细粒度的行为模式。

![Alt Text](https://raw.githubusercontent.com/qiangsiwei/hangzhou_SSTD/master/figure/04.png)