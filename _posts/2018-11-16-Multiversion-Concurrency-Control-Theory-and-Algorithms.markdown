---
layout: post
title:  "論文紹介: Multiversion Concurrency Control - Theory and Algorithms"
date:   2018-11-16 12:00:00 +0900
categories: jekyll update
---

# FASTER

書いた人: [@nikezono](http://github.com/nikezono)<br>

### TL;DR
- Multiversion Concurrency Control(MVCC)というものがある.
    - 同じデータxについて，データベースが複数のバージョン(x1, x2, x3,...,xn)を保持できることを指す．
    - PostgreSQLもMySQLもOracleも猫も杓子もMVCCである．
- このMVCCで使われるべき，マルチバージョンの直列化可能性(Multiversion Serializability, a.k.a. 1SR)をはじめて定義した論文．
    - この論文以前は，より許容しうる組み合わせのパターンが少ない(空間が狭い)直列化可能性しかなかった．
    - ただ，未だにほぼ全てのソフトウェア/論文がそっちの(古いほうの)直列化可能性を使っている...計算量/エンジニアリング的に．
    - 取りうる最大の理論的な性能が定義された，という意味では価値のある論文.
- トランザクションの理論面で最重要レベルの論文であり，同時に，最も難しい論文のひとつ．


#### Reference
>Philip A. Bernstein and Nathan Goodman. 1983. Multiversion concurrency control—theory and algorithms. ACM Trans. Database Syst. 8, 4 (December 1983), 465-483. DOI=http://dx.doi.org/10.1145/319996.319998 <br>
Link: [https://sites.fas.harvard.edu/~cs265/papers/bernstein-1983.pdf](https://sites.fas.harvard.edu/~cs265/papers/bernstein-1983.pdf)<br>

本文はこちらで書きました:[https://scrapbox.io/nikezono/Multiversion_Concurrency_Control_-_Theory_and_Algorithms](https://scrapbox.io/nikezono/Multiversion_Concurrency_Control_-_Theory_and_Algorithms)
