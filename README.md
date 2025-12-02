# GetScutumWAFAlertsToSentinel
このレポジトリは Scutum WAF を API で取得して、Microsoft Sentinel に Azure Monitor Log IngestionAPI で送信するテンプレートを公開しています。

# テンプレート説明

- ScutumWAFSentinelShort
  - 定期ポーリングで動作します（初期値: 1時間）
  - Scutum WAF に対して API で ``https://api.scutum.jp/api/v1/alert`` をポーリングします
    - API の仕様から、``HTTP Request``/ ``HTTP Response`` は含まれません
    - ``HTTP Request``/ ``HTTP Response`` が必要であれば、後述の ScutumWAFSentinelDetail を利用して下さい
  - 取得した API のデータが含まれていれば、データを成型して Azure Monitor DCR に API で転送します

- ScutumWAFSentinelDetail
 - 定期ポーリングで動作します（初期値: 1時間）
  - Scutum WAF に対して API で ``https://api.scutum.jp/api/v1/alert`` をポーリングします
  - 取得した API のデータが含まれていれば、アラートデータの ``log_id`` をキーに、``https://api.scutum.jp/api/v1/alert_detail`` を取得します
    - ``HTTP Request``/ ``HTTP Response`` を取得します
  - 最後にデータを成型して Azure Monitor DCR に API で転送します

# テンプレート導入時のパラメータ
- 本テンプレートは導入時にパラメータを指定します

| Parameter Name | 説明 | 例 |
| --- | --- | --- |
| Play Book Name | Logic Apps テンプレート名 | ``ScutumWAFSentinelDetail`` |
| Scutum Api Key | Scutum WAF の API キー | ``xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`` |
| Scutum User Id | Scutum WAF の ユーザー ID | ``XXXXXXXX`` |
| Scutum Host Fqdn | Scutum WAF のホスト FQDN | ``www.xxx.co.jp`` |
| Dce Ingest Url | Azure Monitor DCE (データ収集エンドポイント) URL | ``https://dce-xxxxx.ingest.monitor.azure.com`` |
| Dcr Immutable Id | Azure Monitor DCR (データ収集ルール) Id | ``dcr-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`` |

- Scutum Api Key / UserId / Host FQDN については、Logic Apps テンプレートの Secret String として保存されるため、導入後テンプレートには表示されません
 - ただし、ロジックアプリの Run history 上では、パラメータが埋め込まれて動作するログが確認できるため、設定した情報は閲覧が出来るようになっています
 - 秘匿情報として扱いたい場合は、エンコードなどを検討して下さい

# 事前準備
- 本テンプレートはカスタムログ / Azure Monitor Log Ingestion API を用いるため、事前に Log Analytics 側にカスタムテーブルの作成、DCE (データ収集エンドポイント) / DCR (データ収集ルール)が必要です。


