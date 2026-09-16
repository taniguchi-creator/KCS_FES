# KCS大分情報専門学校 学園祭備品貸出管理システム デプロイ手順書

スマートフォン（学生のスマホ）や外部環境から常時アクセス可能なWebサイトとしてデプロイするためのガイドです。
以前Netlify等で「フロントだけデプロイされてDBがくっついてこない」状態になった原因と、無料かつ安全にデータベース付きでデプロイする最もおすすめな方法を記載しています。

---

## 1. なぜ以前DBが動かなかったのか？ (Netlify / Vercelの仕様)
NetlifyやVercelなどの「静的ホスティング」サービスは、HTML/CSS/JavaScript等の**フロントエンド画面のみ**を配布します。
ローカルのファイルベースDB（SQLiteやJSONファイルなど）は、静的ホスティングサーバー上に永続的なサーバープロセスがないため自動的には連携されません。

---

## 2. 最も簡単で確実なデプロイ構成（完全無料）

おすすめの構成は **【フロントエンド: Vercel / Netlify】＋【データベース: Supabase (無料PostgreSQL)】** です。

```
[学生のスマホ (Web)] 
       │ 
       ├─── (画面表示) ────> Vercel / Netlify (GitHub連携)
       │
       └─── (データ保存/更新) ─> Supabase (無料クラウドDB)
```

---

## 3. 手順ステップ・バイ・ステップ

### ステップ A: GitHub へのコードプッシュ
1. 本フォルダ（`学園祭_備品`）を Git リポジトリとして初期化し、GitHubにプッシュします。
   ```bash
   git init
   git add .
   git commit -m "Initial commit for KCS Festival Equipment System"
   git remote add origin https://github.com/あなたのユーザー名/kcs-equipment-app.git
   git push -u origin main
   ```

### ステップ B: Supabase で無料DBを作成する（5分）
1. [Supabase公式](https://supabase.com) にアクセスして無料アカウントを作成。
2. 「New Project」をクリックし、プロジェクト名（例: `kcs-festival-db`）を設定。
3. 作成完了後、Settings -> API から **Project URL** と **anon public API Key** をコピーします。
4. コピーした接続情報を、以下のいずれかの方法でアプリに設定します。

#### 【設定方法 1】環境変数 `.env` を使用する場合（推奨）
プロジェクト直下に `.env` ファイルを作成（または `.env.example` をコピーしてリネーム）し、以下のように入力します：

```env
VITE_SUPABASE_URL=https://your-project-id.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

> **※ Vercel / Netlify にデプロイする場合:**  
> Vercelの「Settings」→「Environment Variables」にて、同じ変数名（`VITE_SUPABASE_URL` と `VITE_SUPABASE_ANON_KEY`）を追加してください。

#### 【設定方法 2】`src/services/storage.js` に直接入力する場合
[`src/services/storage.js`](file:///c:/Users/tani/Desktop/KCS_FES/src/services/storage.js#L14-L17) の 14〜17 行目にある `SUPABASE_CONFIG` オブジェクトに直接ペーストします：

```javascript
export const SUPABASE_CONFIG = {
  url: 'https://your-project-id.supabase.co',      // ← コピーした Project URL を貼り付け
  anonKey: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpX...', // ← コピーした anon key を貼り付け
};
```

### ステップ C: Vercel または Netlify にデプロイする（3分）
1. [Vercel](https://vercel.com) または [Netlify](https://netlify.com) にログイン。
2. 「Add New Project」-> 「Import Git Repository」から GitHubの `kcs-equipment-app` リポジトリを選択。
3. Build Settings:
   - Framework Preset: **Vite**
   - Build Command: `npm run build`
   - Output Directory: `dist`
4. 「Deploy」をクリック。約1分でデプロイ完了！

---

## 4. 完成後のURL共有
デプロイが完了すると `https://kcs-equipment-app.vercel.app` のような公開URLが発行されます。
このURL（またはQRコード）をLINEや学区内掲示板で共有すれば、**学生のスマートフォンからいつでも備品検索・貸出申請・承認通知の受信**が可能になります！

---

## 5. ローカルでの開発・動作確認コマンド
ローカル環境で即座に動作テストを行う場合は以下を実行します：

```bash
# 依存パッケージのインストール
npm install

# 開発用Webサーバーの起動 (http://localhost:3000)
npm run dev
```
