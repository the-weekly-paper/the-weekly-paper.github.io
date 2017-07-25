---
layout: post
title:  "論文紹介: Feral Concurrency Control: An Empirical Investigation of Modern Application Integrity"
date:   2017-07-23 18:00:00 +0900
---

書いた人: @nikezono

# 概要

Webアプリケーションの興隆に伴って，多くのプログラミングフレームワークが生まれた．

データベースの見地から見て特に大きいのはORM(Object-Relational Mapping)を行うソフトウェアやフレームワークの出現と流行である．
ORMでは，従来のような `BEGIN TRANSACTION...` から始まるステートメントでデータベースアクセスを行うことはない．
また，ユニーク制約や外部キー制約といった**データベースに対する制約も，アプリケーションコードに記述する**といったプログラミングパラダイムが普及している．

このような環境において，並行性の制御はどのように行われているのだろうか？
歴史あるトランザクション研究が示してきたように，同じデータへの並行するアクセスは，データの一貫性を失わせる可能性がある．
となれば，一貫性を保証するための制約条件は，データベースに記述するのが当然だ，と考えられてきた．

しかし，[ActiveRecord][]などのORMは，データベースの制約条件やトランザクションを一切用いずに，独自の並行性制御(`Feral Concurrency Control`)を行う機構を実装している．
この論文では，このようなアプローチの有効性を検証している．
評価実験では，OSSアプリケーションを取り上げ，既に`BEGIN TRANSACTION...`のような明示的にトランザクションを実行するインタフェースは殆ど使われていないこと，
また，全評価セットの86.9%の制約条件はORMのみの並行性制御実装で守ることができていたことを報告している．

# ACM Refs

```
Peter Bailis, Alan Fekete, Michael J. Franklin, Ali Ghodsi, Joseph M. Hellerstein, and Ion Stoica. 2015.
Feral Concurrency Control: An Empirical Investigation of Modern Application Integrity.
In Proceedings of the 2015 ACM SIGMOD International Conference on Management of Data (SIGMOD '15). ACM, New York, NY, USA, 1327-1342.
DOI: https://doi.org/10.1145/2723372.2737784
```

# この論文を紹介する理由

データベースやトランザクションの理論は，直列化可能性(Serializability)を定理として，並行アクセスに関する異常(Anomaly)を除去することを可能としてきた．

しかし実態として，[ANSI SQL 分離レベルの標準](https://blog.acolyer.org/2016/02/24/a-critique-of-ansi-sql-isolation-levels/)が存在し，**異常が発生する代わりに性能が出る，というトレードオフの選択肢** をユーザに与えている．

今年開催された国際会議SIGMOD 2017の[Keynote](https://twitter.com/andy_pavlo/status/864473432850919424)においては，トランザクション研究がしばしば前提とする `Serializable` 分離レベルはほぼ使われていないことが報告された．

![](https://i.gyazo.com/bfdf49f2012eb58154ed2ae38a1936cf.png)
> What Are We Doing With Our Lives? Nobody Cares About Our Research on Transactions (sgm272k) Andy Pavlo (CMU)


これが性能のためか，MySQLやPostgreSQLのデフォルト設定であるためか，という問題は鶏と卵である．
特に分散システムにおいて，読取りロックやレンジクエリ用のロックを必要とする `Serializable` はスケーラビリティに乏しい．
対して，`Read Commited` などの弱い一貫性レベルは，結果整合性等の分散システムにおける一貫性と妥協点がある程度近しいため，分散システムにおいて選択しやすい(異論反論があれば教えて頂きたい)．
デフォルト設定が異常を含む設定であるのはこの性能を確保するためであるから，現状の `Serializable` は「わかっている人」でもためらう選択肢となってしまう．

さて，では現実のトランザクション異常はどこで対処されているか？
私の少ない人生経験の実感から言ってそれはアプリケーションのレベルである．少なくともデータベースではない．
特に，結果整合性でもアプリケーションのセマンティクスに矛盾が起きないようなWebアプリケーションの界隈では，データベースの性能は可能な限り引き出しておいて，異常の中でも起こってはいけない異常は，丁寧に開発者が潰してまわり，日夜バッチ処理でデータのサニタイズを行う，という手法が普及しているように思う．

このような世界では，優先されるのはバグがないことではなく，バグをどれだけ早く潰せるか，という開発スピードである．であれば，スケーラビリティをより高められる弱い一貫性レベルは，妥当な選択となる．

このような市場の選択に適合したのが[ActiveRecord][]をはじめとするORMである．いわゆる[オブジェクト指向モデリングとリレーショナルモデリングのインピーダンスミスマッチの解決](https://ja.wikipedia.org/wiki/%E3%82%A4%E3%83%B3%E3%83%94%E3%83%BC%E3%83%80%E3%83%B3%E3%82%B9%E3%83%9F%E3%82%B9%E3%83%9E%E3%83%83%E3%83%81)に加えて，本来データベーストランザクションの責務であるところの並行性制御まで実装されている．

本論文は，こうした「亜流/野生/Feral」のトランザクション実装をまがい物と切って捨てずに，性能評価を行うことで真摯に向き合っている．

## 背景

### インピーダンスミスマッチ

ORMはオブジェクト指向モデリングとリレーショナルモデリングとの間のインピーダンスミスマッチを埋めるシステムである．最も典型的には，[Ruby on Rails][]の重要なコンポーネントである[ActiveRecord][]は，TwitterやAirbnb，GitHub，Hulu,Shopify，Groupon，SoundCloud，Twitch，Goodreads，Zendesk，エトセトラエトセトラ，と言った多くの `central player`たちの間で利用実績がある．
Rails等のWAFやORMを通して，現状のWeb技術者というのはデータベース技術の恩恵を授かる `consumer` としてはとても大きいパイを占めている．

さて，Railsの面白い点は「Railsが `opinionated software` であることだ．Railsを開発したDHHは，以下のように述べている．

>  I don't want my database to be clever!... I consider stored procedures and constraints vile and reckless destroyers of coherence. No, Mr. Database, you can not have my business logic. Your procedural ambitions will bear no fruit and you'll have to pry that logic from my dead, cold object-oriented hands.

> Before the DBA-induced side of your brain explodes at that statement, please do read Martin Fowler's article on the difference between application and integration databases. And realize that my opinions are confined to dealing with application databases (and that doing integration through the database belongs in a time where Beverly Hills 90210 was a hit show on TV). Hopefully that calmed you down again.

> In other words, I want a single layer of cleverness: My domain model.

> Choose a single layer of cleverness, http://david.heinemeierhansson.com/arc/2005_09.html

これはいわゆる"NoSQL"ムーブメントの興隆とも一致する見解だろう．
データベースに制約条件やインデックスを書くというのは，ビジネスロジックの記述に片足(パフォーマンスの見地では，両足を)突っ込む行為だ．対して，肝心のアプリケーションコードのほうにも，もちろんビジネスロジックはある．
この構造は往々にしてアプリケーションかデータベースのどちらかが更新に追従できなくなり，しだいに腐っていく(`database decay`)原因であることは[Stonebraker御大も語っている通り](https://cacm.acm.org/blogs/blog-cacm/208958-database-decay-and-what-to-do-about-it/fulltext)である．

であればいっそ，抽象化のレイヤは一つにして，アプリケーション開発者に全てを書かせて，SQLクエリと格闘するDBAは必要ない，というのもわかる話ではある．

さて，そこでデータベース研究者が持つ一つの疑問は，「**データベースに制約を記述せずに，データベースのACID特性を満たすことはできるのか？**」である．

### 制約の記述

> 以下注:
> この論文の公開当時の議論を記述しています．
> 最新のActiveRecord等のORMの実装とは異なっているかもしれません．

ActiveRecord等のORMは `uniqueness` や `belongs_to` といったDSLで，DBMSのユニーク制約や外部キー制約にあたるものを提供している．
しかし，これらの制約はDBMSに対して透過的である．DBMSのDDLを追加するのではなく，**アプリケーション層でそのまま並行制御や制約の検証を実行する** 仕組みである．

たとえば，RailsにおいてUniquness制約は以下のように実装されている．

![](https://gyazo.com/63eec18ee41884d613e7e2efeabfc7c2.png)

`SELECT` 文を発行し，その結果を検証することでバリデーションを行う．結果が空であれば，`INSERT`を発行する，といった具合である．

言うまでもなく，ここでは二つのトランザクションを実行している．`SELECT` と `INSERT` の二つのクエリを実行する間に，他の`INSERT`が実行されれば，この制約は容易に崩れる．
教科書的には，この二つのクエリは一つのトランザクション文として実行されるべきである．また，一つのトランザクションとして実行していても，バリデーションをアプリケーション層で行ってしまうと，`READ COMMITTED`等の分離レベルにおいては同様の問題が起こりうる．

このような特性はデータベースの一貫性という観点では明らかに問題である．
が，しかし，このような制約条件やバリデーションをアプリ側で実行することにはメリットがある．完全にステートレスなアプリケーションであれば，いくらでもインスタンスを増やすことができる． 一方で，状態を保存するデータベースというのは原理的にスケールアウトには向かない． アプリ側でバリデーションを実行すれば，ある程度の不確実性とは引き換えに，スケールアウトの可能性を得る，というトレードオフがある．

Railsには，ユニーク制約以外にもいくつかの制約が提供されている．

* *Transaction*: DBMSのTransaction句を呼ぶ．範囲はブロックスコープと一致する．分離レベル等はDBMS依存．
* *Lock*:
    * *Pessimistic*: `SELECT FOR UPDATE`を呼ぶ．
    * *Optimistic*: `lock_version` フィールドを作り,commit時にincrementする.　読み出し時にチェックする．
* *Validation*:                   
    * ユーザ定義関数で記述する. built-inのDSLとしてunique制約なども提供する．
* *Associations*:
    * RDBMSの外部キーと同等の機能を提供するが、実際にDBMSの外部キーは宣言されない．

このうちTransactionとLockはDBMSの機能を利用しているが，ValidationとAssociationはDBMSとは一切連携をせず，アプリケーション層でのみ完結するアプローチである．        

これらのORM上に実装された **Feral Concurrency Control** の実際的な効果と，その普及の理由について探るのがこの論文の本旨である．

## 実験

論文では，OSSのアプリケーション67を対象として，**並行アクセスが行われた際に，ORM(アプリケーション側)で記述した制約条件がどれだけ守られるか** を調査した．

まず，67のアプリケーション中で行われた制約条件の宣言から:

![](https://gyazo.com/e4bdcfa36417c215378309b54405e62d.png)

各プロジェクト内で宣言されていたモデル/トランザクション/バリデーション/アソシエーションの一覧である．
モデル数順にソートしてある．プロジェクトによって満たすべき機能要件が異なるため，モデル数とバリデーションやアソシエーション数に相関があるというわけではない．が，**トランザクションを明示的に記述するプログラミングスタイルがもはや受け入れられていない** ことは明らかである．

続いてユニーク制約．前述した仕様で実装されているため，並列アクセス数が高まれば高まるほど，ユニーク制約違反のレコード数は増えてくると考えられる．

![](https://gyazo.com/0a887052e731b5e16aaf8966c0b1bc97.png)

図ではRailsのプロセス数の増加に伴ってユニーク制約に違反するレコード数が増大することがわかる．興味深いのは，この実験では，**2並列で動作している時点で，既に100件ものレコードがユニーク制約に違反していること**，それに加えて， **Feral Concurrency Control無しでは，桁違いの制約違反が発生している** ことだ．
Feral Concurrency Controlは完全ではないが有効なアプローチだということである．

Association制約も同様の傾向である．
![](https://gyazo.com/c1cef201a4b08871c7d996c653c548e7.png)


一方で，著者Peter Bailisの研究成果である[i-Confluence][]テストによれば，全体の86.9%の制約条件は，Feral concurrency Controlを用いても守ることが可能なものであった．

![](https://gyazo.com/e1ba9b3bfdaddfc2da54f5618a917c72.png)

i-Confluence Testingの説明は省略するが，ユニーク制約などは難しくとも，データベース層で行わずとも分散実行して有効なバリデーション(たとえば，「入力フォームの内容が電話番号として正しいか」等)は存在する．Railsビルトインの制約条件の多くはこの類型に該当するものであった．


## 結論

Feral Concurrency Controlが普及しているからと言って，データベースのACID特性はもはや必要ない，ということは意味しない． ACIDを満たすことは依然として重要である.
このような不完全なフレームワークの普及から透けて見えるのは，以下のような知見である．

* アプリケーションの制約条件をいかに満たすか，が何よりも重要．
    * データベースとアプリケーションの両レイヤにこの責務を追わせるのは，開発者フレンドリーではない．
* データベーストランザクション研究の金字塔である`Serializable`の考え方はオーバーヘッドが大きすぎる．
    * 分散環境ではまともに動かない．
* だからこそ，Web scaleのアプリケーションでは弱い分離レベルとFeral concurrency controlが用いられる.
    * 86.9%のcorrectnessでもスケールアウトできるなら良いという考え方.
    * 結果として起こる少数のバグは，開発者がすぐに対処できればよいのだから．

このような現状において、市場に足りない技術は以下であると考えられる．
1. **Express correctness criteria in the language of their domain
model, with minimal friction, while permitting their automatic
enforcement.**
    * トランザクションの明示的な宣言はもはや誰も使っておらず，アプリケーションの制約条件をバリデーション等の形で記述することは開発者に優しい．
    * このような開発者によるクライテリアの記述がそのまま厳格に履行されるシステムであること．
2. **Only pay the price of coordination when necessary.**
    * スケールアウトできることが何よりも重要である．古き良き`Serializable`のような保守的な理論に基づく実装では，全ての`read``write`命令を全ノードで協調する必要が出てくる．
    * 同期が必要な制約のみ同期する，というアプローチが適切だろう．
3. **Easily deploy to multiple database backends.**
    * 既に市中に存在するORMはこれを満たしている．

これら全てを併せ持つ技術が，ORMないしORMフレンドリーなDBMSとして出てくることが求められる．
このような領域で研究していくことは新しいチャレンジとなる．

# 所感

本論文の著者[Peter Bailis](http://www.bailis.org/)はSIGMOD 2017でもJim Gray Awardを受賞した気鋭の研究者である．
他の論文を見ても，「トランザクション研究の新たな研究フィールドは，よりアプリケーションに近い場所，すなわちORM等のレイヤである」という意識を随所から感じられる．

その問題意識を端的に表現しているのが論文７章の一節である:


>Summary. In all, the wide gap between research and current prac-
tice is both a pressing concern and an exciting opportunity to revisit
many decades of research on alternatives to serializability with an
eye towards current operating conditions, application demands, and
programmer practices. Our proposal here is demanding, but so are
the framework and application writers our databases serve. Given
the correct primitives, database systems may yet have a role to play
in ensuring application integrity.

これは全く持ってその通りである．この問題意識はWeb開発者やアプリケーション開発者からすると当然だが，データベース研究者やDBAからするとそうではない．

逆に，Web開発者にとっては，アプリ層のバリデーションをすり抜けて，問題のあるレコードが日に数件保存されてしまう，ということは，問題でもなんでもない．その程度はバッチ処理やアドホックスクリプトで対処できる問題だし，これを回避するためにDBMSの設定を変えて，スキーマを変更して，結果として性能が落ちる，ということのほうが大問題である．(少なくとも私は開発者としてはそう考える．)
が，DBMS研究の，特にトランザクションの理論は，このような数件のレコードを **完全にオンラインで** ゼロにすることを前提，目標として研究を進めてきた．

現在のDBMSの大きな消費者であるWeb開発者の意識と，DBMS研究者の思想の乖離．
この論文は，この問題意識が現実のものであることを研究界隈に知らしめるという意義でも非常に重要だと思われる．


# 参考資料


* Peter Bailis, Alan Fekete, Michael J. Franklin, Ali Ghodsi, Joseph M. Hellerstein, and Ion Stoica. 2014. Coordination avoidance in database systems. Proc. VLDB Endow. 8, 3 (November 2014), 185-196. DOI=http://dx.doi.org/10.14778/2735508.2735509
    * i-Confluence Testingの詳細．
* [2017 Jim Gray Award Talk: Coordination Avoidance in Distributed Databases](https://speakerdeck.com/futuredata/2017-jim-gray-award-talk-coordination-avoidance-in-distributed-databases)
    * Bailis氏のJim Gray Award受賞発表スライド．上記論文の内容．
* [Ruby on Rails][]
* [ActiveRecord][]
* [Database Decay and What To Do About It](https://cacm.acm.org/blogs/blog-cacm/208958-database-decay-and-what-to-do-about-it/fulltext)
    * Stonebraker先生のコラム．問題意識が通底している．

[Ruby on Rails]: http://rubyonrails.org/
[ActiveRecord]: https://github.com/rails/rails/tree/master/activerecord
[i-Confluence]: http://dx.doi.org/10.14778/2735508.2735509
