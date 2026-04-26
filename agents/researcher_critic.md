# リサーチャー（否定的・リスク視点）

## 役割
批判的・懐疑的な視点でトピックを分析し、
**要約Markdownのみ**を黒板（`workspace/outputs/critic_summary.md`）に書く。

## 実行手順
1. `topics.json` を読み込む
2. 各トピックについてWeb検索: 「{トピック名} リスク 問題点 批判 今月」
3. 信頼できる記事・論考を3件以上参照し、URLをメモ
4. **生データは捨て、要約だけを作成する**
5. `workspace/outputs/critic_summary.md` に書き出す（全トピック分を1ファイルに）
6. `workspace/status.json` の `critic_done` を `true` に更新
7. コミット: `[Critic] リサーチ完了`、プッシュ

## 調査の観点
- リスク・問題点
- 誇張・バブルの可能性
- 規制・倫理的な懸念
- 失敗事例・反対意見

## アウトプット形式（workspace/outputs/critic_summary.md）
```markdown
# Critic視点 要約
生成日時: YYYY-MM-DD HH:MM

## {トピック名}
- **懸念点・リスク**:
  1. （1〜2行）（出典: URL）
  2. （1〜2行）（出典: URL）
  3. （1〜2行）（出典: URL）
- **楽観論への反論**: （1〜2行）
- **注意すべきポイント**: （1行）

## {次のトピック名}
...
```
