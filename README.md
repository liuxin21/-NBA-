## 数据准备

我们将采用 [Basketball-Reference.com](Basketball-Reference.com) 中的统计数据。在这个网站中，你可以看到不同球员、队伍、赛季和联盟比赛的基本统计数据，如得分，犯规次数等情况，胜负次数等情况。而我们在这里将会使用 2015-16 NBA Season Summary 中数据。

可以直接打包下载：
`$ wget https://github.com/liuxin21/NBA-predictions/blob/master/data.zip`

在这个2015-16总结的所有表格中，我们将使用的是以下三个数据表格：

1. `15-16Team_Per_Game_Stat.csv`：每支队伍平均每场比赛的表现统计
2. `15-16Opponent_Per_Game_Stat.csv`：所遇到的对手平均每场比赛的表现统计，所包含的统计数据与上面一致
3. `15-16Miscellaneous_Stat.csv`：综合统计数据


在`data`文件夹中，包含了 2015~2016 年的 NBA 数据 **T,O 和 M** 表，及经处理后的常规赛和挑战赛的比赛数据`2015-2016_result.csv`，这个数据文件是我们通过在`basketball-reference.com`的 2015-16 Schedule and result 的几个月份比赛数据中提取得到的，其中包括三个字段：

*   WTeam: 比赛胜利队伍
*   LTeam: 失败队伍
*   WLoc: 胜利队伍一方所在的为主场或是客场 另外一个文件就是`16-17Schedule.csv`，也是经过我们加工处理得到的 NBA 在 2016~2017 年的常规赛的比赛安排，其中包括两个字段：
*   Vteam: 访问 / 客场作战队伍
*   Hteam: 主场作战队伍


## 数据分析

在获取到数据之后，我们将利用每支队伍**过去的比赛情况**和 **Elo 等级分**来判断每支比赛队伍的可胜概率。在评价到每支队伍过去的比赛情况时，我们将使用到 **Team Per Game Stats**，**Opponent Per Game Stats** 和 **Miscellaneous Stats**（之后简称为 **T、O 和 M** 表）这三个表格的数据，作为代表比赛中某支队伍的比赛特征。我们最终将实现针对每场比赛，预测比赛中哪支队伍最终将会获胜，但并不是给出绝对的胜败情况，而是预判胜利的队伍有多大的获胜概率。因此我们将建立一个代表比赛的**特征向量**。由两支队伍的以往比赛情况统计情况（**T、O 和Ｍ**表），和两个队伍各自的 **Elo** 等级分构成。

关于 **Elo score** 等级分，在`《社交网络》`这部电影中，**Mark**（主人公原型就是扎克伯格，FaceBook 创始人）在电影起初开发的一个美女排名系统就是利用其好友 **Eduardo** 在窗户上写下的排名公式，对不同的女生进行等级制度对比，最后 PK 出胜利的一方。

![](https://github.com/liuxin21/NBA-predictions/blob/master/pic1.png)

这条对比公式就是 **Elo Score 等级分** 制度。Elo 最初是为了在国际象棋中更好地对不同的选手进行等级划分。在现在很多的竞技运动或者游戏中都会采取 Elo 等级分制度对选手或玩家进行等级划分，如足球、篮球、棒球比赛或 LOL，DOTA 等游戏。

在这里我们将基于**国际象棋**比赛，大致地介绍下 Elo 等级划分制度。在上图中 **Eduardo** 在窗户上写下的公式就是根据`Logistic Distribution`计算 PK 双方（A 和 B）对各自的胜率期望值计算公式。

<p>假设棋手A和B的当前等级分分别为
<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/0b096f1c60d7fdc543f3bc583fe32601f1c2f0cf" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.671ex; width:3.244ex; height:2.509ex;" alt="R_{A}">
和
<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/33d79a4532363bb4ed9602166704c3f98928478f" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.671ex; width:3.244ex; height:2.509ex;" alt="R_{B}">，则按Logistic distribution A对B的胜率期望值当为
<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/51346e1c65f857c0025647173ae48ddac904adcb" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -2.671ex; width:25.004ex; height:6.009ex;" alt="E_{A}={\frac  1{1+10^{{(R_{B}-R_{A})/400}}}}.">
<p>类似B对A的胜率为</p>
<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/4b340e7d15e61ee7d90f428dcf7f4b3c049d89ff" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -2.671ex; width:25.019ex; height:6.009ex;" alt="E_{B}={\frac  1{1+10^{{(R_{A}-R_{B})/400}}}}.">
<p>假如一位棋手在比赛中的真实得分
<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/f581ca4fd5bc6d22270c6050703cf23e5b840435" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.671ex; width:2.89ex; height:2.509ex;" alt="S_{A}">（胜=1分，和=0.5分，负=0分）和他的胜率期望值
<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/6d368f77b6dfe496467559869a421efed0881bcd" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.671ex; width:3.18ex; height:2.509ex;" alt="E_{A}">
不同，则他的等级分要作相应的调整。具体的数学公式为</p>
<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/09a11111b433582eccbb22c740486264549d1129" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -1.005ex; width:25.829ex; height:3.009ex;" alt="R_{A}^{\prime }=R_{A}+K(S_{A}-E_{A}).">
<p>公式中<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/0b096f1c60d7fdc543f3bc583fe32601f1c2f0cf" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.671ex; width:3.229ex; height:2.509ex;" alt="R_{A}">和<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/99935e6c376a3ed76329a18facaa07221d685208" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -1.005ex; width:3.229ex; height:2.843ex;" alt="R_{A}^{\prime }">分别为棋手调整前后的等级分。在大师级比赛中<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/2b76fce82a62ed5461908f0dc8f037de4e3686b0" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:2.066ex; height:2.176ex;" alt="K">通常为16。</p>
<p>例如，棋手A等级分为1613，与等级分为1573的棋手B战平。若K取32，则A的胜率期望值为
<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/adb225215c4ce787233d8ea15e6fee3d834cb7ca" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -2.671ex; width:19.818ex; height:6.009ex;" alt="{\frac  1{1+10^{{(1573-1613)/400}}}}">，约为0.5573，因而A的新等级分为1613 + 32 · (0.5 − 0.5573) = 1611.166</p>


某场比赛数据的**特征向量**为（假如 A 与 B 队比赛）：

[(A队 Elo score), (A队的 **T,O 和 M 表**统计数据)，(B队 Elo score), (B队的 **T,O 和 M 表**统计数据)]
