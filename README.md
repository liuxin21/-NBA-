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

![](https://dn-anything-about-doc.qbox.me/document-uid291340labid2647timestamp1489325997201.png)

这条对比公式就是 **Elo Score 等级分**制度。Elo 的最初为了提供国际象棋中，更好地对不同的选手进行等级划分。在现在很多的竞技运动或者游戏中都会采取 Elo 等级分制度对选手或玩家进行等级划分，如足球、篮球、棒球比赛或 LOL，DOTA 等游戏。

在这里我们将基于国际象棋比赛，大致地介绍下 Elo 等级划分制度。在上图中 **Eduardo** 在窗户上写下的公式就是根据`Logistic Distribution`计算 PK 双方（A 和 B）对各自的胜率期望值计算公式。假设 A 和 B 的当前等级分为 <math><semantics><mrow><msub><mi>R</mi><mi>A</mi></msub></mrow><annotation encoding="application/x-tex">R_A</annotation></semantics></math>R​A​​何 <math><semantics><mrow><msub><mi>R</mi><mi>B</mi></msub></mrow><annotation encoding="application/x-tex">R_B</annotation></semantics></math>R​B​​，则 A 对 B 的胜率期望值为：

![](https://dn-anything-about-doc.qbox.me/document-uid291340labid2647timestamp1490108812711.png)

B 对 A 的胜率期望值为

![](https://dn-anything-about-doc.qbox.me/document-uid291340labid2647timestamp1490108823190.png)

如果棋手 A 在比赛中的真实得分 <math><semantics><mrow><msub><mi>S</mi><mi>A</mi></msub></mrow><annotation encoding="application/x-tex">S_A</annotation></semantics></math>S​A​​（胜 1 分，和 0.5 分，负 0 分）和他的胜率期望值 <math><semantics><mrow><msub><mi>E</mi><mi>A</mi></msub></mrow><annotation encoding="application/x-tex">E_A</annotation></semantics></math>E​A​​不同，则他的等级分要根据以下公式进行调整：

![](https://dn-anything-about-doc.qbox.me/document-uid291340labid2647timestamp1490108835437.png)

在国际象棋中，根据等级分的不同 K 值也会做相应的调整：

*   <math><semantics><mrow><mo>≥</mo><mn>2</mn><mn>4</mn><mn>0</mn><mn>0</mn></mrow><annotation encoding="application/x-tex">\ge2400</annotation></semantics></math>≥2400，K=16
*   2100~2400 分，K=24
*   <math><semantics><mrow><mo>≤</mo><mn>2</mn><mn>1</mn><mn>0</mn><mn>0</mn></mrow><annotation encoding="application/x-tex">\le2100</annotation></semantics></math>≤2100，K=32

因此我们将会用以表示某场比赛数据的**特征向量**为（加入 A 与 B 队比赛）：[A 队 Elo score, A 队的 **T,O 和 M 表**统计数据，B 队 Elo score, B 队的 **T,O 和 M 表**统计数据]