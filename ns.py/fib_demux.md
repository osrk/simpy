# FIBDemux

- members
  - fib: dict
    - 転送情報ベース。Key： flow id, value: output port
  - outs: list
    - 出力ポートに対応するダウンストリームエレメントのリスト
  - ends: dict
    - 出力ポートに対応するダウンストリームエレメントのリスト
  - default:
    - デフォルトのダウンストリームエレメント
- 動作
  - put() メソッド
    - 受信したパケットのflow_idに対応するends[flow_id] が存在するときは、そこにput()
    - outs[fib[flow_id]] に put()
