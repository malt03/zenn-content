---
title: "ディープラーニングの基礎 〜2Dポイント分類モデルで学ぶ〜"
---

# 開発環境

[uv](https://zenn.dev/malt03/books/d3f6cd9caa864a/viewer/999_environment#uv) をインストールしてください。

### ディレクトリ

本章では、 `/point_classifier_2d` 配下のコードを解説します。  
ディレクトリに移動し、 `sync` を実行してください。

```sh
cd ./point_classifier_2d
uv sync
```

# 実行してみよう

まずは実行してみましょう。

```sh
uv run src/train.py ./data/linear.csv simple
```

以下のような出力とグラフが表示されれば、正しく実行できています。  
この時点でうまくいかない場合は、すぐに[筆者に連絡](https://zenn.dev/malt03/books/d3f6cd9caa864a/viewer/0_intro#discord)してください。

```
Epoch 0, Loss: 0.7215992212295532
Epoch 1000, Loss: 0.2623043358325958
Epoch 2000, Loss: 0.20442548394203186
Epoch 3000, Loss: 0.1847211867570877
Epoch 4000, Loss: 0.17646907269954681
Epoch 5000, Loss: 0.1731029897928238
Epoch 6000, Loss: 0.17198911309242249
Epoch 7000, Loss: 0.1717546284198761
Epoch 8000, Loss: 0.1717332899570465
Epoch 9000, Loss: 0.1717328578233719
2024-11-11 21:01:04.386 python3[16646:3955627] +[IMKClient subclass]: chose IMKClient_Legacy
2024-11-11 21:01:04.386 python3[16646:3955627] +[IMKInputSession subclass]: chose IMKInputSession_Legacy
```

![linear](/images/d3f6cd9caa864a/1_firstdeeplearning/linear.png)
