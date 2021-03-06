![](RackMultipart20210718-4-1lpo92u_html_ffb1dd0639f22f75.jpg)

# 深層学習（後編）

ラビットチャレンジ

竹田稔 | 個人向けコース | 2021-07-18

# 深層学習day３

# １．再帰型ニューラルネットワークの概念

## １－１．ＲＮＮ全体像

## １－１－１．ＲＮＮとは

**時系列データ** に対応可能な、ニューラルネットワークである。

## １－１－２．時系列データ

時間的順序を追って一定間隔ごとに観察され、しかも相互に統計的依存関係が認められるようなデータの系列である。音声データ、テキストデータなど。

## １－１－３．ＲＮＮについて

初期の状態と過去の時間t-1の状態を保持し、そこから次の時間を再帰的に求める再帰構造が必要になる。

確認テスト：ＲＮＮの3つの重み

・入力から現在の中間層を定義する際にかけられる重み

・中間層から出力を定義する際にかけられる重み

・中間層から次の中間層に渡される際にかけられる重み

演習：

![3_1_simple_RNN](img/Web_capture_24-6-2021_115218_localhost.jpeg)

[try] weight\_init\_stdやlearning\_rate, hidden\_layer\_sizeを変更してみよう

![3_1_simple_RNN](img/Web_capture_24-6-2021_12626_localhost.jpeg)

[try] 重みの初期化方法を変更してみよう

・Xavier

Web\_capture\_25-6-2021\_11573\_localhost
![3_1_simple_RNN](img/Web_capture_25-6-2021_11573_localhost.jpeg)

・He

![3_1_simple_RNN](img/Web_capture_25-6-2021_12010_localhost.jpeg)

[try] 中間層の活性化関数を変更してみよう

・ ReLU(勾配爆発を確認しよう)

![3_1_simple_RNN](img/Web_capture_25-6-2021_12750_localhost.jpeg)

・ tanh(numpyにtanhが用意されている。導関数をd\_tanhとして作成しよう)

![3_1_simple_RNN](img/Web_capture_25-6-2021_122242_localhost.jpeg)

## １－２．ＢＰＴＴ

## １－２－１．ＢＰＴＴとは

ＲＮＮにおいてのパラメータ調整方法の一種 ～ 誤差逆伝播の一種。

誤差は時間をさかのぼって逆伝播していることになる(Backpropagation Through Time)。

## １－２－２．ＢＰＴＴの数学的記述

確認テスト：y1 = sigmoid(z1Wout + c) z1 = sigmoid(s0W + x1w(u) + b)

## １－２－３．ＢＰＴＴの全体像

演習問題：

def bptt(xs, ys, W, U, V):

hiddens, outputs = rnn\_net(xs, W, U, V)

dW = np.zeros\_like[W]

dU = np.zeros\_like[U]

dV = np.zeros\_like[V]

do = \_catculate\_do(outputs, ys)

batch\_size, n\_seq = ys.shape[:2]

for t in reversed(range(n\_seq)):

dV += np.dot(do[:,t],t).T, hiddens8:, t]) / batch\_size

delta\_t = do[:, t].dot(V)

for bptt\_step in reversed(range(t+1)):

dW += np.dot(delta\_t.T, xs[:,batt\_step]) / batch\_size

dU += np.dot(delta\_t.T, hidden[:,batt\_step-1]) / batch\_size

delta\_t = delta\_t.dot(U)

return dW, dU, dV

# ２．ＬＳＴＭ

モデルの構造自体を変えて勾配消失を解決する。

確認テスト：シグモイド関数を微分した時、最大値は0.25である。

演習問題：

def gradient\_clipping(grad, threshold):

norm = np.linalg.norm(grad)

rate = threshold / norm

if rate \&lt; 1:

return gradient \* rate

return grad

## ２－１．ＣＥＣ

勾配消失および勾配爆発の解決方法として、勾配を1とすることで解決する。

問題点：入力データについて、時間依存度に関係なく重みが一律となるので、学習特性がなくなる。

## ２－２．入力ゲートと出力ゲート

入力ゲートと出力ゲートを追加することで、それぞれのゲートへの入力値の重みを、重み行列で可変可能とすることで、ＣＥＣの問題点を解決する。

## ２－３．忘却ゲート

過去の情報が要らなくなったタイミングで、情報を忘却する。

確認テスト：なくなっても影響のない要素に作用するのは、忘却ゲートである。

演習チャレンジ：

def lstm(x, prev\_h, prev\_c, w, u, b):

lstm\_in = \_activation(x.dot(W.T) + prev\_h.dot(U.T) + b)

a, i, f, o = np.hsplit(lstm\_in, 4)

a = np.tanh(a)

input\_gate = \_sigmoid(i)

forget\_gate = \_sigmoid(f)

output\_gate = \_sigmoid(o)

c = input\_gate \* a + forget\_gate \* c

h = output\_gate \* np.tanh(c)

return c, h

## ２－４．覗き穴結合

ＣＥＣ自身の値に、重み行列を介して伝播可能にした（ゲート制御する）構造となっている。

# ３．ＧＲＵ（Gated Recurrent Unit）

## ３－１．ＧＲＵ

ＬＳＴＭと同様にＲＮＮの一種であり、単純なＲＮＮにおいて問題となる勾配消失問題を解決し、長期的な依存関係を学習することができる。

ＬＳＴＭのパラメータ（変数の数やゲートの数）を大幅に削減して、精度は同等またはそれ以上が望める様になった構造となっている。

メリット：計算負荷が低い。

確認テスト：

ＬＳＴＭの課題：パラメータ数が多いため、計算負荷が大きい。

ＣＥＣの課題：入力データについて、時間依存度に関係なく重みが一律となるので、学習特性がなくなる。

ハンズオン：

![predict_word](img/Web_capture_24-6-2021_121551_localhost.jpeg)

演習チャレンジ：

def gru(x, h, W\_r, U\_r, W\_z, U\_z, W, U):

r = \_sigmoid(x.dot(W\_r.T) + h.dot(U\_r.T)

z = \_sigmoid(x.dot(W\_z.T) + h.dot(U\_z.T)

h\_bar = np.tanh(x.dot(W.T) + (r \* h).dot(U.T))

h\_new = (1–z) \* h + z \* h\_bar

return h\_new

確認テスト：ＬＳＴＭのパラメータを大幅に削減して、精度は同等またはそれ以上が望める様にしたのがＣＥＣである。

# ４．双方向ＲＮＮ

## ４－１．双方向ＲＮＮ

**過去** の情報だけでなく、 **未来** の情報を加味することで、精度を向上させるためのモデルである。文章の推敲や機械翻訳などで使用される。

演習チャレンジ：

def bidirectional\_rnn\_net(xs, W\_f, U\_f, W\_b, U\_b, V)

xs\_f = np.zeros\_like(xs)

xs\_s = np.zeros\_like(xs)

for I, x in enumerate(xs):

xs\_f[i] = x

xs\_b[i] = x[::-1]

hs\_f = \_rnn(xs\_f, W\_f, U\_f)

hs\_b = \_rnn(xs\_b, W\_b, U\_b)

hs =np.concatenate([h\_f, h\_b[::-1]], axis=1)

ys =hs.dot(V.T)

return ys

# ５．Ｓｅｑ２Ｓｅｑ

Encoder-Decoderモデルの一種を指します。機械対話や、機械翻訳などに使用されている。Encoder、DecoderともにＲＮＮであり、入力の時系列データをEncoderが解析した結果を中間層にベクトルとして蓄積して、その情報がDecoderの出力生成に利用される。

課題：一問一答しかできない。

## ５－１．Ｅｎｃｏｄｅｒ　ＲＮＮ

ユーザーがインプットしたテキストデータを、単語等のトークンに区切って渡す構造になっている。文の意味をベクトルデータとして集約する。

## ５－２．ＤＥｃｏｄｅｒ　ＲＮＮ

システムがアウトプットデータを、単語等のトークンごとに生成する構造になっている。蓄積した情報から別の文脈を生成する。

※確認テスト：

Ｓｅｑ２ＳｅｑはEncoder-Decoderモデルの一種を指します。機械対話や、機械翻訳などに使用されている。

演習チャレンジ：

def encode(words, E, W, U, b):

hidden\_size = W.shape[0]

h = np.zeros(hidden\_size)

for w inwords:

e = E.dot(W)

h = \_activation(W.dot€ + U.dot(h) + b)

return h

## ５－３．ＨＲＥＤ

前の単語の流れに即して応答されるため、より人間らしい文章が生成される。

Ｓｅｑ２ＳｅｑにContext ＲＮＮを追加した構造によって、過去の発話履歴を加味した返答ができる。

課題：回答が短く情報量の乏しいものになりがちである。

※Context ＲＮＮ：Encoderのまとめて各文章の系列をまとめて、これまでの会話コンテキスト全体を表すベクトルに変換する構造になっている（文章の時系列を予測する）。

## ５－４．ＶＨＲＥＤ

ＨＲＥＤにＶＡＥの潜在変数の概念を追加することによって、ＨＲＥＤの課題を解決している。

※確認テスト：

・Ｓｅｑ２Ｓｅｑは一文の一問一答の回答を返すが、ＨＲＥＤはさらに文脈の意味ベクトル解釈に加えることで文脈の意味組み取った文の変換を出るようにした。

・ＨＲＥＤが当たり障りのない回答しか返さないが、ＶＨＲＥＤはＶＡＥの考えを取り入れて短い単語以上の回答を返せるようにした。

## ５－５．ＶＡＥ

## ５－５－１．オートエンコーダー（自己符号化器：Autoencoder）

教師なし学習の一つ。

入力されたデータを一度圧縮し、重要な特徴量だけを残した後、再度もとの次元に復元する。

入力データから潜在変数zに変換する（何かしら潜在変数zにデータを押し込める）ニューラルネットワークをEncoderといい、逆に潜在変数を入力としてもとの次元に復元するニューラルネットワークをDecoderという。

次元削減（特徴抽出）が行える。

事前学習（ニューラルネットワークの初期値に、オートエンコーダで学習させたものを用いる）を行うことで、勾配消失による学習速度低下を防止する。

オートエンコーダによりデータを粗な状態にすることで、過学習を防止する。

## ５－５－２．ＶＡＥ（Variational Autoencoder）

潜在変数zに確率分布z-N(0,1)を仮定したもの。

正規分布に従う乱数を学習時に取り入れているので、教師データのない教師なし学習にもかかわらず同じクラスラベルのデータが近いところに集まる。

※確認テスト：自己符号化器の潜在変数に確率分布を導入したもの。

# ６．Ｗｏｒｄ２ｖｅｃ

## ６－１．Ｗｏｒｄ２ｖｅｃ

学習データからボキャブラリを作成する。可変長の単語を取り扱うことができる。

One-hotベクトルでは1単語に1次元を割り当てるので、ボキャブラリ数が増えると非常に高次元になる。単語分散表現（単語ベクトル）は、単語を低次元の実数値ベクトルで表すので、単語間の類似度を計算することができる。

# ７．Ａｔｔｅｎｔｉｏｎ　Ｍｅｃｈａｎｉｓｍ

## ７－１．Ａｔｔｅｎｔｉｏｎ　Ｍｅｃｈａｎｉｓｍ

「入力と出力のどの単語が関連しているのか」の関連度を学習する仕組みです。

長い文章のシーケンスの内部表現の次元を抑えて学習の精度を上げます。

歴史的には、ＲＮＮのモデルに対して組み込まれて、大きな注目を集めた。近年では、機械翻訳のために提案された Transformerとそれをベースにした BERTなどの高度なモデルで全面的に使用され、様々な自然言語処理のタスクで最高精度を更新し続けています。

※確認テスト：

ＲＮＮ：時系列を処理するのに適したニューラルネットワーク

Ｗｏｒｄ２ｖｅｃ：単語の分散表現ベクトルを得る手法

Ｓｅｑ２Ｓｅｑ：一つの時系列データから別の時系列データ得るネットワーク

Ａｔｔｅｎｔｉｏｎ：時系列データの中身にたいして関連性に重みを付ける手法

演習チャレンジ：

def traverse(node):

if not isinstace(node, dict):

v = node

else:

left = traverse(node[&#39;left&#39;])

right = traverse(node[&#39;right&#39;])

v = \_activation(W.dot(np.concatenate([left, right]))

return v

# 深層学習day４

# １．強化学習

## １－１．強化学習とは

長期的に報酬を最大化できるように環境のなかで行動を選択できるエージェントを作ることを目標とする機械学習の一分野です。

**行動の結果** として与えられる利益(報酬)をもとに、 **行動を決定する** 原理を改善していく仕組みです。

## １－２．強化学習の応用例

販売促進部（環境）において、キャンペーンをする（行動）によって、売上（正の報酬）とコスト（負の報酬）受け取り、報酬がプラスに大きくなるように、キャンペーンの内容（方策）を試行錯誤しながら（観測：知識を得て経験とする）、最適なキャンペーン内容を見つける。

## １－３．探索と利用のトレードオフ

環境について **事前に完璧な知識があれば** 、最適な行動を予測し決定することは可能であるので、どのような顧客にキャンペーンメールを送信すると、どのような行動を行うのかが既知である状況といえる。

強化学習の場合、上記仮定は成り立たないとする。 **不完全な知識を元に行動** しながら、データを収集。 **最適な行動** を見つけていく。

探索が足りない状態（過去のデータだけで、ベストとされる行動のみを常に取り続ければ、他にもっとベストな行動を見つけることはできない）

⇕トレードオフの関係性

利用が足りない状態（未知の行動のみを常に取り続ければ、過去の経験が活かせない）

## １－４．強化学習のイメージ

強化学習の学習の対象

- 方策：方策関数
- 価値：行動価値関数（行動関数、価値関数）

## １－５．強化学習の差分

強化学習と通常の教師あり、教師なし学習との違い

結論:目標が違う

- 教師なし、あり学習では、データに含まれる **パターンを見つけ** 出すおよびその **データから予測する** ことが目標である。

- 強化学習では、 **優れた方策** を見つけることが目標である。

## １－６．強化学習の歴史

## １－６－１．強化学習について

- 冬の時代があったが、 **計算速度の進展** により大規模な状態をもつ場合の、強化学習を可能としつつある。

- 関数近似法と、Q学習を組み合わせる手法の登場

## １－６－２．Ｑ学習

- 行動価値関数を、行動する毎に更新することにより学習を進める方法です。

## １－６－３．関数近似法

- 価値関数や方策関数を関数近似する手法のことです。

## １－６．価値関数

価値関数とは

- 価値を表す関数としては、状態価値関数と行動価値関数の2種類がある。

ある **状態の価値** に注目する場合は、状態価値関数（環境の状態だけが価値を決める対象になっている：V(s)）

**状態と価値を組み合わせた価値** に注目する場合は、行動価値関数（状態と行動の２つの情報をもとに価値を決める:Q(s,a)）

## １－７．方策関数

方策関数とは、方策ベースの強化学習手法において、ある状態でどのような行動を採るのかの確率を与える関数のことです。

## １－８．方策勾配法

方策勾配法：方策をモデル化して最適化する手法である（方策反復法）

定義方法

- 平均報酬

- 割引報酬和

上記の定義に対応して、行動価値関数:Q(s,a)の定義を行い。

方策勾配定理が成り立つ。

# ２．ＡｌｐｈａＧｏ

## ２－１．ＡｌｐｈａＧｏ

ＡｌｐｈａＧｏ Ｌｅｅ

- PolicyNet：方策関数、畳み込みニューラルネットワークであり、19×19で48チャネルの入力データから、19×19マスの着手予想確率が出力される。

- ValueNet：価値関数、畳み込みニューラルネットワークあり、19×19で49チャネル(「手番」が追加)の入力データから、勝率（-1～1）が出力される。

- 教師あり学習（過去の人間の対局を学習する）のあと、強化学習を行う。

- RollOutPolicy：線形の方策関数であり、探索中に高速に着手確率を出すために使用される。

- RollOutPolicy用いたモンテカルロ木探索で、強化学習を行う。

ＡｌｐｈａＧｏ Ｚｅｒｏ

- 教師あり学習を行わない。

- 特徴入力からヒューリスティックな要素を排除し、石の配置のみにした。

- PolicyNetとValueNetを１つのネットワークに統合した。

- ResidualNetを導入した。

- モンテカルロ木探索からRollOutシミュレーションをなくした。

※ ResidualNet：ショートカットのある基本層の39層からなる。

ＡｌｐｈａＧｏ Ｚｅｒｏでの工夫

- Bottleneck：

- PreActivation：

- WideResNet：

- pyramidNet：

ＡｌｐｈａＧｏ Ｚｅｒｏの学習法

- 自己対局による教師データの作成

- 教師データによる学習

- ネットワークの更新

# ３．軽量化・高速化技術

※分散深層学習とは

- 深層学習は多くのデータを使用したり、パラメータ調整のために多くの時間を使用したりするため、高速な計算が求められる。（データが増えている）

- 複数の計算資源(ワーカー)を使用し、並列的にニューラルネットを構成することで、効率の良い学習を行いたい。（コンピュータ１台の性能向上がモデルに追いつかない）

- データ並列化、モデル並列化、GPUによる高速技術は不可欠である。

## ３－１．モデル並列

- 親モデルを各ワーカーに分割し、それぞれのモデルを学習させる。

- 全てのデータで学習が終わった後で、一つのモデルに復元する。

- モデルが大きい時はモデル並列化を、データが大きい時はデータ並列化をすると良い。

※モデル並列化の効果

- モデルのパラメータ数が多い（大きいモデル）ほど、スピードアップの効率も向上する。

## ３－２．データ並列

- 親モデルを各ワーカーに子モデルとしてコピーする。

- データを分割し、各ワーカーごとに計算させる。

## ３－２－１．データ並列化: 同期型

- データ並列化は各モデルのパラメータの合わせ方で、同期型か非同期型か決まる。

※同期型のパラメータ更新の流れ。

- 各ワーカーの計算が終わるのを待つ。

- 全ワーカーの勾配が出たところで勾配の平均を計算する。

- 親モデルのパラメータを更新する。

## ３－２－２．データ並列化: 非同期型

※非同期型のパラメータ更新の流れ。

- 各ワーカーはお互いの計算を待たず、各子モデルごとに更新を行う。

- 学習が終わった子モデルはパラメータサーバにPushされる。

- 新たに学習を始める時は、パラメータサーバからPopしたモデルに対して学習していく。

## ３－２－３．同期型と非同期型の比較

- 処理のスピードは、お互いのワーカーの計算を待たない **非同期型の方が早い** 。

- 非同期型は最新のモデルのパラメータを利用できないので、学習が不安定になりやすい。

    -\&gt; Stale Gradient Problem

- 現在は **同期型の方が精度が良いことが多い** ので、主流となっている。

## ３－３．ＧＰＵ

## ３－３－１．GPGPU (General-purpose on GPU)

- 元々の使用目的であるグラフィック以外の用途で使用されるGPUの総称

## ３－３－２．ＣＰＵ

- 高性能なコアが少数

- 複雑で連続的な処理が得意

## ３－３－３．ＧＰＵ

- 比較的低性能なコアが多数

- 簡単な並列処理が得意

- ニューラルネットの学習は単純な行列演算が多いので、高速化が可能

## ３－３－４．GPGPU開発環境

### ３－３－４―１．CUDA

- GPU上で並列コンピューティングを行うためのプラットフォームである。

- NVIDIA社が開発しているGPUのみで使用可能である。

- Deep Learning用に提供されているので、使いやすい。

### ３－３－４－２．OpenCL

- オープンな並列コンピューティングのプラットフォーム

- NVIDIA社以外の会社(Intel, AMD, ARMなど)のGPUからでも使用可能である。

- Deep Learning用の計算に特化しているわけではない。

### ３－３－４－３．Deep Learningフレームワーク

- フレームワーク(Tensorflow, Pytorch)内でＧＰＵを使用したモデルが実装されているので、使用する際は指定すれば良い。

## ３－４．モデルの軽量化

## ３－４－１．モデルの軽量化とは

- モデルの精度を維持しつつパラメータや演算回数を低減する手法の総称である。

- 高メモリ負荷、高い演算性能が求められる

- 通常は低メモリ低演算性能での利用が必要とされるIotなど

参照論文：山本康平橘素子前野蔵人ディープラーニングのモデル軽量化技術https://www.oki.com/jp/otr/2019/n233/pdf/otr233\_r11.pdf

## ３－４－２．モデルの軽量化の利用

- モデルの軽量化はモバイル, IoT 機器において有用な手法である。

- モバイル端末やIoT はパソコンに比べ性能が大きく劣る。

- 主に計算速度と搭載されているメモリ

- モデルの軽量化は計算の高速化と省メモリ化を行うためモバイル,IoT 機器と相性が良い手法になる。

## ３－４－３．軽量化の手法

代表的な手法として下記の3 つがある

- 量子化

- 蒸留

- プルーニング

## ３－５．量子化(Quantization)

## ３－５－１．量子化(Quantization)とは

- ネットワークが大きくなると大量のパラメータが必要となり、学習や推論に多くのメモリと演算処理が必要となる。

- 通常のパラメータの64 bit 浮動小数点を32 bit など下位の精度に落とすことでメモリと演算処理の削減を行う。

## ３－５－２．量子化の利点と欠点

## ３－５－２－１．利点

- 計算の高速化

- 省メモリ化

## ３－５－２－２．欠点

- 精度の低下

## ３－５－３．計算の高速化

- 倍精度演算(64 bit)と単精度演算(32 bit)は演算性能が大きく違うため、量子化により精度を落とすことにより、より多くの計算をすることができる。

## ３－５－４．省メモリ化

- ニューロンの重みを浮動小数点のbit数を少なくして有効桁数を下げることでニューロンのメモリサイズを小さくすることができ、多くのメモリを消費するモデルのメモリ使用量を抑えることができる。

## ３－５－５．精度の低下

先に述べたようにニューロンが表現できる小数の有効桁が小さくなる。

↓

モデルの表現力が低下する。

↓

学習した結果ニューロンが、小数の有効桁未満の値になる場合など重みを表現できなくなる。

↓

ただし、実際の問題では倍精度を単精度にしてもほぼ精度は変わらない。

## ３－５－６．極端な量子化

極端な量子化を考える。表現できる値が0,1 の1 bitの場合、

a = 0.1 が真値の時、関数y(x) = axを近似する場合を考える際、学習によってa が0.1 を得る必要があるしかし、量子化によってa が表現できる値が0,1 のため求められる式は、

y(x) = 0, y(x) =x

のようになり、誤差の大きな式になってしまう。

量子化する際は極端に精度が落ちない程度に量子化をしなければならない。

## ３－５－７．量子化の実験結果

下記の論文を参照する。

PenghangYin , Shuai Zhang , YingyongQi , and Jack Xin（2017）「Quantization and Training of Low Bit-Width Convolutional Neural Networks for Object Detection」

[https://arxiv.org/pdf/1612.06052.pdf](https://arxiv.org/pdf/1612.06052.pdf)

### ３－５－７－１．速度の実験結果

下記の表は、32bitと6bitのモデルで検出を行った実験の検出時間である。

| 32 bit  | 6 bit  |
|---------|--------|
| 0.507s  | 0.098s |
| 0.441s  | 0.106s |
| 32.269s | 6.113s |

### ３－５－７－２．精度の実験結果

下記の図は量子化を行わない場合と量子化を行った際の精度になる。

| R-FCN, ResNet-50      | mAP    | R-FCN, ResNet-101     | mAP   　|
|-----------------------|--------|-----------------------|---------|
| 4-bit LBW             | 74:37% | 4-bit LBW             | 76:79%　|
| 5-bit LBW             | 76:99% | 5-bit LBW             | 77:83%　|
| 6-bit LBW             | 77:05% | 6-bit LBW             | 78:24%　|
| 32-bit full-precision | 77:46% | 32-bit full-precision | 78:94%　|

## ３－６．蒸留(Distillation)

## ３－６－１．蒸留とは

**精度の高い** モデルはニューロンの **規模が大きなモデル** になっている。そのため、推論に多くのメモリと演算処理が必要です。

↓

規模の大きなモデルの知識を使い、 **軽量なモデルの作成** を行う。

## ３－６－２．モデルの簡約化

学習済みの精度の高いモデルの知識を軽量なモデルへ **継承** させる。

知識の継承により、軽量でありながら複雑なモデルに匹敵する精度のモデルを得ることが期待できる。

## ３－６－３．教師モデルと生徒モデル

蒸留は教師モデルと生徒モデルの2つで構成される

教師モデル：予測精度の高い、複雑なモデルやアンサンブルされたモデル（大きく精度が高い）

生徒モデル：教師モデルをもとに作られる軽量なモデル

### ３－６－３－１．教師モデルと生徒モデル

- 教師モデルの重みを固定し、生徒モデルの重みを更新していく。

- 誤差は教師モデルと生徒モデルのそれぞれの誤差を使い重みを更新していく。

## ３－６－４．蒸留の利点

Cifar 10 データセットで学習を行ったレイヤー数と精度のグラフが参考論文に掲載されている。

表のback propagation は通常の学習、Knowledge Distillation は先に説明した蒸留手法、Hint Taraing は蒸留は引用論文で提案された蒸留手法が比較されてあり、この図から蒸留によって少ない学習回数でより精度の良いモデルを作成することができている。

参照論文：https://arxiv.org/pdf/1412.6550.pdf

## ３－７．プルーニング（Pruning）

## ３－７－１．プルーニングとは

ネットワークが大きくなると大量のパラメータになるが、すべてのニューロンの計算が精度に寄与しているわけではない。

↓

モデルの精度に **寄与が少ないニューロンを削減** することでモデルの軽量化、高速化が見込まれる。

## ３－７－２．計算の高速化

- 寄与の少ないニューロンの削減を行いモデルの圧縮を行うことで高速に計算を行うことができる。

参照論文：https://arxiv.org/pdf/1506.02626.pdf

## ３－７－３．ニューロンの削減

- ニューロンの削減の手法は、「重みが閾値以下の場合、ニューロンを削減」し、再学習を行う。

## ３－７－４．ニューロン数と精度

- 閾値を高くするとニューロンは大きく削減できるが精度も減少する。

参照論文：https://db-event.jpn.org/deim2017/papers/62.pdf

## ３－７－５．モデルの軽量化まとめ

量子化：重みの精度を下げることにより計算の高速化と省メモリ化を行う技術。

蒸留：複雑で精度の良い教師モデルから軽量な生徒モデルを効率よく学習を行う技術。

プルーニング：寄与の少ないニューロンをモデルから削減し高速化と省メモリ化を行う技術。

# ４．応用技術

## ４－１．Ｍｏｂｉｌｅｎｅｔ

## ４－１－１．論文タイトル

- MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications

## ４－１－２．提案手法

- ディープラーニングモデルは精度は良いが、その分ネットワークが深くなり計算量が増える。

- 計算量が増えると、多くの計算リソースが必要で、お金がかかってしまう。

- ディープラーニングモデルの軽量化・高速化・高精度化を実現した。画像認識の軽量化モデルである。(その名の通りモバイルなネットワーク)

- https://qiita.com/HiromuMasuda0228/items/7dd0b764804d2aa199e4

- 近年の画像認識タスクに用いられる最新のニューラルネットワークアーキテクチャは、多くのモバイルおよび組み込みアプリケーションの実行環境を上回る高い計算資源を必要とされる。

## ４－１－３．一般的な畳み込みレイヤー（比較用）

- 入力特徴マップ(チャネル数)：Ｈ×Ｗ×C

- 畳込みカーネルのサイズ：Ｋ×Ｋ×Ｃ

- 出力チャネル数(フィルタ数)：Ｍ

- ストライド１でパディングを適用した場合の畳み込み計算の計算量：Ｈ×Ｗ×Ｋ×Ｋ×Ｃ×Ｍ

## ４－１－４． MobileNets

- 一般的な畳み込みレイヤーは計算量が多い

- MobileNetsはDepthwise ConvolutionとPointwise Convolutionの組み合わせで軽量化を実現

## ４－１－５．Depthwise Convolution

### ４－１－５－１．仕組み

- 入力マップのチャネルごとに畳み込みを実施する。（M＝１）

- 出力マップをそれらと結合(入力マップのチャネル数と同じになる)

- 通常の畳み込みカーネルは全ての層にかかっていることを考えると計算量が大幅に削減可能である。

- 各層ごとの畳み込みなので層間の関係性は全く考慮されない。通常はPW畳み込みとセットで使うことで解決する。

- 畳み込み計算の計算量：Ｈ×Ｗ×Ｃ×Ｋ×Ｋ

## ４－１－６．Pointwise Convolution

### ４－１－６－１．仕組み

- 1 x 1 convとも呼ばれる(正確には1 x 1 x C、カーネルが１×１)

- 入力マップのポイントごとに畳み込みを実施

- 出力マップ(チャネル数)はフィルタ数分だけ作成可能(任意のサイズが指定可能)

- 畳み込み計算の計算量：Ｈ×Ｗ×Ｃ×Ｍ

## ４－１－７．MobileNetのアーキテクチャ

- Depthwise Separable Convolutionという手法を用いて計算量を削減している。通常の畳込みが空間方向とチャネル方向の計算を同時に行うのに対して、Depthwise Separable ConvolutionではそれらをDepthwise ConvolutionとPointwise Convolutionと呼ばれる演算によって個別に行う。

- Depthwise Convolitionはチャネル毎に空間方向へ畳み込む。すなわち、チャネル毎にDK×DK×１のサイズのフィルターをそれぞれ用いて計算を行うため、その計算量は（Ｈ×Ｗ×Ｃ×Ｋ×Ｋ）となる。

- 次にDepthwise Convolutionの出力をPointwise Convolutionによってチャネル方向に畳み込む。すなわち、出力チャネル毎に１×１×Mサイズのフィルターをそれぞれ用いて計算を行うため、その計算量は（Ｈ×Ｗ×Ｃ×Ｍ）となる。

## ４－２．Ｄｅｎｓｅｎｅｔ

## ４－２－１．論文タイトル

- Densely Connected Convolutional Networks. G. Huang et., al. 2016

- [https://arxiv.org/pdf/1608.06993.pdf](https://arxiv.org/pdf/1608.06993.pdf)

- [https://www.slideshare.net/harmonylab/densely-connected-convolutional-networks](https://www.slideshare.net/harmonylab/densely-connected-convolutional-networks)

## ４－２－２．概要

- Dense Convolutional Network（以下、DenseNet）は、畳込みニューラルネットワーク（以下、CNN）アーキテクチャの一種である。ニューラルネットワークでは層が深くなるにつれて、学習が難しくなるという問題があったが、Residual Network（以下、ResNet）などのCNNアーキテクチャでは前方の層から後方の層へアイデンティティ接続を介してパスを作ることで問題を対処した。DenseBlockと呼ばれるモジュールを用いた、DenseNetもそのようなアーキテクチャの一つである。

- 初期の畳み込み

- Denseブロック

- 変換レイヤー

- 判別レイヤー

## ４－２－３．Denseブロック

### ４－２－３－１．出力層に前の層の入力を足しあわせる。

- 層間の情報の伝達を最大にするために全ての同特徴量サイズの層を結合する

### ４－２－３－２．特徴マップの入力に対し、下記の処理で出力を計算する。

- Batch正規化

- Relu関数による変換

- 3 x 3畳み込み層による処理

### ４－２－３－３．前スライドで計算した出力に入力特徴マップを足し合わせる。

- 入力特徴マップのチャンネル数がℓ x kだった場合、出力は(ℓ+1) x kとなる。

### ４－２－３－４．第ℓ層の出力をとすると　Xℓ　＝　Hℓ（[x0,x1,・・・,xℓ-1]）

### ４－２－３－５．kをネットワークの成長率（growth rate）と呼ぶ。

- kが大きくなるほど、ネットワークが大きくなるため、小さな整数に設定するのがよい

## ４－２－４．Transition Layer

- CNNでは、中間層でチャネルサイズを変更し、特徴マップのサイズを変更し、ダウンサンプリングを行うため、Transition Layerと呼ばれる層でDence blockをつなぐ。

## ４－２－５．DenseNetとResNetの違い

- DenseBlockでは **前方の各層** からの出力全てが、後方の層への入力として用いられる。

- RessidualBlockでは **前１層** の入力のみ後方の層へ入力として用いられる。

## ４－２－６．成⻑率(Growth Rate)

- DenseNet内で使用されるDenseBlockと呼ばれるモジュールでは成⻑率(Growth Rate)と呼ばれるハイパーパラメータが存在する。

- DenseBlock内の各ブロック毎にk個ずつ特徴マップのチャネル数が増加していく時、kを成⻑率と呼ぶ。

## ４－３．Ｌａｙｅｒ正規化/Ｉｎｓｔａｎｃｅ

## ４－３－１．Ｂａｔｃｈｎｏｒｍ

### ４－３－１－１．BatchNormの概要

- レイヤー間を流れるデータの分布を、 **ミニバッチ単位** で平均が0・分散が1になるように **正規化** する。

- Batch Normalizationはニューラルネットワークにおいて学習時間の短縮や初期値への依存低減、過学習の抑制など効果がある。

### ４－３－１－２．の問題点

- Batch Sizeが小さい条件下では、学習が収束しないことがあり、代わりにLayer Normalizationなどの正規化手法が使われることが多い。

- Batch Sizeがマシン性能に依存するため、比較検討が難しい。

### ４－３－１－３．Batch Norm以外の正規化との比較

●Batch Norm

○ミニバッチに含まれるsample（画像）の同一チャネルが同一分布に従うよう正規化

●Layer Norm

○それぞれのsample（画像）の全てのpixelsが同一分布に従うよう正規化

●Instance Norm.

○さらにchannelも同一分布に従うよう正規化

論文参照:https://arxiv.org/pdf/1607.08022.pdf

●Group Norm

-1つのデータの任意のチャネル数に対して正規化 したもの。

※あくまでもバッチサイズが小さい時にしかBatch Normよりも良い効果を発揮しないので、使用例は下記の通り。

- Layer Norm: RNN系やTransformer

- Instance Norm: GAN系(StyleGAN)

- Group Norm: 画像認識系(参照論文：https://arxiv.org/pdf/1803.08494.pdf)

### ４－３－１－４．Batch Norm.

●H x W x CのsampleがN個あった場合に、N個の同一チャネルが正規化の単位となる。

○RGBの3チャネルのsampleがN個の場合は、それぞれのチャンネルの平均と分散を求め正規化を実施(図の⻘い部分に対応)。チャンネルごとに正規化された特徴マップを出力する。

●ミニバッチのサイズを大きく取れない場合には、効果が薄くなってしまう。

参照論文: Ioffe, S. et al. &quot;Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift&quot; (2015)

## ４－３－２．Ｌａｙｅｒ ｎｏｒｍ

### ４－３－２－１．Layer Norm

- N個のsampleのうち一つに注目。H x W x Cの全てのpixelが正規化の単位。

- RGBの3チャネルのsampleがN個の場合は、あるsampleを取り出し、全てのチャネルの平均と分散を求め正規化を実施(図の⻘い部分に対応)。特徴マップごとに正規化された特徴マップを出力

- **ミニバッチの数に依存しない** ので、上記の問題を解消できていると考えられる。

- 参照論文: https://arxiv.org/pdf/1607.06450.pdf

●Layer Normは、入力データや重み行列に対して、以下の操作を施しても、出力が変わらないことが知られている。

- 入力データのスケールに関してロバスト

- 重み行列のスケールやシフトに関してロバスト

- 詳細は下記のスライド参照

※https://www.slideshare.net/KeigoNishida/layer-normalizationnips

## ４－３－３．Instance Norm

### ４－３－２－１．Instance Norm

- 各sampleのchannelごとに正規化する。

- コントラストの正規化に寄与・画像のスタイル転送やテクスチャ合成タスクなどで利用される。

## ４－４．Ｗａｖｅｎｅｔ

## ４－４－１．Wavenet

- 「AlphaGo」を作った、Google傘下のDeepMindが開発した、生の音声波形を生成する深層学習モデルである。2016年にモデルが発表され、2017年に実用化段階にいたった。最初のリリース時点（2016年）ではWaveNetは非常に計算コストが高かった。その後、新しく改良され2017年には、実時間に比べて20倍の速さで音声を生成するようになった。

- Ｐｉｘｅｌ　ＣＮＮ（高解像度の画像を精密に生成できる手法）を音声に応用した。

- 時系列データに対して畳み込み（Ｄｉｌａｔｅｄ　ｃｏｎｖｏｌｕｔｉｏｎ）を適用した。

- コンピュータによる音声合成・音声認識に対して大きな衝撃を与えた技術といわれる。

※確認問題：

- 深層学習を用いて結合確率を学習する際に、効率的に学習が行えるアーキテクチャを提案したことがWaveNet の大きな貢献の1 つである。提案された新しいConvolution 型アーキテクチャは、 **Dilated causal convolution** と呼ばれ、結合確率を効率的に学習できるようになっている。

- Dilated causal convolutionを用いた際の大きな利点は、単純なConvolution layer と比べて、 **パラメータ数に対する受容野が広い** ことである。

## ４－５．Ｓｅｑ２Ｓｅｑ

## ４－５－１．Ｓｅｑ２Ｓｅｑの概要

- Ｅｎｃｏｄｅｒ-Ｄｅｃｏｄｅｒモデルとも呼ばれる。

- 入力系列がＥｎｃｏｄｅ（内部状態に変換）され、内部状態からＤｅｃｏｄｅ（系列に変換）する。

- 実応用上も、入力・出力共に系列情報なものは多い。

## ４－５－１．Ｓｅｑ２Ｓｅｑの構成

- ＲＮＮ：系列データを読み込むために、再帰的に動作するニューラルネットワーク（出力が再び入力になる）。

系列データを舐めて内部状態に変換できる（内部状態ベクトルを出力する）。

- 言語モデル：単語の並びに確率（尤度）を与える。事前の情報により事後確率を求める。

※実習：

![lecture_chap1_exercise_public](img/Web_capture_11-7-2021_183216_localhost.jpeg)

                ※実習５．評価の出力が空欄となる部分は未だ調査中です。

## ４－６．Ｔｒａｎｓｆｏｒｍｅｒ

## ４－６－１．Transformerの概要

- 計算速度向上のため、 **再帰的ニューラルネットワーク (RNN) を使わずに、 Attention機構を使って** 並列計算を可能にするモデルです。

- Attention （注意機構）は従来の RNN のモデル Seq2Seq などでも使われていました。

- 合計が１となる重みを設定して、各隠れ層に分配していく仕組みである。

-　** 辞書オブジェクト** の機能となっている。入力であるqueryをkeyの検索により得られる類似の度合を、ソフトマックス関数で処理して、得られる正規化ベクトルによって、一致するValueを求めている。（Key Value Attention）

- Attentionには、Source Target AttentionとSelf-Attentionがあり、Transformer は **Self-Attention** を使用している。Self-Attentionは入力を全て同じにして学習的に注意箇所を決めていきます。文脈を反映した自己表現が得られます。

- Transformer は文章などのシーケンスから別の文章などのシーケンスを予測するモデルとして発表されましたが、 Transformer の Encoder 部分を使ったモデルは文章分類などシーケンスからカテゴリを予測する問題等でも高い性能を出しており、特に最近発表された同じく Google の BERT (Bidirectional Encoder Representations from Transformers) は言語理解の様々なタスクのベンチマークで圧倒的なスコアを達成しました。

## ４－６－２．Transformerの構成 ～ Encoder-Decoder

- Transformer-EncoderはSelf-Attentionにより、文脈を考慮して、各単語をエンコードする。

- Position-Wise Feed-Forward Networksにより、 **位置情報を保持** したまま準伝播させます。

- Scaled dot product attentionにより、全単語に関するAttentionをまとめて計算する。（次元に応じてスケーリングしている）

- Multi-Head attentionでは、重みパラメータの異なる８個のヘッドを使用して、それぞれのヘッドが異なる種類の情報を収集して、役割分担をしている。

- Transformer-Decoderは６層（Encoderと同じ）あり、 **各層で２種類の Attention**を使用している。

## ４－６－３．Transformerの特徴

- Aｄｄ（Residual Connection）：入出力の差分を学習することで、学習を効率化する。

- Ｎｏｒｍ（Layer Normalization）：入力のバラツキを正則化して抑えることで、処理を高速化する。

- RNNを用いないので単語列の **語順情報** （位置情報）を追加する必要があるので **Position Encoding** を行っている。アドレス情報が２進数で付加される。

※実習：

![lecture_chap2_exercise_public](img/Web_capture_11-7-2021_195258_localhost.jpeg)

## ４－７．物体検知

入力は画像で、出力の違いによって、タスクが分類、物体検知、意味領域分割、個体領域分割の４種類ある。

- Classification（分類）：画像に対して、ラベルを出力する。

- OBJECT DELECTION（物体検知）：バウンディングボックス（物体の検出位置）

- SEMANTIC SEGMENTATION（意味領域分割）：各ピクセルにラベル出力する。

- INSTANCE SEGMENTATION（個体領域分割）：各ピクセルにラベル出力し、個体も識別する。

## ４－７－１．Object Delection（物体検知）

- 一般的な出力は、何処（バウンディングボックス）に何（ラベル）がどんなコンフィデンス（確率）で存在しているかを示す。

- 共通のデータセットによる結果が精度評価の指標となる。

- データセットの特徴としては、クラス数（物体の種類）、サンプル画像数、Box/画像（１画像当たり物体数）の３つが挙げられる。

### ４－７－１－１．２段階検出器：フレームワーク～ＳＳＤ、ＹＯＬＯなど

- 候補領域の検出とクラスの推定を別々に行う。

- 相対的に精度が高い傾向がある。

- 相対的に計算量が大きく推論が遅い傾向がある。

### ４－７－１－２．１段階検出器：フレームワーク～ＦａｓｔＲＣＮＮ、ＭａｓｋＲＣＮＮなど

- 候補領域の検出とクラスの推定を同時に行う。

- 相対的に精度が低い傾向がある。

- 相対的に計算量が小さく推論が早い傾向がある。

### ４－７－１－３．ＳＳＤ（Ｓｉｎｇｌｅ Ｓｈｏｔ Ｄｅｔｅｃｔｏｒ）

### ４－７－１－３－１．ＳＳＤの概要

- Default Boxを設置する。学習によって検出したい物体に合わせて変形し、コンフィデンスを出力する。

- ベースネットワークとしてＶＧＧ１６が使用される。

- マルチスケール特徴マップ

## ４－８．セグメンテーション

## ４－８－１．Semantic segmentation（意味領域分割）

### ４－８－１－１．アップサンプリング

- 解造像度が落とすことによって、画像のサイズが元の画像より小さくなっている。

- アップサンプリングする（プーリングによって解像度を落として領域分割した画像のサイズを元に戻すこと）。

- プーリングで失われた情報が復元されるわけではない。

- アップサンプリング前に、低レイヤからの出力の加算によって、ローカルな情報：輪郭を補完する。

### ４－８－１－２．FCN(Fully Convolutional Network)

- ＶＧＧ１６の最後の３つのＦＣ層を、ＣＮＮに置き換えることで、特徴のヒートマップ画像にしている。

### ４－８－１－３．Deconvolution/Transposed convolution

- 特徴マップのPixel間隔をstrideだけ空ける。

- 特徴マップの周りに(kernel size – 1) – paddingだけ余白を作る。

- 畳み込み演算を行う。

### ４－８－１－４．Unpooling

- プーリング時に位置情報を残しておいて、アップサンプリングに再利用する。

### ４－８－１－５．Dilated Convolution

- カーネルのpixelに隙間を設けることで、Convolutionの段階で受容野を広げている。