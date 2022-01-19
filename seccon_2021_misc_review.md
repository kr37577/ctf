# hitchhike
コードは以下のコードで与えられている<br>
なおこのコードはpythonで記述されている<br>
```
#!/usr/bin/env python3.9
import os
def f(x):
    print(f'value 1: {repr(x)}')
    v = input('value 2: ')
    if len(v) > 8: return
    return eval(f'{x} * {v}', {}, {})
if __name__ == '__main__':
    print("+---------------------------------------------------+")
    print("| The Answer to the Ultimate Question of Life,      |")
    print("|                the Universe, and Everything is 42 |")
    print("+---------------------------------------------------+")
    for x in [6, 6.6, '666', [6666], {b'6':6666}]:
        if f(x) != 42:
            print("Something is fundamentally wrong with your universe.")
            exit(1)
        else:
            print("Correct!")
    print("Congrats! Here is your flag:")
    print(os.getenv("FLAG", "FAKECON{try it on remote}"))
```

このコードを読むと入力された2つの数の積が42になればFLAGを獲得できると読むことができる<br>
現在(2022/1/19)サーバーがまだ開いているため以下のコードで問題に挑戦できる<br>
```
nc hiyoko.quals.seccon.jp 10042
```

※repr関数とは
>引数付きのコンストラクタ（または初期化子）を文字列で返してくれる関数
https://gammasoft.jp/blog/use-diffence-str-and-repr-python/

1つ目の6は7を入力すれば次のステップに進むことができる<br>
2つ目の6.6は42/6.6を入力すれば次のステップに進むことができる<br>
ここでソースコードをもう一度読むとeval関数を使って2つの数の積を求めていることが分かる<br>
よって3つ目の'666'は 0 or 42 と入力することで次のステップに進むことができる<br>
4つ目の[6666]も 0 or 42 と入力することで次のステップに進むことができる<br>
5つ目の{b'6':6666}はdict型と呼ばれるデータ型で普通に計算しても42と出ることはないのでこの方針だと問題を解くことができない<br>
またdict型の詳細について知りたいなら以下のURLで学ぶことができる<br>
https://qiita.com/yokotanaohiko/items/85f6b9e119f05c6103b8　<br>
https://www.headboost.jp/python-everything-to-know-about-dict/ <br>

eval関数とは<br>
>文字列をPythonのコードとして実行するための関数
https://techacademy.jp/magazine/40662

ここでもう一度ソースコードを読むと以下のコードから8文字以下の入力ならば受け付けていることがわかる
```
 if len(v) > 8: return
    return eval(f'{x} * {v}', {}, {})
```
よって8文字以下の関数を読み出せることがわかるのでpythonの組み込み関数であるhelp関数を引数を渡さずに使ってみる<br>
すると対話型のhelpシステムを呼び出すことができる<br>
対話型のhelpシステムの使い方は以下のURLから知ることができる<br>
https://python.keicode.com/devenv/how-to-use-help.php <br>
この後何らかの関数を打つと関数の使い方が出てきてその後に'--More--'というページャーが出てくる<br>

ページャーとは<br>
>ファイルの中身を1ページずつ見ていけるコマンド（ソフト）のこと
https://wa3.i-3-i.info/word12463.html

ページャーはエクスクラメーションマークから始まる入力をコマンドとして認識して実行するため以下のように打つとFLAGを獲得できる<br>
```
!cat /proc/self/environ
```
/proc/self/environを使った理由<br>
ソースコードを見ると環境変数からFLAGを呼び出していることがわかる<br>
```
    print(os.getenv("FLAG", "FAKECON{try it on remote}"))
```
よってサーバー側の環境変数が保存されている所をcatするとFLAGが得られるため<br>

参考<br>
https://ptr-yudai.hatenablog.com/entry/2021/12/19/232158#Misc-227pts-hitchhike