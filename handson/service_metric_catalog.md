# AWS Service Metric Catalog

この表は、動画台本とスライド設計の元になるサービス別メトリクスカタログです。全メトリクスの網羅ではなく、初動で見る候補に絞る。

## 使い方

- `Alarm`: すぐ通知候補になるもの
- `Dashboard`: 状況把握に置くもの
- `Investigation`: 原因分析で見るもの
- `Event/Log/Audit`: CloudWatch MetricsよりEventBridge、CloudTrail、ログ、設定監査で見るもの

## Core Catalog

| Service | First metrics / signals | Primary use | Notes |
| --- | --- | --- | --- |
| AWS Lambda | `Errors`, `Invocations`, `Duration`, `Throttles`, `IteratorAge` | Alarm / Dashboard / Investigation | 業務エラーは`Errors`ではなく構造化ログ、metric filters、EMF、カスタムメトリクスで扱う。 |
| Amazon API Gateway | `Count`, `4XXError`, `5XXError`, `Latency`, `IntegrationLatency` | Alarm / Dashboard | 4xxは必ずしも障害ではない。5xxとLatencyを入口にする。 |
| Elastic Load Balancing (ELB: ALB / NLB) | `RequestCount`, target 5xx, load balancer 5xx, `TargetResponseTime`, `HealthyHostCount`, connection error系 | Alarm / Dashboard | ユーザー入口の可用性とバックエンド健全性を見る。 |
| Amazon EC2 | `StatusCheckFailed`, `CPUUtilization`, `NetworkIn/Out`, disk系はCloudWatch AgentまたはAmazon Elastic Block Store (EBS)側 | Alarm / Investigation | CPUだけをSLIにしない。Status checkとアプリ/ELB指標を合わせる。 |
| Amazon Elastic Block Store (EBS) | read/write ops, bytes, latency, `VolumeQueueLength`, `BurstBalance`, throughput/IOPS utilization系 | Dashboard / Investigation | Amazon EC2遅延の原因候補として見る。EBS単体でユーザー影響を断定しない。 |
| Amazon EC2 Auto Scaling | desired/in service/pending/terminating capacity, scaling activity, lifecycle hook failure | Alarm / Event | 「台数が足りない」「増えない」「置換が止まる」を見る。 |
| Amazon Elastic Container Service (ECS) / AWS Fargate | `CPUUtilization`, `MemoryUtilization`, running/pending task count, service deployment events, ELB target health | Alarm / Dashboard | サービス影響はELB/API側、原因候補はtaskとresource側で見る。 |
| Amazon Elastic Kubernetes Service (EKS) | Container Insights pod/node CPU/memory, restarts, node health, control plane logs, kube-state metrics if installed | Dashboard / Investigation | EKSは導入している監視基盤で見える範囲が変わる。CloudWatchだけに閉じない。 |
| Amazon RDS | `CPUUtilization`, `DatabaseConnections`, `FreeStorageSpace`, `FreeableMemory`, read/write latency, read/write IOPS, `DiskQueueDepth`, replica lag | Alarm / Dashboard | 通常RDSとして扱う。容量、接続、遅延、レプリカ遅延を分ける。 |
| Amazon Aurora | CPU, connections, `AuroraReplicaLag`, `BufferCacheHitRatio`, deadlocks, commit latency, serverless capacity/ACU系 | Alarm / Dashboard | Amazon RDSと分ける。クラスタ/インスタンス、Serverless、レプリカ遅延の観点が違う。 |
| Amazon DynamoDB | consumed read/write capacity, throttled requests, successful request latency, system errors, user errors | Alarm / Dashboard | throttleとlatencyを優先。user errorsは設計判断が必要。 |
| Amazon ElastiCache | engine CPU, memory usage, current connections, evictions, cache hit rate, replication lag, swap | Alarm / Dashboard | evictionsとmemory、replication lag、hit rateを分ける。 |
| Amazon S3 | `BucketSizeBytes`, `NumberOfObjects`, request count, 4xx/5xx, first byte latency, total request latency | Dashboard / Alarm | request metricsは設定が必要な場合がある。4xxはアプリ仕様と分ける。 |
| Amazon Elastic File System (EFS) | `PercentIOLimit`, `BurstCreditBalance`, `ClientConnections`, metered IO bytes, permitted throughput | Dashboard / Investigation | 遅延の原因候補として、I/O上限とburst creditを見る。 |
| AWS Backup | backup/copy/restore job success/failure, recovery point状態, EventBridge job events | Alarm / Event | メトリクス単体よりジョブ失敗イベントを重視する。 |
| AWS Transfer Family | bytes/files transferred, transfer failure, auth failure, server/user activity logs | Dashboard / Log | 転送失敗と認証失敗はログ/EventBridgeも見る。 |
| Amazon Simple Queue Service (SQS) | `ApproximateAgeOfOldestMessage`, visible/not visible messages, sent/received/deleted, DLQ depth | Alarm / Dashboard | ageとDLQを強く見る。件数だけでは遅延を判断しない。 |
| Amazon Simple Notification Service (SNS) | messages published, notifications delivered, notifications failed, delivery status logs | Alarm / Log | 配信失敗は通知候補。詳細はdelivery logsで確認する。 |
| Amazon Simple Email Service (SES) | send, delivery, bounce, complaint, reject, rendering failure, reputation系 | Alarm / Dashboard | bounce/complaint/rejectを強く見る。送信量だけでは健全性を判断しない。 |
| Amazon EventBridge | invocations, failed invocations, throttled rules, DLQ/dead-letter invocations | Alarm / Event | failed invocationsとDLQを入口にする。イベントパターン不一致はメトリクスだけでは分からない。 |
| Amazon Kinesis Data Streams | incoming records/bytes, iterator age, read/write throughput exceeded, put success | Alarm / Dashboard | `IteratorAge`は消費遅延の重要指標。 |
| Amazon Data Firehose | incoming bytes/records, delivery success, data freshness, throttled records, destination-specific delivery metrics | Alarm / Dashboard | destinationごとに見る指標が変わる。data freshnessを重視する。 |
| Amazon VPC / NAT Gateway / AWS Transit Gateway | NAT bytes, packets drop, error port allocation, TGW bytes/packets, Amazon VPC Flow Logs rejects | Dashboard / Log | Amazon VPC自体ではなくNAT/TGW/Flow Logsなど観測点を分ける。 |
| AWS Site-to-Site VPN | tunnel state, tunnel data in/out | Alarm / Dashboard | 2トンネルの片系断と両系断を分ける。 |
| AWS Direct Connect | connection state, bps/pps ingress/egress, error/CRC/light level系 | Alarm / Dashboard | 回線状態とトラフィック量を分けて見る。 |
| Amazon CloudFront | requests, bytes, total error rate, 4xx/5xx error rate, origin latency, cache hit rate | Alarm / Dashboard | viewer側とorigin側を分ける。 |
| Amazon Route 53 | health check status, child health check count, query volume/logs where enabled | Alarm / Dashboard | DNSそのものはログ/health checkと合わせる。 |
| AWS Global Accelerator | endpoint health, new flow count, processed bytes, dropped/error flow candidates | Alarm / Dashboard | endpoint healthとtrafficを分ける。 |
| AWS WAF | allowed requests, blocked requests, counted requests, CAPTCHA/challenge系, rule labels | Dashboard / Investigation | block増加は攻撃と誤検知を分けて判断する。 |
| Amazon Elastic Container Registry (ECR) | image scan findings, push/pull events, lifecycle policy results, repository activity | Event/Log/Audit | 通常のリクエストメトリクス監視より、スキャン結果とイベントが主役。 |
| AWS CodeCommit | repository events, approval/merge activity, CloudTrail | Event/Log/Audit | 通常はCloudWatch Metricsで追わない。イベント/監査が主役。 |
| AWS CodeBuild | build count, duration, succeeded/failed builds, queued/build phase failures | Alarm / Event | failed buildとdurationを入口にする。 |
| AWS CodeDeploy | deployment success/failure, rollback events, alarm-based rollback | Event / Alarm | deployment failureイベントを重視する。 |
| AWS CodePipeline | pipeline/stage/action execution success/failure, duration | Event / Dashboard | pipeline failure通知が主役。 |
| AWS X-Ray | traces, latency distribution, errors, faults, throttles, service map | Investigation | CloudWatch Metricsというよりトレース分析の入口。 |
| Amazon Athena | query success/failure, processed bytes, execution time, queue/planning/service time, workgroup usage | Dashboard / Cost | 失敗、処理量、実行時間をworkgroup単位で見る。 |
| Amazon GuardDuty | finding count, severity, account/region, EventBridge/Security Hub | Event/Audit | メトリクス監視というよりfinding対応が主役。 |
| Amazon Inspector | vulnerability finding count, severity, affected resource, EventBridge/Security Hub | Event/Audit | findingの優先度付けが主役。 |
| AWS Certificate Manager (ACM) | certificate expiry, renewal events, validation failure | Alarm / Event | 証明書期限は見る価値が高いが、通常の性能メトリクスではない。 |
| AWS Key Management Service (KMS) | access denied/throttle/error in CloudTrail, key rotation/material expiration, quota pressure | Event/Log/Audit | 通常はCloudTrail、Service Quotas、設定監査が主役。 |
| AWS Directory Service | directory health, domain controller health/resource, authentication/DNS/logs | Dashboard / Log | 深い監視は環境依存。通常はhealthとログを入口にする。 |

## Metrics-Light Services

以下は「監視不要」ではないが、通常のCloudWatchメトリクスアラームを増やす対象としては優先度が低い。講座最後のセッションでまとめて扱う。

| Service | Why metrics are not the main entry | Better first check |
| --- | --- | --- |
| Amazon GuardDuty | findingが主成果物で、性能メトリクスではない | EventBridge、Security Hub、finding severity |
| Amazon Inspector | vulnerability findingが主成果物 | EventBridge、Security Hub、Amazon ECR/Amazon EC2/AWS Lambda findings |
| Amazon Elastic Container Registry (ECR) | サービス性能よりスキャン結果、push/pullイベント、鮮度が重要 | image scan findings、EventBridge、lifecycle policy |
| AWS CodeCommit | リポジトリ操作の監査が中心 | EventBridge、CloudTrail |
| AWS CodeDeploy / AWS CodePipeline | 実行失敗イベントが中心 | EventBridge、pipeline/deployment status |
| AWS Key Management Service (KMS) | 利用エラーや権限/クォータ/監査が中心 | CloudTrail、Service Quotas、key rotation/material expiry |
| AWS Directory Service | 監視深度が環境依存 | health events、directory logs、domain controller metrics |

## Production Notes

- 動画化前にAWS公式ドキュメントで各メトリクス名を最終確認する。
- メトリクス名をスライドに詰め込みすぎない。各講義は3〜5個の初動指標に絞る。
- 「見る必要なし」という言い方は、受講者に誤解されやすい。ナレーションでは「CloudWatchメトリクスアラームの主役ではない」と補足する。
