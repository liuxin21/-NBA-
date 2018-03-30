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

![](https://wikimedia.org/api/rest_v1/media/math/render/svg/0b096f1c60d7fdc543f3bc583fe32601f1c2f0cf)

和
<span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle R_{B}}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <msub>
          <mi>R</mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mi>B</mi>
          </mrow>
        </msub>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle R_{B}}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/33d79a4532363bb4ed9602166704c3f98928478f" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.671ex; width:3.244ex; height:2.509ex;" alt="R_{B}"></span>，则按Logistic distribution A对B的胜率期望值当为</p>
<dl>
<dd><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle E_{A}={\frac {1}{1+10^{(R_{B}-R_{A})/400}}}.}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <msub>
          <mi>E</mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mi>A</mi>
          </mrow>
        </msub>
        <mo>=</mo>
        <mrow class="MJX-TeXAtom-ORD">
          <mfrac>
            <mn>1</mn>
            <mrow>
              <mn>1</mn>
              <mo>+</mo>
              <msup>
                <mn>10</mn>
                <mrow class="MJX-TeXAtom-ORD">
                  <mo stretchy="false">(</mo>
                  <msub>
                    <mi>R</mi>
                    <mrow class="MJX-TeXAtom-ORD">
                      <mi>B</mi>
                    </mrow>
                  </msub>
                  <mo>−<!-- − --></mo>
                  <msub>
                    <mi>R</mi>
                    <mrow class="MJX-TeXAtom-ORD">
                      <mi>A</mi>
                    </mrow>
                  </msub>
                  <mo stretchy="false">)</mo>
                  <mrow class="MJX-TeXAtom-ORD">
                    <mo>/</mo>
                  </mrow>
                  <mn>400</mn>
                </mrow>
              </msup>
            </mrow>
          </mfrac>
        </mrow>
        <mo>.</mo>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle E_{A}={\frac {1}{1+10^{(R_{B}-R_{A})/400}}}.}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/51346e1c65f857c0025647173ae48ddac904adcb" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -2.671ex; width:25.004ex; height:6.009ex;" alt="E_{A}={\frac  1{1+10^{{(R_{B}-R_{A})/400}}}}."></span></dd>
</dl>
<p>类似B对A的胜率为</p>
<dl>
<dd><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle E_{B}={\frac {1}{1+10^{(R_{A}-R_{B})/400}}}.}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <msub>
          <mi>E</mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mi>B</mi>
          </mrow>
        </msub>
        <mo>=</mo>
        <mrow class="MJX-TeXAtom-ORD">
          <mfrac>
            <mn>1</mn>
            <mrow>
              <mn>1</mn>
              <mo>+</mo>
              <msup>
                <mn>10</mn>
                <mrow class="MJX-TeXAtom-ORD">
                  <mo stretchy="false">(</mo>
                  <msub>
                    <mi>R</mi>
                    <mrow class="MJX-TeXAtom-ORD">
                      <mi>A</mi>
                    </mrow>
                  </msub>
                  <mo>−<!-- − --></mo>
                  <msub>
                    <mi>R</mi>
                    <mrow class="MJX-TeXAtom-ORD">
                      <mi>B</mi>
                    </mrow>
                  </msub>
                  <mo stretchy="false">)</mo>
                  <mrow class="MJX-TeXAtom-ORD">
                    <mo>/</mo>
                  </mrow>
                  <mn>400</mn>
                </mrow>
              </msup>
            </mrow>
          </mfrac>
        </mrow>
        <mo>.</mo>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle E_{B}={\frac {1}{1+10^{(R_{A}-R_{B})/400}}}.}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/4b340e7d15e61ee7d90f428dcf7f4b3c049d89ff" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -2.671ex; width:25.019ex; height:6.009ex;" alt="E_{B}={\frac  1{1+10^{{(R_{A}-R_{B})/400}}}}."></span></dd>
</dl>
<p>假如一位棋手在比赛中的真实得分<span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle S_{A}}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <msub>
          <mi>S</mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mi>A</mi>
          </mrow>
        </msub>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle S_{A}}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/f581ca4fd5bc6d22270c6050703cf23e5b840435" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.671ex; width:2.89ex; height:2.509ex;" alt="S_{A}"></span>（胜=1分，和=0.5分，负=0分）和他的胜率期望值<span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle E_{A}}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <msub>
          <mi>E</mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mi>A</mi>
          </mrow>
        </msub>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle E_{A}}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/6d368f77b6dfe496467559869a421efed0881bcd" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.671ex; width:3.18ex; height:2.509ex;" alt="E_{A}"></span>不同，则他的等级分要作相应的调整。具体的数学公式为</p>
<dl>
<dd><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle R_{A}^{\prime }=R_{A}+K(S_{A}-E_{A}).}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <msubsup>
          <mi>R</mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mi>A</mi>
          </mrow>
          <mrow class="MJX-TeXAtom-ORD">
            <mi class="MJX-variant" mathvariant="normal">′<!-- ′ --></mi>
          </mrow>
        </msubsup>
        <mo>=</mo>
        <msub>
          <mi>R</mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mi>A</mi>
          </mrow>
        </msub>
        <mo>+</mo>
        <mi>K</mi>
        <mo stretchy="false">(</mo>
        <msub>
          <mi>S</mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mi>A</mi>
          </mrow>
        </msub>
        <mo>−<!-- − --></mo>
        <msub>
          <mi>E</mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mi>A</mi>
          </mrow>
        </msub>
        <mo stretchy="false">)</mo>
        <mo>.</mo>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle R_{A}^{\prime }=R_{A}+K(S_{A}-E_{A}).}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/09a11111b433582eccbb22c740486264549d1129" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -1.005ex; width:25.829ex; height:3.009ex;" alt="R_{A}^{\prime }=R_{A}+K(S_{A}-E_{A})."></span></dd>
</dl>
<p>公式中<span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle R_{A}}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <msub>
          <mi>R</mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mi>A</mi>
          </mrow>
        </msub>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle R_{A}}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/0b096f1c60d7fdc543f3bc583fe32601f1c2f0cf" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.671ex; width:3.229ex; height:2.509ex;" alt="R_{A}"></span>和<span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle R_{A}^{\prime }}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <msubsup>
          <mi>R</mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mi>A</mi>
          </mrow>
          <mrow class="MJX-TeXAtom-ORD">
            <mi class="MJX-variant" mathvariant="normal">′<!-- ′ --></mi>
          </mrow>
        </msubsup>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle R_{A}^{\prime }}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/99935e6c376a3ed76329a18facaa07221d685208" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -1.005ex; width:3.229ex; height:2.843ex;" alt="R_{A}^{\prime }"></span>分别为棋手调整前后的等级分。在大师级比赛中<span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle K}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mi>K</mi>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle K}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/2b76fce82a62ed5461908f0dc8f037de4e3686b0" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:2.066ex; height:2.176ex;" alt="K"></span>通常为16。</p>
<p>例如，棋手A等级分为1613，与等级分为1573的棋手B战平。若K取32，则A的胜率期望值为<span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle {\frac {1}{1+10^{(1573-1613)/400}}}}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mrow class="MJX-TeXAtom-ORD">
          <mfrac>
            <mn>1</mn>
            <mrow>
              <mn>1</mn>
              <mo>+</mo>
              <msup>
                <mn>10</mn>
                <mrow class="MJX-TeXAtom-ORD">
                  <mo stretchy="false">(</mo>
                  <mn>1573</mn>
                  <mo>−<!-- − --></mo>
                  <mn>1613</mn>
                  <mo stretchy="false">)</mo>
                  <mrow class="MJX-TeXAtom-ORD">
                    <mo>/</mo>
                  </mrow>
                  <mn>400</mn>
                </mrow>
              </msup>
            </mrow>
          </mfrac>
        </mrow>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle {\frac {1}{1+10^{(1573-1613)/400}}}}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/adb225215c4ce787233d8ea15e6fee3d834cb7ca" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -2.671ex; width:19.818ex; height:6.009ex;" alt="{\frac  1{1+10^{{(1573-1613)/400}}}}"></span>，约为0.5573，因而A的新等级分为1613 + 32 · (0.5 − 0.5573) = 1611.166</p>
<h2><span id=".E5.9B.BD.E9.99.85.E8.B1.A1.E6.A3.8B.E4.B8.AD.E7.9A.84.E7.AD.89.E7.BA.A7.E5.88.86"></span><span class="mw-headline" id="国际象棋中的等级分">国际象棋中的等级分</span><span class="mw-editsection"><span class="mw-editsection-bracket">[</span><a href="/w/index.php?title=%E7%AD%89%E7%BA%A7%E5%88%86&amp;action=edit&amp;section=3" title="编辑小节：国际象棋中的等级分">编辑</a><span class="mw-editsection-bracket">]</span></span></h2>
<p>国际象棋中，等级分和棋联称号的大致对应为</p>
<ul>
<li>2500分以上：国际特级大师</li>
<li>2400-2499分：国际大师</li>
<li>2300-2399分：棋联大师</li>
</ul>
<p>历史上，只有6位棋手的等级分曾超过2800分。他们是<a href="/wiki/%E5%8D%A1%E6%96%AF%E5%B8%95%E7%BE%85%E5%A4%AB" class="mw-redirect" title="卡斯帕羅夫">卡斯帕罗夫</a>、<a href="/wiki/%E5%85%8B%E6%8B%89%E5%A7%86%E5%B0%BC%E5%85%8B" class="mw-redirect" title="克拉姆尼克">克拉姆尼克</a>、<a href="/wiki/%E6%89%98%E5%B8%95%E6%B4%9B%E5%A4%AB" class="mw-redirect" title="托帕洛夫">托帕洛夫</a>、<a href="/wiki/%E9%98%BF%E5%8D%97%E5%BE%B7" class="mw-redirect" title="阿南德">阿南德</a>、<a href="/wiki/%E5%88%97%E5%86%AF%C2%B7%E9%98%BF%E7%BD%97%E5%B0%BC%E6%89%AC" title="列冯·阿罗尼扬">列冯·阿罗尼扬</a>和<a href="/wiki/%E9%A6%AC%E6%A0%BC%E5%8A%AA%E6%96%AF%C2%B7%E5%8D%A1%E7%88%BE%E6%A3%AE" title="馬格努斯·卡爾森">马格努斯·卡尔森</a>。</p>
<table class="wikitable">
<caption><b>国际棋联等级分排名（2013年12月）</b><sup id="cite_ref-1" class="reference"><a href="#cite_note-1">[1]</a></sup></caption>
<tbody><tr>
<th>位次</th>
<th>棋手</th>
<th>国籍</th>
<th>等级分</th>
</tr>
<tr>
<td>1</td>
<td><a href="/wiki/%E9%A6%AC%E6%A0%BC%E5%8A%AA%E6%96%AF%C2%B7%E5%8D%A1%E7%88%BE%E6%A3%AE" title="馬格努斯·卡爾森">马格努斯·卡尔森</a></td>
<td><span class="flagicon"><img alt="" src="//upload.wikimedia.org/wikipedia/commons/thumb/d/d9/Flag_of_Norway.svg/22px-Flag_of_Norway.svg.png" width="22" height="16" class="thumbborder" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/d/d9/Flag_of_Norway.svg/33px-Flag_of_Norway.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/d/d9/Flag_of_Norway.svg/44px-Flag_of_Norway.svg.png 2x" data-file-width="1100" data-file-height="800">&nbsp;</span><a href="/wiki/%E6%8C%AA%E5%A8%81" title="挪威">挪威</a></td>
<td>2872</td>
</tr>
<tr>
<td>2</td>
<td><a href="/wiki/%E5%88%97%E5%86%AF%C2%B7%E9%98%BF%E7%BD%97%E5%B0%BC%E6%89%AC" title="列冯·阿罗尼扬">列冯·阿罗尼扬</a></td>
<td><span class="flagicon"><img alt="" src="//upload.wikimedia.org/wikipedia/commons/thumb/2/2f/Flag_of_Armenia.svg/22px-Flag_of_Armenia.svg.png" width="22" height="11" class="thumbborder" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/2/2f/Flag_of_Armenia.svg/33px-Flag_of_Armenia.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/2/2f/Flag_of_Armenia.svg/44px-Flag_of_Armenia.svg.png 2x" data-file-width="1200" data-file-height="600">&nbsp;</span><a href="/wiki/%E4%BA%9E%E7%BE%8E%E5%B0%BC%E4%BA%9E" title="亞美尼亞">亞美尼亞</a></td>
<td>2803</td>
</tr>
<tr>
<td>3</td>
<td><a href="/wiki/%E5%BC%97%E6%8B%89%E5%9F%BA%E7%B1%B3%E5%B0%94%C2%B7%E9%B2%8D%E9%87%8C%E7%B4%A2%E7%BB%B4%E5%A5%87%C2%B7%E5%85%8B%E6%8B%89%E5%A7%86%E5%B0%BC%E5%85%8B" title="弗拉基米尔·鲍里索维奇·克拉姆尼克">弗拉基米尔·克拉姆尼克</a></td>
<td><span class="flagicon"><img alt="" src="//upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Flag_of_Russia.svg/22px-Flag_of_Russia.svg.png" width="22" height="15" class="thumbborder" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Flag_of_Russia.svg/33px-Flag_of_Russia.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Flag_of_Russia.svg/44px-Flag_of_Russia.svg.png 2x" data-file-width="900" data-file-height="600">&nbsp;</span><a href="/wiki/%E4%BF%84%E7%BE%85%E6%96%AF" class="mw-redirect" title="俄羅斯">俄羅斯</a></td>
<td>2793</td>
</tr>
<tr>
<td>4</td>
<td><a href="/wiki/%E4%B8%AD%E6%9D%91%E5%85%89_(%E6%A3%8B%E6%89%8B)" title="中村光 (棋手)">中村光</a></td>
<td><span class="flagicon"><img alt="" src="//upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Flag_of_the_United_States.svg/22px-Flag_of_the_United_States.svg.png" width="22" height="12" class="thumbborder" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Flag_of_the_United_States.svg/33px-Flag_of_the_United_States.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Flag_of_the_United_States.svg/44px-Flag_of_the_United_States.svg.png 2x" data-file-width="1235" data-file-height="650">&nbsp;</span><a href="/wiki/%E7%BE%8E%E5%9C%8B" class="mw-redirect" title="美國">美國</a></td>
<td>2786</td>
</tr>
<tr>
<td>5</td>
<td><a href="/wiki/%E7%BB%B4%E5%A1%9E%E6%9E%97%C2%B7%E6%89%98%E5%B8%95%E6%B4%9B%E5%A4%AB" title="维塞林·托帕洛夫">维塞林·托帕洛夫</a></td>
<td><span class="flagicon"><img alt="" src="//upload.wikimedia.org/wikipedia/commons/thumb/9/9a/Flag_of_Bulgaria.svg/22px-Flag_of_Bulgaria.svg.png" width="22" height="13" class="thumbborder" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/9/9a/Flag_of_Bulgaria.svg/33px-Flag_of_Bulgaria.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/9/9a/Flag_of_Bulgaria.svg/44px-Flag_of_Bulgaria.svg.png 2x" data-file-width="1000" data-file-height="600">&nbsp;</span><a href="/wiki/%E4%BF%9D%E5%8A%A0%E5%88%A9%E4%BA%9A" title="保加利亚">保加利亚</a></td>
<td>2785</td>
</tr>
<tr>
<td>6</td>
<td><a href="/wiki/Alexander_Grischuk" class="mw-redirect" title="Alexander Grischuk">Alexander Grischuk</a></td>
<td><span class="flagicon"><img alt="" src="//upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Flag_of_Russia.svg/22px-Flag_of_Russia.svg.png" width="22" height="15" class="thumbborder" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Flag_of_Russia.svg/33px-Flag_of_Russia.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Flag_of_Russia.svg/44px-Flag_of_Russia.svg.png 2x" data-file-width="900" data-file-height="600">&nbsp;</span><a href="/wiki/%E4%BF%84%E7%BE%85%E6%96%AF" class="mw-redirect" title="俄羅斯">俄羅斯</a></td>
<td>2783</td>
</tr>
<tr>
<td>7</td>
<td><a href="/wiki/%E6%B3%95%E6%AF%94%E4%BA%9A%E8%AF%BA%C2%B7%E5%8D%A1%E9%B2%81%E9%98%BF%E7%BA%B3" title="法比亚诺·卡鲁阿纳">法比亚诺·卡鲁阿纳</a></td>
<td><span class="flagicon"><img alt="" src="//upload.wikimedia.org/wikipedia/commons/thumb/0/03/Flag_of_Italy.svg/22px-Flag_of_Italy.svg.png" width="22" height="15" class="thumbborder" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/0/03/Flag_of_Italy.svg/33px-Flag_of_Italy.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/0/03/Flag_of_Italy.svg/44px-Flag_of_Italy.svg.png 2x" data-file-width="1500" data-file-height="1000">&nbsp;</span><a href="/wiki/%E7%BE%A9%E5%A4%A7%E5%88%A9" class="mw-redirect" title="義大利">義大利</a></td>
<td>2782</td>
</tr>
<tr>
<td>8</td>
<td><a href="/wiki/%E9%B2%8D%E9%87%8C%E6%96%AF%C2%B7%E6%A0%BC%E5%B0%94%E5%87%A1%E5%BE%B7" title="鲍里斯·格尔凡德">鲍里斯·格尔凡德</a></td>
<td><span class="flagicon"><img alt="" src="//upload.wikimedia.org/wikipedia/commons/thumb/d/d4/Flag_of_Israel.svg/22px-Flag_of_Israel.svg.png" width="22" height="16" class="thumbborder" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/d/d4/Flag_of_Israel.svg/33px-Flag_of_Israel.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/d/d4/Flag_of_Israel.svg/44px-Flag_of_Israel.svg.png 2x" data-file-width="660" data-file-height="480">&nbsp;</span><a href="/wiki/%E4%BB%A5%E8%89%B2%E5%88%97" title="以色列">以色列</a></td>
<td>2777</td>
</tr>
<tr>
<td>9</td>
<td><a href="/wiki/%E7%BB%B4%E6%96%AF%E7%93%A6%E7%BA%B3%E5%9D%A6%C2%B7%E9%98%BF%E5%8D%97%E5%BE%B7" title="维斯瓦纳坦·阿南德">维斯瓦纳坦·阿南德</a></td>
<td><span class="flagicon"><img alt="" src="//upload.wikimedia.org/wikipedia/commons/thumb/4/41/Flag_of_India.svg/22px-Flag_of_India.svg.png" width="22" height="15" class="thumbborder" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/4/41/Flag_of_India.svg/33px-Flag_of_India.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/4/41/Flag_of_India.svg/44px-Flag_of_India.svg.png 2x" data-file-width="1350" data-file-height="900">&nbsp;</span><a href="/wiki/%E5%8D%B0%E5%BA%A6" title="印度">印度</a></td>
<td>2773</td>
</tr>
<tr>
<td>10</td>
<td><a href="/wiki/%E5%BD%BC%E5%BE%97%C2%B7%E6%96%AF%E7%BB%B4%E5%BE%B7%E5%8B%92" title="彼得·斯维德勒">彼得·斯维德勒</a></td>
<td><span class="flagicon"><img alt="" src="//upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Flag_of_Russia.svg/22px-Flag_of_Russia.svg.png" width="22" height="15" class="thumbborder" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Flag_of_Russia.svg/33px-Flag_of_Russia.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Flag_of_Russia.svg/44px-Flag_of_Russia.svg.png 2x" data-file-width="900" data-file-height="600">&nbsp;</span><a href="/wiki/%E4%BF%84%E7%BE%85%E6%96%AF" class="mw-redirect" title="俄羅斯">俄羅斯</a></td>
<td>2758</td>
</tr>
</tbody></table>
<h2><span id=".E5.8F.82.E8.A7.81"></span><span class="mw-headline" id="参见">参见</span><span class="mw-editsection"><span class="mw-editsection-bracket">[</span><a href="/w/index.php?title=%E7%AD%89%E7%BA%A7%E5%88%86&amp;action=edit&amp;section=4" title="编辑小节：参见">编辑</a><span class="mw-editsection-bracket">]</span></span></h2>
<ul>
<li><a href="/wiki/%E4%B8%AD%E5%9B%BD%E5%9B%B4%E6%A3%8B%E7%AD%89%E7%BA%A7%E5%88%86" title="中国围棋等级分">中国围棋等级分</a></li>
<li><span class="ilh-all" data-orig-title="Glicko评分系统" data-lang-code="en" data-lang-name="英语" data-foreign-title="Glicko rating system"><span class="ilh-page"><a href="/w/index.php?title=Glicko%E8%AF%84%E5%88%86%E7%B3%BB%E7%BB%9F&amp;action=edit&amp;redlink=1" class="new" original-title="Glicko评分系统（页面不存在）">Glicko评分系统</a></span><span class="noprint ilh-comment">（<span class="ilh-lang">英语</span><span class="ilh-colon">：</span><span class="ilh-link"><a href="https://en.wikipedia.org/wiki/Glicko_rating_system" class="extiw" title="en:Glicko rating system"><span lang="en" dir="auto" xml:lang="en">Glicko rating system</span></a></span>）</span></span></li>
</ul>
<h2><span id=".E6.B3.A8.E9.87.8B"></span><span class="mw-headline" id="注釋">注釋</span><span class="mw-editsection"><span class="mw-editsection-bracket">[</span><a href="/w/index.php?title=%E7%AD%89%E7%BA%A7%E5%88%86&amp;action=edit&amp;section=5" title="编辑小节：注釋">编辑</a><span class="mw-editsection-bracket">]</span></span></h2>
<ol class="references">
<li id="cite_note-1"><span class="mw-cite-backlink"><b><a href="#cite_ref-1"><span class="cite-accessibility-label">跳转 </span>^</a></b></span> <span class="reference-text"><a rel="nofollow" class="external text" href="http://ratings.fide.com/topstat.phtml">國際西洋棋聯盟排名（英文）</a></span></li>
</ol>
<h2><span id=".E5.BB.B6.E4.BC.B8.E9.96.B1.E8.A6.BD"></span><span class="mw-headline" id="延伸閱覽">延伸閱覽</span><span class="mw-editsection"><span class="mw-editsection-bracket">[</span><a href="/w/index.php?title=%E7%AD%89%E7%BA%A7%E5%88%86&amp;action=edit&amp;section=6" title="编辑小节：延伸閱覽">编辑</a><span class="mw-editsection-bracket">]</span></span></h2>
<ul>
<li><a rel="nofollow" class="external text" href="http://www.gameres.com/thread_228018_1_1.html">ELO算法教程 - GameRes游资网</a></li>
</ul>




假设 A 和 B 的当前等级分为 ![](http://latex.codecogs.com/gif.latex?R_A) 和 ![](http://latex.codecogs.com/gif.latex?R_B)，则 A 对 B 的胜率期望值为：

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