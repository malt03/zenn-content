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
Epoch 0, Loss: 0.7446361780166626
Epoch 1000, Loss: 0.4672893285751343
Epoch 2000, Loss: 0.4629960358142853
Epoch 3000, Loss: 0.4629831314086914
Epoch 4000, Loss: 0.462983101606369
Epoch 5000, Loss: 0.46298304200172424
Epoch 6000, Loss: 0.462983101606369
Epoch 7000, Loss: 0.46298304200172424
Epoch 8000, Loss: 0.46298304200172424
Epoch 9000, Loss: 0.462983101606369
Epoch 10000, Loss: 0.46298304200172424
```

![simple](/images/d3f6cd9caa864a/1_2_firstdeeplearning/simple.png)

損失値があまり下がらず、画像を見ても正しく境界線を探索できているとは言えません。
学習がうまくいっていないことがわかります。

それでは、学習するモデルを変更してみましょう。

```sh
uv run ./src/train.py ./data/curve.csv complex
```

```
Epoch 0, Loss: 0.6921066045761108
Epoch 1000, Loss: 0.06626128405332565
Epoch 2000, Loss: 0.04576430469751358
Epoch 3000, Loss: 0.04641619697213173
Epoch 4000, Loss: 0.02098308503627777
Epoch 5000, Loss: 0.016660088673233986
Epoch 6000, Loss: 0.01085667498409748
Epoch 7000, Loss: 0.006349646020680666
Epoch 8000, Loss: 0.0188666433095932
Epoch 9000, Loss: 0.015938470140099525
Epoch 10000, Loss: 0.011187952011823654
```

![complex](/images/d3f6cd9caa864a/1_2_firstdeeplearning/complex.png)

今度は学習がうまくいきました。

# 解説

### なぜ学習がうまくいったか？（いかなかったか？）

まず最初に試したモデルでは、データが複雑な境界線を学習できませんでした。
モデルの構造が単純すぎて直線的な境界線しか表現できず、複雑なデータに適応できませんでした。

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

#### `forward` メソッド

このメソッドでは、データが各層を通過して最終的な出力が得られるまでの流れが定義されています。
`relu` という新しい活性化関数が出てきました。

https://www.google.com/search?q=relu+activation+function&udm=2

連続的でなく、すごく変な形をしている関数です。
この「連続的でなさ」によって、モデルが非線形で複雑な境界線を学習できます。
最もよく使われる活性化関数の一つで、とりあえず全結合層の間には挟んでおきましょう。

やっていることはシンプルで、ひたすら作成した層の間に `relu` を挟んでデータを流していっているだけです。

# まとめ

モデルを複雑化する方法はいたってシンプルです。  
全結合層の数と、各層が持てるパラメータを増やし、間に活性化関数を挟みました。人間がやるのはそれだけです。

どんなに複雑で大量のパラメータに対しても、損失関数がパラメータの調整方向を計算し、オプティマイザが調整の大きさを決定することで勝手に学習が進みます。
これこそがディープラーニングが革命を起こした所以です。
