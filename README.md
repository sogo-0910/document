# 🐶ペットの体調管理アプリ

## 🏁概要

- 認証機能
  - 会員登録
    - 指定したメアドにフォームを飛ばしパスワードを設定する
  - ログイン
    - 会員登録が完了していたら指定したメアド・PWでログイン
  - ログアウト
  - アカウント削除
  - ログインされていないときは/login（ログインページ）に飛ばす
- 管理するペットのルーム（ログインしているときのみ有効）
  - タイムツリーを参考にしています
  - ペットのルームは作成するか招待を受けないと入れない
    - ルームIDは必ず重複しないようにランダムな複数文字で生成されるようにしたい
  - ルームでできること
  - 1か月ごとのカレンダーUIになっていて日付のセルをクリックすると項目と時間と内容を登録できる
    - 誰が登録したかわかるようにして
      - それぞれ編集・削除可能
      - カレンダー一覧には項目名のみ表示
        - 項目名のラベルの背景色は変更可能
  - 体重の登録
    - 体重の登録ができる
      - 登録後に編集・削除ができる
    - 体重をグラフで週・月・年ごとに見れる
  - 写真の共有
    - 写真の投稿
      - 編集・削除可能
      - 投稿するときにコメントを追加して投稿できる

## ⚡技術スタック（Next.js + Supabase版）

### フロントエンド

| 項目           | 技術・ライブラリ                                    | 備考                                          |
| -------------- | --------------------------------------------------- | --------------------------------------------- |
| フレームワーク | **Next.js 14**                                | App Router、React Server Components活用       |
| 言語           | **TypeScript**                                | 型安全にする                                  |
| 認証           | **NextAuth.js（Auth.js）**+**Supabase** | Email、OAuth認証（Google, GitHub など）も簡単 |
| UIライブラリ   | **Tailwind CSS**                              | 開発効率UP                                    |
| カレンダー表示 | **react-calendar**or**FullCalendar**    | 使いやすい方で                                |
| グラフ表示     | **recharts**                                  | 体重推移グラフ                                |
| 状態管理       | **Zustand** （必要に応じて）                  | 軽量で使いやすい                              |

---

### バックエンド（BaaS）

| 項目         | 技術・ライブラリ                 | 備考                          |
| ------------ | -------------------------------- | ----------------------------- |
| データベース | **Supabase（PostgreSQL）** | RDBでデータ管理しやすい       |
| ストレージ   | **Supabase Storage**       | 写真・画像アップロード        |
| 認証         | **Supabase Auth**          | NextAuth.jsとも組み合わせ可能 |
| API通信      | **supabase-js**            | Supabase公式SDK               |

---

### 開発・運用

| 項目           | 技術・サービス                          | 備考                                |
| -------------- | --------------------------------------- | ----------------------------------- |
| デプロイ       | **Vercel**                        | Next.js公式デプロイ先。無料枠も強い |
| 環境変数管理   | **Next.js環境変数（.env.local）** | APIキーなど                         |
| 型チェック     | **ESLint + Prettier**             | コード品質向上                      |
| バージョン管理 | **GitHub**                        | ソース管理                          |

---

# 🔥 Next.js + Supabase 開発スタート時の流れ

## ✅ ① 全体設計・準備

### 1. 目的・機能・画面を整理

まず、「何を作るのか」「必要な機能・画面は？」をざっくり洗い出す。

#### 例：タスク管理アプリの場合

| 機能                   | 画面               | 備考                        |
| ---------------------- | ------------------ | --------------------------- |
| 認証                   | ログイン・登録     | NextAuth.js + Supabase Auth |
| タスク一覧・登録・編集 | ダッシュボード     | CRUD操作                    |
| タスク詳細表示         | タスク詳細ページ   | コメント機能追加もありかも  |
| ファイルアップロード   | タスク添付ファイル | Supabase Storage利用        |

---

### 2. データベース設計

SupabaseはPostgreSQL。必要なテーブルを考えておく。

#### 例：タスク管理アプリ用テーブル設計

| テーブル名 | カラム例                                                        | 備考                           |
| ---------- | --------------------------------------------------------------- | ------------------------------ |
| users      | id, email, name, created_at                                     | Supabase Authで自動生成あり    |
| tasks      | id, user_id, title, description, status, created_at, updated_at | タスク情報                     |
| comments   | id, task_id, user_id, content, created_at                       | タスク詳細に紐づくコメント     |
| files      | id, task_id, file_url, created_at                               | Supabase Storageでファイル管理 |

---

## ✅ ② 環境構築

### 1. Next.js プロジェクト作成

```
npx create-next-app@latest my-task-app
cd my-task-app
```

### 2. 必要パッケージインストール

```
npm install @supabase/supabase-js next-auth tailwindcss postcss autoprefixer
```

### 3. TailwindCSS 初期設定

```npx
npx tailwindcss init -p
```

`tailwind.config.js`に以下追記

```
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./pages/**/*.{js,ts,jsx,tsx}', './components/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

`globals.css`に以下追記

```
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

## ✅ ③ Supabase設定

### 1. Supabaseでプロジェクト作成

[https://supabase.com/](https://supabase.com/)
→ プロジェクト作成 → APIキーとURL取得

### 2. `.env.local` 作成

```
NEXT_PUBLIC_SUPABASE_URL=YOUR_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
```

### 3. Supabaseクライアント作成

`lib/supabaseClient.ts`

```
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!
const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!

export const supabase = createClient(supabaseUrl, supabaseKey)
```

---

## ✅ ④ 認証（NextAuth.js + Supabase）

### 1. `pages/api/auth/[...nextauth].ts` 作成

```
import NextAuth from 'next-auth'
import EmailProvider from 'next-auth/providers/email'
import { SupabaseAdapter } from '@next-auth/supabase-adapter'

export default NextAuth({
  providers: [
    EmailProvider({
      server: process.env.EMAIL_SERVER!,
      from: process.env.EMAIL_FROM!,
    }),
  ],
  adapter: SupabaseAdapter({
    url: process.env.NEXT_PUBLIC_SUPABASE_URL!,
    secret: process.env.SUPABASE_SERVICE_ROLE_KEY!,
  }),
})
```

#### 補足：

* `EMAIL_SERVER`、`EMAIL_FROM`はMagic Linkなどメール認証用。
* `SUPABASE_SERVICE_ROLE_KEY` はSupabaseのService Role Key（.envに追加しておく）

---

## ✅ ⑤ DBマイグレーション

Supabase Studio（管理画面）でテーブルを手動作成するか、SQLクエリで実行。

例：`tasks`テーブル作成

```
create table tasks (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references auth.users(id),
  title text not null,
  description text,
  status text default 'todo',
  created_at timestamp default now(),
  updated_at timestamp default now()
);
```

---

## ✅ ⑥ フロントエンド（画面開発）

### 1. タスク一覧ページ

`pages/index.tsx`

```
import { supabase } from '../lib/supabaseClient'

export default function Home({ tasks }: { tasks: any[] }) {
  return (
    <div className="p-4">
      <h1 className="text-xl font-bold">タスク一覧</h1>
      <ul>
        {tasks.map((task) => (
          <li key={task.id} className="border-b py-2">
            {task.title}
          </li>
        ))}
      </ul>
    </div>
  )
}

export async function getServerSideProps() {
  const { data: tasks } = await supabase.from('tasks').select('*')
  return { props: { tasks } }
}
```

---

## ✅ ⑦ ファイルアップロード

### 1. Supabase Storage バケット作成

Supabase Studioで「Storage」→「新しいバケット作成」

### 2. ファイルアップロード処理

```
const uploadFile = async (file: File) => {
  const { data, error } = await supabase.storage.from('task-files').upload(`tasks/${file.name}`, file)

  if (error) {
    console.error(error)
    return null
  }

  return data
}
```

---

## ✅ ⑧ デプロイ

### 1. Vercelでデプロイ（Next.jsと相性◎）

* GitHub連携 → `vercel` CLIでデプロイも可能

---

## ✅ ⑨ 進め方の例

### STEP1. 認証（NextAuth.js + Supabase）

→ Magic Link or メール認証確認

### STEP2. タスクCRUD

→ 一覧・作成・編集・削除

### STEP3. ファイルアップロード

→ Storage使ってアップロード・表示

### STEP4. コメント機能

→ リアルタイム購読（Supabase Realtime）も検討

---

## ✅ よく使う公式ドキュメント

* [Next.js公式]()
* [Supabase公式]()
* [NextAuth.js公式]()
* [Supabase Adapter（NextAuth公式）]()

---

## ✅ 技術スタック例まとめ

| 項目           | 使用技術                                 |
| -------------- | ---------------------------------------- |
| フロントエンド | Next.js 14（App Router or Pages Router） |
| スタイリング   | TailwindCSS                              |
| 認証           | NextAuth.js + Supabase Auth              |
| データベース   | Supabase（PostgreSQL）                   |
| ストレージ     | Supabase Storage                         |
| デプロイ       | Vercel                                   |
