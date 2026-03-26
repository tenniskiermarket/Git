# memo.html — Webメモ帳アプリ 実装計画

## Context

ユーザーからのリクエストに基づき、新規ファイル `memo.html` としてWebメモ帳アプリを作成する。
既存の `index.html`（サービス紹介LP）はそのまま残し、ダークテーマのデザインシステムを踏襲する。

**必要機能：** メモのCRUD / リアルタイム検索 / タグ・カテゴリ / 写真添付ライフログ
**データ保存：** localStorage（サーバー不要）
**技術スタック：** Vanilla HTML/CSS/JS のシングルファイル（外部ライブラリなし）

---

## 対象ファイル

| ファイル | 操作 |
|---|---|
| `c:\Git\memo.html` | **新規作成**（本計画の全実装先） |
| `c:\Git\index.html` | 参照のみ（CSSデザインシステムを踏襲） |

---

## デザインシステム（index.htmlから踏襲）

```css
:root {
  --bg:          #06060f;
  --bg-alt:      #0d0d1a;
  --surface:     #131326;
  --border:      rgba(99, 102, 241, .25);
  --accent:      #818cf8;
  --accent-glow: #6366f1;
  --cyan:        #22d3ee;
  --text:        #f1f5f9;
  --muted:       #94a3b8;
  --radius:      14px;
  /* 追加 */
  --sidebar-w:   300px;
  --header-h:    56px;
  --danger:      #f87171;
}
```

---

## UIレイアウト

```
┌─────────────────────────────────────────────────────┐
│  Header: [Memo ロゴ]                  [+ 新規メモ]  │
├──────────────────┬──────────────────────────────────┤
│  Sidebar(300px)  │  Editor Panel                    │
│  ─────────────   │  ─────────────────────────────   │
│  [🔍 検索バー]   │  [タイトル入力]                  │
│                  │  [日付]              [削除ボタン]  │
│  タグフィルター  │  [タグチップ] [タグ入力欄]       │
│  #tag1 #tag2     │  [📷 写真エリア（アップ/プレビュ│
│                  │  [本文テキストエリア]             │
│  メモ一覧        │                    [保存ボタン]  │
│  ┌──────────┐    │                                  │
│  │ タイトル  │    │                                  │
│  │ プレビュー│    │                                  │
│  └──────────┘    │                                  │
└──────────────────┴──────────────────────────────────┘
```

---

## データ構造（localStorage）

```json
{
  "id": "uuid-v4",
  "title": "タイトル",
  "content": "本文テキスト",
  "tags": ["tag1", "tag2"],
  "photo": "data:image/png;base64,...（またはnull）",
  "createdAt": "2026-03-26T00:00:00.000Z",
  "updatedAt": "2026-03-26T00:00:00.000Z"
}
```

**Storage Key：** `memo_app_v1`

---

## JavaScriptモジュール構成（IIFE内）

```
[A] 定数・設定          STORAGE_KEY, AUTOSAVE_DELAY(800ms), MAX_PHOTO_SIZE(2MB)
[B] State管理          memos[], selectedId, searchQuery, activeTag, pendingDeleteId
[C] localStorage I/O   loadMemos() / saveMemos() ※QuotaExceededError を try-catch
[D] UUID生成           crypto.randomUUID() フォールバック付き
[E] フィルタリング     getFilteredMemos() / getAllTags()
[F] レンダリング関数   renderSidebar / renderMemoList / renderTagFilter
                       renderEditor / renderEditorTags / renderPhotoPreview
[G] アクション関数     createMemo / saveMemo / deleteMemo / selectMemo
                       addTag / removeTag / attachPhoto / removePhoto
[H] イベント登録       全DOMイベントをここで一括登録
[I] 初期化             loadMemos() → renderSidebar() → renderEditor()
```

---

## 主要機能の実装方針

### CRUD
- **新規作成：** 検索・タグフィルターをリセット → uuid生成 → state先頭に追加 → エディタを開く
- **自動保存：** title/content の `input` イベントに 800ms デバウンス
- **削除：** 削除確認モーダルを経由（誤操作防止）
- **メモ切り替え時：** autosaveタイマーをキャンセルして即時保存してから切り替え

### 検索
- title + content を小文字化して部分一致
- 表示フィルタリングのみ（ストレージ変更なし）

### タグ
- Enter / `,` キーで確定、重複チェックあり
- Backspace（空欄時）で直前タグ削除
- サイドバーのタグチップクリックでフィルター ON/OFF トグル

### 写真添付
- `<input type="file" accept="image/*">` → FileReader API → base64 で memo.photo に格納
- 2MB 超過はアラートで弾く
- `onerror` で読み込み失敗をハンドリング

### レンダリング最適化
- 全体再描画を避け、変更範囲に応じた個別 render 関数を呼び出す
- 自動保存時は `renderMemoListItem(id)` でリストの該当行だけ更新

---

## エッジケース対処

| ケース | 対処 |
|---|---|
| localStorage 容量超過 | `QuotaExceededError` を catch してアラート |
| メモ0件 | 「メモがありません」の空状態UI表示 |
| エディタ未選択 | 「メモを選択または新規作成してください」プレースホルダー |
| ページリロード後 | loadMemos() で復元、selectedId は最新メモを選択 |
| 画像ファイル以外 | `file.type.startsWith('image/')` でチェック |

---

## 実装順序

1. **骨格（HTML/CSS）** — 2カラムレイアウト、共通コンポーネントCSS
2. **データ層** — localStorage I/O、State、UUID、初期描画
3. **CRUD** — 新規作成、自動保存、削除モーダル
4. **検索・タグ** — リアルタイム検索、タグ追加/削除/フィルター
5. **写真添付** — FileReader、プレビュー、容量チェック
6. **仕上げ** — 空状態UI、モバイル対応、保存インジケーター

---

## 検証方法

1. ブラウザで `memo.html` を直接開く
2. 新規メモを作成し、タイトル・本文を入力 → 自動保存 → リロード後も残ることを確認
3. 写真を添付してリロード後も表示されることを確認
4. タグを追加 → サイドバーのタグフィルターに反映されることを確認
5. 検索バーに文字入力 → 該当メモだけが絞り込まれることを確認
6. 削除ボタン → モーダル → 確認後に一覧から消えることを確認
7. DevTools の Application → Local Storage でデータ構造を確認
