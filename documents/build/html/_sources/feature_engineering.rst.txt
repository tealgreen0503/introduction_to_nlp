===============================================
特徴量エンジニアリング
===============================================

　本章では特徴量エンジニアリングについて説明します．

　以下のリンクでソースコードを試すことができます．

`feature_engineering.ipynb <https://colab.research.google.com/drive/15hQZ2f8QVLTmC6cU81OuMZ2dxmpahpFM?usp=sharing>`_

.. contents:: 目次

特徴量エンジニアリングとは？
===============================================

　**特徴量エンジニアリング** とは， **機械学習アルゴリズムの性能が高くなるような特徴（feature）を生データから作成するプロセス** のことです．特徴量エンジニアリングは機械学習を使ったシステムを作る際の基礎であり，ここで良い特徴量を得られれば，良い性能を出すことができます．

　特徴量エンジニアリングは自然言語処理をする場合だけだけでなく，どのタイプのデータに対する機械学習でも行われるものです．しかし，扱うデータのタイプにより，適切な特徴量エンジニアリングの手法は異なり，さらに同じタイプのデータに対しても，そのデータの特色により様々な手法の特徴量エンジニアリングが考えられます．

　今回は，多種多様な特徴量エンジニアリングの手法の中でも，自然言語処理における代表的な特徴量エンジニアリングの手法を紹介していきます．

Bag of Words (BoW)
===============================================

　前の章ではテキストデータに前処理を施しましたが，ただテキストデータを前処理しただけでは，それはただのトークンの羅列です．トークンの羅列をシステムが扱えるように表現する，つまり特徴量エンジニアリングを施していきましょう．

　最も単純なテキストデータの表現手法として **Bag of Words(BoW)** が挙げられます．BoWは，テキストの中の **それぞれの単語が現れる回数** を数えるだけの手法です．単語の出現頻度を数えることで，それぞれのテキストを数値の集まりとして表現，つまり **ベクトル** として表現します．

　簡単な例として，"when in Rome, do as the Romans do" と "all roads lead to Rome" の2つの英文を考えてみましょう．

.. code-block:: python

    corpus = ["when in Rome, do as the Romans do.",
              "all roads lead to Rome."]

　BoWはシンプルなので自力で実装もできそうですが，Pythonのライブラリの一つである :code:`scikit-learn` の :code:`CountVectorizer` を使えば簡単に実装できます．

.. code-block:: python

    from sklearn.feature_extraction.text import CountVectorizer

    count_vec = CountVectorizer()
    count_vec.fit(corpus)

　BoWに限らず，テキストデータをベクトル化するにはまず，コーパス全体に含まれる単語にそれぞれ番号を割り当てた **Vocaburary** （辞書）が必要です．上のコードで :code:`CountVectorizer` の :code:`fit` メソッドにコーパスを渡すことでVcaburaryを作成しています．Vocaburaryは :code:`vocabulary_` 属性で確認できます．

.. code-block:: python

    print("Vocabulary size: {}".format(len(count_vec.vocabulary_)))
    print("Vocabulary content:\n {}".format(count_vec.vocabulary_))

.. code-block::

    Vocabulary size: 11
    Vocabulary content:
    {'when': 10, 'in': 3, 'rome': 7, 'do': 2, 'as': 1, 'the': 8, 'romans': 6, 'all': 0, 'roads': 5, 'lead': 4, 'to': 9}

　では，BoWによってベクトル化された2つのデータを見てみましょう．

.. code-block:: python

    bag_of_words = count_vec.transform(corpus)
    bag_of_words = bag_of_words.toarray()

    print(corpus[0], "->\t", bag_of_words[0])
    print(corpus[1], "->\t\t", bag_of_words[1])

.. code-block:: none 

    when in Rome, do as the Romans do. ->   [0 1 2 1 0 0 1 1 1 0 1]
    all roads lead to Rome. ->              [1 0 0 0 1 1 0 1 0 1 0]

　このようにして，2つのテキストをBoWを用いてベクトル化することができました．2つのベクトルが似たような数値になっていれば，その2つのテキストの中には共通する単語が登場することを表しており，2つのテキストは似たような意味を持つと（ある程度は）考えられそうです．Vocaburaryを参照すれば元の文を復元できます．

.. note:: 

    　Vocaburaryを参照すればBoWを元の文に復元できますが，語順は元の文と同じには戻りません．このように，BoWは単語の並び順を全く考慮できていない点に注意しましょう．

　ところで，"I"，"you"，"the"などの一般的な単語はどの文にも多く登場しそうです．これらの単語が多く登場すれば，その単語ばかりが重視されることになり，私たちの意図する分析結果に悪い影響を与えそうです．ストップワードを用いて除去してもいいですが，ここででは，重要でない情報を削ぎ落とすのではなく，どの単語がどの程度情報を持っていそうかに応じて重み付けをしていく手法を試してみましょう．


tf–idf
===============================================

　単純なBoWの欠点を軽減する手法として， **tf-idf** （term frequency–inverse document frequency）があります．tf-idfとは，直観的に説明すると、特定の文書にだけ頻繁に現れる単語に大きな重みを与え、コーパス中の多数の文書に現れる単語にはあまり重みを与えない手法，ということになります．特定の文書にだけ頻出し、他の文書にはあまり現れない単語は、その文書の内容をよく示しているのではないか，という発想です．数式で表すと以下のようになります．

.. math::

    \mathrm{tfidf}(t,d)=\mathrm{tf}(t,d)×\mathrm{idf}(t)

    \mathrm{idf}(t)=\log\left(\frac{N}{1+\mathrm{df}(t)}\right)

　難しそうな数式が出てきましたが，順に説明しましょう．

　まず， :math:`\mathrm{tf}(t,d)` は文 :math:`d` における単語 :math:`t` の出現頻度を表しています．つまりデフォルトのBoWと同じです．

　tf-idfの重要な部分は :math:`\mathrm{idf}(t)` の部分です．上の式の :math:`N` はコーパスの中の全文数です．この値はどの単語に関わらず定数（今回の場合は2）ですから，重要な部分は :math:`\mathrm{df}(t)` の方です．:math:`\mathrm{df}(t)` は **単語** :math:`t`  **が現れる文の数** （document frequency）です．例えば"the"のような一般的な語句の場合，多くの文に登場するので，値が大きくなるはずです．そしてその :math:`\mathrm{df}(t)` が分母にあるので"the"という単語に対する :math:`\mathrm{idf}(t)` の値（そして :math:`\mathrm{tfidf}(t,d)` の値）は相対的に小さくなるはずです．

　このようにして，一般的な語句に対してはtf-idfの値は小さく，反対にその文特有の語句に対してはtf-idfの値は大きくなります．

.. note:: 

    :math:`\mathrm{idf}(t,d)` の分母に+1があるのは，分母が0で割られるのを防ぐためです． :math:`\log` に関してはあまり気にないでください．
    　
    また，tf-idfにはこれの他にも様々な計算方法があります．

　ではPythonでtf-idfを実装してみましょう．tf-idfは， :code:`CountVectorizer` の代わりに :code:`TfidfVectorizer` を使えば得ることができます．

.. code-block:: python

    from sklearn.feature_extraction.text import TfidfVectorizer

    tfidf_vec = TfidfVectorizer()
    tfidf_vec.fit(corpus)

　tf-idfによってベクトル化された2つのデータを見てみましょう．また，ここでは :code:`Pandas` を利用して単語ごとの数値をわかりやすくしてみましょう．

.. code-block:: python

    import pandas as pd

    vocab = tfidf_vec.get_feature_names()
    pd.DataFrame(tfidf.toarray(), columns=vocab).round(2)

.. image:: ./images/tfidf.png

　"rome"の数値を見てみましょう．"rome"は両方の文に共通して登場するので，tf-idfの値は他の単語と比べて，小さくなっています．また"do"に関しては，片方の文にしか登場せず，かつ1つ目の文には2回登場しているので，値が大きくなっています．この"do"は1つ目の文を表す単語として，大きな影響を与えていそうです．

n-gram
===============================================

　BoWの欠点は単語の順番の情報が完全に失われてしまうことです．このため正反対の意味を持つ "it's bad, not good at all" と "it's good, not bad at all" が全く同じ表現になってしまいます．このBoWの欠点を補う手法として **n-gram** が挙げられます．

　n-gram では任意の数 :math:`n` 個の連続するトークンを人まとまりとして考えます．一般に :math:`n=2` のとき **バイグラム（bigram）** :math:`n=3` のとき **トリグラム（trigram）** と呼びます．

　Pythonでn-gramを再現するには :code:`CountVectorizer` や :code:`TfidfVectorizer` の :code:`ngram_range` パラメータを設定します．

.. code-block:: python

    count_vec = CountVectorizer(ngram_range=(2, 2))  # bigramのとき
    count_vec.fit(corpus)

    print("Vocabulary size: {}".format(len(count_vec.vocabulary_)))
    print("Vocabulary content:\n {}".format(count_vec.vocabulary_))

    bi_gram = count_vec.transform(corpus)
    bi_gram = bi_gram.toarray()

    print()
    print(corpus[0], "->\t", bi_gram[0])
    print(corpus[1], "->\t\t", bi_gram[1])

.. code-block:: none

    Vocabulary size: 11
    Vocabulary content:
    {'when in': 10, 'in rome': 3, 'rome do': 7, 'do as': 2, 'as the': 1, 'the romans': 8, 'romans do': 6, 'all roads': 0, 'roads lead': 5, 'lead to': 4, 'to rome': 9}

    when in Rome, do as the Romans do. ->    [0 1 1 1 0 0 1 1 1 0 1]
    all roads lead to Rome. ->               [1 0 0 0 1 1 0 0 0 1 0]

　2つの文章に共通のトークンは登場しなくなりました．今回は語彙数が変化していませんが，一般的にn-gramを利用すると語彙数が爆発的に増加することが知られています．