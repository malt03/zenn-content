---
title: "ディープラーニングの基礎 - 複雑な2Dポイント分類モデル"
---

# ゴール

この章のゴールは、ディープラーニングでどのように複雑なデータを学習するか理解することです。

# 開発環境

:::details 前章と同じです。

[uv](https://zenn.dev/malt03/books/d3f6cd9caa864a/viewer/999_environment#uv) をインストールしてください。

### ディレクトリ

本章では、`./point_classifier_2d` 配下のコードを解説します。  
ディレクトリに移動し、 `sync` を実行してください。

```sh
cd ./point_classifier_2d
uv sync
```

:::

# 実行してみよう

今度は、少し複雑なデータを学習してみます。  
まずは学習データを表示してみましょう。

```sh
uv run ./src/dump_points.py ./data/curve.csv
```

前章の `./data/linear.csv` に比べて境界線が定かではありません。
学習してみましょう。

```sh
uv run ./src/train.py ./data/curve.csv simple
```

```
Epoch 0, Loss: 0.7329819202423096
Epoch 1000, Loss: 0.5453845858573914
Epoch 2000, Loss: 0.486764520406723
Epoch 3000, Loss: 0.4681168496608734
Epoch 4000, Loss: 0.46361517906188965
Epoch 5000, Loss: 0.4630098044872284
Epoch 6000, Loss: 0.46298325061798096
Epoch 7000, Loss: 0.462983101606369
Epoch 8000, Loss: 0.462983101606369
Epoch 9000, Loss: 0.462983101606369
Epoch 10000, Loss: 0.46298304200172424
Epoch 11000, Loss: 0.462983101606369
Epoch 12000, Loss: 0.46298304200172424
Epoch 13000, Loss: 0.462983101606369
Epoch 14000, Loss: 0.462983101606369
Epoch 15000, Loss: 0.4629831314086914
Epoch 16000, Loss: 0.4629831314086914
Epoch 17000, Loss: 0.462983101606369
Epoch 18000, Loss: 0.462983101606369
Epoch 19000, Loss: 0.4629831314086914
Epoch 20000, Loss: 0.46298304200172424
```

![simple](/images/d3f6cd9caa864a/1_2_firstdeeplearning/simple.png)

損失値があまり下がらず、画像を見ても正しく境界線を探索できているとは言えません。
学習がうまくいっていないことがわかります。

それでは、学習するモデルを変更してみましょう。

```sh
uv run ./src/train.py ./data/curve.csv complex
```

```
Epoch 0, Loss: 0.6947018504142761
Epoch 1000, Loss: 0.08028826117515564
Epoch 2000, Loss: 0.051741085946559906
Epoch 3000, Loss: 0.03521064668893814
Epoch 4000, Loss: 0.02702275663614273
Epoch 5000, Loss: 0.024927878752350807
Epoch 6000, Loss: 0.0199274979531765
Epoch 7000, Loss: 0.015367915853857994
Epoch 8000, Loss: 0.01430653315037489
Epoch 9000, Loss: 0.011918473988771439
Epoch 10000, Loss: 0.009658046066761017
Epoch 11000, Loss: 0.007121877279132605
Epoch 12000, Loss: 0.011210339143872261
Epoch 13000, Loss: 0.00868938583880663
Epoch 14000, Loss: 0.0063851880840957165
Epoch 15000, Loss: 0.0047729965299367905
Epoch 16000, Loss: 0.004584208596497774
Epoch 17000, Loss: 0.003644748358055949
Epoch 18000, Loss: 0.002768190111964941
Epoch 19000, Loss: 0.00197925278916955
Epoch 20000, Loss: 0.0012651399010792375
```

![complex](/images/d3f6cd9caa864a/1_2_firstdeeplearning/complex.png)

今度は学習がうまくいきました。

# 解説

### なぜ学習がうまくいったか？（いかなかったか？）

最初に試したモデルでは、データが複雑な境界線を学習できませんでした。
モデルの構造が単純すぎて直線的な境界線しか表現できなかったのです。

一方複雑なモデルでは、より多くの層や活性化関数を追加することで、複雑な境界線を表現できるようになっています。

# コードを読む

### `PointClassifier2DComplex`

`PointClassifier2DComplex` は、前章で使った `PointClassifier2DSimple` よりも複雑なモデルです。
以下のコードを `PointClassifier2DSimple` と見比べてみましょう。

```python:train.py
class PointClassifier2DComplex(torch.nn.Module):
    def __init__(self):
        super(PointClassifier2DComplex, self).__init__()
        self.fc1 = torch.nn.Linear(2, 128)
        self.fc2 = torch.nn.Linear(128, 128)
        self.fc3 = torch.nn.Linear(128, 1)

    def forward(self, data):
        data = self.fc1(data)
        data = torch.relu(data)
        data = self.fc2(data)
        data = torch.relu(data)
        data = self.fc3(data)
        data = torch.sigmoid(data)
        return data.squeeze()
```

#### `__init__`メソッド

3 つの全結合層を作成しています。

- `self.fc1 = torch.nn.Linear(2, 128)`
  - 最初の層 fc1 は、2 つの入力 `(x, y)` を 128 個の値に変換します。
- `self.fc2 = torch.nn.Linear(128, 128)`
  - 2 番目の層 fc2 は、前の層から来た 128 個の値を、また 128 個の値に変換します。
- `self.fc3 = torch.nn.Linear(128, 1)`
  - 最後の層 fc3 は、128 個の値を 1 つの値に変換します。

全結合層は、`input * output + output` 個のパラメータを持ちます。
つまり、上記の変更により、`PointClassifier2DSimple` ではたった 3 個だったパラメータ数が、`PointClassifier2DComplex` では 17,025 個になりました。

#### `forward` メソッド

このメソッドでは、データが各層を通過して最終的な出力が得られるまでの流れを定義しています。
`relu` という新しい活性化関数が出てきました。

https://www.google.com/search?q=relu+activation+function&udm=2

連続的でなく、すごく変な形をしている関数です。
この「連続的でなさ」によって、モデルが非線形で複雑な境界線を学習できます。
最もよく使われる活性化関数の一つで、とりあえず全結合層の間には挟んでおきましょう。

やっていることはシンプルで、ひたすら作成した層の間に `relu` を挟んでデータを流していっているだけです。

# まとめ

モデルを複雑化する方法はいたってシンプルです。  
全結合層の数と、各層が持てるパラメータ数を増やし、間に活性化関数を挟みました。人間がやるのはそれだけです。

17,025 個もの変数に対して、損失関数が調整の方向を計算し、オプティマイザが調整の大きさを決定することで勝手に学習を進めることができます。
どんなにパラメータが増えてもこれが可能なことこそが、ディープラーニングが革命を起こした所以です。

### なぜこんなことが可能なのか？

:::details 興味があれば読んでください

ディープラーニングが自動で学習を進められるのは、勾配降下法という手法によります。
勾配降下法により、モデルは最適なパラメータを見つけ、損失を小さくしていくことができます。

#### 微分

微分は、ある値が変化したときに、別の値がどのように変化するかを示すものです。
ディープラーニングで使われる全ての関数は微分可能になっています。
`loss = loss_fn(model(input))` という関数を微分することで、この関数の傾きを計算できます。
この傾きのことを「勾配」と呼びます。

#### 勾配降下

勾配降下法は、この勾配を使って、損失を小さくする方向にパラメータを更新していく手法です。
具体的には、計算された勾配に基づいて、パラメータを少しずつ調整し、損失が小さくなる方向へとモデルを改善していきます。
N 次元空間上の複雑な山を、低い方、低い方へと下っていくイメージです。

#### オプティマイザの役割

この時、ただ下れば良いわけではありません。山には起伏があり、局所的に低いところを見つけても、ちょっと上った先にもっと低いところがあるかもしれません。

オプティマイザは、損失関数の勾配に従ってパラメータを更新し、より低い損失を目指して「山を下っていく」役割を担っています。

山を転がるボールをイメージしてみてください。ボールは山を下る中で加速し、途中窪みにたどり着いても慣性によって山を登ることができます。
そして登った先でもっと低いところがあれば、さらに下に転げ落ちていくことが可能です。

あくまで比喩表現ではありますが、このボールのような機能がオプティマイザに備わっていることで、局所的な最小値に止まらずに本当の最小値を探索することが可能なのです。

#### 学習率を振り返る

学習率は、値が大きいと学習が不安定になり、小さいと学習が遅くなります。  
これも山下りを意識すると理解しやすくなります。

|                               学習率が低い                               |                                学習率が適切                                |                                学習率が高い                                |
| :----------------------------------------------------------------------: | :------------------------------------------------------------------------: | :------------------------------------------------------------------------: |
| ![lr-low](/images/d3f6cd9caa864a/1_2_firstdeeplearning/lr-low.png =200x) | ![lr-good](/images/d3f6cd9caa864a/1_2_firstdeeplearning/lr-good.png =200x) | ![lr-high](/images/d3f6cd9caa864a/1_2_firstdeeplearning/lr-high.png =200x) |

:::

# 手を動かしてみよう

ここで、`PointClassifier2DComplex` や `train_model` 内を色々変更し、学習を走らせてみてください。どこをどのように変更すると、学習結果がどう変化するかを試し、なぜそのように変化するのかを考察してみましょう。
特に、学習がうまくいかなくなるパターンを見つけ、その差分を考えてみると理解が深まると思います。

### 変更してみてほしい項目

- 学習回数
- 学習率
- 全結合層の数
- 全結合層の入力数、出力数
- 活性化関数の有無

余裕があれば、損失関数やオプティマイザを変更してみるのも学びになるでしょう。
試行を通して、さまざまな要素がモデルの学習にどのように影響するかを確認してみてください。

### トラブルシューティング

モデルを変更する中で各層の入出力の大きさを変更していると、以下のようなエラーによく遭遇します。

```
RuntimeError: mat1 and mat2 shapes cannot be multiplied (200x64 and 128x128)
```

モデルを以下のように変更して発生させました。

```python:train.py
class PointClassifier2DComplex(torch.nn.Module):
    def __init__(self):
        super(PointClassifier2DComplex, self).__init__()
        self.fc1 = torch.nn.Linear(2, 64)
        self.fc2 = torch.nn.Linear(128, 128)
        self.fc3 = torch.nn.Linear(128, 1)
```

このエラーは、層に対して期待するサイズの行列が入力されなかったために計算できない(`cannot be multiplied`)と発生します。
モデルを調整する際、最初のうちはこのエラーを頻繁に発生させてしまうと思います。

そんな時は、前章でも紹介した公式リファレンスを確認しましょう。
https://pytorch.org/docs/stable/generated/torch.nn.Linear.html

注目すべきは `Shape` の項目です。
ここには期待する入力の行列の大きさと、出力する行列の大きさが記載されています。
ちょっと難しげですが、書いてあることは単純なので、踏ん張りどころです。

全結合層は `(*, in_features)` の大きさの行列を入力すると、 `(*, out_features)` の行列を出力する、と記載されています。
試しに、行列の大きさを表示してみましょう。

```python:train.py
    def forward(self, data):
        print(data.shape)
        data = self.fc1(data)
        print(data.shape)
        data = torch.relu(data)
        data = self.fc2(data)
        data = torch.relu(data)
        data = self.fc3(data)
        data = torch.sigmoid(data)
        return data.squeeze()
```

```
torch.Size([200, 2])
torch.Size([200, 64])
```

関数に入力された `data` の大きさは `(200, 2)`、 `fc1` で変換された後は `(200, 64)` になっていることがわかります。
今回は学習データが全部で 200 件あり、その全てを一気に学習しています。
各行に 2 つの要素を持っていますから、`(200, 2)` のデータが入力されています。
そして、それを `fc1` で変換し、 `(200, 64)` のサイズに変換していることが確認できました。

`fc1` の出力数が 64 になっており、`fc2` の入力数である 128 と一致していません。
それでは、`fc2` の入力サイズも変更しましょう。

```python:train.py
class PointClassifier2DComplex(torch.nn.Module):
    def __init__(self):
        super(PointClassifier2DComplex, self).__init__()
        self.fc1 = torch.nn.Linear(2, 64)
        self.fc2 = torch.nn.Linear(64, 128)
        self.fc3 = torch.nn.Linear(128, 1)
```

これで無事動くようになったはずです。

エラーが発生した場合は、以下の手順で原因を特定して修正しましょう。

1. **公式リファレンスを確認**して、層の入力と出力のサイズを理解する。
2. **コードにデバッグ用の`print`を追加**して、実際の行列のサイズを確認する。
3. **エラーが出ている層の入力サイズや出力サイズを修正**し、再実行する。
