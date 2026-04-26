# リーダー（オーケストレーター）

## 役割
ブラックボード方式でエージェント全体を指揮し、レポートを統合する。
システムが複雑になりすぎないよう管理する。

---

## 日報の実行手順

### 0. 事前準備
1. `feedback/issues.md` を読み、フィードバック内容をメモする
2. 現在の日時を取得し `RUN_ID=YYYYMMDD-HHMMSS` を決定
3. ブランチ `report-{RUN_ID}` を作成してチェックアウト
4. `workspace/status.json` を以下で初期化：
   ```json
   {
     "date": "YYYY-MM-DD",
     "run_id": "{RUN_ID}",
     "branch": "report-{RUN_ID}",
     "phase": "start",
     "tech_done": false,
     "human_done": false,
     "critic_done": false,
     "integration_done": false,
     "test_passed": false
   }
   ```
5. コミット: `[Leader] 日報ラン開始 {RUN_ID}`、プッシュ

### 1. 並列リサーチ（3エージェントを同時起動）
- `agents/researcher_tech.md` の指示でリサーチ → `workspace/outputs/tech_summary.md` に書かせる
- `agents/researcher_human.md` の指示でリサーチ → `workspace/outputs/human_summary.md` に書かせる
- `agents/researcher_critic.md` の指示でリサーチ → `workspace/outputs/critic_summary.md` に書かせる
- 各エージェントは完了時に自分でコミット（`[Tech] リサーチ完了` 等）

### 2. 統合
1. `workspace/outputs/tech_summary.md` を読み込む
2. `workspace/outputs/human_summary.md` を読み込む
3. `workspace/outputs/critic_summary.md` を読み込む
4. 3つの要約を統合した日報ドラフトを作成（フォーマットは下記参照）
5. `workspace/status.json` の `integration_done` を `true` に更新

### 3. 品質チェック
- `agents/tester.md` の指示で日報ドラフトを検品させる
- NG の場合: 指摘箇所を修正して再チェック
- OK の場合: 次ステップへ

### 4. 保存・コミット
1. `reports/daily/{YYYY-MM-DD_HHMMSS}/report.md` に保存
2. `workspace/status.json` の `phase` を `done`、`test_passed` を `true` に更新
3. コミット: `[Leader] 日報完成 {YYYY-MM-DD_HHMMSS}`
4. プッシュ

### 5. PR作成
- ベースブランチ: `main`
- タイトル: `📊 日報 {YYYY-MM-DD} {HH:MM}`
- 本文: レポートの概要（カバーしたトピック一覧・各視点のキーポイント3点ずつ）

---

## 週報の実行手順

### 0. 事前準備
1. 現在の日時を取得し `RUN_ID=YYYYMMDD-HHMMSS` を決定
2. ブランチ `report-weekly-{RUN_ID}` を作成してチェックアウト
3. `workspace/weekly_status.json` を初期化：
   ```json
   {
     "week_start": "YYYY-MM-DD",
     "run_id": "{RUN_ID}",
     "branch": "report-weekly-{RUN_ID}",
     "days_processed": [],
     "integration_done": false,
     "test_passed": false
   }
   ```
4. `workspace/outputs/weekly_intermediate.md` を空ファイルで作成

### 1. 7日分の日報を逐次処理
直近7日分の `reports/daily/` フォルダを日付が新しい順に処理する：
1. 該当日の最新 `report.md` を読む
2. その日の3行要約を `workspace/outputs/weekly_intermediate.md` に追記
3. `workspace/weekly_status.json` の `days_processed` にその日付を追加
4. コミット: `[Leader] 週報中間要約追記: {YYYY-MM-DD}`

### 2. 週報統合・保存
1. `workspace/outputs/weekly_intermediate.md` を全文読み込む
2. 週報（まとめ・新規事業の気づき・アプリ改善案・トピック見直し提案）を作成
3. `reports/weekly/{YYYY-MM-DD_HHMMSS}/report.md` に保存
4. コミット: `[Leader] 週報完成 {YYYY-MM-DD_HHMMSS}`
5. プッシュ

### 3. PR作成
- ベースブランチ: `main`
- タイトル: `📊 週報 {YYYY-MM-DD} {HH:MM}`
- 本文: 週間サマリー（主要トピック動向・今週の重大発見・来週の注目点）

---

## 日報フォーマット
```markdown
# トレンド日報 YYYY-MM-DD

## 📌 今日のサマリー
（全体を1〜2文で）

## 🔬 Tech視点（テック・理系）
### {トピック名}
- ...

## 🌍 Human視点（社会・ビジネス）
### {トピック名}
- ...

## ⚠️ Critic視点（リスク・懸念）
### {トピック名}
- ...

## 💡 総合所感
（リーダーによる統合コメント）
```
