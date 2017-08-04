---
layout: post
title:  "On Optimistic Methods for Concurrency Control"
date:   2017-06-11 18:17:58 +0900
categories: jekyll update
---

書いた人: @nikezono

# 要約

並行処理において読み書き両方でロックを使うアプローチを「悲観的並行制御」と呼び,
読み込みの際はロックを用いず, バリデーションをもってロックを使った際と挙動が等価であることを判定するアプローチ「楽観的並行制御」を提案する.

これにより参照ヘビーなワークロードでほぼロックを用いないためオーバヘッドを大きく削減できる.
しかしLong Transactionに対する耐性が課題として生まれた.

# ACM Refs

```
H. T. Kung and John T. Robinson. 1981. 
On optimistic methods for concurrency control. 
ACM Trans. Database Syst. 6, 2 (June 1981), 213-226. 
DOI=http://dx.doi.org/10.1145/319566.319567
```

# 楽観的ロック

このブログの最初のエントリは「楽観的ロック」の起源となる1981年のこの論文.

「楽観的ロック」といえば[Ruby on Railsでの実装パターン](http://ruby-rails.hatenadiary.com/entry/20141209/1418123425)など,
ユーザランドで実装する「ロックをバージョン比較で代用するアプローチ」だと思われがちである.
かくいう私もそうだった.

しかしこの論文で紹介された "Optimistic Concurrency Control(以下OCC)" はより汎用的な概念であり, OSやDBはもちろんトランザクショナルメモリ等の実装にも用いられるテクニックである. 

この論文では, OCCの基本アイデアとその実装手法を二種類, さらにOCCの問題点を説明している.

## 1. Introduction

以下の前提を置く.

* [ページング](https://ja.wikipedia.org/wiki/%E3%83%9A%E3%83%BC%E3%82%B8%E3%83%B3%E3%82%B0%E6%96%B9%E5%BC%8F)がなされ, メモリに載っているデータが二次記憶にスワップアウトしうる
* データベースとしては全てのデータがメモリに載っている(インメモリDB). かつマルチコアである


この場合, 二次記憶のデータを読みに行くのはトランザクション的に正しくない場合がある.
他のプロセスが管理している, スワップアウトしただけで書きだされていないページが実は最新のデータを持っているかもしれない.
となるとページとは別に**ロック**を使うことになる. 

ロックの実装方法にはいくつかのプロトコルがあるが, とにかく他のプロセスからのアクセスを拒否し, 専有状態を作り出すものである.

言うまでもなくロックはオーバヘッドがある.

1. Read only Transactionだけが発行されるのであればデータの一貫性は必ず壊れない. が, writeが挟み込まれた時には[一貫性は壊れうる](http://qiita.com/kumagi/items/5ef5e404546736ebac49).
    1. ということはReadでもロックを取らなければならない.
    1. さらにデッドロック回避までしなければならない. オーバヘッドは計り知れない.
2. デッドロックフリーかつ高性能なロックプロトコルが存在しない. 
3. 二次記憶に触りに行ったりすると, B-Treeの根(root)のロックをずっと握っている状況などが起きうる.
    1. これは全体のパフォーマンスへの影響が大きい.
4. トランザクションの失敗時にも, トランザクションが終わるまでロックを手放せない.
5. ここがこの論文で一番重要な部分: **ロックはworst caseでしか必要がないかもしれない**.
    1. 同じデータにアクセスするトランザクションが複数いてはじめてロックが必要となる.
    2. クエリによるが、参照が多く更新が少ないワークロードであればなおさらロックは必要ない.

この論文が出るまでの研究は**デッドロックフリーなロックをいかに効率的に実装するか**に注目があたっていたが、
この論文の論旨は**ロックを除去するアルゴリズム**を指向している.

そこで`Optimistic Approach for Concurrency Control` つまり楽観的並行制御というものを提案する.

アプローチはシンプル.

1. 読み出しは常に成功する. 何のオーバヘッドもかからない.
1. 書込みは二つのフェーズを持つ. `Validation` と `Write` である.
    1. `Validation` では読みだした値が変更されているかどうか, 一貫性を保っているかどうかを調べる.
        2. 厳密には, "仮に読み込みでロックを取っていた際とデータベースの状態が同じか" を判定する.
    1. `Write` ではvalidationに成功している場合書込み可能ということになる. 書込みを行う.

![](https://gyazo.com/f02094028b365da64709a741792a3a2c.png)

これだけでRead Lockを排除できる.



## 2. THE READ AND WRITE PHASE

ここからは実装の話になる. ここで出てくる擬似コードは現在のプログラミング環境やパラダイムとは少しズレているので, 要旨のみ説明して他を割愛する.

### Read Phase

**Read Phaseではディープコピーを行う**. これがOCC最大のポイントである.
OCCでは読み書きは直接In-placeに行わない. 
そのトランザクションが**一貫性のある読み書きをできているかどうかは`Validation`フェーズまでわからない**ため, `read` も `write` もローカルのコピーに対して行う. 
これを`read set`や`write set`と呼ぶ.

これが`validation`を通過した際には晴れて実際に`write`となる. 
これは普通にロックを取って実行していけばよい. もちろんポインタを書き換えてテーブルをまるっと入れ替えてもいいし, テーブルロックをとってもいい. そこは従来通り.

readしたものをまた読んだ際はrepeatable readのために`read set`から読むとか, それを書く場合`read set`から消して`write set`に入れるとか, 少し細かいプロトコルがあるが割愛.

## 3. THE VALIDATION PHASE

バリデーションと証明が書いてある.
依存関係グラフとトランザクションについての知識がなければ理解はかなり難しいため, 読み飛ばししても構わない.
要旨はシンプルで, 「ロックを取った場合では起こらない状態遷移になったらアボート」と考えれば良い.

### theorem

* 単調増加するユニークなTransaction Idを発行し、トランザクションに割り当てる.
* `if(n < m)`のとき, 先行関係は`Tx(n) -> Tx(m)`.
* ↑を詳細に書くと以下のどれか.
    1. nがmのRead Phaseより前にwrite phaseを終わらせる. (実時間的に直列実行)
    2. nのw/setがmのr/setと重複するエントリを持たない. かつnのw-phaseがmのw-phaseより先に終わる.
        * [出力依存](https://ja.wikipedia.org/wiki/%E5%91%BD%E4%BB%A4%E3%83%AC%E3%83%99%E3%83%AB%E3%81%AE%E4%B8%A6%E5%88%97%E6%80%A7)の関係が`n->m`で存在しうるが, トランザクションIDはそれを表現できているのでOK.
    3. nのw/setがmのr/w setとintersectしない. かつnのr-phaseがmのr-phaseより先に終わる.
        * nが読んだ値をmが書いている可能性があり, [逆依存](https://ja.wikipedia.org/wiki/%E5%91%BD%E4%BB%A4%E3%83%AC%E3%83%99%E3%83%AB%E3%81%AE%E4%B8%A6%E5%88%97%E6%80%A7)の関係が`n->m`で存在しうるが, `m->n`では存在しえないのでOK. 循環はしない.

この条件を満たすようバリデーションを実装すればよい.

1,2,3の条件は以下の図にあたる.

![](https://gyazo.com/ab78279a1f6471cd65dda6bce761f6e9.png)

#### トランザクションID

このときトランザクションIDをいつ払い出すか？ということが性能面でかなり重要.
ナイーブに考えるとRead Phaseの最初に払い出すことになるが, その場合事実上トランザクションの実行順序を開始時に決定しているのと同じである. ロックを取らない旨味はあるが, 実行スピードが違うトランザクションの先行関係を最初に決定してしまうのはOptimisticとはいえない. 速いトランザクションは若いIDを取るべきである.

厳密には, Valdiation Phaseの開始時にIDを払い出すのが最大限遅らせることができる限界となる.
上述した`(3)`の条件に含まれる、`nのr-phaseがmのr-phaseより先に終わる`はこの方法で自動的に満たせる.

## 実装

ここでは二種の実装を紹介している.
後世の論文で `forward validation` と呼ばれるものと `backward validation` と呼ばれるものである.
論点は "同時に実行されているトランザクションのread/write setに触ることがvalidationの大前提であるが、それをどうやって実現するか" である.

非常にシンプルな違いしかない:

* Backward Validation: 既に終わったトランザクションのr/w setを調べる.
    * 自分の `read set`と他人の `write set` の中身を比較して, 同じエントリに誰も触っていないことを確認する.
        * これが真であればread lockを取ったのと等価.
    * このとき調べる範囲は, 自分のTIDから現在最新のTIDまで.
        * 調べている最中にTIDが更新されると困るので, TIDの更新をストップさせる. 

![](https://gyazo.com/f4c6a247b78ce38c384ed47ef149a785.png)

* Forward Validation: 現在走っているトランザクションのr/w setを調べる.
    * 現在走っているトランザクションの`write set`に自分の`read/write set`と重複するエントリがないことを確認する
    * このとき調べる範囲は, 自分のTIDから現在最新のTIDまで.
    * TIDの更新をストップさせなくともよい.

![](https://gyazo.com/7390ff6e7620f10fa55690cc40acf4e8.png)

これはどちらも難しく書かれているが、現在では, backward validationのほうが圧倒的に容易に実装できるためそちらが使われている.

この論文のアルゴリズムは共有メモリではなくプロセス間でのロックやページの扱いを想定して書かれているため, **他のトランザクションの`read/write set`に触る**というのがまわりくどい書き方をされている.

が,インメモリDBでは共有メモリを扱えるため, Backward Validationはシンプルに以下のように実装できる.

以下は`x`と`y`を読み, `x`をインクリメントするトランザクションの実装.

```ruby
# Read Phase
my_tid = get_tid()

local_x = read("x")
local_y = read("y")

local_x_dirty = local_x++

# Validation Phase
lock(x);

latest_x = read("x")
latest_y = read("y")

if (latest_x.version != local_x.version)
  return ABORT
if (latest_y.version != local_y.version)
  return ABORT

# Write Phase
write("x", local_x, my_tid)
increment_tid()
```

現在OCCといえばだいたいこのように実装されることが多い. これはどちらかというとBackward Validationである.
つまり**他人のwrite setの中身を調べる**というのを共有メモリ上のデータの構造体のメンバのチェックで済ませることで実装している.

#### なぜバージョンを使うのか

validation phaseでバージョン比較をせず, 値が変わっているかどうかだけ調べれば良いという考えも当然ある. しかし, それでは厳密にvalidationをできないパターンがある.

詳しくは[ABA問題](https://ja.wikipedia.org/wiki/ABA%E5%95%8F%E9%A1%8C)と呼ばれる問題であるため, そちらを参照.

この理由によりOCCではロックの代わりに単調増加するIDというのは必ず必要である.
この単調増加するIDの払い出しというのも新たなボトルネックとなっており, [Silo][]ではそれを解決している.

## OCCの問題

OCCの問題は一言で言い表されている. `Starving Transaction` である. つまり「ロングトランザクションが全く通らない」ことにある.

実装の章で見たように, 仮にトランザクションIDの払い出しを`end of read phase or beginning of validation phase`とした場合, ロングトランザクションはIDを払いだされた時点で既に相当古い`read set`を持っている. これがvalidationを通過する可能性は薄い.

かりにread phaseの最初にトランザクションIDを払い出した場合でも, このトランザクションのread phaseの内容に引っ張られて、同じレコードを操作する他のトランザクションは軒並みabortしていく.
これでは普通の悲観的ロックと同じになってしまう.


# 所感

OCCはここ数年のデータベース界において注目度の非常に高い技術である.
その理由はインメモリデータベースの興隆にある.

従来のような[ディスクベースのデータベースのボトルネックはとにもかくにもディスクI/O回数](http://dl.acm.org/citation.cfm?id=1376713)であり, これを削減することがデータベース研究の主眼であった.

![](https://gyazo.com/f1752f2c9bab1f1699f666fbeb3ba0d0.png)
```
Stavros Harizopoulos, Daniel J. Abadi, Samuel Madden, and Michael Stonebraker. 2008. 
OLTP through the looking glass, and what we found there. 
In Proceedings of the 2008 ACM SIGMOD international conference on Management of data (SIGMOD '08).
ACM, New York, NY, USA, 981-992. DOI=http://dx.doi.org/10.1145/1376616.1376713
```
DBMSは多くの時間をI/Oやロック, ログ等に割いており, CPUが有効に扱える時間はほぼ無かった.

一方で, インメモリデータベースの時代には, [ボトルネックはCPUであり, キャッシュコヒーレンスである](http://dl.acm.org/citation.cfm?id=2735511)ことが報告されている. CPUの使用率が100%に張り付くのはインメモリDBでは正常の動作であり, その際にはキャッシュヒット率が極めて重要な性能指標となる.

この世界観では, 「Read Lockを取らない == Readでキャッシュが汚れない」ことを意味する.
Read Lockを実装するためには「値を読んでいる最中であること」を何かしらの方法で保存する必要があり, そのためにはレジスタに書き込むことが不可欠であるからだ. OCCを用いれば, 値やロックオブジェクトは読みだすだけでよく, キャッシュラインを`shared`ステートに保てる. これは非常にキャッシュヒット率, つまり現在のボトルネック解消に貢献する. 

もちろん, このアプローチを行った場合にもトランザクションの一貫性は変わらず守れる, OCCは実装のみで完結するhackである. ここがこの論文の最大の貢献である.

OCCは2013年に[Silo][]という名前でインメモリDB向けの実装としてリバイバルされ, いわゆるBackword Validationを行うOCCをベースラインとした手法がここ数年間で多く発表されている.

しかし, このSiloベースのOCCが市中のDBMSに搭載された例は(私の観測上で)ほぼ無い.
これはOCCが持つ本質的な問題である「Starvation」が解決されていないことが原因であると[指摘している論文もある](http://dl.acm.org/citation.cfm?id=2882903.2882905).

たとえば典型的なOLTPとして `INSERT` や `UPDATE` クエリが流星群のように飛んでくる環境で,

```
SELECT COUNT(*) FROM table WHERE ~
```

といったクエリが**ほぼ間違いなく通らない**となると、そのデータベースってどうなの? と思うことは想像に難くない.
MySQLやPostgreSQLといったOSSのRDBMSはこのようなクエリが両方共扱えることを想定しているソフトウェアであるため, OCCを導入することはあまり現実的ではない.

この問題(Starvation)に対処するために[適応的に悲観的ロックに切り替える手法](http://dl.acm.org/citation.cfm?id=3015276)が去年提案されるなど, このあたりは今現在でもホットな話題である.


また, ロックを取らない代わりにローカルなread setを使うためにdeep copyを行う必要があり, この`malloc`にかかるコストもけして安くはないということも[先日のSIGMODで主張された](http://dl.acm.org/citation.cfm?id=3064015)らしい.
`malloc`のコストがロックを取るコストより高くつくのであれば, 確かにOCCは全くの無駄である.

しかし, 1981年に発表されたこの論文がインメモリ && メニーコアの時代を予見して書かれたものであるということは非常に面白い. DBの研究の理論自体は30年前に終わっているとよく言われる(?)が, これもその証左の一つか.

# 参考

* [Optimistic Concurrency Controlについて](http://qiita.com/kumagi/items/aad314574e1986f7243b): @kumagi さんの説明
* [どこかの大学の講義資料](https://pdfs.semanticscholar.org/b9b2/e39c26f9870491bb770e4608fcd197d34edb.pdf)
* [その2](http://www.cse.scu.edu/~jholliday/COEN317S05/RamirezSlides.ppt)

[Silo]:http://dl.acm.org/citation.cfm?id=2522713
