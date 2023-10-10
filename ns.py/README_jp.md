
# ns.py: Pythonic 離散事象ネットワークシミュレータ

https://github.com/TL-System/ns.py

この離散イベントネットワークシミュレータは、Python用の汎用離散イベントシミュレーションフレームワークであるsimpyに基づいています。 ns.py は、柔軟で再利用できるように設計されており、パケットジェネレータ、ネットワークリンク、スイッチ要素、スケジューラ、トラフィックシェーパー、トラフィックモニター、デマルチプレクシング要素など、複数のネットワークコンポーネントを簡単に接続するために使用できます。

## インストール
まず、ターミナルを起動し、新しい`conda`環境を作成します(たとえば、 `ns.py`)

```
$ conda update conda
$ conda create -n ns.py python=3.9
$ conda activate ns.py
```

次に、`pip`を使用してns.pyをインストールします。

```
$ pip install ns.py
```

これで終わりです！
これで、examples/ディレクトリのサンプルを実行できます。
現在の環境で Python パッケージをアップグレードするには、次のコマンドを実行します。
```
$ python upgrade_packages.py
```

## 現状のネットワークコンポーネント
すでに実装されているネットワークコンポーネントは次のとおりです。

### Packet
- Packet:作成時間、サイズ、パケットID、フローID、送信元、および宛先を運ぶネットワークパケットの単純な表現。

### Generator
- DistPacketGenerator: 到着間隔とパケット サイズの指定された分布に従ってパケットを生成します。

- TracePacketGenerator: トレース ファイルに従ってパケットを生成し、トレース ファイルの各行はパケットを表します。

- TCPPacketGenerator: TCP をトランスポート プロトコルとして使用してパケットを生成します。

- ProxyPacketGenerator: 実世界のパケット(固定パケットサイズ)をシミュレーション環境にリダイレクトします。

### Sink
- PacketSink: パケットを受信し、遅延統計を記録します。

- TCPSink: パケットを受信し、遅延統計を記録し、TCP 送信側に確認応答を生成します。

- ProxySink: 受信したすべてのパケットを宛先の実世界の TCP サーバーにリダイレクトします。

### Port

- Port:単純なテールドロップメカニズムを使用してパケットをドロップする、特定のレートとバッファサイズ(バイト単位またはパケット数)を持つスイッチの出力ポート。

- REDPort:早期ランダム検出(RED)メカニズムを使用してパケットをドロップする、特定のレートとバッファサイズ(バイト単位またはパケット数)を持つスイッチの出力ポート。

### Wire
- Wire: 特定の分布に続く伝搬遅延を持つネットワークワイヤ(ケーブル)。ワイヤの帯域幅は、アップストリーム Port、サーバまたはスケジューリング サーバでモデル化できるため、モデル化する必要はありません。

### Splitter
- Splitter: 元のパケットをポート 1 から送信し、パケットのコピーをポート 2 から送信するだけのスプリッタ。

- NWaySplitter: パケットのコピーを n 個のダウンストリーム要素に送信する n 方向スプリッタ。

### Marker
- TrTCM: パケットを緑、黄、または赤としてマークする 2 レート 3 カラー マーカー (詳細については、RFC 2698 を参照)。

### Demultiplexing
- RandomDemux: 出力ポートをランダムに選択するデマルチプレキシング要素。

- FlowDemux: パケット ストリームをフロー ID で分割するデマルチプレキシング要素。

- FIBDemux: フロー情報ベース(FIB)を使用してフロー ID に基づいてパケット転送を決定するデマルチプレキシング要素。

### Shaper
- TokenBucketShaper: トークンバケットシェイパー。

- TwoRateTokenBucketShaper: コミットレートとピークレート/バーストサイズの両方を備えた2レート3色トークンバケットシェイパー。

### Scheduler
- SPServer: スタティック プライオリティ (SP) スケジューラ。

- WFQServer: 重み付け均等化キューイング (WFQ) スケジューラ。

- DRRServer: デフィシットラウンドロビン (DRR) スケジューラ。

- VirtualClockServer: 仮想クロックスケジューラ。

### Switch

- SimplePacketSwitch: 各送信ポートに FIFO 制限バッファを持つパケットスイッチ。

- FairPacketSwitch:WFQ、DRR、スタティックプライオリティ、または仮想クロックスケジューラ、および各発信ポート上の有界バッファを選択できる公平なパケットスイッチ。また、単純なハッシュ関数を使用して (flow_id、node_id、および port_id) のタプルをクラス ID にマップし、このパラメーターを使用してflow_basedスケジューリングではなくクラスベースのスケジューリングをアクティブにする方法の例も示します。

### Monitor

- PortMonitor: Port のパケット数を記録します。モニター間隔は、指定された分布に従います。

- ServerMonitor: スケジューリング・サーバ (WFQServer, VirtualClockServer, SPServer, DRRServer) のパフォーマンス統計を記録します。

## 現状のユーティリティ
- TaggedStore:タグに基づいてソートされたsimpy.Storeであり、WFQと仮想クロックの実装に役立ちます。

- Config: 構成ファイルからパラメーター設定を読み取るグローバル シングルトン インスタンス。Config()を使用して、インスタンスにグローバルにアクセスします。

## 現状のサンプル(複雑さ順)
これらの例の一部では、matplotlib をインストールする必要があります。ns.py の依存関係のリストには含まれていないため、現在の Python 環境に個別にインストールする必要があります。

- basic.py: 2 つのパケット ジェネレータを伝搬遅延分散を使用してネットワーク ワイヤに接続し、次にパケット シンクに接続する基本的な例。DistPacketGenerator, PacketSink, Wireを示します。

- overloaded_switch.py: ダウンストリーム スイッチ ポートに接続されたパケット ジェネレータを含み、パケット シンクに接続される例。DistPacketGenerator、PacketSink、およびPort を示します。

- mm1.py: この例では、パケットの到着間隔が指数関数的に分散されたパケット サイズを使用してポートをシミュレートする方法を示します。DistPacketGenerator、PacketSink、PortおよびPortMonitor を示します。

- tcp.py: この例は、単純なパケット転送スイッチを介して送信者から受信者への 2 ホップの単純なネットワークを設定する方法と、同じスイッチを介して受信側から送信者に確認応答パケットを送信する方法を示しています。送信側はトランスポート プロトコルとして TCP を使用し、輻輳制御アルゴリズムは構成可能です (TCP Reno や TCP CUBIC など)。TCPPacketGenerator、CongestionControl、TCPSink、WireおよびSimplePacketSwitch を示します。

- token_bucket.py: この例では、バケットサイズがパケットサイズと同じで、バケットレートが入力パケットレートの半分であるトラフィックシェーパーを作成します。DistPacketGenerator、PacketSink、および TokenBucketShaperを示します。

- two_rate_token_bucket.py: この例では、2 レートの 3 色のトラフィック シェーパーを作成します。DistPacketGenerator、PacketSink、および TwoRateTokenBucketShaperを示します。

- static_priority.py: この例では、2 つの静的優先度 (SP) スケジューラを使用して、より複雑な 2 層スケジューラを構築し、アップストリーム スケジューラとダウンストリーム スケジューラをオンにする方法を示します。zero_downstream_buffer、zero_buffer、DistPacketGeneratorおよび PacketSinkSPServerを示します。

- wfq.py: この例では、重み付け均等化キューイング (WFQ) スケジューラを使用する方法と、サーバー モニターを使用して、サンプリング分布を使用してより細かい粒度でパフォーマンス統計を記録する方法を示します。DistPacketGenerator、PacketSink、Splitter、WFQServerServer、およびMonitorを示します。

- virtual_clock.py: この例では、Virtual Clock スケジューラを使用する方法と、サーバー モニターを使用して、サンプリング分布を使用してパフォーマンス統計をより細かい粒度で記録する方法を示します。DistPacketGenerator、PacketSink、Splitter、VirtualClock、QServerServerおよびMonitor を示します。

- drr.py: この例では、デフィシットラウンドロビン (DRR) スケジューラの使用方法を示します。DistPacketGenerator、PacketSink、Splitterおよび DRRServerを示します。

- two_level_drr.py,two_level_wfq.py , two_level_sp.py: これらの例では、赤字ラウンドロビン(DRR)、重み付け均等化キューイング(WFQ)、およびスタティックプライオリティ(SP)サーバで構成される2レベルのトポロジを構築する方法を示しています。また、フロー ID に文字列を使用する方法、およびディクショナリを使用してフローごとの重みを DRR、WFQ、または SP サーバに提供する方法も示し、グループ ID とグループごとのフロー ID を使用してグローバルに一意のフロー ID を簡単に作成できるようにする方法も示します。

- red_wfq.py: この例では、ランダム早期検出 (RED) バッファー (またはテール ドロップ バッファー) と WFQ サーバーを組み合わせる方法を示します。RED またはテールドロップバッファはアップストリーム入力バッファとして機能し、ダウンストリーム要素がゼロバッファ構成であることを認識するように構成されます。WFQ サーバーは、RED またはテールドロップバッファの後のダウンストリーム要素としてゼロバッファリングで初期化されます。ダウンストリーム WFQ サーバーがボトルネックになっている場合、パケットはドロップされます。DistPacketGenerator、PacketSink、Port、REDPort、WFQServerおよび、Splitter基本エレメントを使用してより複雑なネットワーク エレメントを構築する方法と、その方法を紹介します。zero_bufferzero_downstream_buffer

- fattree.py: ネットワーク フロー シミュレーションに FatTree トポロジを構築して使用する方法を示す例。DistPacketGenerator、PacketSink、SimplePacketSwitchおよびFairPacketSwitch を示します。フローごとの公平性が必要な場合は、FairPacketSwitch を用い、重み付け均等化キューイング、赤字ラウンドロビン、または仮想クロックとともに、スイッチの各発信ポートのスケジューリング規範として使用されます。

## エミュレーションモード
ns-3 シミュレーターのエミュ