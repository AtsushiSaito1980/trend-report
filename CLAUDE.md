# Trend Report System — システム憲法 v3（スカウト＆アナリスト方式）

## 基本ルール
- すべての出力は日本語
- シンプルさを最優先。複雑にしない
- 作業完了後は git commit（2回のみ）→ PR作成

## アーキテクチャ：スカウト＆アナリスト方式
**Web検索はリーダーが1回のみ実施**。リサーチャーはスカウト報告を読み「分析」に専念する。
これによりトークン消費を90%削減しつつ、3視点の並列分析で多角的なレポートを生成する。

```
[リーダー＝スカウト]
  1. topics.json からランダムに3トピック選択
  2. 各トピックでWebSearch（上位3件のみ）→ scout_report.md に要約
  3. commit #1: [Scout] リサーチ完了

         ↓ 並列起動
[Tech] [Human] [Critic]
  - scout_report.md を読む（Web検索は禁止）
  - 専門視点で分析し workspace/outputs/{role}_analysis.md に保存
  - チャット報告: "[Role] Done." のみ

         ↓
[リーダー] → 3つの分析ファイルを統合 → 日報ドラフト
[テスター] → 品質チェック → OK
[リーダー] → reports/daily/YYYY-MM-DD_HHMMSS/report.md に保存
  → commit #2: [Leader] レポート完成
  → push → gh pr create → main
```

## ディレクトリ構成
```
trend-report/
├── CLAUDE.md
├── topics.json
├── agents/
│   ├── leader.md           # スカウト＆ディレクター
│   ├── researcher_tech.md  # 分析専念（検索禁止）
│   ├── researcher_human.md # 分析専念（検索禁止）
│   ├── researcher_critic.md# 分析専念（検索禁止）
│   ├── tester.md           # 品質チェック
│   └── developer.md        # アプリ改善
├── workspace/
│   ├── outputs/
│   │   ├── scout_report.md      ← リーダーが書く（唯一の検索結果）
│   │   ├── tech_analysis.md     ← Tech分析
│   │   ├── human_analysis.md    ← Human分析
│   │   └── critic_analysis.md   ← Critic分析
│   ├── status.json
│   └── weekly_status.json
├── reports/
│   ├── daily/YYYY-MM-DD_HHMMSS/
│   └── weekly/YYYY-MM-DD_HHMMSS/
└── feedback/
    └── issues.md
```

## スカウト報告フォーマット（workspace/outputs/scout_report.md）
```markdown
# スカウト報告
実行日時: YYYY-MM-DD HH:MM
選択トピック: [トピックA, トピックB, トピックC]

## {トピックA}
1. {記事タイトル} — {1行要約}（URL）
2. {記事タイトル} — {1行要約}（URL）
3. {記事タイトル} — {1行要約}（URL）

## {トピックB}
...

## {トピックC}
...
```

## 分析ファイルフォーマット（workspace/outputs/{role}_analysis.md）
```markdown
# {Role}視点 分析
分析日時: YYYY-MM-DD HH:MM

## {トピックA}
- **{観点}**: ...

## {トピックB}
...
```

## Gitルール（2コミットのみ）
| タイミング | コマンド |
|---|---|
| commit #1（リサーチ完了後） | `git commit -m "[Scout] リサーチ完了 {RUN_ID}"` |
| commit #2（最終レポート完了後） | `git commit -m "[Leader] レポート完成 {RUN_ID}"` |

## 保存ルール（重複回避）
- 日報: `reports/daily/YYYY-MM-DD_HHMMSS/report.md`
- 週報: `reports/weekly/YYYY-MM-DD_HHMMSS/report.md`
- 時分秒付きフォルダにより、1日複数回実行しても衝突しない

## 自動PRルール
- ブランチ: `report-YYYYMMDD-HHMMSS`
- PRタイトル: `📊 日報 YYYY-MM-DD HH:MM`
- PR本文: 選択トピック・各視点のキーポイント
- ベースブランチ: `main`

## 状態管理（workspace/status.json）
```json
{
  "date": "YYYY-MM-DD",
  "run_id": "YYYYMMDD-HHMMSS",
  "branch": "report-YYYYMMDD-HHMMSS",
  "selected_topics": [],
  "phase": "idle|scouting|analyzing|integrating|done",
  "scout_done": false,
  "tech_done": false,
  "human_done": false,
  "critic_done": false,
  "test_passed": false
}
```

## 週報フロー
1. `report-weekly-{RUN_ID}` ブランチを作成
2. 直近7日の日報を1日ずつ読み、`workspace/outputs/weekly_intermediate.md` に3行要約を追記
3. 全日分揃ったら週報に統合 → `reports/weekly/YYYY-MM-DD_HHMMSS/report.md`
4. 1回のcommitでpush → PR作成

## フィードバック反映
実行開始時に必ず `feedback/issues.md` を読み、内容を指示に反映する。
