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
  0. reports/index.json の直近7日エントリを読み、使用済みトピックをリストアップ
  1. topics.json（約100件）から直近7日と被らない候補リストを作成
  2. 候補リストからランダムに3トピックを選定
  3. 各トピックでWebSearch（上位3件のみ）→ scout_report.md に要約
  4. commit #1: [Scout] リサーチ完了

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
│   ├── index.json               ← 図書目録（全レポートの索引）
│   ├── daily/YYYY-MM-DD_HHMMSS/
│   └── weekly/YYYY-MM-DD_HHMMSS/
└── feedback/
    └── issues.md
```

## トピックランダム選定（直近7日と重複しないトピックを選ぶ）
- リーダーが `reports/index.json` の直近7日のエントリから `topics` フィールドを抽出し「除外リスト」を作成
- `topics.json` の全候補（約100件）から除外リストを引いた「候補リスト」を作成
- 候補リストからランダムに **3件** を選定する

## スカウト報告フォーマット（workspace/outputs/scout_report.md）
```markdown
# スカウト報告
実行日時: YYYY-MM-DD HH:MM
選択トピック: [トピックA, トピックB, トピックC]

## {トピックA}
1. [{記事タイトル}]({URL}) — {1行要約}
2. [{記事タイトル}]({URL}) — {1行要約}
3. [{記事タイトル}]({URL}) — {1行要約}

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

## 図書目録（reports/index.json）
レポート生成の最後に必ず `reports/index.json` を更新する。
- 最新エントリを**先頭に追加**（Unshift）する配列形式
- 日報・週報の両方を追記対象とする

```json
[
  {
    "date": "YYYY-MM-DD",
    "time": "HH:MM:SS",
    "type": "daily",
    "topics": ["トピックA", "トピックB", "トピックC"],
    "path": "reports/daily/YYYY-MM-DD_HHMMSS/report.md"
  }
]
```

## レポートスタイルガイド
すべてのレポート（日報・週報・分析ファイル）に適用する。

### 強調
- 重要な結論・数字は `**太字**` で強調する
- 特に重要な一文は `<mark>蛍光ペン効果</mark>` でマークする

### 絵文字の配置
| 用途 | 絵文字 |
|---|---|
| セクション見出し | 🚀 新技術  📊 データ  💡 洞察  ⚠️ リスク  🌍 社会影響 |
| 状態・評価 | ✅ ポジティブ  ❌ ネガティブ  🔍 要注目  💰 ビジネス機会 |
| 週報・サマリー | 📈 成長  📉 下落  🎯 重点  🗓️ 週次まとめ |

### Mermaidによる図解（**必須**）
各分析ファイルにはトピックごとに **必ず1つ以上** `mermaid` コードブロックを含める：

| 分析ファイル | 図の種類 | 用途 |
|---|---|---|
| tech_analysis.md | `graph LR` | 技術関係・依存図 |
| human_analysis.md | `mindmap` | ステークホルダーマップ |
| critic_analysis.md | `graph TD` | リスク連鎖フロー |

````markdown
```mermaid
graph LR
  A["技術A"] --> B["応用領域"]
  B --> C["市場インパクト"]
```
````

### Markdownテーブル（**必須**）
各分析ファイルにはトピックごとに **必ず1つ以上** テーブルを含める：

````markdown
| 項目 | 値 | 備考 |
|------|-----|------|
| データA | XX | ... |
````

### エグゼクティブ・サマリー形式
レポートは「結論から書く」構造を徹底する：
1. **📋 エグゼクティブ・サマリー**（全体を3〜5行で）
2. **本文**（各視点の詳細）
3. **💡 総合所感**（アクション提言）

## フィードバック反映
実行開始時に必ず `feedback/issues.md` を読み、内容を指示に反映する。
