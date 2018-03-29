我们将采用 [Basketball-Reference.com](Basketball-Reference.com) 中的统计数据。在这个网站中，你可以看到不同球员、队伍、赛季和联盟比赛的基本统计数据，如得分，犯规次数等情况，胜负次数等情况。而我们在这里将会使用 2015-16 NBA Season Summary 中数据。

可以直接打包下载：
`$ wget https://github.com/liuxin21/NBA-predictions/blob/master/data.zip`

在这个2015-16总结的所有表格中，我们将使用的是以下三个数据表格：

1. `15-16Team_Per_Game_Stat.csv`：每支队伍平均每场比赛的表现统计
2. `15-16Opponent_Per_Game_Stat.csv`：所遇到的对手平均每场比赛的统计信息，所包含的统计数据与Team Per Game Stats中的一致，只是代表的该球队对应的对手的
3. `15-16Miscellaneous_Stat.csv`：综合统计数据