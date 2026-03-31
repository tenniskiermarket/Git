# メタ認知日記アプリ実装計画

## Context

ユーザーが日々のメタ認知を記録するためのスマートフォン向け日記アプリを作る。既存プロジェクト（index.html / memo.html）と同じ技術スタック（バニラHTML/CSS/JS）で新規ファイル `diary.html` を作成。週次分析機能で「繰り返す壁」を可視化し、自己改善のサイクルを支援する。

---

## 作成対象

- **新規ファイル**: `c:\Git\diary.html`（1ファイル完結）
- **参照元**: `memo.html`（LocalStorage/CSS設計パターン）、`index.html`（デザイントークン）

---

## データ構造（LocalStorage）

```js
// キー: 'diary_v1'
{
  "2026-03-31": {
    goal:      string,  // 本日の目標
    done:      string,  // 実際にやった事・得られた成果
    obstacles: string,  // ぶつかった壁・詰まったポイント ★週次分析対象
    tomorrow:  string,  // 明日変える事
    updatedAt: number   // Date.now()
  }
}
```

---

## 画面構成

### ヘッダー（固定上部）
```
[ ← ]  2026年3月31日（火）  [ → ]  [今日]
```

### タブバー（固定下部）
```
[ ✏ 日記 ]  [ 📊 週次分析 ]
```

### タブ1: 日記入力
- 状態バッジ（未記入 / 記入済）
- 4つのテキストエリア（自動リサイズ）
  - 🎯 本日の目標
  - ✅ 実際にやった事・得られた成果
  - 🧱 ぶつかった壁・詰まったポイント
  - 🔄 明日変える事
- 700ms デバウンス自動保存 + 保存ボタン

### タブ2: 週次分析
- 期間選択: [7日] [14日] [21日] [28日]
- 統計サマリー: 記録日数・達成率・連続記録日数
- obstacles 時系列一覧（日付 + テキスト全文）
- キーワード頻度チップ（繰り返しパターン可視化）

---

## デザイントークン

```css
:root {
  --bg: #020208;
  --surface: #0a0a18;
  --accent: #818cf8;         /* 紫系（index.htmlに統一） */
  --accent-glow: #6366f1;
  --cyan: #22d3ee;
  --text: #f1f5f9;
  --muted: #94a3b8;
  --border: rgba(0,255,200,.2);
  --radius: 12px;
}
```

---

## 週次分析アルゴリズム

1. **データ収集**: 過去N日間の `obstacles` フィールドを時系列で収集
2. **キーワード抽出**: 助詞・助動詞・形式名詞などのストップワードを除去、2〜15文字の語を抽出
3. **頻度集計**: 出現回数でソートし上位20件を表示
4. **統計計算**: 記録日数/対象日数で達成率、連続記録日数をカウント

---

## JS関数構成（即時関数でカプセル化）

| カテゴリ | 関数 |
|---|---|
| Storage | `loadData()`, `saveData()`, `getEntry(key)` |
| 日付ユーティリティ | `toDateKey(d)`, `todayKey()`, `offsetDate(key,delta)`, `formatDateLabel(key)`, `escHtml(s)` |
| 日記画面 | `renderDiaryView()`, `updateHeader()`, `scheduleAutosave()`, `saveCurrentEntry()`, `showSaveIndicator()` |
| 週次分析 | `renderAnalysisView()`, `collectObstaclesData(days)`, `buildWordFrequency(entries)`, `extractKeywords(text)`, `calcStreak()` |
| イベント | `initEventListeners()`, `switchTab(tab)`, `navigateDate(delta)`, `goToToday()`, `init()` |

---

## モバイル対応

- `max-width: 600px` + 中央寄せ
- 全 `textarea` に `font-size: 16px`（iOSズーム防止）
- タップターゲット最小44×44px
- `env(safe-area-inset-bottom)` でiPhoneホームバー対応
- `body { padding-bottom: 72px }` でタブバーと被らない余白

---

## 実装順序

1. HTML骨格（ヘッダー・タブバー・2ビュー）
2. CSS（トークン定義→レイアウト→コンポーネント→アニメーション）
3. Storage層（loadData/saveData/getEntry）
4. 日付ユーティリティ
5. 日記入力画面（renderDiaryView→日付ナビ→自動保存）
6. 週次分析画面（データ収集→キーワード分析→描画）
7. イベント統合
8. スマホ調整・動作確認

---

## 検証方法

1. ブラウザで `diary.html` を開き日記入力 → 保存確認
2. DevTools > Application > LocalStorage で `diary_v1` キーのデータを確認
3. 前日/翌日ナビゲーションで正しく日付が切り替わるか確認
4. 7日分のデータを手動で入れて週次分析タブを確認
5. スマートフォン実機またはDevToolsのモバイルエミュレーションでレイアウト確認
