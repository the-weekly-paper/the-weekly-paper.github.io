---
layout: post
title:  "論文紹介: HCI for Recommender Systems: the Past, the Present and the Future"
date: 2017-06-20 05:27:02 +0900
categories: jekyll update
---


## 要約
どのようにして自分の興味にあったものを見つけるか。
情報爆発の時代の進みに伴って90年代から情報推薦システムの研究は歩みを続けてきた。
研究初期にテーマとなったのは推薦アルゴリズムとF値を用いた予測評価の精度であった。その後、焦点はヒューマンファクターな部分、特にUXや結果透明性のためのビジュアライゼーションやユーザコントロールに移った。
本論文では推薦システムのHCIに関する部分を対象にしつつ、これまでの研究分野の解析や今後のトピックに関して論ずる。
さらに今後の焦点として、ユーザコントロール性、適応的なインタフェース、感情を加味した推薦、ハイリスクな対象領域に対する応用といったものを紹介する。

### ACM Refs
```
André Calero Valdez, Martina Ziefle, and Katrien Verbert. 2016. 
HCI for Recommender Systems: the Past, the Present and the Future. 
In Proceedings of the 10th ACM Conference on Recommender Systems (RecSys '16). ACM, New York, NY, USA, 123-126. 
DOI: https://doi.org/10.1145/2959100.2959158
```

## 概要

### 推薦システムのこれまで
推薦システムで最も成功したのはAmazonの「この商品を買った人はこんな商品も買っています」という商品推薦だろう。

推薦システムは大きく分けるとコンテンツベースのものと協調フィルタリングの2つに分けられるが、ドメイン知識を使ったKnowledgeベースのものやいくつかのアプローチを組み合わせたハイブリッドなものも存在する。

歴史的経緯を振り返ると、Tapestry[1]は協調フィルタリングという概念を最初に提唱したシステムとして知られている。これはEmailとNetnewsから有益な情報をフィルタリングするためのシステムであり、ユーザーがドキュメントにアノテーションを付けそれをTQL(Tapestry Query Language)と呼ばれるDSL(Domain Specic Language)とアノテーションを利用して情報を抽出するものであった。

Tapestryの二年後の1994年にResnickら[2]により、アイテム評価値からユーザー間の類似度を計算しその情報を用いて未知のアイテムの評価値を算出する、という現在の協調フィルタリングのベースとなるアイデアが提案された。

そこから20数年に渡って推薦分野の研究は進んできたが、論文数の遷移としては以下の図のように年々増えて来ており、特にReview論文は2005年と2010年頃の二回スパイクがある。
スパイクの理由として推薦システムのトピックに変化があったことがある。2005年頃にAdaptiveな推薦やメモリベースでなくユーザモデルを用いた推薦アルゴリズムが登場したこと、2009年頃にはSNSの情報を用いた信頼性指向の推薦システムが現れたことによるものと考えられる。

![](https://gyazo.com/707fe3d93f29e7ecb38a6bd6a83378f1.png)

### 推薦システムの評価指標の移り変わり
推薦システムの評価手法に関する最初のReview論文としてHerlockerらがまとめたもの[3]がある。当初は精度(Pricision)や再現率(Recall)をカバレッジとする手法が一般的であった。一方でAdomaviciusらの論文では、推薦システムの有用性や品質と同時に、現実のアプリケーションの中では、説明性,信頼性,スケーラビリティ,プライバシー問題がより大きな焦点となることが論じられた[4]。


### 現在
近年の研究の焦点は、推薦精度というよりも推薦の効用に関するものに遷移している。例えばビジュアライゼーションを組み合わせることで推薦の説明性や透明性を付け加えることによって、推薦されたものへの興味を増強するといったものになっている。

このような推薦をBlackboxにせず、ユーザインタフェースを通してアイテムに対する説明性をあげたり、推薦のコントロール性を提供している例としてTasteWeights[5]がある。
TasteWeightsはユーザに対して、現在の推薦で使われているアイテムの特徴と協調フィルタリングで使われる特徴を以下のように視覚化する。
さらに推薦に使われる重みを調節できるようなインタフェースも提供し、適合性フィードバックを通して嗜好を抽出して更新することができる。（図は[5]から引用）

![](https://gyazo.com/3cd13bea6b41ac0800cdc46ba719c4c1.png)

### 今後の焦点

これまでの推薦システムに関する論文で使われた著者キーワードを解析したところ以下のような結果となった。
前述したように2005年頃にadaptiveやuser modelといったキーワードが増えており、snsの普及に伴ってtrustが増え2009年頃にピークを迎えていることが見て取れる。

![](https://gyazo.com/0cd4da9c0efe8b0527d77f2f10cd41fc.png)

推薦アルゴリズムの改善を訴える研究者が多い一方で、HCIに関するものも未だに根強く続いていることが見て取れる。近年のReviewではユーザに焦点を当てたものもいくつか存在している([9]など)。

最後に上記コーパス解析から得た結果を踏まえて、HCIに関連するキーワードのなかから重要であるが未だに議論が不十分だと我々が考える4つのトピックについて論じることにしたい。

#### User control
現在のインタラクティブな推薦システムの研究は、推薦に使われるパラメータやユーザプロファイルをコントロール可能とすることに焦点を当てている。しかしこのようなコントロール性だけでは高い信頼性やプライバシーに関する問題の対処としては不十分である。さらなる研究として、どのような目的でデータを追跡したり考慮するのかをより高いレベルでユーザが調整可能にできるものが求められる。

#### Adaptive
推薦システムにおいて、レコメンデーションの精度を向上させる目的で、ユーザによるフィードバックを使ってコントローラブルなシステムを提供する場合がある。
しかし、いくつかの研究ではユーザの満足度に関して必ずしも高い推薦精度がユーザの知識レベル・興味・必要性といった要素と相関するわけではないことが示されている。
そのため既存のようなStaticなインタフェースでなくユーザのレベルに合わせた適合的なインタフェースに関する研究が求められている。

#### Affective
感情は意思決定に関して極めて重大な役割を果たしているといわれている。いくつかの研究グループによって実験的な環境における感情測定は行われているが、より現実的な感情認知モデルによる研究は未踏領域として存在している。
次世代の推薦システムのために、意思決定で重要となる感情情報と、文脈を加味した必要性を加味した研究が望まれる。

#### High-risk domains
電子商取引において最もユーザがリスクを思うのは特に必要でないものにお金を掛けることであり、そのようなものを推薦することは高いリスクを伴う。現在推薦システムが使われている分野は日用的に使うものが多く、医療や原子力プラントや株取引の計画といった不確実だったり危険性を伴う領域での利用は進んでいない。
このような難度の高い領域への応用を考えた際にどのように不確実性や危険性を視覚化したりユーザをコミュニケートしていけばよいかといった調査研究が行われるだろう。


## 所感
そもそもこの論文を読むきっかけとして、推薦システムを現実世界にデプロイしていくなかで問題になるだろうことをHCIの観点から一旦整理したいという目的があった。数年ぶりに推薦システム関連の論文を読んでみて、気になった点がいくつかある。

まず第一にこの論文では多様性や新規性、フィルターバブルといったトピックに触れられていない点がある。推薦システムにおける多様性に注目した研究は多数存在する。

有名どころをいくつか紹介すると、Lathiaら[6]は推薦システムで同じような推薦リストを提示し続けるとユーザーの満足度が低下すること、複数回推薦を行う場合はアイテムの多様性が重要であることを示しているし、Zieglerら[7]は協調フィルタリングにおけるトピックの多様性がユーザーに与える影響を調べ、トピックの多様性はユーザーベースの場合は効果がほとんどないが、アイテムベースの場合は約40%満足度をあげるとの報告を行っている。

この論文では精度を超えた推薦システムの効用について述べているが、そこで多様性に触れられていないのはなぜなのかという疑問がある。新規性やフィルターバブルについても同様のことがいえると思う。

考えられる理由としてはこのようなトピックが客観的に定義しにくい種類のものなので定量的に評価しにくいというのがあるかもしれない。
本当に新規性のあるものをユーザに提示した際のデータセットとなるようなものは存在できるのだろうか。


またHCIという文脈で推薦システムを考えた場合、提示されたアイテムを表示するユーザインタフェースにも目を向けるべきだと思っている。その意味でSocial Annotationにも触れておいて欲しかった。

Social Annotationとは以下の図のようなWeb上のコンテンツに付加される人物のアイコンやコメントといった社会的側面を持ったメタデータである。主に検索システムの領域ではあるが、情報を選別する際の一要素として重要な役割を果たしているという研究結果がいくつか発表されている。
また実際のWebアプリケーションとしてもFacebookやはてなブックマークといったソーシャルメディアで既に活用されている。

![](https://gyazo.com/5a483ed0f5d5e62daa4c513b848bb9f1.png)

Kulkarniら[8]はSocial Annotationがニュースリーディングに及ぼす影響についての評価実験を行い、

- Social Annotationは見知らぬ人物だと効果はないが、アルゴリズムや会社・組織によるannotationは一定の効果がある
- 友人からのannotation はクリック率・評価付け共に良い影響を与える
- 親しい人物、エキスパート、特定のコンテキストを持つ人物の場合に特に効果を発揮する

というような報告をしている。

推薦システムはユーザに対して何らかのアイテムを提供するが、そのアイテムに対してどのようなメタデータを付加すれば効果的なのか・どう行動を変えるのか、というような側面でもっと論じられてもいい。この点はこの論文の最後にあったAffectiveとHigh-risk domainsの観点からもっと深掘りされてもいい分野だと思う。


さて最後にちゃぶ台返しになってしまうが所感をひとつ。
残念ながら実用化されている情報検索システムや推薦システムでHCIの研究成果がクリティカルに影響を及ぼしている例を僕は今までみたことがない。GoogleにしてもAmazonにしても20年近く経過するが、サービス当初と比較してUIやインタラクションは大きくは変わっていない。

近年はDeepLearningなどの機械学習分野の発展が著しいこともあるとは思うが、この論文でも触れられていたように推薦システムの分野でのHCIに関する研究は下火になりつつある。
情報検索にしても情報推薦にしても、やはりメインになるのはそこで返される内容であり結果のリストであるというのはおそらく真理なのではないか。とはいえGoogleやAmazonの登場で細かいながらもHCIの研究が進んだことも事実だろう。

この論文が取り上げているユーザコントロール性・適応的なインタフェースといったトピックが現実世界に応用される可能性があるとすれば、情報を取得するためのユーザインタラクション、それ自体に何か変化をもたらすこと、つまりこれまでに得られた知見を生かした全く新しい情報推薦アプリケーションをHCIとバックエンドにあるアルゴリズムを組み合わせることによって作り上げるしかない気がするがどうなんだろうか。


## 引用
[1] David Goldberg, David Nichols, Brian M. Oki, and Douglas Terry. Using collaborative filtering to weave an information tapestry. Commun. ACM, Vol. 35, No. 12, pp. 61-70, December 1992.

[2] Paul Resnick, Neophytos Iacovou, Mitesh Suchak, Peter Bergstrom, and John Riedl. Grouplens: An open architecture for collaborative filtering of netnews. In Proceedings of the 1994 ACM Conference on Computer Supported Cooperative Work, CSCW '94,
pp. 175-186, New York, NY, USA, 1994. ACM.

[3] Jonathan L. Herlocker, Joseph A. Konstan, Loren G. Terveen, and John T. Riedl. 2004. Evaluating collaborative filtering recommender systems. ACM Trans. Inf. Syst. 22, 1 (January 2004), 5-53. DOI=http://dx.doi.org/10.1145/963770.963772

[4] Gediminas Adomavicius and Alexander Tuzhilin. 2005. Toward the Next Generation of Recommender Systems: A Survey of the State-of-the-Art and Possible Extensions. IEEE Trans. on Knowl. and Data Eng. 17, 6 (June 2005), 734-749. DOI: https://doi.org/10.1109/TKDE.2005.99

[5] Svetlin Bostandjiev, John O'Donovan, and Tobias Höllerer. 2012. TasteWeights: a visual interactive hybrid recommender system. In Proceedings of the sixth ACM conference on Recommender systems (RecSys '12). ACM, New York, NY, USA, 35-42. DOI=http://dx.doi.org/10.1145/2365952.2365964

[6] Neal Lathia, Stephen Hailes, Licia Capra, and Xavier Amatriain. Temporal diver-
sity in recommender systems. In Proceedings of the 33rd International ACM SIGIR
Conference on Research and Development in Information Retrieval, SIGIR '10, pp.210{217, New York, NY, USA, 2010. ACM.

[7] Cai-Nicolas Ziegler, Sean M. McNee, Joseph A. Konstan, and Georg Lausen. Improv-
ing recommendation lists through topic diversification. In Proceedings of the 14th
International Conference on World Wide Web, WWW '05, pp. 22{32, New York, NY, USA, 2005. ACM.

[8] Chinmay Kulkarni and Ed Chi. All the news that's fit to read: A study of social annotations for news reading. In Proceedings of the SIGCHI Conference on Human Factors in Computing Systems, CHI '13, pp. 2407{2416, New York, NY, USA, 2013. ACM.

[9] Joseph A. Konstan and John Riedl. 2012. Recommender systems: from algorithms to user experience. User Modeling and User-Adapted Interaction 22, 1-2 (April 2012), 101-123. DOI=http://dx.doi.org/10.1007/s11257-011-9112-x

記事を書いたひと:@ytanaka
