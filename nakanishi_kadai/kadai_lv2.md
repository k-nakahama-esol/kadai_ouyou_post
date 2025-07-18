# 課題レベル２

generatorを使ってみる

# 内容：  

引数でファイル名をもらって、
ファイルの内容に行数を入れて、標準出力に出す
(引数については、argparserを使う)

## 基礎編

```python
import argparse
import sys

def count_lines_in_file(filename):
    try:
        with open(filename, 'r') as file:
            lines = file.readlines()
            for line in lines:
                line = line.strip()  # 改行文字を削除
                line_count = len(line)
                yield (f"'{filename}' : {line_count}: {line}")
    except FileNotFoundError:
        print(f"ファイル '{filename}' が見つかりません。") 
    except Exception as e:
        print(f"エラーが発生しました: {e}")

def main():
    parser = argparse.ArgumentParser(description='ファイルの行数をカウントします。')
    parser.add_argument('filename', type=str, help='行数をカウントするファイル名')
    args = parser.parse_args()
    
    for msg in count_lines_in_file(args.filename):
        print(msg)

if __name__ == "__main__":
    sys.argv = ['script_name', r"C:\Users\9004033494\Downloads\20250528\output.txt"]
    main()
```

----
# 課題レベル２ 基礎編
Generator を使って、ファイルの行数をカウントし、各行の内容とともに出力します。  
コマンドライン引数でファイル名を受け取り、そのファイルの内容を行ごとに読み込み、行数と内容を出力していますが
その際に、ファイル内容を１行ずつ読み出す関数 count_lines_in_file は generator として実装されています。  
さて、generator という聞きなれない単語が出てきました。 generater とは何でしょうか？  
generator とは、Python のイテレータの一種で、値を一つずつ生成する関数です。  
通常の関数は値を一度に返しますが、generator は `yield` キーワードを使って値を一つずつ返します。  

...何言ってるか分からないって？　

generator を使った関数のより分かりやすい例として、自作のrange 関数を使って説明していこうと思います。  


## range関数を自作関数として書き直す。

```python
def my_range(n):
    """指定された数値までの整数を生成するジェネレータ関数"""
    i = 0
    while i < n:
        yield i
        i += 1

# range関数を使ったいつものforループ
for i in range(10):
    print(f"range(10)の{i}番目の値は、{i}です。")

# my_range関数を使った同様のforループ
for i in my_range(10):
    print(f"my_range(10)の{i}番目の値は、{i}です。")

```

このコードは、Pythonの組み込み関数であるrange関数を自作のジェネレータ関数my_rangeで再実装しています。  
(本当は、range関数はもっと複雑な機能を持っていますが、ここでは説明に必要な、基本的な部分だけを実装しています。)  
実行結果は次のようになります。  

```text
range(10)の0番目の値は、0です。
range(10)の1番目の値は、1です。
range(10)の2番目の値は、2です。
range(10)の3番目の値は、3です。
range(10)の4番目の値は、4です。
range(10)の5番目の値は、5です。
range(10)の6番目の値は、6です。
range(10)の7番目の値は、7です。
range(10)の8番目の値は、8です。
range(10)の9番目の値は、9です。
my_range(10)の0番目の値は、0です。
my_range(10)の1番目の値は、1です。
my_range(10)の2番目の値は、2です。
my_range(10)の3番目の値は、3です。
my_range(10)の4番目の値は、4です。
my_range(10)の5番目の値は、5です。
my_range(10)の6番目の値は、6です。
my_range(10)の7番目の値は、7です。
my_range(10)の8番目の値は、8です。
my_range(10)の9番目の値は、9です。
```

同じですね。やったぜ！  
my_range関数を読んでみれば分かりますが、0からn-1までの整数を一つずつ作って、それを yield というキーワードで返しています。  
yield というキーワードは、return と似ていますが、return は関数を終了させて値を返すのに対し、yield は関数の実行を一時停止して値を返します。  
my_range関数は、呼び出されるたびに中断した位置から再開し、次の値を生成します。  
通常、関数は 与えられた引数のみに依存して、一貫して常に同じ戻り値を返す実装が理想とされますが、my_range関数は内部に状態を持ち、呼ばれるたびに戻り値を生成するような特殊な関数を作ることができるイメージです。  
この機能のことを generator と呼びます。 上のケースでは、yield を使うことで my_range 関数は generator として動作するようになります。  
なんとなく generator のイメージは掴めましたか？  

でも、こんな単純な機能ならわざわざ generator とかいう、わけわからんヤツを連れてこなくてもいいのでは？ なんて思うかもしれません。  
だいたい、yield ってなんやねん。  
なんか読み方もヨクワカランし、文字数も多すぎて（５文字もある！）スペルも覚えられません。  
中高大と英語が壊滅的にダメだった鳥頭の私に喧嘩を売っているに違いない。

プログラミングは自由であるべきだ。使いたくないステートメントを使わない自由が私にはある！  
ためしに書いてみましょう。例えば、こんな感じ。

## yieldを使わない range関数を書いてみる

```python
# 【注】動かないコード
my_range_cnt = 0  # おぞましいグローバル変数を使って、カウントを管理します。
def my_range_simple(n):
    global my_range_cnt
    if my_range_cnt<n:
        i = my_range_cnt
        my_range_cnt += 1
    else:
        raise StopIteration("my_range_simpleの値は、指定された数値を超えました。")
    return i

for i in my_range_simple(10):
    print(f"my_range_simple(10)の{i}番目の値は、{i}です。")

```

なるほど、my_range_simple関数は、引数で指定された数値までの整数を返すように実装されています。  
しかし、残念ながら、これ、動かないコードです。  
なぜなら、for文の中で my_range_simple(10) を呼び出しても、my_range_simple(10) は単に整数を返すだけだからです。  

for文の in の後に続くのは、for文が反復処理できる ナニカ である必要があります。  
その ナニカ のことを、pythonでは イテラブル（反復可能）なオブジェクトと呼んでいます。  
イテラブルなオブジェクトとは、例えば次のようなやつらです。  

- リスト [1, 2, 3]
- タプル (1, 2, 3)
- 文字列 "abc"
- 辞書 {'a': 1, 'b': 2, 'c': 3}
- セット {1, 2, 3}
- range(10) などの組み込み関数
- ジェネレータ関数 （例：my_range(10)）の呼び出し戻り値

これらのオブジェクトは、反復処理が可能で、for文の in の後に続くことができます。  
my_range_simple関数は、呼び出すごとに違う値を返すので、range関数っぽくうごきそうではありますが、戻り値としては単に整数を返すだけなので、イテラブルなオブジェクトではありません。  
そのため、for文の in の後に続けることができません。  

my_range_simple関数がイテラブルなオブジェクトを返すようにすればいいんじゃん？ 例えばリストとかさ。 って思われた方。天才です。  
その通りです。こうすればいい。  

# イテラブルオブジェクトを返すmy_range_simple関数
```python
def my_range_simple_list(n):
    ret = []
    for i in range(n):
        ret.append(i)
    return ret

for i in my_range_simple_list(10):
    print(f"my_range_simple_list(10)の{i}番目の値は、{i}です。")

# 実行結果は想定通りです。
# my_range_simple_list(10)の0番目の値は、0です。
# my_range_simple_list(10)の1番目の値は、1です。
# my_range_simple_list(10)の2番目の値は、2です。
# my_range_simple_list(10)の3番目の値は、3です。
# my_range_simple_list(10)の4番目の値は、4です。
# my_range_simple_list(10)の5番目の値は、5です。
# my_range_simple_list(10)の6番目の値は、6です。
# my_range_simple_list(10)の7番目の値は、7です。
# my_range_simple_list(10)の8番目の値は、8です。
# my_range_simple_list(10)の9番目の値は、9です。
```

やった！ できたやん！ やっぱりgeneratorなんて要らなかったんや！  
でも、ちょっと待ってください。 これホントに大丈夫なんですかね？  
例えば、引数で指定された数値が 10000000とかだったらどうなりますか？  

```python
def my_range_simple_list(n):
    ret = []
    for i in range(n):
        ret.append(i)
    # n=10000000 のとき [0,1,2,3,......9999999] を返す 
    return ret

# 1000万個の整数をリストに格納するので、かなりの量のメモリを消費します。
target_list = my_range_simple_list(10000000)

# 1000万個の整数が含まれるリストって、何バイトくらいの大きさになるんでしょうか？
import sys
print(sys.getsizeof(target_list))

# 0～9999999 までを数え上げるだけの処理が見たいですか？ 私は見たくありません。
# ご覧になりたい方は、以下のコメントアウトを外して実行してください。
# for i in target_list:
#     print(f"my_range_simple_list(10)の{i}番目の値は、{i}です。")

```

sys.getsizeof(target_list) は target_list がメモリ上で何バイトの領域を使っているかを返します。  
実行結果は「 89,095,160 バイト 」でした。    
つまり、約89MB。メモリの無駄遣いも甚だしいし、このリストを生成する処理負荷もバカになりません。  
generator を使えば、こんな苦労はしません。 my_range関数使ったほうが、メモリも処理速度も圧倒的に有利です。  

## どうしても yield のスペルが覚えられなかった私は…
そんなの嫌だ！ 俺はこんな yield とかいう、わけわからんヤツは使いたくない！  
なんとしてでも、メモリ的にも実行速度的にも優れていて、かつ for文の in の後に続ける。つまりイテラブルなオブジェクトを作りたいんだよ！  
yield なんて使わなくても、イテラブルなオブジェクトを作る方法はないのか？  
そう思ったあなたは勉強に勉強を重ね、pythonのイテレータ（イテラブルオブジェクトの処理）について学びました。  
そして、ついにイテレータを実装するためのクラスを作成することに成功します。  

```python
class MyRangeIterator:
    """イテレータクラス"""
    def __init__(self, n):
        self.n = n
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.current < self.n:
            value = self.current
            self.current += 1
            return value
        else:
            raise StopIteration("MyRangeIteratorの値は、指定された数値を超えました。")

for i in MyRangeIterator(10):
    print(f"MyRangeIterator(10)の{i}番目の値は、{i}です。")

```

やった！ できたやん！ 今度こそ本当に、yield 使わなくても for文の in の後で使えるイテラブルオブジェクトが書けたぞ！！  
やっぱり generatorなんて要らなかったんや！  

そっすね。 おめでとうございます。  
でも、これ、いちいち書くのめちゃくちゃ面倒くさくないですか？  
これ書くためにいろいろ勉強するくらいなら、なんとなく yield とかいうヤツを使って、my_range関数を書いたほうがよっぽど楽じゃないですか？  
スペルもさ「 y i e l d （いるーど） 」とかってふせんに書いて貼っておけば、忘れないんじゃない？

そうなんです。 generator を使うと、イテラブルなオブジェクトを簡単に作ることができるんです。  
しかも、余談ですが、メモリの消費量も、処理速度も、上で書いた複雑なコードと同等かそれ以上に優れています。  
同じようなことをやってはいるんですが、yield を使うことで、Cレベルのコードが走るようなオブジェクトが生成されるため高速で  
コードがシンプルになり、可読性も向上すると、こういうことらしいです。  

結論といたしましては・・・  
generator 便利だから、ちゃんと理解して使いましょう！  
ということでした。
