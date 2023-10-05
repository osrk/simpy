# Comparision of Network-Simulators
https://ti.tuwien.ac.at/ecs/people/albeseder/simcomp/simcomp

## まとめ
- SimPy が使いやすい
  - Good
    - 無料で少量利用に制限がない。
    - プログラマビリティが高い
  - Bad
    - 性能は低いため、小規模な実験に適している

## シミュレーションツール
### OMNet++ V3.1
- 通信ネットワークのシミュレーションを目的としたオープンソースのシミュレーション環境
- GUIのサポートやIPv4, IPv6, Ethernet, MPLSなどの一般的なモデルが利用可能
- 学術や非営利目的では無料だが、**商用利用にはライセンスが必要**
- NEDという独自のネットワーク定義言語でモジュールを構造化し、C++で機能を実装
- Unix系システムやWindowsに対応
### ns-2 V2.28
- ネットワーク研究によく使われる離散事象シミュレータ
- モデルの実装にはTCLのオブジェクト指向拡張であるOTclを使用
- モジュールの機能はOTclやC++でコーディング可能
- 一般的なネットワーク技術やアプリケーションを含み、簡単かつ迅速にネットワークの仕様とシミュレーションができる
- **研究や教育目的では無料**で利用可能
- Unix系システム向けに設計されているが、WindowsのCygWinでも動作する
### QualNet V3.8
- UCLAで開発されたGloMoSimを基にした商用シミュレータ
- モデルや仕様の作成に**GUI**を備えており、手動でファイルに記述するよりも簡単に小規模から中規模のネットワークを指定できる
- **無線ネットワークのシミュレーションを主な対象としている**が、有線ネットワークもサポートしている
- 環境やライブラリが充実しており、データ分析やチャート作成などが容易にできる
- GUIには主にJavaを使用し、LinuxやWindowsに対応している。シミュレータ自体は、指定されたターゲットシステムに最適化されたCプログラムである
### SimPy V1.6
- スクリプト言語Pythonの拡張であり、簡単な離散事象シミュレーションができる
- 制限なくシミュレーションを変更できるという利点があるが、Pythonはスクリプト言語であるため、**シミュレーションの性能は低い**
- GNU LibraryまたはLGPLライセンスの下で公開されており、**誰でも自由に利用できる**
- Pythonの拡張であるため、Pythonが動作するすべてのシステムで利用可能であり、LinuxやWindowsも含まれる
### Enterprise Dynamics V6.1 Build 497
- **産業プロセスのシミュレータ**であり、モデルの開発やシミュレーションに優れたGUIを持つ
- 独自のスクリプト言語4DScriptでコアモジュールを拡張できる
- **ネットワークシミュレーションは主な対象ではない**ため、性能は最適化されていないが、可能である
- QualNetと同様に商用製品であるが、Windowsのみで利用可能である
### AnyLogic V5.3.1
- もう一つの**商用製品**であり、**離散、連続、混合のシミュレーション方法をサポートする唯一のシミュレータ**である
- しかし、この評価では離散部分のみを使用した
- GUIは非常に洗練されており、モデルの設計やインタラクティブな実験が速やかに行える
- Javaに基づいており、すべてのモジュールはカスタムJava関数で拡張できる
- シミュレータはWindowsのみで利用可能である

## Implementation
SimPy以外は省略
### SimPy
- 以前のシミュレータはすべてネットワーク シミュレーション専用でした。
- ただし、SimPy は一般的なプロセスベースの離散イベント シミュレータです。
- そこで、2 つの異なる種類のプロセスが作成されました。 
- 「Host」と呼ばれる 1 つのプロセスは、指数関数的な出発間隔を持つメッセージを作成するだけです。
- 次に、このプロセスは 2 番目のプロセス「Router」のプロシージャを呼び出してメッセージを送信します。 
- 「Router」プロセスはキューを管理し、ルーター キューから 1 秒あたり 20 パケットの速度でパケットを取り出します。 
- 「Router」プロセスは、受信およびドロップされたパケットを追跡するため、シミュレーションの終了時にパケット損失率を提供できます。

## Performance evaluation

- このパフォーマンス比較では **OMNet++ が圧倒的な勝者**
- ただし、OMNet++ で**シミュレーションを作成するために必要な労力は、他のすべてのシミュレータに比べてかなり大きくなる**。しかし、OMNet++ では、速度を最適化してシミュレーションを最初から作成することができ、包括的な INET パッケージも利用できる。
- **商用シミュレータである QualNet と Enterprise Dynamics** は両方とも**グラフィカル デザイン モード**を提供しており、これは QualNet がネットワーク シミュレーションの初心者でも問題なく使用できることを示しているが、それでも多くの落とし穴があります。 
  - **QualNet の主な目的はワイヤレス シミュレーション**であるため、デフォルトでどの設定が有効になるか注意する必要があります。実践者を対象としたツールでもあります。そのため、論理シミュレーションには必要なく、さらには削除するのが困難な、多くの高度で包括的な機能が提供されます。
  - **Enterprise Dynamics は産業プロセスのシミュレーションを目的としている**ため、それほど優れたパフォーマンスは得られない。優れた洗練されたグラフィカル ユーザー インターフェイスを備えているため、問題をモデル化するための最も簡単なツールだった。 Enterprise Dynamics のパフォーマンスはやや残念。キュー サイズ 200 のホスト 18 台のみに対する別のパフォーマンス実行は、3 分 17 秒しか続きませんでした。そのため、Enterprise Dynamics は拡張性があまり高くないか、使用率を高めるために使用されることが多く、非常に非効率な機能がいくつかあります。
- SimPy
  - 唯一のPython 用の**離散イベント シミュレーション拡張機能**であり、あらゆる離散シミュレーションに使用る。
  - Python は洗練されたメソッドとデータ構造も提供するため、**プログラミングは非常に簡単**。
  - ただし、**小規模なプロジェクトや、シミュレーション モデルの小規模で迅速な実験に適している**。
- OMNeT++
  - 私の個人的なお気に入りは OMNet++ です。
  - 最も明確でオープンなプログラミング インターフェイスを備えている (一方、ns-2 にはいくつかの部分でいくつかの制限がある)
  - コンパイル済みのグラフィカル ユーザー インターフェイス (TK を使用) も提供されている
  - シミュレーションの実行中にパケット フローを観察したり、データを変更したりできます。
  - したがって、実験機能も ns-2 より優れています。
  - ただし、ns-2 は OTcl スクリプト インターフェイスを提供し、モデルと一部のモジュールを極めて高速にコーディングできるため、モデルの変更は ns-2 よりも手間がかかる。

職業的な調整は、統合 GUI デザイナー モードを備えた 2 つの商用製品で最も低くなります。

## SimPy 版のソース解析

### 使用ライブラリなど
- psyco はPythonのJITコンパイラ

```python:smallpcnet.py
#!/usr/bin/env python

from SimPy.Simulation import *
from random import expovariate,Random
import time,sys

try:
    import psyco
    psyco.full()
except:
    pass
```

### Msg クラス
- ホスト、ルーター間で送受信するメッセージ

```python:smallpcnet.py
class Msg:
    def __init__(self, name):
        self.name = name
```

### Router クラス
#### コンストラクタ
- self.queuesize: キューサイズ
- self.rate: レート
- self.queue: キュー
- self.event: メッセージを受信したときに発生させるイベント
- self.precv: 受信メッセージカウンタ
- self.pdropped: 廃棄メッセージカウンタ

```python:smallpcnet.py        
class Router(Process):
    def __init__(self, queuesize, rate):
        Process.__init__(self)
        self.queuesize = queuesize
        self.rate = rate
        self.queue = []
        self.event = SimEvent("newmsg")
        self.precv = 0
        self.pdropped = 0
        activate(self, self.send())
```
#### `receive`
- メッセージを受け取る
  - キューの長さがキューサイズより小さいとき
    - キューにメッセージを挿入する
    - `self.event.signal()` を実行する。send()が起きる。
  - キューの長さがキューサイズ以上の時
    - メッセージをドロップする (ドロップカウントを+1し、メッセージを削除する)

```python:smallpcnet.py   
    def receive(self, message):
        self.precv += 1
        if DEBUG: print "received msg '%s' at time %f" % (message.name, now())
        if len(self.queue) < self.queuesize:
            self.queue.insert(0,message)
            self.event.signal()
        else:
            self.pdropped +=1
            if DEBUG: print "dropped packet!"
            del message
```
#### `printpl`
- 統計情報 (ドロップ数/受信数) をprintする

```python:smallpcnet.py   
    def printpl(self):
        print "%f" % (float(self.pdropped)/float(self.precv))
```
#### `send`
- キューにメッセージがある時
  - キューからメッセージを取り出す
  - 一定時間待つ (`yield hold, self, self.rate`)
  - メッセージを削除する
- キューにメッセージがない時
  - イベントを待つ (`yield waitevent, self, self.event`)

```python:smallpcnet.py   
    def send(self):
        while True:
            if len(self.queue) > 0:
                m = self.queue.pop()
                if DEBUG: print "sent message '%s' at time %f" %(m.name, now())
                yield hold, self, self.rate
                del m
                
            # if queue is empty wait for event from receiver process
            if len(self.queue) == 0:
                yield waitevent, self, self.event
```

### Host クラス
- パラメータは rate, router, name
- self.g 乱数発生のインスタンス
- 実行ステップ
  - 指数分布の時間だけ待つ
    - `yield hold, self, self.g.exploraiate`
      - `yield hold, self, time` は`time`分だけ待つときに使う
      - expovariate(self.rate) は指数分布 (平均self.rate)
  - メッセージmを作成
  - ルーターにメッセージを送る (`router.receive(m)`)

```python:smallpcnet.py 
class Host(Process):
    def __init__(self, rate, router, name):
        Process.__init__(self)
        self.rate = rate
        self.g = Random(time.time()*hash(name))
        self.router = router
        self.name = name

    def execute(self):
        cnt = 0
        while True:
            yield hold, self, self.g.expovariate(self.rate)
            cnt += 1
            m = Msg("%d/%s" % (cnt, self.name))
            self.router.receive(m)
```

### Main プログラム
- パラメータは ホスト数 キューサイズ
```python:smallpcnet.py
# MAIN

if len(sys.argv) != 3:
    print "USAGE: %s <# hosts> <queue size>" % sys.argv[0]
    sys.exit(1)

hosts = int(sys.argv[1])
qsize = int(sys.argv[2])

initialize()

DEBUG = 0

linkrate = 0.05  # 20 packets per second

router = Router(qsize, linkrate)

host=[]
for i in range(hosts):
    # generate host which sends a message with a poisson process with mean
    # of 1 second to the object router
    h = Host(1, router, "host%d" % i)
    activate(h, h.execute())
    host.append(h)

simulate(until=100000)

router.printpl()
```
