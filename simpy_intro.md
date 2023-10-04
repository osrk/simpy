
## Install

```
pip install simpy
```
```
$ pip install simpyDefaulting to user installation because normal site-packages is not writeable
Collecting simpy
  Downloading simpy-4.0.2-py2.py3-none-any.whl (30 kB)
Installing collected packages: simpy
Successfully installed simpy-4.0.2
```

## Basic Concepts

SimPy は離散イベント シミュレーション ライブラリです。アクティブなコンポーネント (車両、顧客、メッセージなど) の動作はプロセスでモデル化されます。すべてのプロセスは環境内に存在します。それらは環境と対話し、イベントを通じて相互に対話します。

プロセスは単純な Python ジェネレーターによって記述されます。通常の関数であるかクラスのメソッドであるかに応じて、プロセス関数またはプロセス メソッドと呼ぶことができます。彼らは生きている間、イベントを作成し、それが起こるのを待つためにそれを提供します。

プロセスがイベントを生成すると、プロセスは一時停止されます。イベントが発生すると、SimPy はプロセスを再開します (イベントが処理されたと呼びます)。複数のプロセスが同じイベントを待つことができます。 SimPy は、イベントが発生したのと同じ順序でそれらを再開します。

重要なイベント タイプはタイムアウトです。このタイプのイベントは、一定の (シミュレートされた) 時間が経過した後に発生します (処理されます)。これらにより、プロセスが指定された時間スリープ (またはその状態を保持) することができます。タイムアウトおよびその他すべてのイベントは、プロセスが存在する環境の適切なメソッド (Environment.timeout() など) を呼び出すことで作成できます。

## Process interaction

Environment.process() によって返される Process インスタンスは、プロセスの対話に利用できます。この最も一般的な 2 つの例は、別のプロセスが終了するのを待つことと、イベントを待っている間に別のプロセスを中断することです。

### Wait for a process
(略)

### Interrupting Another Process

想像してみてください。電気自動車が完全に充電されるまで待つのではなく、充電プロセスを中断して、そのまま運転を開始したいと考えているとします。

SimPy では、`interrupt()` メソッドを呼び出すことで、実行中のプロセスに割り込むことができます。

```python
def driver(env, car):
    yield env.timeout(3)
    car.action.interrupt()
```

`driver`のプロセスには、車の動作プロセスへの参照があります。 3 つのタイム ステップを待った後、そのプロセスが中断されます。

割り込みは、割り込み例外としてプロセス関数にスローされ、割り込みを受けたプロセスによって処理できる (はずです)。その後、プロセスは次に何をするかを決定できます (元のイベントを待ち続けるか、新しいイベントを生成するなど)。

```python
class Car(object):
    def __init__(self, env):
        self.env = env
        self.action = env.process(self.run())

    def run(self):
        while True:
            print('Start parking and charging at %d' % self.env.now)
            charge_duration = 5
            # We may get interrupted while charging the battery
            try:
                yield self.env.process(self.charge(charge_duration))
            except simpy.Interrupt:
                # When we received an interrupt, we stop charging and
                # switch to the "driving" state
                print('Was interrupted. Hope, the battery is full enough ...')

            print('Start driving at %d' % self.env.now)
            trip_duration = 2
            yield self.env.timeout(trip_duration)

    def charge(self, duration):
        yield self.env.timeout(duration)
```

Note: 
- `charge_duration` 待つところを、driver プロセスが car.action (Carインスタンスのプロセスを保持) に interrupt() を投げることで割り込みが入る。
- `try:` で通常処理をしておき、`except simpy.Interrupt:` で例外処理(インタラプト)をキャッチする。

このシミュレーションの出力を前の例と比較すると、車が時刻 5 ではなく 3 に運転を開始していることがわかります。

```
env = simpy.Environment()
car = Car(env)
env.process(driver(env, car))
<Process(driver) object at 0x...>
env.run(until=15)
Start parking and charging at 0
Was interrupted. Hope, the battery is full enough ...
Start driving at 3
Start parking and charging at 5
Start driving at 10
Start parking and charging at 12
```


## Resouce

サンプルコード。`bcs` (battery charging station) がリソース。
```python
def car(env, name, bcs, driving_time, charge_duration):
    # Simulate driving to the BCS
    yield env.timeout(driving_time)

    # Request one of its charging spots
    print('%s arriving at %d' % (name, env.now))
    with bcs.request() as req:
        yield req

        # Charge the battery
        print('%s starting to charge at %s' % (name, env.now))
        yield env.timeout(charge_duration)
        print('%s leaving the bcs at %s' % (name, env.now))
```

リソースの `request()` メソッドは、リソースが再び利用可能になるまで待機できるイベントを生成します。再開された場合は、解放するまでそのリソースを「所有」します。

上記のように `with` ステートメントでリソースを使用すると、リソースは自動的に解放されます。 `with` を使用せずに `request()` を呼び出した場合は、リソースの使用が完了したら `release()` を呼び出す必要があります。

リソースを解放すると、次の待機プロセスが再開され、リソースのスロットの 1 つを「所有」します。基本的なリソースは、待機中のプロセスを FIFO (先入れ先出し) 方式で並べ替えます。

リソースには、作成時に`Environment`への参照と容量が必要です。

```python
import simpy
env = simpy.Environment()
bcs = simpy.Resource(env, capacity=2)
```

これで、`car` のプロセスを作成し、リソースへの参照といくつかの追加パラメータを渡すことができます。

```python
for i in range(4):
    env.process(car(env, 'Car %d' % i, bcs, i*2, 5))
<Process(car) object at 0x...>
<Process(car) object at 0x...>
<Process(car) object at 0x...>
<Process(car) object at 0x...>
```

最後に、シミュレーションを開始できます。このシミュレーションでは自動車のプロセスはすべて自動的に終了するため、終了時間を指定する必要はありません。イベントがなくなるとシミュレーションは自動的に停止します。

```
env.run()
Car 0 arriving at 0
Car 0 starting to charge at 0
Car 1 arriving at 2
Car 1 starting to charge at 2
Car 2 arriving at 4
Car 0 leaving the bcs at 5
Car 2 starting to charge at 5
Car 3 arriving at 6
Car 1 leaving the bcs at 7
Car 3 starting to charge at 7
Car 2 leaving the bcs at 10
Car 3 leaving the bcs at 12
```

最初の 2 台の車両は BCS に到着するとすぐに充電を開始できますが、2 台目と 3 台目の車両は待つ必要があることに注意してください。

## How to proceed

SimPy が要件を満たしているかどうかまだわからない場合、またはより多くの機能の動作を確認したい場合は、提供されているさまざまな例を参照してください。

SimPy の特定の側面や機能についてのより詳細な説明を探している場合は、「トピック ガイド」セクションが役立つかもしれません。 
