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

## 1.カスタムテーブル (ScutumWAF_CL) の作成
Scutum WAF アラートを格納するため、専用のカスタムテーブルを作成します。GUI でも作成可能ですが、以下 Powershell での作成例です。

```powershell
$tableParams = @'
{
    "properties": {
        "schema": {
            "name": "ScutumWAF_CL",
            "columns": [
                {
                    "name": "TimeGenerated",
                    "type": "datetime",
                    "description": "The time at which the data was generated"
                },
                {
                    "name": "log_id",
                    "type": "string",
                    "description": "Scutum WAF Alert log_id"
                },
                {
                    "name": "ip",
                    "type": "string",
                    "description": "Scutum WAF Request source IP Address"
                },
                {
                    "name": "block",
                    "type": "boolean",
                    "description": "Scutum WAF block status"
                },
                {
                    "name": "category",
                    "type": "string",
                    "description": "Scutum WAF attack category"
                },
                {
                    "name": "uri",
                    "type": "string",
                    "description": "Scutum WAF request URI"
                },
                {
                    "name": "host",
                    "type": "string",
                    "description": "Scutum WAF host"
                },
                {
                    "name": "ts2",
                    "type": "Datetime",
                    "description": "Scutum WAF timestamp"
                },
                {
                    "name": "request",
                    "type": "string",
                    "description": "Scutum WAF HTTP request data"
                },
                {
                    "name": "response",
                    "type": "string",
                    "description": "Scutum WAF HTTP response data"
                }
                ]
        }
    }
}
'@

Invoke-AzRestMethod -Path "/subscriptions/<SubscriptionId>/resourcegroups/<リソースグループ名>/providers/microsoft.operationalinsights/workspaces/<LogAnalytics名>/tables/ScutumWAF_CL?api-version=2022-10-01" -Method PUT -payload $tableParams
```
- 各フィールドの意図は以下の通りです。

| Field Name | 説明
| --- | --- |
| TimeGenerated | ログインジェスチョン時間 (Default) |
| log_id | Scutum WAF Log ID |
| block | Scutum WAF 防御/検知判定 |
| category | Scutum WAF 検知カテゴリ |
| uri | Scutum WAF 検知 URI |
| host | Scutum WAF API 発行対象の FQDN ホスト |
| ts2 | Scutum WAF 検知時間 (JST) |
| request | Scutum WAF HTTP リクエスト情報 (API ``alert_detail`` で取得可能) |
| response | Scutum WAF HTTP レスポンス情報 (API ``alert_detail`` で取得可能) |

## 2. Azure Monitor DCE (データ収集エンドポイント) の作成
- [Azure Monitor Log Ingestion API](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/logs-ingestion-api-overview) を用いるため、データ収集ルールを作成します
- Log Ingestion API を想定した DCR 作成は Azure GUI では作成出来ないため、MS Learn の手順に従って ARM テンプレートで作成します。






