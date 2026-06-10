# NewsLingo

英語ニュースを使って、読む、訳す、語彙を貯める学習アプリです。  
記事一覧から気になるニュースを開き、AI翻訳と重要語句の抽出で読解をサポートします。

公開URL: https://news-lingo.vercel.app

## 主な機能

- 英語ニュース記事の一覧表示（カテゴリ切り替え、キーワード検索）
  - カテゴリ: technology, business, science, world, politics, environment, culture, sport
- 記事詳細表示（原文リンク、公開日、出典情報）
- AI翻訳（タイトル・説明文・本文要約の日本語化）
- 重要語句の抽出（意味・英日例文つき）
- Googleアカウントでのログイン（Supabase Auth）
- 解析済み記事のキャッシュ（一度解析した記事はDBから再利用し、再解析を行わない）
- 語彙保存と単語帳表示（ログインユーザーの単語帳はSupabaseに保存）
- 閲覧履歴の表示（ログインユーザーのみ）

## 技術スタック

- Framework: Next.js 16.2.2 (App Router)
- Language: TypeScript
- UI: Tailwind CSS v4, shadcn/ui, Radix UI, motion
- Icons: lucide-react, react-icons
- Backend: Supabase (Postgres, Auth / Google OAuth, Row Level Security)
- API: The Guardian API, OpenAI API

## 必要な環境変数

プロジェクトルートに .env.local を作成してください。

```env
GUARDIAN_API_KEY=your_guardian_api_key
OPENAI_API_KEY=your_openai_api_key
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

メモ:

- GUARDIAN_API_KEY は記事の取得・詳細表示に必須です。
- OPENAI_API_KEY が未設定でもアプリは動作します（翻訳・キーワード抽出はフォールバック動作）。
- NEXT_PUBLIC_SUPABASE_* はログイン、記事キャッシュ、閲覧履歴、単語帳の保存に必須です。

## セットアップ

1. 依存関係をインストール

```bash
npm install
```

2. 開発サーバーを起動

```bash
npm run dev
```

3. ブラウザで確認

http://localhost:3000

## データベース（Supabase）のセットアップ

`supabase/migrations` 配下のSQLをSupabaseプロジェクトに適用してください。

- `articles`: 解析済み記事（要約・翻訳・キーワード）のキャッシュ
- `view_history`: ユーザーの閲覧履歴
- `vocabulary`: ユーザーが保存した語彙（Row Level Securityにより本人のみ閲覧・編集可能）

また、Supabase Authで Google プロバイダーを有効化し、リダイレクトURLに `/auth/callback` を設定してください。

## スクリプト

- npm run dev: 開発サーバー起動
- npm run build: 本番ビルド
- npm run start: 本番サーバー起動
- npm run lint: ESLint実行

## 画面構成

- /: ホーム
- /article: 記事一覧（カテゴリ切り替え・キーワード検索）
- /article/[...id]: 記事詳細（Guardian記事IDがスラッシュを含むため catch-all ルート）
- /article/reading_history: 閲覧履歴（要ログイン）
- /vocabulary: 単語帳
- /auth/callback: Googleログイン後のコールバック

## APIエンドポイント

- GET /api/article: 記事一覧取得（category または keyword、The Guardian APIから取得）
- GET /api/article/detail/[id]: 記事詳細取得（Guardian記事ID）
- GET /api/article/cache/[id]: 解析済み記事のキャッシュ取得（Supabase）
- POST /api/article/save: 解析結果（要約・翻訳・キーワード）を記事キャッシュとしてSupabaseに保存
- POST /api/article/analyze: AIによる要約・翻訳・キーワード抽出（OpenAI）
- POST /api/view-history: 閲覧履歴を記録
- GET /api/vocabulary: ログインユーザーの単語帳を取得（要認証）
- POST /api/vocabulary: 語彙を保存・更新（要認証）
- DELETE /api/vocabulary: 語彙を削除（要認証）

## データ保存について

- ログインしていない場合、記事一覧・記事詳細の閲覧は可能ですが、閲覧履歴と単語帳は利用できません。
- ログイン（Googleアカウント）すると、Supabase Authでユーザーが管理されます。
- 解析済み記事は `articles` テーブルにキャッシュされ、再訪問時はAI解析を再実行しません。
- 閲覧履歴は `view_history` テーブルに保存されます。
- 語彙は `vocabulary` テーブルに保存されます（旧バージョンでLocal Storageに保存されていた `savedVocabulary` は初回ログイン時に自動移行されます）。
