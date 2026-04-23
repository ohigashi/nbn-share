# nbn-share

`ohgs.link` で公開する、north by north のコンテンツ共有リポジトリ。
イベント参加者や関係者向けに、インタラクティブマップ、資料、ランディングページなどを配信する。

---

## このリポジトリが担う役割

- **独自ドメイン下のコンテンツ置き場**：`northbynorth.jp`（将来的にGhostで本体サイト運用）とは別軸の、**軽量な静的コンテンツ配信**を担う。
- **イベント単位のサブディレクトリ運用**：各イベント・プロジェクトごとに独立したディレクトリを持ち、URLがそのまま意味を持つ設計。
- **参加者配布用・限定共有用**：検索インデックスには載せない想定（`noindex,nofollow` をHTMLに付ける）。URL直リンクで届ける。

## ドメインと思想

- ドメイン：`ohgs.link`（大東洋克の個人ブランド用短縮ドメイン）
- 配信基盤：**Cloudflare Pages**（GitHub連携による自動デプロイ）
- 配置原則：**Owned infrastructure**。プラットフォーム依存を避け、自前で運用する。

---

## ディレクトリ構造

```
nbn-share/
├─ CLAUDE.md                   ← このファイル
├─ nbn/                        ← north by north 関連
│  └─ caravan-1/               ← 第1回 道北アクティビティキャラバン
│     └─ map/
│        └─ index.html         ← インタラクティブマップ
└─ （今後追加していく）
```

### URLとディレクトリの対応

| URL | ファイルパス |
|---|---|
| `https://ohgs.link/nbn/caravan-1/map/` | `nbn/caravan-1/map/index.html` |

### 新規コンテンツ追加時の命名ルール

- **イベントシリーズ**：`nbn/`, `voyage/`, `largo/` など、ブランド/プロジェクト単位
- **イベント回**：`caravan-1`, `caravan-2`, `vol-3` のように連番
- **コンテンツ種別**：`map/`, `guide/`, `photos/`, `lp/` のように用途

例：
- `nbn/caravan-2/map/index.html` → `ohgs.link/nbn/caravan-2/map/`
- `voyage/vol-9/recap/index.html` → `ohgs.link/voyage/vol-9/recap/`

---

## デプロイフロー

GitHub → Cloudflare Pages の連携により、`main` ブランチへの push が自動デプロイのトリガーになる。

```bash
# 編集したら
cd ~/projects/nbn-share
git add .
git commit -m "何を変えたか"
git push

# → 1〜2分で ohgs.link に反映
```

手動操作は不要。Cloudflareダッシュボードを触る必要もない。

---

## インフラ構成

| 項目 | 内容 |
|---|---|
| ドメインレジストラ | Cloudflare Registrar（`ohgs.link`） |
| DNS | Cloudflare（自動管理） |
| 配信 | Cloudflare Pages（プロジェクト名：`nbn-share`） |
| ソース | GitHub（`ohigashi/nbn-share`、Public） |
| SSL | Cloudflare自動発行（Let's Encrypt相当） |
| ビルド | なし（静的HTMLをそのまま配信） |

### Cloudflare Pages プロジェクト設定

- **Production branch**: `main`
- **Build command**: （空）
- **Build output directory**: （空、ルートそのまま配信）

---

## 新規コンテンツを追加する手順（標準ワークフロー）

### 1. ディレクトリとHTMLを作成

```bash
cd ~/projects/nbn-share
mkdir -p nbn/caravan-2/map
# index.html を配置
```

### 2. ローカルで動作確認

```bash
# Python3標準のシンプルサーバ
python3 -m http.server 8000
# → http://localhost:8000/nbn/caravan-2/map/ で確認
```

### 3. コミット＆プッシュ

```bash
git add .
git commit -m "add caravan-2 map"
git push
```

### 4. デプロイ確認

- Cloudflareダッシュボードで自動デプロイを確認（またはpush後1〜2分待つ）
- 本番URL：`https://ohgs.link/nbn/caravan-2/map/`

---

## HTML設計の共通ルール

このリポジトリに置くHTMLは、以下の方針で統一する。

### デザイン

north by north ブランドの配色・タイポグラフィに揃える。参照：`nbn/caravan-1/map/index.html` の `:root` CSS変数セクション。

```css
--cream:       #f5efe4;
--forest:      #2d4a36;
--gold:        #c8a15a;
--copper:      #8b5a3c;
/* … */
```

- **フォント**：日本語は Shippori Mincho B1（見出し）と Noto Sans JP（本文）、英語は DM Serif Display（見出しイタリック）と IBM Plex Mono（ラベル）
- **トーン**：editorial、落ち着いた、フィールドジャーナル的。観光マーケティング的な派手さを避ける

### メタタグ必須項目

```html
<meta name="robots" content="noindex,nofollow" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

### 外部依存

- CDN利用は最小限に（Leaflet、Google Fonts程度までOK）
- 可能なら単一HTMLファイルで完結させる（CSS・JSをインライン）

### SNSポリシー

north by north関連コンテンツでは、フッターに以下の方針を明記する：

> 写真や感想のSNSシェアは大歓迎です。ただし、行程表や具体的なアクティビティの場所の公開はお控えください。地元の方々とガイドが守り育ててきた繊細なフィールドを大切にしながら、道北のアクティビティ文化を丁寧に育てていくためのお願いです。

---

## Claude Code への依頼時のガイド

### やってほしいこと

- 新規コンテンツのHTMLファイル作成（既存デザインのトーンを踏襲）
- マップ、ランディングページ、資料ページなどのUI構築
- 既存コンテンツの修正・リファクタ
- git の commit & push まで一気通貫で進める

### 注意してほしいこと

- **デザインは既存のnbn/caravan-1/map/index.htmlを参照してトーンを揃える**
- **外部CDN依存は最小限に**。Leaflet、Google Fontsは可。独自APIサーバ前提の実装は避ける
- **コミットメッセージは日本語で簡潔に**（例：「add: caravan-2 mapの初版」「fix: 詳細パネルの閉じるボタン」）
- **push前にローカルサーバで動作確認**を提案する
- **`CLAUDE.md` 自体の更新も気づいたら提案する**（運用ルール変更時など）

### 作業の前に確認してほしいこと

- 追加するコンテンツが既存ディレクトリ構造のどこに入るか
- 配色・フォント・トーンをどこまで既存に揃えるか
- 公開範囲（誰に配布するか、`noindex`でOKかなど）

---

## 背景：なぜこの構成にしたか

- **Ghost（本体サイト）とは独立**：本体サイトは記事コンテンツ中心、こちらはインタラクティブ素材・参加者配布用素材を担う。役割を分けることで、本体サイトの運用を重くしない。
- **GitHub連携を選んだ理由**：手動アップロードのドラッグ&ドロップはUIの変更に振り回される。git push で完結する運用にすることで、Cloudflareダッシュボードを触る頻度を最小化。
- **`ohgs.link` を使う理由**：短く覚えやすい、個人ブランドのドメイン。northbynorth.jp と切り分けることで、運用主体と発信主体の違いを構造で表現できる。

---

## 履歴

- 2026-04-23：リポジトリ初期化、第1回アクティビティキャラバンのマップ公開（`ohgs.link/nbn/caravan-1/map/`）、CLAUDE.md 整備
