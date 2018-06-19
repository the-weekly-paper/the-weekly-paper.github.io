---
layout: post
title:  "論文紹介: FASTER: A Concurrent Key-Value Store with In-Place Updates"
date:   2018-06-20 12:00:00 +0900
categories: jekyll update
---

# FASTER

書いた人: [@nikezono](http://github.com/nikezono)<br>

### TL;DR
- 大量のデータがモバイル/IoT機器/ブラウザ/etc..から生成され，保存されている．これらを**ステート**という
- とにかくUpdateが多く，後からまとめて分析のためにReadされる，という特徴がある
- ステートを保持するためのストレージとしては，RDB/KVS/Streaming DBなどが使われている
- どれも性能が低い．数百万Request/Sec(RPS)出れば御の字
- そこで，Microsoftから高速なKVSを提案する．
- [Make the common case fast](https://scrapbox.io/nikezono/Make_the_common_case_fast)を設計原則とする．
- とにかく**キャッシュ効率**を限界まで追求している

#### Reference
> FASTER: A Concurrent Key-Value Store with In-Place Updates Badrish Chandramouli, Guna Prasaad, Donald Kossmann, Justin Levandoski, James Hunter, Mike Barnett 2018 ACM SIGMOD International Conference on Management of Data (SIGMOD '18), Houston, TX, USA ACM June 10, 2018<br>
Link: [https://www.microsoft.com/en-us/research/project/faster/#!publications](https://www.microsoft.com/en-us/research/project/faster/#!publications)<br>

本文はこちらで書きました:[https://scrapbox.io/nikezono/FASTER](https://scrapbox.io/nikezono/FASTER)
