# AWSサービスメトリクスカタログ

この表は、動画台本とスライド設計の元になるサービス別メトリクスカタログです。全メトリクスの網羅ではなく、初動で見る候補に絞る。

## 使い方

- `アラーム`: すぐ通知対象になるもの
- `ダッシュボード`: 状況把握に置くもの
- `調査`: 原因分析で見るもの
- `イベント・ログ・監査`: CloudWatchメトリクスよりEventBridge、CloudTrail、ログ、設定監査で見るもの

## 主要カタログ

| サービス | 最初に見る指標 / シグナル | 主な用途 | 補足 |
| --- | --- | --- | --- |
| AWS Lambda | `Errors`, `Invocations`, `Duration`, `Throttles`, `IteratorAge` | アラーム / ダッシュボード / 調査 | 業務エラーは`Errors`ではなく構造化ログ、metric filters、EMF、カスタムメトリクスで扱う。 |
| Amazon API Gateway | `Count`, `4XXError`, `5XXError`, `Latency`, `IntegrationLatency` | アラーム / ダッシュボード | 4xxは必ずしも障害ではない。5xxとレイテンシを入口にする。 |
| Elastic Load Balancing (ELB: ALB / NLB) | `RequestCount`, target 5xx, load balancer 5xx, `TargetResponseTime`, `HealthyHostCount`, 接続エラー系 | アラーム / ダッシュボード | 利用者入口の可用性とバックエンド健全性を見る。 |
| Amazon EC2 | `StatusCheckFailed`, `CPUUtilization`, `NetworkIn/Out`, ディスク系はCloudWatch AgentまたはAmazon Elastic Block Store (EBS) 側 | アラーム / 調査 | CPUだけをSLIにしない。Statusチェックとアプリ/ELB指標を合わせる。 |
| Amazon Elastic Block Store (EBS) | 読み取り/書き込み処理数、バイト数、レイテンシ、`VolumeQueueLength`, `BurstBalance`, スループット/IOPS 利用率系 | ダッシュボード / 調査 | Amazon EC2遅延の原因候補として見る。EBS単体でユーザー影響を断定しない。 |
| Amazon EC2 Auto Scaling | 目標台数、稼働台数、待機/終了中の台数、スケール実行、ライフサイクルフック失敗 | アラーム / イベント | 「台数が足りない」「増えない」「置換が止まる」を見る。 |
| Amazon Elastic Container Service (ECS) / AWS Fargate | `CPUUtilization`, `MemoryUtilization`, 実行中/待機中タスク数、サービスデプロイイベント、ELBターゲット正常性 | アラーム / ダッシュボード | サービス影響はELB/API側、原因候補はタスクとリソース側で見る。 |
| Amazon Elastic Kubernetes Service (EKS) | Container InsightsのPod/Node CPU/メモリ、再起動、ノード健全性、コントロールプレーンログ、kube-stateメトリクス（導入時） | ダッシュボード / 調査 | EKSは導入している監視基盤で見える範囲が変わる。CloudWatchだけに閉じない。 |
| Amazon RDS | `CPUUtilization`, `DatabaseConnections`, `FreeStorageSpace`, `FreeableMemory`, 読み取り/書き込みレイテンシ、読み取り/書き込みIOPS, `DiskQueueDepth`, レプリカ遅延 | アラーム / ダッシュボード | 通常RDSとして扱う。容量、接続、遅延、レプリカ遅延を分ける。 |
| Amazon Aurora | CPU、接続数、`AuroraReplicaLag`, `BufferCacheHitRatio`, デッドロック、コミットレイテンシ、Serverless容量/ACU系 | アラーム / ダッシュボード | Amazon RDSと分けて扱う。クラスタ/インスタンス、Serverless、レプリカ遅延の観点が違う。 |
| Amazon DynamoDB | 消費済み読み取り/書き込み容量、スロットル要求、リクエスト成功時レイテンシ、システムエラー、利用者エラー | アラーム / ダッシュボード | スロットルとレイテンシを優先。利用者エラーは設計判断が必要。 |
| Amazon ElastiCache | エンジンCPU、メモリ使用量、現在接続数、追い出し件数、キャッシュヒット率、レプリケーション遅延、スワップ | アラーム / ダッシュボード | 追い出し、メモリ、レプリケーション遅延、ヒット率を分ける。 |
| Amazon S3 | `BucketSizeBytes`, `NumberOfObjects`, リクエスト数、4xx/5xx、FirstByteレイテンシ、全体レイテンシ | ダッシュボード / アラーム | リクエスト系メトリクスは設定が必要な場合がある。4xxはアプリ仕様と分けて扱う。 |
| Amazon Elastic File System (EFS) | `PercentIOLimit`, `BurstCreditBalance`, `ClientConnections`, メータリングIOバイト、許容スループット | ダッシュボード / 調査 | 遅延の原因候補として、I/O上限とburst creditを見る。 |
| AWS Backup | バックアップ/コピー/復元ジョブの成功/失敗、リカバリポイント状態、EventBridgeジョブイベント | アラーム / イベント | メトリクス単体よりジョブ失敗イベントを重視する。 |
| AWS Transfer Family | 送受信バイト数/ファイル数、転送失敗、認証失敗、サーバー/利用者アクティビティログ | ダッシュボード / ログ | 転送失敗と認証失敗はログ/EventBridgeも見る。 |
| Amazon Simple Queue Service (SQS) | `ApproximateAgeOfOldestMessage`, 表示可/非表示メッセージ、送信/受信/削除数、DLQ深度 | アラーム / ダッシュボード | 最古メッセージの遅延とDLQを優先して見る。件数だけでは遅延を判断しない。 |
| Amazon Simple Notification Service (SNS) | 発行数、配信成功数、配信失敗数、配信ステータスログ | アラーム / ログ | 配信失敗は通知候補。詳細は配信ログで確認する。 |
| Amazon Simple Email Service (SES) | 送信、配信、バウンス、苦情、拒否、レンダリング失敗、レピュテーション系 | アラーム / ダッシュボード | バウンス/苦情/拒否を重視する。送信量だけでは健全性を判断しない。 |
| Amazon EventBridge | 呼び出し数、失敗呼び出し数、スロットルルール数、DLQ/デッドレター呼び出し数 | アラーム / イベント | 失敗呼び出しとDLQを入口にする。イベントパターン不一致はメトリクスだけでは分からない。 |
| Amazon Kinesis Data Streams | 受信レコード/バイト、イテレータエイジ、読み取り/書き込みスループット超過、登録成功数 | アラーム / ダッシュボード | `IteratorAge`は消費遅延の重要指標。 |
| Amazon Data Firehose | 受信レコード/バイト、配信成功、データ鮮度、スロットルレコード、宛先別配信メトリクス | アラーム / ダッシュボード | 宛先によって見る指標が変わる。データ鮮度を重視する。 |
| Amazon VPC / NAT Gateway / AWS Transit Gateway | NATバイト数、パケットドロップ、エラー用ポート割り当て、TGWバイト数/パケット数、Amazon VPC Flow Logsの拒否 | ダッシュボード / ログ | Amazon VPC自体ではなくNAT/TGW/Flow Logsなど観測点を分ける。 |
| AWS Site-to-Site VPN | トンネル状態、トンネル入出力 | アラーム / ダッシュボード | 2トンネルの片系断/両系断を分ける。 |
| AWS Direct Connect | 接続状態、入出力量（bps/pps）、エラー/CRC/光レベル系 | アラーム / ダッシュボード | 回線状態とトラフィック量を分けて見る。 |
| Amazon CloudFront | リクエスト数、バイト数、総エラー率、4xx/5xxエラー率、オリジンレイテンシ、キャッシュヒット率 | ダッシュボード / アラーム | ビューア側とオリジン側を分ける。 |
| Amazon Route 53 | ヘルスチェック状態、子ヘルスチェック数、クエリ量/ログ（有効時） | ダッシュボード / アラーム | DNSそのものはログ/ヘルスチェックと合わせる。 |
| AWS Global Accelerator | エンドポイント健全性、新規フロー数、処理済みバイト数、破棄/エラーフロー候補 | アラーム / ダッシュボード | エンドポイント健全性とトラフィックを分けて見る。 |
| AWS WAF | 許可リクエスト数、拒否リクエスト数、計測対象リクエスト数、CAPTCHA/チャレンジ系、ルールラベル | ダッシュボード / 調査 | 拒否増加は攻撃と誤検知を分けて判断する。 |
| Amazon Elastic Container Registry (ECR) | イメージスキャン結果、push/pullイベント、ライフサイクルポリシー結果、リポジトリアクティビティ | イベント・ログ・監査 | 通常のリクエストメトリクス監視より、スキャン結果とイベントが主役。 |
| AWS CodeCommit | リポジトリイベント、承認/マージ活動、CloudTrail | イベント・ログ・監査 | 通常はCloudWatchメトリクスで追わない。イベントと監査が主役。 |
| AWS CodeBuild | ビルド数、実行時間、成功/失敗ビルド数、待機/実行フェーズ失敗 | アラーム / イベント | 失敗ビルドと実行時間を入口にする。 |
| AWS CodeDeploy | デプロイ成功/失敗、ロールバックイベント、アラーム起因ロールバック | イベント / アラーム | デプロイ失敗イベントを重視する。 |
| AWS CodePipeline | パイプライン/ステージ/アクション実行の成功/失敗、処理時間 | イベント / ダッシュボード | パイプライン失敗通知が主役。 |
| AWS X-Ray | トレース、レイテンシ分布、エラー、障害、スロットル、サービスマップ | 調査 | CloudWatchメトリクスというよりトレース分析の入口。 |
| Amazon Athena | クエリ成功/失敗、処理バイト数、実行時間、キュー/計画/サービス時間、ワーキンググループ利用状況 | ダッシュボード / コスト | 失敗、処理量、実行時間をワーキンググループ単位で見る。 |
| Amazon GuardDuty | 検知件数、重大度、アカウント/リージョン、EventBridge/Security Hub | イベント / 監査 | メトリクス監視より検知対応が主役。 |
| Amazon Inspector | 脆弱性検知件数、重大度、影響リソース、EventBridge/Security Hub | イベント / 監査 | 検知の優先度付けが主役。 |
| AWS Certificate Manager (ACM) | 証明書有効期限、更新イベント、検証失敗 | アラーム / イベント | 証明書期限は重要だが、通常の性能メトリクスではない。 |
| AWS Key Management Service (KMS) | CloudTrail上のアクセス拒否/スロットル/エラー、キー回転・資材期限、クォータ圧迫 | イベント・ログ・監査 | 通常はCloudTrail、Service Quotas、設定監査が主役。 |
| AWS Directory Service | ディレクトリ健全性、ドメインコントローラー健全性/リソース、認証/DNS/ログ | ダッシュボード / ログ | 監視深度は環境依存。通常はhealthとログを入口にする。 |

## メトリクス軽量サービス

以下は「監視不要」ではないが、通常のCloudWatchメトリクスアラームを増やす対象としては優先度が低い。講座最後のセッションでまとめて扱う。

| サービス | メトリクスが主観測軸にならない理由 | まず最初に確認するもの |
| --- | --- | --- |
| Amazon GuardDuty | 検知結果が主成果物で、性能メトリクスではない | EventBridge、Security Hub、重大度 |
| Amazon Inspector | 脆弱性検知結果が主成果物 | EventBridge、Security Hub、Amazon ECR / Amazon EC2 / AWS Lambdaの検知 |
| Amazon Elastic Container Registry (ECR) | サービス性能よりスキャン結果、push/pullイベント、鮮度が重要 | イメージスキャン結果、EventBridge、ライフサイクルポリシー |
| AWS CodeCommit | リポジトリ操作の監査が中心 | EventBridge、CloudTrail |
| AWS CodeDeploy / AWS CodePipeline | 実行失敗イベントが中心 | EventBridge、パイプライン/デプロイ状態 |
| AWS Key Management Service (KMS) | 利用エラーや権限/クォータ/監査が中心 | CloudTrail、Service Quotas、キー回転と暗号化資材期限 |
| AWS Directory Service | 監視深度が環境依存 | 健全性イベント、ディレクトリログ、ドメインコントローラー指標 |

## 運用メモ

- 動画化前にAWS公式ドキュメントで各メトリクス名を最終確認する。
- メトリクス名をスライドに詰め込みすぎない。各講義は3〜5個の初動指標に絞る。
- 「見る必要なし」という言い方は、受講者に誤解されやすい。ナレーションでは「CloudWatchメトリクスアラームの主役ではない」と補足する。
