---
title: "ディープラーニング実践 - 手書き数字の分類"
---

この章のゴールは、ディープラーニングを使ったモデル作成の基本的な流れを理解することです。
実際に画像分類モデルを作成しながら学びます。

# 開発環境

:::details 前章と同じです。

[uv](https://zenn.dev/malt03/books/d3f6cd9caa864a/viewer/999_environment#uv) をインストールしてください。

:::

### ディレクトリ

本章では、`./mnist` 配下のコードを解説します。  
ディレクトリに移動し、 `sync` を実行してください。

```sh
cd ./mnist
uv sync
```

# 実行してみよう

### 学習データの表示

今回は、MNIST と呼ばれる手書き数字の画像データを使って、画像分類モデルを作成します。
まずは、学習データを表示してみましょう。

```
uv run ./src/show_mnist.py
```

初回実行時はデータのダウンロードが必要なため、少し時間がかかります。
その後、手書き数字にラベル付けされたデータが表示されます。

今回は、このデータを使って 4 と 9 を分類するモデルを作成します。
4 と 9 は形が似ているため、分類が特に難しい数字の組み合わせです。

![4-9](/images/d3f6cd9caa864a/2_mnist/4-9.png)
_どっち？_

### 学習

それでは、学習を実行します。
今回は少し時間がかかります。

```
uv run ./src/train.py
```

40 エポック実行され、以下のような画像が表示されます。

![result](/images/d3f6cd9caa864a/2_mnist/result.png)

# 解説

### 学習曲線

出力された学習曲線について説明していきます。

このグラフには、各エポックごとの学習の状況を記録して表示しています。
こういったグラフは、実際の学習でも必ず作成して学習がうまくいっているかを確認します。

グラフには 3 つの曲線があります。

| 線の色 | 説明                             |
| :----: | :------------------------------- |
|   青   | トレーニングデータに対する損失値 |
|   緑   | テストデータに対する損失値       |
|   赤   | テストデータに対する分類の正解率 |

新しいキーワードとして「トレーニングデータ」と「テストデータ」が出てきました。

### トレーニングデータとテストデータ

ディープラーニングでは、学習に使うデータを 2 つに分けて使います。1 つ目をトレーニングデータ、2 つ目をテストデータと呼びます。

トレーニングデータとは、モデルを学習させるために使われるデータです。モデルはこのデータを使ってパターンを見つけ、分類や予測を行えるようになります。いわば「勉強用の教材」のようなものです。
前章の 2D ポイント分類モデルでは、全てのデータをトレーニングデータとして使っていました。

一方、テストデータは、学習したモデルを評価するために使われるデータです。
テストデータはトレーニングそのものには使わず、各エポックで学習が終わった後にモデルの性能を確認するために用いられます。

トレーニングデータとテストデータを分けて用いる理由は、モデルが勉強した内容を暗記するだけではなく、新しいデータに対しても適切な予測を行えるかをチェックするためです。
これを確認することで、モデルの汎化性能（新しいデータに対する対応力）を評価することができます。

### 具体的な学習曲線の解釈

- **青の線（トレーニングデータの損失値）**  
  学習が進むにつれて、トレーニングデータに対する損失値が徐々に下がるのが理想です。
  モデルがトレーニングデータを学習できているか確認できます。
- **緑の線（テストデータの損失値）**  
  テストデータの損失値も、トレーニングデータと同じように下がるのが望ましいです。トレーニングデータよりもやや高い値になるのが一般的で、汎化性能を確認できます。
  学習が進むと、テストデータの損失値は増加に転じます。これは**過学習**が始まったことを意味します。
- **赤の線（テストデータの分類の正解率）**  
  損失値だけでは、学習が進んでいるかを直感的に判断しにくい場合があります。正解率は、モデルがテストデータをどれだけ正確に分類できたかを割合で表したもので、損失値の補助的な指標として活用されます。  
  正解率が高いほどモデルの分類能力が高いことを示します。

#### 過学習

ディープラーニングでは、学習が進むと必ず**過学習**が発生します。過学習とは、モデルがトレーニングデータに過剰に適応し、テストデータに対する損失値が増加し始める現象です。

過学習の状態では、モデルはトレーニングデータ特有のパターンまで学習してしまうため、汎化性能が低下します。トレーニングデータに対しては高精度なモデルに見えますが、テストデータに対する性能は下がります。

しかし、**過学習はネガティブなことではありません**。ディープラーニングのモデルは学習が進めば必ず過学習を起こすため、逆に過学習が全く起きていないということは、まだモデルがトレーニングデータを学習しきれていないということです。
この場合、さらに学習を進めたり、モデルをより複雑にすることで、トレーニングデータから多くのパターンを学習させる必要があります。

### ディープラーニングの基本的なフロー

ここで、ディープラーニングのモデルを作成していくときの基本的なフローを解説しておきます。
モデルを作成する際、最初から学習がうまくいくことはありません。学習がうまくいかない場合に対処すべき問題を、解決すべき順番に、対処方法とともに解説します。

#### 1. トレーニングデータの損失値が下がらない

この問題は簡単に対処が可能です。
ディープラーニングでは、ランダムな値さえも学習できることを思い出してください。
あなたが試すことはただ一つ、モデルのパラメータを増やしてみることです。
層を増やしたり、各層が持つパラメータを増やしてみましょう。
学習時間は増加しますが、トレーニングデータに対する損失値は**必ず**下げることが可能です。

:::message
計算リソースが足りない場合は、高性能なマシンやクラウドサービス（例: Google Colab）を活用すると良いでしょう。
:::

#### 2. テストデータに対する正解率がベースライン以上にならない

学習の際には、**ベースライン**を定めることが重要です。  
ベースラインは、モデルが最低限達成すべき性能の基準であり、ランダムに選択した場合に達成できる値を基に設定します。

今回の場合は 2 クラス分類であるため、ランダムに選択しても正解率は 0.5 になることから、**「正解率 0.5 以上」をベースライン**とするべきです。
トレーニングデータに対する損失値は下がるのに正解率がベースラインを超えられない場合、モデルが入出力を記憶しているだけで、汎用的なパターンを学習できていないことを示しています。この状態ではモデルがランダムな値を出力しているのと変わりません。

これはかなり深刻な問題です。このような場合、以下の点を確認しましょう。

- **この問題は客観的に見て学習が可能な問題か**  
  例えば、円周率の次の桁の予測をするモデルは学習することはできません。
- **学習データは正しくラベリングされているか**  
  データのラベルに誤りがあると、モデルが正しいパターンを学習できません。データセットを一部確認し、ラベルが正確かどうかをチェックしましょう。
- **実装は正しく行われているか**  
  損失関数が問題に適したものになっているか、データの入力は正しく行われているか再確認しましょう。

#### 3. 過学習が起きない

上記の通り、ディープラーニングでは学習が十分に進むと必ず過学習が発生します。過学習が全く起きない場合、モデルがトレーニングデータに含まれるパターンを十分に学習しきれていない可能性があります。

この状態では、モデルの予測性能が十分に高まらないため、以下のような確認と対処が必要です。

- **学習率を上げる**
  単純に学習が遅すぎる可能性があるので、学習率を調整しましょう。
- **エポック数を増やす**
  学習回数が不十分だと、モデルがトレーニングデータを十分に学習しきれません。エポック数を増やして学習を継続し、モデルがデータに含まれるパターンをさらに学習できるようにしましょう。
- **モデルのパラメータをさらに増やす**
  モデルの複雑さが足りない場合、データ内の複雑なパターンを学習することができません。層を追加したり、各層のパラメータ数を増やすことで、モデルの学習能力を向上させることができます。
- **テストデータにトレーニングデータが紛れ込んでしまっていないか確認する**
  テストデータとトレーニングデータが重複している場合、過学習が起きないように見えることがあります。これは、モデルがトレーニングデータを暗記しているためです。データセットを再確認し、重複がないように分割を適切に行ってください。

#### 4. すぐに過学習に陥る

モデルがベースラインはクリアするものの、すぐに過学習に陥ってしまいモデルが必要な性能を満たさない場合です。ディープラーニングでモデルを作成する際には、多くの労力はここに費やされます。ここからが本番だと言ってもよいでしょう。
一般的に以下のような対処方法がありますが、これ以外にもたくさんの手法が存在します。

- **データを増やす**
  学習に利用するデータが少ない場合、十分にパターンを学習できません。
  可能であれば、新しいデータを収集したり、既存のデータセットを拡張したりしてください。
- **学習率を下げる**
  学習率が大きすぎるとモデルが振動し過学習にも陥りやすくなります。適切な値を見つけることが重要です。
- **モデルのパラメータを減らす**
  モデルが必要以上に複雑な場合、すぐにトレーニングデータ特有のパターンを学習してしまいます。層の数や各層のパラメータ数を減らしてみましょう。
- **モデルの設計を考える**
  あらゆるデータは全結合層のみでも学習可能ですが、多くの場合、データの特徴によってそれに最適化された層や構造が存在します。学習させたいデータの特徴を理解し、それに合わせた先行実装がないか調査しましょう。
  以下に例を挙げます。

  | データの特徴 | 具体例                 | 実装                                                                               |
  | :----------- | :--------------------- | :--------------------------------------------------------------------------------- |
  | 2 次元       | 画像、オセロ           | [畳み込み層](https://pytorch.org/docs/stable/generated/torch.nn.Conv2d.html)       |
  | 順序を持つ   | 株価、天気、文章       | [Transformer](https://pytorch.org/docs/stable/generated/torch.nn.Transformer.html) |
  | グラフ構造   | ソーシャルネットワーク | [GNN](https://pytorch-geometric.readthedocs.io/en/latest/)                         |

- **正則化を実装する**
  過学習を抑えるための数学的な手法で、特に以下の二つの手法は必ずと言って良いほど使われます。本書でもオセロ AI を実装する中で出てくるので、キーワードだけ押さえておいてください。
  - L2 正則化: オプティマイザの `weight_decay` というパラメータを調整します
  - ドロップアウト: ランダムに一部ニューロンを無効化します
- **損失関数を見直す**
  損失関数には様々なバリエーションがあります。例えば回帰問題で使われる損失関数にも以下のような種類があり、データの特徴に合わせて使い分けます。

  |                                      名前                                      | ペナルティの計算方法              | こんなデータに使う                     |
  | :----------------------------------------------------------------------------: | :-------------------------------- | :------------------------------------- |
  |   [MSELoss](https://pytorch.org/docs/stable/generated/torch.nn.MSELoss.html)   | 誤差の二乗                        | ノイズや外れ値が少ない                 |
  |    [L1Loss](https://pytorch.org/docs/stable/generated/torch.nn.L1Loss.html)    | 誤差の絶対値                      | ノイズや外れ値が含まれる               |
  | [HuberLoss](https://pytorch.org/docs/stable/generated/torch.nn.HuberLoss.html) | 小さいときは MSE、大きいときは L1 | 外れ値が含まれるが、それほど極端でない |

- **活性化関数を見直す**
  活性化関数には一般的に [`ReLU`](https://pytorch.org/docs/stable/generated/torch.nn.ReLU.html) が使われてきましたが、それに変わる様々な活性化関数が開発されています。筆者が試す中では [`Mish`](https://pytorch.org/docs/stable/generated/torch.nn.Mish.html) は良い性能を発揮することが多いです。`ReLU` に比べて計算コストが増加しますが、可能ならば試してみるのが良いでしょう。
- **オプティマイザを見直す**
  活性化関数と同様に、オプティマイザも様々なものが開発されています。[`Adam`](https://pytorch.org/docs/stable/generated/torch.optim.Adam.html) はとても良いオプティマイザなので、とりあえず選ぶには最適な選択でしょう。
  他にも [`AdamW`](https://pytorch.org/docs/stable/generated/torch.optim.AdamW.html) や [`RAdam`](https://pytorch.org/docs/stable/generated/torch.optim.RAdam.html)、最新のものでは [`Lion`](https://github.com/lucidrains/lion-pytorch)（PyTorch には未実装）というオプティマイザが開発されたりしています。

# コードを読む

今回も main 関数から見ていきます。

```python:train.py
def main():
    train_data, test_data = load_data()

    model = Mnist()
    train_losses, test_losses, test_accuracies = train_model(
        model, train_data, test_data
    )
    show_results(train_losses, test_losses, test_accuracies)
```

ここの実装は前章と変わりません。
今回も `load_data` の中身は重要ではありませんが、 `train_data` と `test_data` が返ってきていることに注目してください。
トレーニングデータとテストデータが、それぞれ画像の入力とそれに対するラベルの配列を持っています。

### `Mnist`

MNIST を分類するためのモデルを作成しています。

#### `__init__` メソッド

```python:train.ppy
def __init__(self):
    super(Mnist, self).__init__()
    self.conv1 = torch.nn.Conv2d(1, 4, kernel_size=3)
    self.conv2 = torch.nn.Conv2d(4, 8, kernel_size=3)
    self.conv3 = torch.nn.Conv2d(8, 16, kernel_size=3)

    self.fc = torch.nn.Linear(16, 1)
```

今回は 2 次元の画像データですから、畳み込み層（Conv2d）を利用します。
畳み込み層を重ねて作成したモデルを畳み込みニューラルネットワーク（CNN）と呼びます。

畳み込みニューラルネットワークでは、データは以下の図のように変換されていきます。

![CNN](/images/d3f6cd9caa864a/2_mnist/CNN.png =150x)

畳み込み層は、画像全体をカーネルと呼ばれる、3x3 や 2x2 の小さなまとまりに分解し、それを深さを持った 1 つのピクセルに変換します。
上記の図では、2x2 のカーネルを 4 の深さを持ったピクセルに変換しています。
変換したデータを、またカーネルに分解してさらに深い 1 つのピクセルへとまとめていきます。

これにより、上の層では画像の部分部分の特徴を学習し、下の層に行くに従って画像全体の特徴を学習することができます。

最後に、1x1 の深いデータを全結合層を使って出力に変換します。

#### `__forward__` メソッド

```python:train.py
def forward(self, data):
    data = self.conv1(data)
    data = torch.relu(data)
    data = torch.max_pool2d(data, kernel_size=2)
    data = self.conv2(data)
    data = torch.relu(data)
    data = torch.max_pool2d(data, kernel_size=2)
    data = self.conv3(data)
    data = torch.relu(data)
    data = torch.max_pool2d(data, kernel_size=2)
    data = data.view(-1, 16)
    data = self.fc(data)
    data = torch.sigmoid(data)
    return data.squeeze()
```

実際にデータを流していきます。

畳み込み層と活性化関数の次に `max_pool2d` という関数が挟まっています。この関数は、与えられたカーネルサイズの中で最も大きい値のみを残す関数です。
CNN でよく使われる関数で、今は詳しく理解する必要はありませんが、これにより画像データを縮小しながら重要な特徴を保持することが可能です。計算量が減り、モデルが効率的に学習できるようになります。

各処理の間に `print(data.shape)` を挟むことで、どのようにデータが畳み込まれていくかを確認することができます。

### `train_model`

前章までと違う点を解説していきます。

#### `for data, label in train_data:`

前章とは異なり、ループをネストしています。
これは学習データが大きいため、一度に全データを学習させると効率が悪くなるからです。
データは小分け（バッチ）にして処理され、バッチサイズは `load_data` 内で定義されています。今回はバッチサイズを 100 に設定しています。

#### `with torch.no_grad():`

テストを行う場合のおまじないその 1 です。
このブロック以降ではテストデータをモデルに流して損失値と正解率を計算しています。

:::details 詳しく
このブロック内以降では、テストデータに対する損失と正解率の計算を行っています。
テストデータに対しては学習を行わないため、`no_grad` ブロックを作成することで、学習を行わないことを PyTorch に対して伝えます。
これにより学習のための計算を行わず、計算コストとメモリ使用量を削減できます。
:::

#### `model.eval()`

テストを行う場合のおまじないその 2 です。

:::details 詳しく
こちらは、モデルに対して学習を行わないことを伝えます。
層や活性化関数によっては、学習時と非学習時で処理が異なる場合があります。
これは多くは正則化のための処理で、正則化は学習を行わない場合にはただ精度を下げることになるため、これらを無効化します。
:::

#### `return train_losses, test_losses, test_accuracies`

最後に、学習した結果どのように損失値と正解率が推移したかの配列を返却します。
これをグラフに表示することで、今回の学習がうまくいったかを判断することが可能になります。

# まとめ

この章では、ディープラーニングを用いて学習を行うための流れを解説してきました。
実際には、エポックを回すたびにモデルのパラメータを保存し、保存したパラメータを読み込むことで本番環境で学習したモデルを動かすことができるようになります。
これらの細かい実装に関しては、オセロ AI を作成する中で学習していくことにしましょう。

:::details 補足
「トレーニングデータとテストデータ」の項で「学習に使うデータを 2 つに分ける」と説明しましたが、実際にはトレーニングデータとテストデータに「バリデーションデータ」を加えた 3 つに分けることがあります。

### ハイパーパラメータ

モデルが持つ「重み」や「バイアス」のことを「パラメータ」と呼びます。それに対して、学習中に変化せず、事前に実装者が決定する必要がある値を「ハイパーパラメータ」と呼びます。具体的には、学習率やバッチサイズ、層の数、層が持つパラメータの数などが該当します。

### バリデーションデータとその必要性

モデルを作成する際には、学習がうまくいくようにハイパーパラメータを調整しながら何度も学習を走らせることになります。
これを繰り返していくと、ハイパーパラメータをテストデータに対して適応させていってしまい、ハイパーパラメータを含めたモデル全体がテストデータに対して過学習に陥ります。

バリデーションデータは、この過学習を防ぐために使用されます。調整を繰り返していく中で、バリデーションデータに対する損失値がテストデータに対する損失値と乖離してきた場合、テストデータに対する過学習が始まっている兆候です。このように、バリデーションデータを利用することで、汎化性能（新しいデータへの適応力）が低下しているかを確認できます。
:::

# 手を動かしてみよう

### 全結合層のみを使った学習

今回、MNIST が二次元データであるため畳み込み層を使いましたが、全結合層のみでも学習が可能です。テストデータに対する正解率は 98% 程度までは出るはずです。
私の実装は `train_with_linear.py` にありますが、可能な限り見ずに自分で実装してみましょう。

:::details ヒント 1 - `foraward` メソッドの一行目
`forward` メソッドの一行目は `data = data.view(-1, 28 * 28)` とするのが良いでしょう。
:::

:::details ヒント 2 - 一つ目の層
一層目は `self.fc1 = torch.nn.Linear(28 * 28, 128)` とすると学習がうまくいきました。
:::

### MNIST 全体に対する学習

今回、MNIST の 4 と 9 二つに対する分類タスクを行うモデルを作成しましたが、0 から 9 すべての分類を行うモデルを作成してみましょう。
こちらも私の実装が `train_all.py` にありますが、可能な限り見ずに実装してみてください。
かなりチャレンジングな課題ですが、つまづいたらヒントを見つつ頑張ってください。
`load_data` 関数だけは本質ではないので見ても良いかもしれません。

また、一回の学習にかなりの時間を要します。ローカルの計算能力が足りない場合は、パラメータを減らして精度を犠牲に、速度重視のモデルを作成しましょう。

:::details ヒント 1 - モデル全体と学習の基本方針
モデルの基本的な設計は二値分類と変わりません。
学習において変更する必要があるのは、モデルの最後の出力と、損失関数だけのはずです。
各層が持つパラメータ数は必要に応じて変更しても良いでしょう。
:::

:::details ヒント 2 - モデルの出力、最後の層
多値分類問題では「ワンホットエンコーディング（One-Hot Encoding）」という手法を使います。
ワンホットエンコーディングは、カテゴリデータをベクトルで表す手法の一つです。
この方法では、各カテゴリをベクトルの 1 要素で表現します。ベクトルの要素 1 つずつが各カテゴリである確率を表します。

例えば、"犬"、"猫"、"鳥"というカテゴリがある時、
これらをワンホットエンコーディングで表現すると以下のようになります。

| カテゴリ | ワンホットベクトル |
| :------- | :----------------- |
| 犬       | [1, 0, 0]          |
| 猫       | [0, 1, 0]          |
| 鳥       | [0, 0, 1]          |

### なぜワンホットエンコーディングを使うのか？

単純にカテゴリに番号を割り当てる（例: 犬 = 0, 猫 = 1, 鳥 = 2）と、モデルがそれらの間に「順序」や「距離」があると誤解します。
この場合では猫と犬、犬と鳥は似ており、犬と鳥は似ていないという表現になってしまいます。
一方で、ワンホットエンコーディングではこのような順序関係がないため、正しくカテゴリデータをモデルが解釈します。
多値分類問題の出力にはワンホットエンコーディングを使うことを覚えておいてください。

今回は 10 のカテゴリに分類するので、最後の層は `self.fc = torch.nn.Linear(INPUT, 10)` となります。
:::

:::details ヒント 3 - 損失関数と最後の活性化関数
損失関数は `CrossEntropyLoss` を使います。
PyTorch の `CrossEntropyLoss` は、ただの損失関数ではなく、最後の活性化関数である `Softmax` 関数（二値分類問題でいうところの `Sigmoid`）が含まれています。
これは、微分をスムーズに行うためにはこの二つの関数をまとめた方が都合が良いためです。
ですから、今回のモデルの `forward` 関数では最後の活性化関数は不要になり、`data = self.fc(data)` が実質最後の行になります。
:::
