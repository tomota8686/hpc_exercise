# ネットワーク系演習II 高能率計算 1回目レポート
## 学籍番号：32114023

## 名前：岩越 智貴

## 使用計算機環境情報
* CSE
* コンパイラ: g++

# 課題１
ソースコードの計測部分前後にタイマー関数を追記して実行速度を計測した．

10回実行した結果の実行時間の例を以下に示す．
mul_timeが行列積の計算時間、mul_addが行列和の計算時間である。

exercise 1: loop = 10, size = 3
|count|mul_time    |add_time   |
|-----|------------|-----------|
|    0|0.000275    |0.001353   |
|    1|0.000102    |0.000171   |
|    2|8.7e-05     |0.000125   |
|    3|6.1e-05     |0.000169   |
|    4|6.7e-05     |0.000129   |
|    5|0.000101    |0.000143   |
|    6|7.7e-05     |0.000121   |
|    7|0.000837    |0.000118   |
|    8|6e-05       |0.000186   |
|    9|6e-05       |0.000139   |
||||
|  avg|0.000161333 |0.000144556|

考察：行列積の計算時間は何度試行しても、7回目の計算に時間がかかった。
1回目の計算に7回目を除くと、時間がかかっていることがわかる。
また、行列和の計算は、1回目の試行に時間が他の10倍程度かかっていることがわかる。

# 課題２
コンパイラオプションをO0~Ofastに変えてプログラムをコンパイルした．

結果は以下のようになった．

exercise 2: loop = 100, size = 1024

|option|time [ms]|
|------|---------|
|none  |38.8705  |
|-O0   |39.0309  |
|-O1   |10.267   |
|-O2   |8.10946  |
|-O3   |3.42826  |
|-Ofast|2.74398  |

またコンパイル時間を下記に示す．
```shell-session 
cmj14023@cs-d42:~/lec/hpc_exercise/src/hpc-exercise > time make
g++ -std=c++0x -fopenmp -Wno-unused-result -march=native -O0    -c -o utils/mat.o utils/mat.cpp
g++ -std=c++0x -fopenmp -Wno-unused-result -march=native -O0    -c -o utils/mat_util.o utils/mat_util.cpp
g++ -std=c++0x -fopenmp -Wno-unused-result -march=native -O0    -c -o main.o main.cpp
g++ -std=c++0x -fopenmp -Wno-unused-result -march=native -O0  -o hpc_exercise utils/mat.o utils/mat_util.o main.o

real    0m1.753s
user    0m1.524s
sys     0m0.156s

cmj14023@cs-d42:~/lec/hpc_exercise/src/hpc-exercise > time make
g++ -std=c++0x -fopenmp -Wno-unused-result -march=native -Ofast    -c -o utils/mat.o utils/mat.cpp
g++ -std=c++0x -fopenmp -Wno-unused-result -march=native -Ofast    -c -o utils/mat_util.o utils/mat_util.cpp
g++ -std=c++0x -fopenmp -Wno-unused-result -march=native -Ofast    -c -o main.o main.cpp
g++ -std=c++0x -fopenmp -Wno-unused-result -march=native -Ofast  -o hpc_exercise utils/mat.o utils/mat_util.o main.o

real    0m3.692s
user    0m3.464s
sys     0m0.164s
```

考察：何もしない場合と-O0、-O1、-O2、-O3、-Ofastを試した。実行速度は、-O0が一番遅く、-Ofastが一番速いという結果になった。
一方、コンパイルにかかった時間は、-OFastが一番遅く、-O0が一番速いという結果になった。

# 課題３

## <はじめに>
古典的な高速化技法では、プログラムを書き換えることにより高速化をはかる。
コンパイラオプションで最適化を図ると、今回の効果を確認しづらくなってしまう。
このことより、課題3から課題12まではコンパイルオプションは以下のとおりとする。
```Makefile
CXXFLAGS = -std=c++0x -fopenmp -Wno-unused-result -march=native
```

以下のようにコードを書き換えた．

書き換え前
```cpp
const float v = x.data[i];
ret.data[i] = 3.f * v * v * v * v * v * v
    + 3.f * v * v * v * v * v
    + 3.f * v * v * v * v
    + 3.f;
```
書き換え後
```cpp
const float v = x.data[i];
y.data[i] = 3.f * ((v * v * v * (v * (v * (v + 1.f) + 1.f) + 1.f)) +1.f);
```

計算速度は以下のとおりとなった。
```shell-session
exercise 3: loop = 100000, size = 64

|method|time [ms]|
|------|---------|
|before|0.0249922|
|after |0.0108397|

info:
default parameter: default_loop = 100000, default_size = 64
diff from ans: 1.63106e-13
```

考察：ホーナー法を使った場合、使わない場合に比べて約1.5倍早くなっている。


# 課題４
小さな行列に対して、各要素xを下記の定数倍するプログラムを作成し、数式の展開前後で計算速度を比較する。
```math
(2\pi+\sqrt{5}+0.5^{2})x
```

毎回計算をするコードは以下のとおりである。
```cpp
for (int i = 0; i < s; i++)
{
    //計算 ansに書き込み
    const float v = x.data[i];
    ans.data[i] = (2.f * M_PI + sqrt(5) + pow(3.f, 2.0)) * v;
}
```

先に共通部分を計算する場合は以下のとおりである。
```cpp
for (int i = 0; i < s; i++)
{
    //計算 yに書き込み
    const float v = x.data[i];
    y.data[i] = tmp * v;
}
```

計算速度は以下に示す。
```shell-session
exercise 4: loop = 10000, size = 64

|method |time [ms]|
|-------|---------|
|inline |0.110791|
|precomp|0.00631822|

info:
default parameter: default_loop = 10000, default_size = 64
diff from ans: 2.45426e-09
```


# 課題２２

画像の張り付けサンプル

<img src="loofline.png" alt="ルーフライン" width="600px">
図ｘ：シングルスレッドとマルチスレッドのルーフライン．

参考までに，CSEは1.3TFLOPSくらい出ます（試すのは絶対に夜中で．）．

．．．

# 課題２９





# 画像処理課題１

# 画像処理課題２

1回目の課題はここまで．
画像処理の共通の課題である上記１，２を忘れずにやること．
これ以降の課題は2回目のレポートです．

