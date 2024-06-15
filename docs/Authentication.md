---
layout: default
title: "セキュリティ"
---

# セキュリティ

<video controls autoplay playsinline muted loop>
  <source src="./img/login.webm" type="video/webm"/>
  <source src="./img/login.mp4" type="video/mp4"/>
  お使いのブラウザはビデオタグをサポートしていません。
</video>

React-adminでは、お好みの認証戦略を使用して管理アプリを保護できます。Basic Auth、JWT、OAuthなど多くの戦略があるため、react-adminは認証ロジックを`authProvider`に委任します。

## 認証機能の有効化

デフォルトでは、react-adminアプリは認証を要求しません。管理アクセスを制限するには、`<Admin>`コンポーネントに`authProvider`を渡します。

```jsx
    // src/App.js
    import authProvider from './authProvider';

    const App = () => (
        <Admin authProvider={authProvider}>
            ...
        </Admin>
    );
```

管理アプリに`authProvider`が設定されると、`/login`ルートに新しいページが追加され、ユーザー名とパスワードを求めるログインフォームが表示されます。

## `authProvider`の構成

`authProvider`とは何でしょうか？`dataProvider`と同様に、`authProvider`は認証と認可のロジックを処理するオブジェクトです。必要に応じてreact-adminが呼び出すメソッドを公開し、専門のフックを通じて手動で呼び出すこともできます。`authProvider`のメソッドはPromiseを返す必要があります。最もシンプルな`authProvider`は次のようになります：

```js
const authProvider = {
    // ユーザー名とパスワードを認証サーバーに送り、認証情報を取得
    login: params => Promise.resolve(),
    // dataProviderがエラーを返したときに、認証エラーかどうかを確認
    checkError: error => Promise.resolve(),
    // ユーザーがナビゲートする際に認証情報がまだ有効かどうかを確認
    checkAuth: params => Promise.resolve(),
    // ローカルの認証情報を削除し、ユーザーがログアウトしたことを認証サーバーに通知
    logout: () => Promise.resolve(),
    // ユーザープロフィールを取得
    getIdentity: () => Promise.resolve(),
    // ユーザーの権限を取得（オプション）
    getPermissions: () => Promise.resolve(),
};
```

既存の認証プロバイダーは[利用可能な認証プロバイダーのリスト](./AuthProviderList.md)で見つけるか、[独自の認証プロバイダーの作成方法](./AuthProviderWriting.md)に従って独自のものを作成できます。

## APIに認証情報を送信する

`authProvider`は認証ロジックを処理しますが、APIとの通信で認証情報を使用するのは`dataProvider`の責任です。

[データプロバイダードキュメント](./DataProviders.md#adding-custom-headers)で説明されているように、`simpleRestProvider`と`jsonServerProvider`は2番目のパラメータとして`httpClient`を受け取ります。ここでリクエストヘッダーやクッキーなどを変更できます。

例えば、`authProvider`が認証トークンをlocalStorageに保存する場合、`dataProvider`を調整してこのトークンを`Authorization`ヘッダーとして渡す方法は次のとおりです：

```jsx
import { fetchUtils, Admin, Resource } from 'react-admin';
import simpleRestProvider from 'ra-data-simple-rest';

const httpClient = (url, options = {}) => {
    if (!options.headers) {
        options.headers = new Headers({ Accept: 'application/json' });
    }
    const { token } = JSON.parse(localStorage.getItem('auth'));
    options.headers.set('Authorization', `Bearer ${token}`);
    return fetchUtils.fetchJson(url, options);
};
const dataProvider = simpleRestProvider('http://localhost:3000', httpClient);

const App = () => (
    <Admin dataProvider={dataProvider} authProvider={authProvider}>
        ...
    </Admin>
);
```

これで管理アプリが保護され、ユーザーは認証され、セキュアなAPIと通信するための認証情報を使用できます。

独自のRESTクライアントを使用する場合、必ず自分で認証情報を追加してください。

## 匿名アクセスの許可

`authProvider`を追加すると、react-adminは`<Resource>`コンポーネントで宣言されたすべてのページへのアクセスを制限します。匿名アクセスを許可する場合は、ページコンポーネントに`disableAuthentication`プロップを設定できます。

例えば、匿名ユーザーに投稿リストビューへのアクセスを許可するには：

```jsx
const PostList = () => (
    <List disableAuthentication>
        // ...
    </List>
);

const App = () => (
    <Admin dataProvider={dataProvider} authProvider={authProvider}>
        <Resource name="posts" list={PostList} />
    </Admin>
);
```

`disableAuthentication`は次のコンポーネントやフックで利用可能です：

* `<Create>`, `<CreateBase>`, `<CreateController>`, `useCreateController`
* `<Edit>`, `<EditBase>`, `<EditController>`, `useEditController`
* `<List>`, `<ListBase>`, `<ListController>`, `useListController`
* `<Show>`, `<ShowBase>`, `<ShowController>`, `useShowController`

## 匿名アクセスの無効化

react-adminアプリの一部のページは匿名アクセスを許可する場合があります。このため、react-adminはユーザーがログインしているかどうかを確認する前にページレイアウトを表示し始めます。すべてのページが認証を必要とする場合、このデフォルトの動作は、ログインしていないユーザーに対して不要な「UIのフラッシュ」を引き起こします。

アプリが匿名アクセスを一切許可しないことがわかっている場合、`<Admin requireAuth>`プロップを設定して、ページレイアウトのレンダリング前に`authProvider.checkAuth()`の解決を待つように強制できます。

```jsx
const App = () => (
    <Admin dataProvider={dataProvider} authProvider={authProvider} requireAuth>
        <Resource name="posts" list={PostList} />
    </Admin>
);
```

## カスタムページへのアクセス制限

カスタムページを追加すると、デフォルトで匿名ユーザーがアクセス可能です。これらを認証されたユーザーのみがアクセスできるようにするには、カスタムページで[useAuthenticatedフック](./useAuthenticated.md)を呼び出します：

```jsx
import { Admin, CustomRoutes, useAuthenticated } from 'react-admin';

const MyPage = () => {
    useAuthenticated(); // 認証されていない場合、ログインページにリダイレクト
    return (
        <div>
            ...
        </div>
    )
};

const App = () => (
    <Admin authProvider={authProvider}>
        <CustomRoutes>
            <Route path="/foo" element={<MyPage />} />
            <Route path="/anoonymous" element={<Baz />} />
        </CustomRoutes>
    </Admin>
);
```

または、[<Authenticated>コンポーネント](./Authenticated.md)を使用することもできます。例えば、ページコンポーネントを変更できない場合や、`<Route element>`プロップで認証を追加したい場合などです：

```jsx
import { Admin, CustomRoutes, Authenticated } from 'react-admin';

const MyPage = () => {
    return (
        <div>
            ...
        </div>
    )
};

const App = () => (
    <Admin authProvider={authProvider}>
        <CustomRoutes>
            <Route path="/foo" element={<Authenticated><MyPage /></Authenticated>} />
            <Route path="/anoonymous" element={<Baz />} />
        </CustomRoutes>
    </Admin>
);
```

## 外部認証プロバイダーの使用

組み込みのログインページの代わりに、Auth0、Cognito、または他のOAuthベースのサービスなどの外部認証プロバイダーを選択できます。これらのサービスはすべて、ログイン後にユーザーをリダイレクトするコールバックURLをアプリに要求します。

React-adminは`/auth-callback`にデフォルトのコールバックURLを提供します。このルートはマウント時に`authProvider.handleCallback`メソッドを呼び出します。つまり、受け取ったパラメーターを使用して将来のAPI呼び出しを認証するのは`authProvider`の役割です。

例えば、Auth0のためのシンプルなauthProviderは次のようになります：

```js
import { Auth0Client } from './Auth0Client';

export const authProvider = {
    async login() { /* ここでは何もしない、この関数は呼ばれません */ },
    async checkAuth() {
        const isAuthenticated = await Auth0Client.isAuthenticated();
        if (isAuthenticated) {
            return;
        }
        // 認証されていない場合、ユーザーをAuth0サービスにリダイレクトし、
        // ログイン後にアプリに戻る
        Auth0Client.loginWithRedirect({
            authorizationParams: {
                redirect_uri: `${window.location.origin}/auth-callback`,
            },
        });
    },
    // ユーザーがAuth0サービスで正常にログインし、
    // アプリの/auth-callbackルートにリダイレクトされた
    async handleCallback() {
        const query = window.location.search;
        if (query.includes('code=') && query.includes('state=')) {
            try {
                // クエリパラメータに基づいてアクセストークンを取得
                await Auth0Client.handleRedirectCallback();
                return;
            } catch (error) {
                console.log('error', error);
                throw error;
            }
        }
        throw new Error('ログインコールバックの処理に失敗しました。');
    },
    async logout() {
        const isAuthenticated = await client.isAuthenticated();
            // react-adminがcheckAuthに失敗した場合にlogoutを呼び出す必要があるため、チェックが必要
        if (isAuthenticated) {
            return client.logout({
                returnTo: window.location.origin,
            });
        }
    },
    ...
}
```

ユーザーをサードパーティ認証サービスにリダイレクトするタイミングは、以下のように決定できます：

* 上記のように`AuthProvider.checkAuth()`メソッド内で直接。
* [カスタムログインページ](#customizing-the-login-component)でユーザーがボタンをクリックしたとき。

## リフレッシュトークンの処理

[リフレッシュトークン](https://oauth.net/2/refresh-tokens/)は重要なセキュリティメカニズムです。これを活用するには、認証をリフレッシュする必要があるかどうかをチェックし、実際にリフレッシュするように`dataProvider`と`authProvider`を装飾する必要があります。

この目的のために、[`addRefreshAuthToDataProvider`](./addRefreshAuthToDataProvider.md)および[`addRefreshAuthToAuthProvider`](./addRefreshAuthToAuthProvider.md)関数を使用できます。これらはそれぞれ`dataProvider`や`authProvider`を受け取り、必要に応じて認証トークンをリフレッシュする関数を受け取ります：

```jsx
// src/refreshAuth.js
import { getAuthTokensFromLocalStorage } from './getAuthTokensFromLocalStorage';
import { refreshAuthTokens } from './refreshAuthTokens';

export const refreshAuth = () => {
    const { accessToken, refreshToken } = getAuthTokensFromLocalStorage();
    if (accessToken.exp < Date.now().getTime() / 1000) {
        // この関数は認証サービスから新しいトークンを取得し、localStorageに更新します
        return refreshAuthTokens(refreshToken);
    }
    return Promise.resolve();
}

// src/authProvider.js
import { addRefreshAuthToAuthProvider } from 'react-admin';
import { refreshAuth } from 'refreshAuth';
const myAuthProvider = {
    // 通常のAuthProviderメソッド
};
export const authProvider = addRefreshAuthToAuthProvider(myAuthProvider, refreshAuth);

// src/dataProvider.js
import { addRefreshAuthToDataProvider } from 'react-admin';
import simpleRestProvider from 'ra-data-simple-rest';
import { refreshAuth } from 'refreshAuth';
const baseDataProvider = simpleRestProvider('http://path.to.my.api/');
export const dataProvider = addRefreshAuthToDataProvider(baseDataProvider, refreshAuth);
```

## ログインコンポーネントのカスタマイズ

ユーザー名とパスワードに依存する認証の場合、`authProvider`を使用するだけで完全な認証システムを実装できます。

しかし、ユーザー名の代わりにメールを使用したい場合や、サードパーティの認証サービスでシングルサインオン（SSO）を使用したい場合、または二要素認証を使用したい場合はどうでしょうか？

これらすべての場合、デフォルトのユーザー名/パスワードフォームの代わりに、`/login`ルートに表示される独自の`LoginPage`コンポーネントを実装する必要があります。このコンポーネントを`<Admin>`コンポーネントに渡します：

```jsx
// src/App.js
import { Admin } from 'react-admin';

import MyLoginPage from './MyLoginPage';

const App = () => (
    <Admin loginPage={MyLoginPage} authProvider={authProvider}>
    ...
    </Admin>
);
```

デフォルトでは、ログインページはグラデーションの背景を表示します。背景だけを変更したい場合は、デフォルトのログインページコンポーネントを使用し、`backgroundImage`プロップに画像URLを渡すことができます。

```jsx
// src/MyLoginPage.js
import { Login } from 'react-admin';

const MyLoginPage = () => (
    <Login
        // 毎日変わるランダムな画像
        backgroundImage="https://source.unsplash.com/random/1600x900/daily"
    />
);
```

ログインページを一から作成する場合、[`useLogin`フック](./useLogin.md)が必要です。

```jsx
// src/MyLoginPage.js
import { useState } from 'react';
import { useLogin, useNotify, Notification } from 'react-admin';

const MyLoginPage = ({ theme }) => {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const login = useLogin();
    const notify = useNotify();

    const handleSubmit = e => {
        e.preventDefault();
        login({ email, password }).catch(() =>
            notify('無効なメールアドレスまたはパスワードです')
        );
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                name="email"
                type="email"
                value={email}
                onChange={e => setEmail(e.target.value)}
            />
            <input
                name="password"
                type="password"
                value={password}
                onChange={e => setPassword(e.target.value)}
            />
        </form>
    );
};

export default MyLoginPage;
```

## ログアウトコンポーネントのカスタマイズ

```jsx
// src/MyLogoutButton.js
import * as React from 'react';
import { forwardRef } from 'react';
import { useLogout } from 'react-admin';
import MenuItem from '@mui/material/MenuItem';
import ExitIcon from '@mui/icons-material/PowerSettingsNew';

// Material UIがキーボードナビゲーションを管理できるようにするため、refを渡すことが重要です
const MyLogoutButton = forwardRef((props, ref) => {
    const logout = useLogout();
    const handleClick = () => logout();
    return (
        <MenuItem
            onClick={handleClick}
            ref={ref}
            // Material UIがキーボードナビゲーションを管理できるようにするため、propsを渡すことが重要です
            {...props}
        >
            <ExitIcon /> ログアウト
        </MenuItem>
    );
});

export default MyLogoutButton;
```

**ヒント**：デフォルトでは、react-adminはログアウト後にユーザーを'/login'にリダイレクトします。これを変更するには、`logout()`関数にリダイレクトするURLをパラメータとして渡します：

```diff
// src/MyLogoutButton.js
// ...
-   const handleClick = () => logout();
+   const handleClick = () => logout('/custom-login');
```

これを使用するには、カスタム`UserMenu`を提供する必要があります：

```jsx
import MyLogoutButton from './MyLogoutButton';

const MyUserMenu = () => <UserMenu><MyLogoutButton /></UserMenu>;

const MyAppBar = () => <AppBar userMenu={<MyUserMenu />} />;

const MyLayout = (props) => <Layout {...props} appBar={MyAppBar} />;

const App = () => (
    <Admin layout={MyLayout}>
        // ...
    </Admin>
);
```


