# 論文
https://research.fb.com/publications/practical-lessons-from-predicting-clicks-on-ads-at-facebook/

# Abstract
* クリック予測はオンライン広告システムにおいて中核的。
* 決定木とロジスティック回帰を結合したモデルを紹介する。
  * 結合したことで、単独の時より3％改善した。
* 最も重要なのは適切な特徴量を取ること。
  * ユーザーや広告に関するヒストリカルな情報を含んだ特徴量は、他よりも重要だ。
* 特徴量とモデルを適切に選ぶことが重要。他の要因は小さい。
  * データ鮮度、learning rate調整、データサンプリングは少ししかモデルを改善しなかった。
  
# 1. Introduction
* 2007年ごろからオークションでの売買形式の広告が提案されていた。
* 広告オークションの効率はクリック予測の精度と較正に依存する。
* クリック予測はロバストで適応性が求められる。大規模データで学習できる必要。
* この論文の目的は、これらの必要条件の元に、現実世界のデータに対して実施した実験から得られた知見を共有すること。
* facebook広告はSponserdSearch（リスティング）とは異なる。
  * SSは（検索）クエリを用いる
  * fb広告はデモグラと興味でターゲティング
  * そのためSSと比べボリュームが出る。 
* この論文では、最終的なクリック予測モデルに焦点を当てる。候補となる広告の予測を返す。
* 決定木とロジスティックリグレッションを結合した、ハイブリッドなモデルを見つけた。これらのメソッド単体と比べて3％効果がよかった。

* 論文の構成
  * 2章で実験設定の概要
  * 3章では異なる確率的線形分類器と多様なオンライン学習アルゴリズムを評価する。
  * 4章はオンライン学習レイヤーにおいて重要となる構成部分
  * 5章はメモリや滞在時間への実践的な対応について。
  * 6章は学習データ量と精度間のトレードオフについて掘り下げる。
  
# 2. EXPERIMENTAL SETUP
* データについて
  * オフラインでの学習用のデータは2013年4四半期の任意の週のデータ。
  * オンラインで観察されたものと似たオフラインデータを準備した（よくわからない）
  * オフラインのデータを、オンライン学習、予測するためのストリーミングデータを模して用いる。
* 評価指標
  * 因子が機械学習モデルに与えるインパクトに最も関心があるので、利益、収入に直接関係する指標の代わりに予測精度を用いた。
  * この論文では、主な評価指標として、NormalizedEntropy(NE)と測定(calibration)を用いた。( `calibration` がよくわからん)
* Normalized Entropy(Normalized CrossEntropy)
  * impあたりの平均loglossを、impごとにbackgroundCTRを予測したときのimpあたり平均loglossの値で割ったものに等しい（謎）
  * 言い換えると、backgroundCTRのエントロピーで標準化した予測loglossである。
  * backgroundCTRは教師データの平均経験CTR(empiricalCTR)である。
    * empirical CTRは、実際のimp数とclick数に基づくCTRと思われる。真のCTRとは異なるということを示しているか。imp数が少なければ、真のCTRとの乖離は大きくなる。[Qiita参照](https://qiita.com/ysekky/items/7dfca3fb4e70b679727d#premiumな広告主をデータセットから取り除く)
  * もしかしたら、Normalized Logarithmic Lossを参照した方がわかりやすいかもしれない
  * 値が低ければ低いほど、モデルによる予測が良くなる
  * 標準化の理由は、backgroundCTRが0または1に近いほど、loglossが良くなりやすいから。
  * backgroundCTRのエントロピーで割るのは、NEをbackgroundCTRの影響を受けにくくする。
  * 数式の説明
    * $y$は-1 or 1
    * $p_i$は予測クリック確率
    * $p$は平均経験CTR
  $$ N E = \frac { - \frac { 1 } { N } \sum _ { i = 1 } ^ { n } \left( \frac { 1 + y _ { i } } { 2 } \log \left( p _ { i } \right) + \frac { 1 - y _ { i } } { 2 } \log \left( 1 - p _ { i } \right) \right) } { - ( p * \log ( p ) + ( 1 - p ) * \log ( 1 - p ) ) }$$
  * NEは本来、RelativeInformationGain (RIG,相対的情報利得)を計算する際の構成要素であり、 $RIG= 1 - NE$である。
* Calibration
  * Calibrationとは予測CTRの平均と経験的CTRの比率。
  * 言い換えると、予測クリック回数と、実際に観測されたクリック回数の比率。
  * Calibrationはオンライン入札、オークションにて重要な指標。
  * 1に近ければ近いほど良いモデル。
* Area-Under-ROC(AUC)
  * calibrationを考慮せずにランク順位を測るためには、AUCもかなり良い
  * 実際の環境では、単に最適なランク順位を得るだけでなく、予測が正確であることを期待している。潜在的な配信不足、または配信超過を避けるために。(そのためにはAUCは)
    * クリック確率を予測し間違えると、想定よりも少ない人しかLPに来てくれなかったり、人が来すぎる（コストかけなければよかった）ことがある、ということを言っていると思う。
  * NEは予測の良さを測り、暗にcalibrationを反映している。
  * 例えば、モデルが実際の2倍の値を予測したとする。calibrationを修正するためにglobal multiplierを0.5に設定したら、対応するNEは改善される。一方でAUCは変わらないままである。
    * 詳しくは[Predictive Model Performance:Offline and Online Evaluations](https://chbrown.github.io/kdd-2013-usb/kdd/p1294.pdf)