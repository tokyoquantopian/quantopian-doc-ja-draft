Quantopianパイプラインのチュートリアルへようこそ。このチュートリアルでは `pipeline API <https://www.quantopian.com/docs/user-guide/tools/pipeline>`__ を紹介します。
もしあなたがQuantopianに不慣れであれば `Getting Started Tutorial <https://www.quantopian.com/tutorials/getting-started>`__ から始め、少なくともPythonの動作知識を学習しておくことをお薦めします。
このチュートリアルはいくつかのレッスンに分かれており、それぞれのレッスンで異なるPipeline APIの機能に触れていきます。
レッスン2から11はResearch環境で動作します。レッスン12はIDE環境で動作します。

なぜ Pipeline なのか？ 
-------------------------
多くの取引アルゴリズムは、以下のような構造を持っています：

1. 既知の（大きな）データセット内に存在する資産に対し、一定区間データに基づくN個のスカラ値を計算する。
2. (1)の計算結果に基づき、取引可能な資産の集合を絞り込む。
3. (2)で絞り込みした資産の集合に対し、望ましい投資比率を計算する。
4. 現在のポートフォリオの投資比率が、(3)で計算した望ましい投資比率となるように発注する。

これらを頑健に実行するためには、いくつかの技術的困難が存在します。そこには、以下のようなものが含まれます。

* 大規模データセットに対する効率的な問い合わせ
* 大規模データセットに対する計算効率
* データ修正作業（株式分割や配当金）
* 資産の上場廃止作業

パイプラインは、さまざまなデータセットのコレクションに対する計算処理を表現するための統一されたAPIを提供することにより、こうした技術的困難を解決します。

Research と IDE
-------------------------

理想的なアルゴリズムデザインワークフローには、調査（Research）段階と、実装（Implementaion）段階が内在します。
Reserch環境内では、`notebook <https://ipython.org/notebook.html>`__ を通じてデータに触れたり、様々なアイデアを素早く試したりすることができます。
アルゴリズムは、バックテストが可能なIDEの中で実装されます。

パイプラインAPIの特徴のひとつは、パイプラインの構築はResearchとIDEの両方で同じであることです。
2つの環境内でパイプラインを使用する際の唯一の違いは、その実行方法です。
この特徴は、パイプラインをReserch環境でデザインし、それをIDE内のアルゴリズムに単純にコピーアンドペーストできることを可能にしています。
このワークフローは、後ほどのレッスンで詳細に議論する予定ですが、チュートリアル全体を通して見ていくことになるでしょう。

計算処理
-------------------------

パイプラインで表現される計算処理には、「ファクター（Factors）」「フィルタ（Filters）」「クラシファイア（Classifires）」の3種類あります。

抽象的にいうと、ファクター、フィルタ、クラシファイアで表されるすべての関数は、とある資産のある一時点における何らかの値を出力します。
ファクター、フィルタ、クラシファイアは、出力する値の種類によって区別されます。

ファクター
^^^^^^^^^^^^^^^^^^^^^^^^^

ファクターとは、とある資産のある一時点から数値を出力する関数のことです。

ファクターの単純な例は、とある証券の直近値段です。証券の銘柄名と特定時点を与えられることによって、その直近の値段が数値として返ってきます。
また別の例として、とある証券の10日間の平均出来高が挙げられます。
ファクターは数値を証券に割り当てるために最も一般的に利用され、いろいろな方法で用いられます。
ファクターは、以下のような処理において使われます。

* ターゲットとなる比率の計算
* アルファシグナルの生成
* より複雑なファクターの作成
* フィルタの作成

フィルタ
^^^^^^^^^^^^^^^^^^^^^^^^^

フィルタとは、とある資産のある一時点からブール型の値を出力する関数のことです。

フィルタの一例は、とある証券の値段が10ドル以下かどうかを表す関数です。証券の銘柄名と特定時点を与えられることによって、TrueまたはFalseによって評価されます。
フィルタは特定の目的に対し、資産の集合が該当するか否かを表現するために最も一般的に利用されます。

クラシファイア
^^^^^^^^^^^^^^^^^^^^^^^^^

クラシファイアとは、とある資産のある一時点から分類型の値を出力する関数のことです。

より明確には、クラシファイアは数値で表現することのできない、string型、あるいはint型（例えば数値でラベリングされた業種コードなど）を返します。
クラシファイアはファクター計算上の複雑な変換を行ううえで、資産をグループ化するために最も一般的に利用されます。
クラシファイアの一例は、とある資産の現在取引可能な取引所を返す関数です。

データセット
-----------------------

パイプライン処理は、`四本値や出来高 <https://www.quantopian.com/docs/data-reference/daily_pricing>`__ 、 `財務データ <https://www.quantopian.com/docs/data-reference/factset_fundamentals>`__ 、そして `コンセンサス予想データ <https://www.quantopian.com/docs/data-reference/estimates_consensus>`__ といった `多様なデータ <https://www.quantopian.com/docs/data-reference/overview>`__ を用いて実行することが可能です。
後ほどのレッスンで、こうしたデータセットを見ていくことにします。

典型的なパイプラインには通常、複数の計算処理とデータセットを含んでいます。このチュートリアルでは、10日間平均値段と30日値段の間で大きな値動きがあった流動性の高い証券を選別するパイプラインを構築していきます。
