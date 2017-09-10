---
layout: post
title:  "論文紹介: Translating Embeddings for Modeling Multi-relational Data"
date:   2017-09-11 03:10:00 +0900
categories: jekyll update
---

書いた人: @ytanaka

## 概要と紹介理由
最近ニュース推薦についてもう一度問い直している。そこで思うのは推薦システムにおいて重要なのは、推薦したいアイテムに対するコンテキストをいかに付加するかということなのではないかということだ。
要は、この記事はあなたが興味がある〇〇と関連してるとか、この記事の背景にあるのは〇〇なんだとか、そういったニュースに対してメタコンテキストになりうる知識が、モノを推薦する際には重要ではないかということである。

推薦システムはBag-of-wordsのような単純なデータ表現モデルが前提となってきたわけだが、それはいい意味ではモデルをシンプルに保つことによる保守性やスケーラビリティの担保につながったが、一方では推薦そのものに対するメタコンテキストの排除にもつながったといえる。

現行ほとんどの推薦システム、特にニュースアプリはタイトルとサムネイルでしか判断材料を提供しない。

ニュースに対する知識表現を見直すことはできないだろうか。

という問題意識でサーベイを始めた。
そしてふとしたきっかけで、そういえばセマンティックWebの研究はどうなったのか？と調べて行き着いたのが本論文である。


[TransE [NIPS'13] を実装（と実験再現）した](http://yamaguchiyuto.hatenablog.com/entry/2016/02/25/080356)

こちらに非常にわかりやすいブログ記事があるためアルゴリズムの解説や実装はこちらに譲りたい。ここでは完結な概要と、推薦システムからみた研究意義という観点から述べてみたい。

## ACM Refs

```
Antoine Bordes, Nicolas Usunier, Alberto Garcia-Durán, Jason Weston, and Oksana Yakhnenko.
2013. Translating embeddings for modeling multi-relational data.
In Proceedings of the 26th International Conference on Neural Information Processing Systems (NIPS'13), C. J. C. Burges, L. Bottou, M. Welling, Z. Ghahramani, and K. Q. Weinberger (Eds.).Curran Associates Inc., , USA, 2787-2795.
```

## 知識表現
知識表現(Knowledge Representation)は「推論を導けるような知識の表現、およびその方法を開発する人工知能研究の領域」とされている[1]。

ここでは深くは立ち入らないが（専門外なので）、意味を表現するためのデータ形式だと思ってもらえばいいと思う。

代表なものではRDFがある。
RDFはWeb上のリソースを表現する形式であり、トリプルという主語・述語・目的語の3つからなるリソースで関係情報を表現する。

例えば

>ytanaka-職業-プログラマ

>ytanaka-趣味-釣り

といった感じになる。

## Embedding
さて上記のようなトリプルと呼ばれるデータ形式は以前はグラフを使って扱われてきたが、それをベクトルを使って表現しようという手法がEmbeddingである。

トリプルの表現構造をベクトル空間にtranslateすることの利点は大きい。なぜならこれまでのuser/itemを使った単純なデータ形式を前提とした推薦システムの研究の枠組みに対しての応用と拡張ができるためだ。

マルチリレーショナルなモデルをデータモデルに取り入れるという研究はこれまでもあったが、シンプルな表現形式でかつ性能も良いことが差分として述べられている。


### 提案されているアルゴリズム

とてもシンプルで、トリプルの各要素をh,l,tとした場合(hとtはEntity,lはrelationshipを示す)

>v(h)+v(l) ≒ v(t)

このような関係が成り立つベクトル空間を算出することを目指す。

そして、この関係が成り立つようにベクトルを調整するには以下のような目的関数Lを最小化するように学習を行えばよい。

![](https://gyazo.com/b7f2c587b00e564bafdd09ca322fbc88.png)

この式のS'の部分がポイントで、これは以下のように、正例Sのhまたはtを別のエンティティにランダムに入れ替えることによって作り出された負例である。

![](https://gyazo.com/0f19b62c3076e88036b44eb23b9bed4d.png)

また、γは予め設定されたマージンパラメータで、d(h+l,t)は正例の距離、d(h'+l,t')は負例の距離を示す。

要は正例の距離が小さく負例の距離が大きくなればよい。
ちなみに論文では最小化のアルゴリズムはmini-batch SGDが使われている。


## 所感
TransEは1対Nや多対多となる関係の学習ができないという弱点があるが、これまでTransMやTransHといった拡張が提案されておりこの問題は克服されているらしい。

KnowledgeBaseの研究におけるTransEはシンプルな形式で精度も良好のためベースラインとしての位置付けとのことだが、個人的にはシンプルゆえの実装コストの低さと別分野への応用の可能性に注目したい。

推薦アイテムに対しての説明性(Explanation)を高めるという研究は以前から行われてきており、それが推薦の効用を高めるという結果は知られているわけだが、メタコンテキストを付加するためにこの論文で提案されているような手法を用いるというのが面白いのではないかなと思っている。

## 参考
[1] https://ja.wikipedia.org/wiki/知識表現
[2] https://ja.wikipedia.org/wiki/Resource_Description_Framework
[3] https://www.slideshare.net/unnonouno/deep-learning-48974928