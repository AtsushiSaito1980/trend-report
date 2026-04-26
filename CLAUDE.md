# Trend Report System — システム憲法

## 基本ルール
- すべての出力は日本語
- シンプルさを最優先。複雑にしない
- 作業完了後は必ずgit commit & push → PR作成

## アーキテクチャ：ブラックボード方式
各エージェントは直接通信せず、共通黒板（`workspace/`）を介して情報を共有する。
これによりトークン消費を最小化し、分散処理を実現する。

```
[リーダー] → 指示
    ↓  ↓  ↓  （並列）
[Tech] [Human] [Critic] → workspace/outputs/ に要約Markdownを書く
                                    ↓
[リーダー] ←── workspace/outputs/ から読み込み統合
    ↓
[テスター] → 品質チェック
    ↓
[リーダー] → reports/ に保存 → git commit & push → PR作成
```

## ディレクトリ構成
```
trend-report/
├── CLAUDE.md                          # システム憲法（このファイル）
├── topics.json                        # 監視トピック一覧
├── agents/
│   ├── leader.md                      # オーケストレーター
│   ├── researcher_tech.md             # テック・理系視点
│   ├── researcher_human.md            # 文系・バイタリティ視点
│   ├── researcher_critic.md           # 否定的・リスク視点
│   ├── tester.md                      # 品質チェック
│   └── developer.md                   # アプリ改善
├── workspace/
│   ├── outputs/                       # 黒板：各エージェントの中間報告
│   ├── status.json                    # 日次進捗管理
│   └── weekly_status.json             # 週次進捗管理
├── reports/
│   ├── daily/YYYY-MM-DD_HHMMSS/      # 日報（時分秒付きフォルダ）
│   └── weekly/YYYY-MM-DD_HHMMSS/    # 週報（時分秒付きフォルダ）
└── feedback/
    └── issues.md                      # フィードバック履歴
```

## エージェント構成
リーダーが以下を指揮する：
- **リサーチャーTech**（テック・理系視点）
- **リサーチャーHuman**（文系・バイタリティ視点）
- **リサーチャーCritic**（否定的・リスク視点）
- **テスター**（品質チェック）
- **開発者**（アプリ改善）

## ブラックボードの使い方

### 中間報告の保存場所
各リサーチャーは以下のファイルに**要約Markdownのみ**を書く（Web検索の生データは捨てる）：
- `workspace/outputs/tech_summary.md`    … Tech視点
- `workspace/outputs/human_summary.md`   … Human視点
- `workspace/outputs/critic_summary.md`  … Critic視点
- `workspace/outputs/weekly_intermediate.md` … 週次中間要約（1日分ずつ追記）

### 状態管理ファイルの構造
`workspace/status.json`（日次）:
```json
{
  "date": "YYYY-MM-DD",
  "run_id": "YYYYMMDD-HHMMSS",
  "branch": "report-YYYYMMDD-HHMMSS",
  "phase": "start|researching|integration|testing|done",
  "tech_done": false,
  "human_done": false,
  "critic_done": false,
  "integration_done": false,
  "test_passed": false
}
```

`workspace/weekly_status.json`（週次）:
```json
{
  "week_start": "YYYY-MM-DD",
  "run_id": "YYYYMMDD-HHMMSS",
  "branch": "report-weekly-YYYYMMDD-HHMMSS",
  "days_processed": [],
  "integration_done": false,
  "test_passed": false
}
```

## 保存ルール（重複回避）
- 日報: `reports/daily/YYYY-MM-DD_HHMMSS/report.md`
- 週報: `reports/weekly/YYYY-MM-DD_HHMMSS/report.md`
- 時分秒付きフォルダにより、1日に何度テスト実行しても衝突しない

## 自動PRフロー
1. **実行開始時**: `report-YYYYMMDD-HHMMSS`（日報）または `report-weekly-YYYYMMDD-HHMMSS`（週報）ブランチを作成
2. **各エージェント完了時**: `[エージェント名] 作業完了` メッセージでコミット＆プッシュ
3. **全工程完了後**: `main` ブランチへのPRを自動作成
   - 日報PRタイトル: `📊 日報 YYYY-MM-DD HH:MM`
   - 週報PRタイトル: `📊 週報 YYYY-MM-DD HH:MM`
   - PR本文: レポートの概要（トピック一覧・各視点のキーポイント）

## 情報収集のルール
- 各トピックについて必ずWeb検索を行う
- 検索は「トピック名 最新情報 今月」で実施
- 検索結果から信頼できる記事を3件以上参照する
- 情報源のURLも記録する
- **黒板に書く内容は要約のみ**（元のWeb検索結果は含めない）

## 日報フロー
1. リーダーが `workspace/status.json` を初期化し、ブランチ作成
2. Tech / Human / Critic が**並列**でリサーチし、各要約を `workspace/outputs/` に書く
3. 各エージェントは完了時にコミット
4. リーダーが3つの要約を読み込み、日報ドラフトを統合
5. テスターが品質チェック（NG → リーダーへ差し戻し）
6. `reports/daily/YYYY-MM-DD_HHMMSS/report.md` に保存してコミット
7. プッシュ → PR作成

## 週報フロー
1. リーダーが `workspace/weekly_status.json` を初期化し、ブランチ作成
2. 直近7日分の日報を1日ずつ読み込み、要約を `workspace/outputs/weekly_intermediate.md` に追記
3. 各日の追記完了時にコミット
4. 7日分が揃ったら週報に統合
5. `reports/weekly/YYYY-MM-DD_HHMMSS/report.md` に保存
6. プッシュ → PR作成

## フィードバック反映
- 実行開始時に必ず `feedback/issues.md` を読む
- フィードバックの内容を各エージェントへの指示に反映する
