# OpenTime Prototype Plan v0.1

## 1. 文書の目的

本書は、OpenTime Server の試作開発に向けた初期計画書である。

OpenTime Server は、オープンソースで再現可能な Stratum 1 時刻サーバーを目指すプロジェクトである。既存の要件定義ではデュアル Raspberry Pi Compute Module 4 を前提としていたが、本計画では今後の新規基板設計を見据え、Raspberry Pi Compute Module 5 を中心に再整理する。

本書の目的は、以下を明確にすることである。

- 何を作るのか
- なぜ作るのか
- どのような構成で試作するのか
- どの程度の予算が必要か
- どの順番で検証するのか
- 何をもって試作成功と判断するのか

本書は完成仕様書ではなく、試作・資金調達・共同開発者募集のための計画書である。部品型番や詳細設計は、試作と検証を通じて更新する。

## 2. 背景と課題

ネットワーク機器、仮想化基盤、ログ基盤、監視基盤では、正確な時刻同期が不可欠である。特にファイアウォール、L3 スイッチ、Proxmox VE などの仮想化基盤、ログサーバーでは、障害解析や証跡管理のために安定した NTP サーバーが必要となる。

一方で、商用の高機能タイムサーバーは高価であり、個人ユーザー、ホームラボ、メイカー、独立系エンジニア、小規模事業者にとって導入しづらい。

一般的な Raspberry Pi + GPS モジュールによる NTP サーバーは安価に構築できるが、以下の課題が残る。

- 電源の信頼性が低い
- microSD カード運用になりやすい
- 単一ノード障害に弱い
- GNSS や PPS の状態が見えにくい
- ラックマウントや常時運用に向かない
- OSS ハードウェアとして再現可能な形にまとまりにくい

OpenTime Server は、これらの課題を解決し、OSS として公開可能な実用的タイムアプライアンスを目指す。

## 3. プロジェクトの目的

OpenTime Server の目的は、以下の条件を満たす時刻サーバーを開発することである。

- GNSS と PPS を利用した Stratum 1 NTP サーバーであること
- 常時稼働を前提にした信頼性設計を行うこと
- Active / Standby 構成によりサービス継続性を高めること
- 電源を 2 系統化し、電源障害に対する耐性を持たせること
- 状態を前面 UI または外部監視から把握できること
- 19 インチラックに搭載可能な筐体を目指すこと
- 回路図、基板、筐体、ソフトウェア、BOM、手順を OSS として公開すること

## 4. 初期ターゲット構成

試作機の初期ターゲットは、以下の構成とする。

| 項目 | 方針 |
|---|---|
| 製品仮称 | OpenTime Pro Prototype |
| コンピュート | Raspberry Pi Compute Module 5 x2 |
| 運用構成 | Active / Standby |
| 電源 | Dual Power Module |
| 時刻源 | Single high-quality GNSS |
| 時刻同期 | GNSS NMEA / UBX + PPS |
| NTP 実装 | chrony |
| PPS 処理 | pps-gpio / pps-tools |
| HA | keepalived / VRRP |
| UI | M5Stack Basic または同等の小型 MCU UI |
| 表示 | LED マトリックス、PPS 同期 LED、状態 LED |
| 筐体 | 1U または 2U ラックマウント |
| OS 方針 | eMMC 起動、読み取り専用 OS、RAM log |
| 公開形態 | GitHub モノレポ |

## 5. 採用予定アーキテクチャ

初期試作では、Dual CM5 + Dual Power + Single GNSS の構成を採用する。

```text
DC IN A -> Power Module A --+
                            +-> Power OR / Ideal Diode -> System Power Rail
DC IN B -> Power Module B --+

System Power Rail
  +-> CM5-A
  +-> CM5-B
  +-> GNSS
  +-> Supervisor / M5Stack
  +-> LED / Front Panel
```

```text
GNSS Antenna
  |
GNSS Module
  +-> NMEA / UBX -> CM5-A
  +-> NMEA / UBX -> CM5-B
  +-> PPS -> Buffer / Fanout -> CM5-A GPIO
  +-> PPS -> Buffer / Fanout -> CM5-B GPIO
```

CM5-A と CM5-B はそれぞれ chrony を実行し、同一 GNSS から取得した NMEA / UBX と PPS を用いて時刻同期を行う。ネットワーク側では keepalived により仮想 IP を共有し、クライアントからは単一の NTP サーバーとして見える構成とする。

## 6. CM5 を採用する理由

本計画では、既存要件の CM4 から CM5 へ更新する。

主な理由は以下である。

- 新規基板設計であるため、CM5 前提で設計できる
- CM5 の方が今後の供給期間と拡張性に期待できる
- 4GB RAM / 32GB eMMC 構成により、開発・監視・将来機能に余裕を持たせられる
- eMMC を使用することで microSD カードを排除できる
- 将来的な Web UI、Prometheus exporter、A/B アップデートなどの実装余地が広がる

初期試作では以下の構成を想定する。

| 項目 | 仕様 |
|---|---|
| Module | Raspberry Pi Compute Module 5 |
| Wireless | なし |
| RAM | 4GB |
| Storage | 32GB eMMC |
| Quantity | 2 |

## 7. 冗長化方針

OpenTime Pro Prototype では、単に CM5 を 2 台搭載するだけではなく、電源も 2 系統化する。

### 7.1 冗長化する対象

- CM5 ノード
- 電源入力
- DC/DC 電源モジュール
- 物理 Ethernet ポート
- NTP サービスの仮想 IP

### 7.2 初期試作で冗長化しない対象

- GNSS モジュール
- GNSS アンテナ
- キャリア基板
- 筐体
- 前面 UI

GNSS まで 2 系統化すると設計難度と費用が大きく上がるため、初期試作では Single GNSS とする。ただし、将来の OpenTime HA 版では Dual GNSS / Dual Antenna を検討する。

## 8. 電源設計方針

電源は信頼性に直結するため、初期試作の重要項目とする。

### 8.1 基本方針

- DC 入力を 2 系統持つ
- 各入力に保護回路を設ける
- 2 つの電源モジュールを ORing する
- 片系電源喪失時もシステムが継続動作する
- Power Good、電圧、電流、温度を監視する
- 前面 UI または監視 API で電源状態を表示する

### 8.2 検討する電源機能

- 逆接保護
- 過電流保護
- TVS ダイオードによるサージ保護
- eFuse
- Ideal diode controller
- 入力電圧監視
- 電源モジュール温度監視
- 電源抜け止め機構

電源コネクタは、試作後期または筐体設計時に決定する。候補は、ねじロック式 DC コネクタ、端子台、ラッチ付きコネクタ、産業用丸型コネクタなどとする。

## 9. GNSS / PPS 方針

OpenTime Server の精度と安定性は、GNSS と PPS の品質に大きく依存する。

### 9.1 GNSS 選定方針

初期試作では、一般的な測位用 GPS モジュールではなく、タイミング用途に適した GNSS モジュールまたは評価基板を優先する。

候補は以下とする。

- u-blox NEO-F10T
- u-blox ZED-F9T
- u-blox LEA-F9T
- 上記または同等品を搭載した評価ボード / ブレイクアウト基板

NEO-F9P など RTK 測位向けモジュールは高精度測位には適するが、時刻サーバー用途ではタイミング向け品番を優先する。

### 9.2 PPS 分配方針

単一 GNSS から出力される PPS を、バッファまたはファンアウト回路を通じて CM5-A と CM5-B の GPIO に入力する。

PPS は時刻同期の心臓部であるため、以下を重視する。

- 3.3V ロジックレベルの整合
- 低ジッタ
- 低スキュー
- ESD 保護
- 配線長の均等化
- ノイズ源からの距離
- オシロスコープによる波形確認

## 10. ネットワーク方針

初期試作では、各 CM5 に独立した Ethernet ポートを割り当てる。

```text
CM5-A -> Ethernet A -> Network
CM5-B -> Ethernet B -> Network
```

内部スイッチ IC による 1 ポート化は、構成を簡単に見せられる一方で、スイッチ IC、RJ45、ケーブルが単一障害点になりやすい。そのため、初期の高信頼構成では 2 ポート構成を優先する。

クライアントは VRRP による仮想 IP を NTP サーバーとして参照する。

## 11. UI / 監視方針

前面 UI は、時刻サーバーの状態を直感的に把握するために設ける。

### 11.1 表示したい情報

- 現在時刻
- PPS lock 状態
- GNSS lock 状態
- Active / Standby 状態
- VRRP 状態
- 電源 A / B 状態
- Ethernet link 状態
- 温度
- chrony offset / jitter の概要
- 障害状態

### 11.2 UI 候補

- M5Stack Basic
- LED マトリックス
- PPS 同期 LED
- ステータス LED
- 将来的な Web UI
- Prometheus exporter

M5Stack または Supervisor MCU は、時刻同期の主経路には入れず、監視・表示・簡易設定に限定する。

## 12. ソフトウェア方針

初期試作では、実装を複雑にしすぎず、Linux 上の標準的な構成を優先する。

### 12.1 主要ソフトウェア

- chrony
- pps-tools
- pps-gpio
- keepalived
- systemd
- monitoring scripts
- optional: Prometheus node exporter
- optional: custom exporter

### 12.2 OS 運用方針

- eMMC 起動
- microSD カード不使用
- 読み取り専用 OS または OverlayFS
- ログは RAM ディスク中心
- 永続ログは必要最小限
- 設定バックアップ機構を用意
- 将来的に A/B アップデートを検討

## 13. 概算予算

OpenTime Pro Prototype の目標予算は 200,000 円前後とする。

| 区分 | 内容 | 概算 |
|---|---|---:|
| Compute | CM5 Wi-Fi なし / 4GB RAM / 32GB eMMC x2 | 68,200 円 |
| 開発用 IO / 変換基板 | CM5 IO Board 等 | 15,000 - 30,000 円 |
| GNSS | タイミング向け GNSS 評価基板 / モジュール | 15,000 - 35,000 円 |
| GNSS アンテナ | アンテナ、SMA、ケーブル、保護部品 | 8,000 - 20,000 円 |
| 電源 | Dual Power、DC/DC、保護、監視 | 15,000 - 30,000 円 |
| UI | M5Stack Basic、LED、表示部品 | 10,000 - 20,000 円 |
| ネットワーク | RJ45、ESD、周辺部品 | 5,000 - 12,000 円 |
| 試作基板 | PCB、実装、小ロット試作 | 25,000 - 50,000 円 |
| 筐体 | 1U / 2U、パネル、固定部材 | 15,000 - 35,000 円 |
| 熱対策 | ヒートシンク、ファン、温度センサー | 4,000 - 10,000 円 |
| 小物 | ケーブル、ネジ、スペーサ、予備 | 5,000 - 10,000 円 |

上記をすべて最大値で積むと 200,000 円を超える可能性がある。初期調達では Phase ごとに購入対象を分け、いきなり全構成を購入しない方針とする。

## 14. フェーズ計画

### Phase 0: 計画・調査

目的は、構成、予算、検証項目を整理し、購入判断ができる状態にすることである。

成果物:

- 試作計画書
- 概算 BOM
- ブロック図
- リスク一覧
- 検証項目一覧

### Phase 1: Single CM5 + GNSS PoC

目的は、CM5 1 台で GNSS / PPS を受け、Stratum 1 NTP サーバーとして成立するか確認することである。

構成:

- CM5 x1
- CM5 IO Board
- GNSS 評価基板
- GNSS アンテナ
- 安定化電源
- LAN 環境

検証項目:

- GNSS が衛星捕捉できること
- NMEA / UBX を CM5 で読めること
- PPS を GPIO で受けられること
- chrony が PPS を参照できること
- LAN 内クライアントが NTP 同期できること
- offset / jitter が許容範囲に収まること

成功条件:

- CM5 1 台で Stratum 1 NTP サーバーとして動作する
- PPS lock 状態を確認できる
- chrony の状態を記録できる

### Phase 2: Dual CM5 + VRRP PoC

目的は、2 台の CM5 で Active / Standby 構成を作り、VIP による NTP サービス継続性を確認することである。

構成:

- CM5 x2
- CM5 IO Board x2
- GNSS x1
- PPS 分配回路
- Ethernet x2
- keepalived / VRRP

検証項目:

- CM5-A / CM5-B の両方が GNSS / PPS を参照できること
- Active 停止時に Standby が VIP を引き継ぐこと
- NTP クライアントから見た同期断時間を測定すること
- ネットワーク断、OS 停止、chrony 停止を試験すること

成功条件:

- Active 側停止時に Standby 側が自動で NTP 応答を継続する
- フェイルオーバー時間を測定し、記録できる

### Phase 3: Dual Power PoC

目的は、電源 2 系統構成を検証することである。

構成:

- DC IN A
- DC IN B
- Power Module A
- Power Module B
- ORing / Ideal diode
- 電圧 / 電流監視

検証項目:

- 電源 A のみで起動すること
- 電源 B のみで起動すること
- 電源 A を抜いても動作継続すること
- 電源 B を抜いても動作継続すること
- 電源障害を UI またはログで検出できること

成功条件:

- 片系電源断で NTP サービスが停止しない
- 電源断イベントを検出・記録できる

### Phase 4: 試作キャリア基板

目的は、PoC で確認した回路を 1 枚の試作キャリア基板に統合することである。

含める予定の機能:

- Dual CM5 connector
- GNSS interface
- PPS fanout
- NMEA / UBX fanout
- Dual Ethernet
- Dual Power
- Power monitoring
- Supervisor / UI interface
- LED / front panel interface
- ESD / surge protection

成功条件:

- 試作基板上で Phase 1 - 3 の検証を再現できる
- 回路図と PCB データを OSS として公開可能な状態にできる

### Phase 5: 筐体・前面 UI 試作

目的は、ラック搭載可能な試作機として成立させることである。

検討項目:

- 1U / 2U 選定
- 前面表示位置
- RJ45 位置
- GNSS SMA 位置
- 電源コネクタ位置
- 電源抜け止め
- 放熱
- ファン交換性
- メンテナンス性

成功条件:

- 机上試作ではなく、筐体に収まった形で連続稼働できる
- 前面 UI から主要状態を確認できる

## 15. 検証項目

### 15.1 時刻同期

- GNSS lock 時間
- PPS lock 時間
- chrony tracking
- offset
- jitter
- root dispersion
- NTP クライアント同期状態

### 15.2 フェイルオーバー

- Active OS 停止
- chrony 停止
- keepalived 停止
- Ethernet link down
- CM5-A 電源断
- CM5-B 電源断
- VIP 切替時間
- NTP 応答停止時間

### 15.3 電源

- DC IN A 単独動作
- DC IN B 単独動作
- 片系電源断
- 瞬断
- 過電流保護
- 電源温度
- 電源状態の UI 表示

### 15.4 熱

- CM5 温度
- 電源モジュール温度
- GNSS 周辺温度
- 筐体内温度
- ファン停止時の挙動
- 24 時間以上の連続稼働

### 15.5 運用

- 再起動後の自動復旧
- 読み取り専用 OS
- RAM log
- 設定バックアップ
- 障害ログ
- 監視 API

## 16. リスクと対策

| リスク | 内容 | 対策 |
|---|---|---|
| GNSS 選定ミス | 測位向けモジュールを選ぶと時刻用途に最適でない可能性がある | タイミング向け品番を優先し、評価基板で検証する |
| PPS 品質不足 | ノイズや配線により PPS が不安定になる | バッファ、ESD、配線長、波形測定を重視する |
| CM5 発熱 | 1U 筐体内で温度が上がる | ヒートシンク、低速ファン、温度監視を設ける |
| 電源単一障害 | 電源断で全停止する | Dual Power と ORing を設計する |
| GNSS 単一障害 | GNSS またはアンテナ障害で Stratum 1 を失う | 初期は状態検出と Holdover 表示、将来版で Dual GNSS を検討 |
| 基板設計難度 | Dual CM5 + 電源 + GNSS で複雑化する | PoC を分けて段階検証する |
| 予算超過 | 筐体・基板・GNSS で費用が増える | Phase 分割し、購入を段階化する |
| OSS 再現性 | 部品入手性が悪いと再現しづらい | 代替部品候補を BOM に含める |

## 17. 成果物

最終的に以下を GitHub モノレポで管理し、OSS として公開する。

- 要件定義
- 試作計画書
- ブロック図
- 回路図
- PCB データ
- 筐体 CAD
- BOM
- 組立手順
- OS セットアップ手順
- chrony 設定
- keepalived 設定
- 監視スクリプト
- UI ファームウェア
- 検証ログ
- デモ動画
- ライセンス

## 18. 初期購入候補

いきなり完成構成をすべて購入せず、Phase 1 に必要なものから購入する。

### Phase 1 購入候補

- Raspberry Pi Compute Module 5 Wi-Fi なし / 4GB RAM / 32GB eMMC x1
- CM5 IO Board x1
- タイミング向け GNSS 評価基板 x1
- GNSS アンテナ x1
- 安定化電源 x1
- GPIO 配線部材
- LAN ケーブル
- 必要に応じて USB-UART 変換器

### Phase 2 購入候補

- Raspberry Pi Compute Module 5 Wi-Fi なし / 4GB RAM / 32GB eMMC x1 追加
- CM5 IO Board x1 追加
- PPS 分配検証用部品
- Ethernet 検証部品

### Phase 3 購入候補

- DC/DC 電源モジュール x2
- ORing / Ideal diode 検証部品
- 電源監視 IC または評価基板
- 電源コネクタ候補

## 19. マイルストーン

| Milestone | 内容 | 成功条件 |
|---|---|---|
| M0 | 計画書作成 | 試作方針と予算を説明できる |
| M1 | Single CM5 Stratum 1 | CM5 1 台で GNSS/PPS NTP が動作する |
| M2 | Dual CM5 HA | VIP フェイルオーバーが動作する |
| M3 | Dual Power | 片系電源断でも継続動作する |
| M4 | 試作基板 | 主要回路を 1 枚の基板に統合する |
| M5 | 筐体試作 | ラック搭載可能な試作機として動作する |
| M6 | OSS 公開 | 回路・ソフト・手順を公開する |

## 20. 現時点の判断

現時点では、以下の方針で進めるのが妥当である。

- いきなり 3D CAD や筐体設計に入らない
- まずは GNSS / PPS / chrony の成立を確認する
- 次に Dual CM5 / VRRP を検証する
- その後に Dual Power を検証する
- PoC の結果をもとに試作キャリア基板を設計する
- 筐体設計は、基板外形とコネクタ位置が見えてから着手する

OpenTime Pro Prototype は、単なる Raspberry Pi NTP サーバーではなく、OSS として再現可能なラックマウント時刻アプライアンスを目指す。そのため、性能よりも信頼性、電源、時刻源、監視、保守性を重視する。
