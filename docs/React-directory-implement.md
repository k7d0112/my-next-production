## Feature-basedの構成
```
src
 ├ app
 │   ├ router
 │   ├ providers
 │   └ store
 │
 ├ features
 │   ├ auth
 │   │   ├ api
 │   │   ├ components
 │   │   ├ hooks
 │   │   └ types
 │   │
 │   ├ courses
 │   │   ├ api
 │   │   ├ components
 │   │   ├ hooks
 │   │   └ types
 │   │
 │   └ dashboard
 │       ├ api
 │       ├ components
 │       └ hooks
 │
 ├ shared
 │   ├ components
 │   ├ hooks
 │   ├ ui
 │   └ utils
```
→ ドメイン単位設計

## 基本構造
```
UI Layer
↓
State Layer
↓
Data Layer
↓
API
```

## 実際のコード
```
Component
↓
Hook
↓
Query
↓
API
```

### 実装例
```
useCourses()
↓
getCoursesQuery()
↓
fetchCourses()
```
- API
```export const fetchCourses = async () => {
  const res = await fetch("/api/courses")
  return res.json()
}
```
- Query
```
export const useCourses = () => {
  return useQuery({
    queryKey: ["courses"],
    queryFn: fetchCourses
  })
}
```
- component
```
const CourseList = () => {
  const { data } = useCourses()

  return (
    <ul>
      {data?.map(c => (
        <li key={c.id}>{c.title}</li>
      ))}
    </ul>
  )
}
```

---

## Tanstack Query設計
- stateの種類
```
server state
client state
url state
```

### server state
APIから来るデータ
例
	•	user
	•	courses
	•	posts

これは
**TanStack Query**
で管理します。

### Tanstack Queryの設計
```
query
mutation
cache
```
#### Query key設計
```
["courses"]
["course", courseId]
["course", courseId, "students"]
```
#### Mutation
```
const mutation = useMutation({
 mutationFn: createCourse
})
```
#### invalidate
```
queryClient.invalidateQueries(["courses"])
```
#### ベストプラクティス
```
query
mutation
queryKey
```
をfeature単位で管理
例)
```
features
 ├ courses
 │   ├ queries
 │   │   ├ useCourses.ts
 │   │   └ useCourse.ts
 │   │
 │   ├ mutations
 │   │   └ useCreateCourse.ts
```

---

## Zustand設計
Zustandは
**client state**
を管理します。

### client stateの例
```
theme
modal
sidebar
filters
```

### store例
```
import { create } from "zustand"

type UIState = {
 sidebarOpen: boolean
 toggleSidebar: () => void
}

export const useUIStore = create<UIState>(set => ({
 sidebarOpen: false,
 toggleSidebar: () =>
   set(state => ({
     sidebarOpen: !state.sidebarOpen
   }))
}))
```

### 使用
```
const sidebarOpen = useUIStore(s => s.sidebarOpen)
```

### Zustand設計ルール
重要です。

- Server state
❌ Zustand
⭕ TanStack Query

- UI state
⭕ Zustand

---

## React上級者のアーキテクチャ
```
Component
↓
Hook
↓
Query
↓
API
```

Step1 React基礎

理解するもの
	•	JSX
	•	Hooks
	•	state
	•	props
	•	useEffect
	•	useMemo

⸻

Step2 SPA理解

理解するもの
	•	CSR
	•	routing
	•	lazy loading

ライブラリ
→ React Router

⸻

Step3 状態管理

- 理解
```
client state
server state
```
- ツール
```
TanStack Query
Zustand
```

⸻

Step4 データ設計

理解
```
API layer
Query layer
UI layer
```

⸻

Step5 パフォーマンス

理解
```
memo
virtualization
code splitting
```

⸻

Step6 アーキテクチャ

理解
```
feature architecture
clean architecture
BFF
```

⸻

7 React上級者が読んでいるもの

おすすめ

docs
	•	React docs
	•	TanStack Query docs

⸻

書籍
	•	Clean Architecture
	•	Frontend Architecture

⸻

つまり

React上級者は
```
state design
data flow
render strategy
```
を考える

⸻

最後に（かなり重要）

Reactを極めるには

この3つが重要です
```
state
data
render
```

## app 配下に置くもの
```
src
 ├ app
 │   ├ router
 │   │   ├ index.tsx
 │   │   ├ routes.tsx
 │   │   └ guards.tsx
 │   │
 │   ├ providers
 │   │   ├ QueryProvider.tsx
 │   │   ├ ThemeProvider.tsx
 │   │   ├ AuthProvider.tsx
 │   │   └ index.tsx
 │   │
 │   ├ store
 │   │   ├ uiStore.ts
 │   │   └ index.ts
 │   │
 │   ├ layouts
 │   │   ├ AppLayout.tsx
 │   │   ├ AuthLayout.tsx
 │   │   └ DashboardLayout.tsx
 │   │
 │   ├ styles
 │   │   ├ globals.css
 │   │   └ reset.css
 │   │
 │   ├ config
 │   │   ├ env.ts
 │   │   ├ constants.ts
 │   │   └ navigation.ts
 │   │
 │   ├ App.tsx
 │   └ main.tsx
```
⸻

1. app は「アプリ全体の初期化」の場所

app には、各 feature の中に入れると不自然なものを置きます。

たとえば次です。
	•	ルーティング
	•	Provider のまとめ
	•	全体レイアウト
	•	グローバルスタイル
	•	環境変数・定数
	•	アプリ起動ファイル
	•	アプリ全体で使う軽いグローバル store

つまり、auth/courses/dashboard のどれか1つの機能ではなく、全体に関係するものです。

⸻

2. router には何を置くか

router には、画面遷移の定義を置きます。
```
// app/router/routes.tsx
import { createBrowserRouter } from "react-router-dom";
import { AppLayout } from "../layouts/AppLayout";
import { LoginPage } from "@/features/auth/pages/LoginPage";
import { CoursePage } from "@/features/courses/pages/CoursePage";
import { DashboardPage } from "@/features/dashboard/pages/DashboardPage";

export const router = createBrowserRouter([
  {
    path: "/",
    element: <AppLayout />,
    children: [
      { path: "dashboard", element: <DashboardPage /> },
      { path: "courses", element: <CoursePage /> },
    ],
  },
  {
    path: "/login",
    element: <LoginPage />,
  },
]);
```
ここで大事なのは、router は画面のつなぎ込み担当であって、ビジネスロジックは持たせないことです。

⸻

3. providers には何を置くか

providers には、アプリ起動時に一度ラップするものをまとめます。

代表例:
	•	QueryClientProvider
	•	ThemeProvider
	•	AuthProvider
	•	BrowserRouter
	•	i18n provider
	•	ErrorBoundary
	•	Toast provider
```
// app/providers/index.tsx
import { ReactNode } from "react";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

type Props = {
  children: ReactNode;
};

export const AppProviders = ({ children }: Props) => {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};
```
main.tsx ではこれを使います。
```
// app/main.tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { App } from "./App";
import { AppProviders } from "./providers";
import "./styles/globals.css";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <AppProviders>
    <App />
  </AppProviders>
);
```

⸻

4. store には何を置くか

ここは少し注意です。
store に何でも入れると、また昔の「巨大グローバル状態管理」に戻ります。

なので app/store に置くのは、本当に全体共通な client state だけです。

例:
	•	テーマ
	•	サイドバー開閉
	•	全画面モーダル
	•	通知UIの状態
	•	ユーザー設定の一部

逆に置かない方がいいもの:
	•	courses 一覧
	•	dashboard の集計結果
	•	認証済みユーザー詳細を何でもかんでも store に入れる
	•	APIレスポンス全般

API由来のデータは、基本 TanStack Query に任せます。

⸻

5. layouts には何を置くか

layouts は、ページ共通の枠組みです。

例:
	•	ヘッダー付きレイアウト
	•	サイドバー付きダッシュボードレイアウト
	•	認証ページ専用レイアウト
```
// app/layouts/AppLayout.tsx
import { Outlet } from "react-router-dom";

export const AppLayout = () => {
  return (
    <div>
      <header>Header</header>
      <main>
        <Outlet />
      </main>
    </div>
  );
};
```
features の責務は個別機能、layouts の責務は画面全体の骨組みです。

⸻

6. config には何を置くか

config には設定値や環境依存値を置きます。

例:
```
// app/config/env.ts
export const env = {
  apiBaseUrl: import.meta.env.VITE_API_BASE_URL,
};

// app/config/navigation.ts
export const NAV_ITEMS = [
  { label: "Dashboard", path: "/dashboard" },
  { label: "Courses", path: "/courses" },
];
```
ここには「静的な設定」を置き、ロジックは入れすぎないのがコツです。

⸻

7. App.tsx と main.tsx の役割

よく混ざるので分けて考えるといいです。

main.tsx
	•	React アプリをマウントする
	•	Provider を巻く
	•	エントリポイント

App.tsx
	•	Router や大枠UIを描画する

例:
```
// app/App.tsx
import { RouterProvider } from "react-router-dom";
import { router } from "./router/routes";

export const App = () => {
  return <RouterProvider router={router} />;
};
```

⸻

8. features に置くべきものとの違い

ここが一番大事です。

app に置く
	•	アプリ全体で1つだけあるもの
	•	全 feature を束ねるもの
	•	起動時に必要なもの

features に置く
	•	auth/courses/dashboard などの機能単位のもの
	•	その機能で閉じるUI・hook・API・型

たとえば auth のログインフォームは features/auth に置きます。
でも「未ログインなら /login に飛ばすルートガード」は app/router に置くことが多いです。

⸻

9. 実務でおすすめの切り分け

かなり実務寄りにいうと、次の基準がわかりやすいです。

app に入れるもの
	•	アプリ起動処理
	•	ルーティング
	•	Provider
	•	レイアウト
	•	全体設定
	•	truly global な UI state

shared に入れるもの
	•	汎用 Button
	•	Modal
	•	formatDate
	•	共通 hooks
	•	共通 UI部品

features に入れるもの
	•	ログイン処理
	•	コース一覧取得
	•	ダッシュボード集計表示
	•	その feature 専用 component / hook / api / types

⸻

10. 迷ったときの判断基準

迷ったらこの質問をします。

そのコードは特定機能に閉じるか？

閉じるなら features

アプリ全体で1回だけ設定されるか？

そうなら app

どの機能でも使える汎用品か？

そうなら shared

この3分割でかなり整理できます。

⸻

11. 学習支援アプリなら app はこうなりやすい

あなたのような学習支援プラットフォーム系なら、たとえばこうです。
```
たとえば student / instructor で UI 骨組みが違うなら layout で分けるのが自然です。

⸻

12. 設計上の注意点

app は便利なので、何でも置きたくなります。
でもそれをやると app が巨大化します。

避けるべき状態はこれです。
	•	app にビジネスロジックが集まる
	•	app/store が全部入りになる
	•	router に認可ロジックや API 呼び出しが増える

app はあくまで composition root に近いです。
つまり「組み立て場所」であって、「機能実装の本体」ではありません。

⸻

「加えて下記の内容」については、このメッセージでは本文が見えていません。続きを貼ってくれれば、そのまま続けて整理します。

必要なら次に、
app / features / shared の実務的な責務分離を、サンプルコード付きで丸ごと解説します。


## sharedの役割
```
src
├ app
├ features
└ shared
```
この3つは、それぞれ役割が違います。
	•	app: アプリ全体の組み立て
	•	features: 機能ごとの実装本体
	•	shared: 機能に依存しない再利用資産

⸻

1. まず結論: それぞれの責務

app

app は アプリ全体を起動・接続する場所 です。

置くもの:
	•	ルーティング
	•	Provider
	•	レイアウト
	•	グローバル設定
	•	エントリポイント
	•	アプリ全体で1つだけ必要な初期化処理

置かないもの:
	•	機能固有のUI
	•	機能固有のAPIロジック
	•	機能固有のhooks

⸻

features

features は 機能の本体 です。

置くもの:
	•	auth
	•	courses
	•	dashboard
	•	notifications
	•	profile

など、ドメインやユースケース単位のコード。

各 feature の中には:
	•	UI
	•	hooks
	•	API呼び出し
	•	query/mutation
	•	types
	•	その機能専用のutils

を置きます。

⸻

shared

shared は どの feature からも使える汎用品 です。

置くもの:
	•	Button
	•	Modal
	•	Input
	•	formatDate
	•	汎用バリデーション
	•	共通定数
	•	APIクライアント
	•	UIライブラリラッパー

置かないもの:
	•	auth専用処理
	•	course専用型
	•	dashboard専用hook

⸻

2. 依存関係のルール

実務ではこれが最重要です。

依存の向きはこうします。
```
app
 ↓
features
 ↓
shared
```
つまり:
	•	app は features と shared を使ってよい
	•	features は shared を使ってよい
	•	shared は features を使ってはいけない
	•	features 同士の直接依存はできるだけ避ける

これが崩れると、すぐに巨大で壊れやすい構造になります。

⸻

3. 実務的なディレクトリ例

たとえば学習支援アプリならこんな形です。
src
├ app
│  ├ router
│  │  ├ routes.tsx
│  │  └ guards.tsx
│  ├ providers
│  │  ├ AppProviders.tsx
│  │  ├ QueryProvider.tsx
│  │  └ ThemeProvider.tsx
│  ├ layouts
│  │  ├ AppLayout.tsx
│  │  ├ AuthLayout.tsx
│  │  └ InstructorLayout.tsx
│  ├ config
│  │  ├ env.ts
│  │  └ navigation.ts
│  ├ App.tsx
│  └ main.tsx
│
├ features
│  ├ auth
│  │  ├ api
│  │  │  ├ login.ts
│  │  │  └ getMe.ts
│  │  ├ hooks
│  │  │  ├ useLogin.ts
│  │  │  └ useCurrentUser.ts
│  │  ├ components
│  │  │  └ LoginForm.tsx
│  │  ├ pages
│  │  │  └ LoginPage.tsx
│  │  ├ types
│  │  │  └ auth.ts
│  │  └ index.ts
│  │
│  ├ courses
│  │  ├ api
│  │  │  ├ fetchCourses.ts
│  │  │  └ createCourse.ts
│  │  ├ hooks
│  │  │  ├ useCourses.ts
│  │  │  └ useCreateCourse.ts
│  │  ├ components
│  │  │  ├ CourseList.tsx
│  │  │  └ CourseCard.tsx
│  │  ├ pages
│  │  │  └ CoursesPage.tsx
│  │  ├ types
│  │  │  └ course.ts
│  │  └ index.ts
│  │
│  └ dashboard
│     ├ api
│     ├ hooks
│     ├ components
│     ├ pages
│     ├ types
│     └ index.ts
│
└ shared
   ├ api
   │  ├ client.ts
   │  └ http.ts
   ├ ui
   │  ├ Button.tsx
   │  ├ Input.tsx
   │  ├ Modal.tsx
   │  └ Spinner.tsx
   ├ hooks
   │  ├ useDebounce.ts
   │  └ useDisclosure.ts
   ├ lib
   │  ├ date.ts
   │  └ validation.ts
   ├ constants
   │  └ queryKeys.ts
   ├ types
   │  └ common.ts
   └ utils
      └ cn.ts
```

⸻

⸻

7. どこに置くべきか迷う例

ここが実務で一番大事です。

例1: LoginForm

これは auth 専用です。
features/auth/components/LoginForm.tsx

⸻

例2: Button

これは汎用です。
shared/ui/Button.tsx

⸻

例3: useCurrentUser

認証ドメインに閉じるなら
features/auth/hooks/useCurrentUser.ts

⸻

例4: apiClient

複数 feature で共通利用するので
shared/api/client.ts

⸻

例5: AppLayout

アプリ全体の骨組みなので
app/layouts/AppLayout.tsx

⸻

例6: RouteGuard

ルーティング制御はアプリ組み立て責務なので
app/router/guards.tsx

ただし、認証状態の取得ロジックそのものは features/auth に寄せることが多いです。

⸻

8. 実務でよくある失敗

失敗1: shared が巨大ゴミ箱になる

何でも shared に入れると壊れます。

たとえば:
	•	shared/components/UserProfileCard
	•	shared/utils/courseFilter
	•	shared/hooks/useDashboardSummary

これは shared ではありません。
機能固有です。

shared は 本当に汎用的なものだけ に絞ります。

⸻

失敗2: app にビジネスロジックが集まる

app/router/routes.tsx に認証判定、API呼び出し、role分岐、メニュー生成などを全部書き始めると、app が太ります。

app はなるべく薄くして、ロジックは feature に寄せます。

⸻

失敗3: features 同士がベタベタ依存する

たとえば dashboard から courses/components/CourseCard を直接使いまくると、境界が崩れます。

共通化したいなら:
	•	本当に汎用なら shared
	•	domain的に共有したいなら設計を見直す

安易な横断依存は避けます。

⸻

9. index.ts を使った公開面の整理

実務では feature の外から何を使ってよいかを制限すると管理しやすいです。

features/courses/index.ts
```
export { CoursesPage } from "./pages/CoursesPage";
export { useCourses } from "./hooks/useCourses";
```
これで外部からは features/courses の公開面だけを見るようにできます。
これをやると、feature の内部構造を後で変えても影響を小さくできます。

⸻

10. ページとコンポーネントの分け方

実務では pages を feature の中に持つことが多いです。
	•	pages: route に直接結びつくもの
	•	components: page の中で使う部品

例:
```
features/courses/
├ pages/
│  └ CoursesPage.tsx
└ components/
   ├ CourseList.tsx
   └ CourseCard.tsx
```
page は feature の入口、components は内部部品です。

⸻

11. 実務でのおすすめ責務分離ルール

かなり使いやすいルールです。

app

「どう組み立てるか」

features

「何を実現するか」

shared

「何度でも使える汎用品」

この3つで考えると迷いにくいです。

⸻

12. 学習支援アプリでの具体例
たとえばあなたのような学習支援プラットフォームなら、こう切れます。

app
	•	student/instructor ルーティング
	•	認証ガード
	•	全体レイアウト
	•	providers

features/auth
	•	ログイン
	•	ログアウト
	•	current user取得
	•	権限判定hook

features/courses
	•	コース一覧
	•	コース作成
	•	コース編集

features/submissions
	•	提出一覧
	•	レビュー
	•	ステータス更新

shared
	•	Button
	•	Modal
	•	Table
	•	date format
	•	API client

⸻

13. 迷ったらこの順で判断する

このコードは:
	1.	アプリ全体を組み立てるためのものか
→ app
	2.	特定の機能に閉じるものか
→ features
	3.	どの機能でも使える汎用品か
→ shared

この順で考えるとかなり整理できます。

⸻

14. 最後に: 一番大事なこと

上級者が見ているのは、フォルダ名ではなく 変更に強い境界 です。

よい構成は、
	•	新機能追加がしやすい
	•	既存修正の影響範囲が小さい
	•	依存関係が追いやすい
	•	featureごとに理解しやすい

という状態を作れます。

逆に悪い構成は、
	•	どこを直せばいいかわからない
	•	shared に何でも入る
	•	app にロジックが集まる
	•	features が互いに依存して崩れる

です。

必要なら次に、この構成を前提に TanStack Query と Zustand をどこにどう置くべきか を、実際のディレクトリ構成とコード付きで続けて整理できます。