---
layout: default
title: "APIのクエリ実行"
---

# APIのクエリ実行

React-adminは、読み取りおよび書き込みクエリを[`dataProvider`](./DataProviders.md)に送信するための特別なフックを提供します。`dataProvider`はあなたのAPIにリクエストを送信します。内部では、[react-query](https://tanstack.com/query/v3/)を使用して`dataProvider`を呼び出し、その結果をキャッシュします。

## `dataProvider`インスタンスの取得

React-adminは`dataProvider`オブジェクトをReactコンテキストに保存しているため、アプリケーションコードのどこからでもアクセスできます。`useDataProvider`フックはそのコンテキストから`dataProvider`を取得し、直接呼び出すことができます。

例えば、現在のユーザープロファイルを`dataProvider`にクエリする方法は次の通りです:

```jsx
    import { useDataProvider } from 'react-admin';

    const MyComponent = () => {
        const dataProvider = useDataProvider();
        // ...
    }
```

詳細については、[`useDataProvider`フックのドキュメント](./useDataProvider.md)を参照してください。

**ヒント**: フックによって返される`dataProvider`は、実際にはあなたの`dataProvider`の*ラッパー*です。このラッパーは、`dataProvider`がエラーを返した場合にユーザーをログアウトさせ、`authProvider`がそのエラーを認証エラーと見なす場合（`authProvider.checkError()`を通じて）にログアウトさせます。

## DataProviderメソッドフック

React-adminは、各Data Providerメソッドに対して1つのフックを提供します。これらはコードをより読みやすくし、より堅牢にするための便利なショートカットです。

クエリフックはマウント時に実行され、次のプロパティを持つオブジェクトを返します: `{ data, isLoading, error }`。クエリフックには次のものがあります:

* [`useGetList`](./useGetList.md)
* [`useGetOne`](./useGetOne.md)
* [`useGetMany`](./useGetMany.md)
* [`useGetManyReference`](./useGetManyReference.md)

ミューテーションフックはコールバックを呼び出したときにクエリを実行し、次の項目を含む配列を返します: `[mutate, { data, isLoading, error }]`。ミューテーションフックには次のものがあります:

* [`useCreate`](./useCreate.md)
* [`useUpdate`](./useUpdate.md)
* [`useUpdateMany`](./useUpdateMany.md)
* [`useDelete`](./useDelete.md)
* [`useDeleteMany`](./useDeleteMany.md)

そのシグネチャは関連する`dataProvider`メソッドと同じです。例えば：

```jsx
// dataProvider.getOne(resource, { id })を呼び出します
const { data, isLoading, error } = useGetOne(resource, { id });
```

例えば、`useGetOne`フックを使用してAPIから1つのレコードを取得する方法は次の通りです:

```jsx
import { useGetOne } from 'react-admin';
import { Loading, Error } from './MyComponents';

const UserProfile = ({ userId }) => {
    const { data: user, isLoading, error } = useGetOne('users', { id: userId });

    if (isLoading) return <Loading />;
    if (error) return <Error />;
    if (!user) return null;

    return (
        <ul>
            <li>Name: {user.name}</li>
            <li>Email: {user.email}</li>
        </ul>
    )
};
```

次に、`useUpdate()`を使用した別の例を示します:

```jsx
import * as React from 'react';
import { useUpdate, useRecordContext, Button } from 'react-admin';

const ApproveButton = () => {
    const record = useRecordContext();
    const [approve, { isLoading }] = useUpdate('comments', { id: record.id, data: { isApproved: true }, previousData: record });
    return <Button label="Approve" onClick={() => approve()} disabled={isLoading} />;
};
```

クエリフックとミューテーションフックの両方は、クエリオプションを上書きするための`options`引数を受け取ります:

```jsx
const { data: user, isLoading, error } = useGetOne(
    'users',
    { id: userId },
    { enabled: userId !== undefined }
);
```

**ヒント**: TypeScriptを使用する場合、レコードタイプを指定してより型安全にすることができます:

```jsx
const { data, isLoading } = useGetOne<Product>('products', { id: 123 });
//        \- dataの型はProductになります
```

## `meta`パラメータ

すべてのData Providerメソッドは`meta`パラメータを受け取ります。React-adminはデフォルトではクエリにこのパラメータを設定しませんが、APIコールに特別な引数やメタデータを渡すための良い方法です。

```jsx
const { data, isLoading, error } = useGetOne(
    'books',
    { id, meta: { _embed: 'authors' } },
);
```

このパラメータを解釈するのはData Provider次第です。

## `useQuery`と`useMutation`

内部的には、react-adminは[react-query](https://tanstack.com/query/v3/)を使用して`dataProvider`を呼び出します。コンポーネントで`dataProvider`からデータを取得する際、どの`dataProvider`メソッドフックも使用できない場合は、そのライブラリを使用することをお勧めします。以下のような利点があります:

1. クエリが実行されているときにAppBarのローダーがトリガーされる。
2. `useState`を使用する必要がないため、ボイラープレートコードが削減される。
3. 多くのオプションをサポートする。
4. 最新のデータを取得する間、古いデータを表示し続けるため、UIがよりスムーズになる。

React-queryは、`dataProvider`と対話するための2つの主要なフックを提供します:

* [`useQuery`](https://tanstack.com/query/v3/docs/react/reference/useQuery): マウント時に`dataProvider`をフェッチします。これは読み取りクエリ用です。
* [`useMutation`](https://tanstack.com/query/v3/docs/react/reference/useMutation): コールバックを呼び出したときに`dataProvider`をフェッチします。これは書き込みクエリ用で、ユーザー操作で実行される読み取りクエリにも使用されます。

これらのフックは、クエリキャッシュ内のクエリを識別するための*キー*と、クエリを実行しPromiseを返す*関数*を受け取ります。内部的には、react-adminは引数の配列をクエリキーとして使用します。

例えば、この章の最初のコードスニペットは、`useQuery`を使用して次のように書き直すことができます:

```jsx
import * as React from 'react';
import { useQuery } from 'react-query';
import { useDataProvider, Loading, Error } from 'react-admin';

const UserProfile = ({ userId }) => {
    const dataProvider = useDataProvider();
    const { data, isLoading, error } = useQuery(
        ['users', 'getOne', { id: userId }], 
        () => dataProvider.getOne('users', { id: userId })
    );

    if (isLoading) return <Loading />;
    if (error) return <Error />;
    if (!data) return null;

    return (
        <ul>
            <li>Name: {data.data.name}</li>
            <li>Email: {data.data.email}</li>
        </ul>
    )
};
```

`useMutation`の使用法を説明するために、コメントの"Approve"ボタンの実装を示します:

```jsx
import * as React from 'react';
import { useMutation } from 'react-query';
import { useDataProvider, useRecordContext, Button } from 'react-admin';

const ApproveButton = () => {
    const record = useRecordContext();
    const dataProvider = useDataProvider();
    const { mutate, isLoading } = useMutation(
        () => dataProvider.update('comments', { id: record.id, data: { isApproved: true } })
    );
    return <Button label="Approve" onClick={() => mutate()} disabled={isLoading} />;
};
```

データプロバイダーメソッドフックを超えて使用したい場合は、[react-queryのドキュメント](https://react-query-v3.tanstack.com/overview)を読むことをお勧めします。

## `isLoading`対`isFetching`

データフェッチングフックは2つのロード状態変数を返します: `isLoading`と`isFetching`。どちらを使用するべきでしょうか？

簡単な答えは: `isLoading`を使用してください。以下にその理由を説明します。

これら2つの変数の出所は[react-query](https://react-query-v3.tanstack.com/guides/queries#query-basics)です。彼らはこれらの2つの変数を次のように定義しています:

* `isLoading`: クエリにデータがなく、現在フェッチ中である
* `isFetching`: クエリが任意の状態でフェッチ中である場合（バックグラウンドフェッチを含む）、`isFetching`はtrueになります。

典型的な使用シナリオでこれらの変数が何を含むかを見てみましょう:

1. ユーザーが最初にページを読み込む。`isLoading`はtrueです。データが一度もロードされておらず、`isFetching`もデータがフェッチされているためtrueです。
2. `dataProvider`がデータを返します。`isLoading`も`isFetching`もfalseになります。
3. ユーザーが離れる
4. ユーザーが最初のページに戻り、新しいフェッチがトリガーされます。`isLoading`はfalseです。古いデータがあり、`isFetching`はデータが`dataProvider`を介してフェッチされているためtrueです。
5. `dataProvider`がデータを返します。`isLoading`も`isFetching`もfalseになります。

コンポーネントは、表示するデータがない場合にロードインジケータを表示するためにロード状態を使用します。上記の例では、ステップ2ではロードインジケータが必要ですが、ステップ4では古いデータを表示できるため不要です。

```jsx
import { useGetOne, useRecordContext } from 'react-admin';

const UserProfile = () => {
    const record = useRecordContext();
    const { data, isLoading, error } = useGetOne('users', { id: record.id });
    if (isLoading) { return <Loading />; }
    if (error) { return <p>ERROR</p>; }
    return <div>User {data.username}</div>;
};
```

そのため、ロードインジケータを表示する必要がある場合は常に`isLoading`を使用する必要があります。

## カスタムメソッドの呼び出し

管理インターフェースでは、CRUDリクエストを超えてAPIにクエリを実行する必要があることがよくあります。例えば、ユーザープロファイルページではユーザーIDに基づいてユーザーオブジェクトを取得する必要があるかもしれません。また、ユーザーがボタンを押してコメントを「承認」したい場合、このアクションは`is_approved`プロパティを更新し、1クリックで更新されたレコードを保存する必要があります。

あなたの`dataProvider`にはカスタムメソッドが含まれているかもしれません。例えば、APIのRPCエンドポイントを呼び出すためのメソッドです。`useQuery`と`useMutation`はこれらのメソッドを呼び出すために特に役立ちます。

例えば、あなたの`dataProvider`が`banUser()`メソッドを公開している場合:

```js
const dataProvider = {
    getList: /* ... */,
    getOne: /* ... */,
    getMany: /* ... */,
    getManyReference: /* ... */,
    create: /* ... */,
    update: /* ... */,
    updateMany: /* ... */,
    delete: /* ... */,
    deleteMany: /* ... */,
    banUser: (userId) => {
        return fetch(`/api/user/${userId}/ban`, { method: 'POST' })
            .then(response => response.json());
    },
}
```

次のように`<BanUser>`ボタンコンポーネント内で呼び出すことができます:

```jsx
const BanUserButton = ({ userId }) => {
    const dataProvider = useDataProvider();
    const { mutate, isLoading } = useMutation(
        () => dataProvider.banUser(userId)
    );
    return <Button label="Ban" onClick={() => mutate()} disabled={isLoading} />;
};
```

## クエリオプション

データプロバイダーメソッドフック（`useGetOne`など）およびreact-queryのフック（`useQuery`など）は、最後の引数としてクエリオプションオブジェクトを受け取ります。このオブジェクトは、クエリの実行方法を変更するために使用できます。多くのオプションがあり、すべて[react-queryのドキュメント](https://tanstack.com/query/v3/docs/react/reference/useQuery)に記載されています:

* `cacheTime`
* `enabled`
* `initialData`
* `initialDataUpdatedA`
* `isDataEqual`
* `keepPreviousData`
* `meta`
* `notifyOnChangeProps`
* `notifyOnChangePropsExclusions`
* `onError`
* `onSettled`
* `onSuccess`
* `queryKeyHashFn`
* `refetchInterval`
* `refetchIntervalInBackground`
* `refetchOnMount`
* `refetchOnReconnect`
* `refetchOnWindowFocus`
* `retry`
* `retryOnMount`
* `retryDelay`
* `select`
* `staleTime`
* `structuralSharing`
* `suspense`
* `useErrorBoundary`

例えば、クエリが完了したときにコールバックを実行したい場合（成功したか失敗したかに関わらず）、`onSettled`オプションを使用できます。これは例えば、dataProviderのすべての呼び出しをログに記録するのに役立ちます:

```jsx
import { useGetOne, useRecordContext } from 'react-admin';

const UserProfile = () => {
    const record = useRecordContext();
    const { data, isLoading, error } = useGetOne(
        'users',
        { id: record.id },
        { onSettled: (data, error) => console.log(data, error) }
    );
    if (isLoading) { return <Loading />; }
    if (error) { return <p>ERROR</p>; }
    return <div>User {data.username}</div>;
};
```

これらのオプションをすべて再説明することはしませんが、react-adminで最も有用なものに焦点を当てます。

**ヒント**: data providerメソッドフックを使用するreact-adminコンポーネントでは、`queryOptions`プロップを使用してクエリオプションを上書きし、`mutationOptions`プロップを使用してミューテーションオプションを上書きすることができます。例えば、`<List>`コンポーネントでdataProvider呼び出しをログに記録するには、次のようにします:

{% raw %}

```jsx
import { List, Datagrid, TextField } from 'react-admin';

const PostList = () => (
    <List
        queryOptions={{ onSettled: (data, error) => console.log(data, error) }}
    >
        <Datagrid>
            <TextField source="id" />
            <TextField source="title" />
            <TextField source="body" />
        </Datagrid>
    </List>
);
```

{% endraw %}

## 依存クエリの同期

すべてのData Providerフックは`enabled`オプションをサポートしています。これは、条件が満たされたときにのみクエリを実行する必要がある場合に便利です。

例えば、次のコードは、少なくとも1つのポストがすでにロードされている場合にのみカテゴリをフェッチします:

```jsx
// ポストをフェッチ
const { data: posts, isLoading } = useGetList(
    'posts',
    { pagination: { page: 1, perPage: 20 }, sort: { field: 'name', order: 'ASC' } },
);

// その後、これらのポストのカテゴリをフェッチ
const { data: categories, isLoading: isLoadingCategories } = useGetMany(
    'categories',
    { ids: posts.map(post => posts.category_id) },
    // 最初のクエリが空でない結果を返した場合にのみ実行
    { enabled: !isLoading && posts.length > 0 }
);
```

## 成功とエラーの副作用

クエリやミューテーションが完了した後にロジックを実行するには、`onSuccess`と`onError`オプションを使用します。React-adminはこのタイプのロジックを「副作用」と呼び、通常はUIの他の部分を変更します。

これは、ミューテーションフック（例: `useUpdate`）を使用する際に非常に一般的であり、通知を表示したり、別のページにリダイレクトするために使用されます。例えば、次の`<ApproveButton>`は、成功または失敗をユーザーに通知するために下部の通知バナーを使用します:

```jsx
import * as React from 'react';
import { useUpdate, useNotify, useRedirect, useRecordContext, Button } from 'react-admin';

const ApproveButton = () => {
    const record = useRecordContext();
    const notify = useNotify();
    const redirect = useRedirect();
    const [approve, { isLoading }] = useUpdate(
        'comments',
        { id: record.id, data: { isApproved: true } },
        {
            onSuccess: (data) => {
                // 成功時の副作用をここに記述
                redirect('/comments');
                notify('Comment approved');
            },
            onError: (error) => {
                // 失敗時の副作用をここに記述
                notify(`Comment approval error: ${error.message}`, { type: 'error' });
            },
        }
    );
    
    return <Button label="Approve" onClick={() => approve()} disabled={isLoading} />;
};
```

React-adminは、最も一般的な副作用を処理するために次のフックを提供します:

* [`useNotify`](./useNotify.md): 通知を表示する関数を返します。
* [`useRedirect`](./useRedirect.md): ユーザーを別のページにリダイレクトする関数を返します。
* [`useRefresh`](./useRefresh.md): 現在のビューを強制的に再レンダリングする関数を返します（リフレッシュボタンを押すのと同等）。
* [`useUnselect`](./useUnselect.md): 渡されたIDに基づいて現在の`Datagrid`の行を選択解除する関数を返します。
* [`useUnselectAll`](./useUnselectAll.md): 現在の`Datagrid`のすべての行を選択解除する関数を返します。

## 楽観的レンダリングとアンドゥ

次の例では、"Approve"ボタンをクリックした後、データプロバイダーがフェッチされている間にローディングスピナーが表示されます。その後、ユーザーはコメントリストにリダイレクトされます。

```jsx
import * as React from 'react';
import { useUpdate, useNotify, useRedirect, useRecordContext, Button } from 'react-admin';

const ApproveButton = () => {
    const record = useRecordContext();
    const notify = useNotify();
    const redirect = useRedirect();
    const [approve, { isLoading }] = useUpdate(
        'comments',
        { id: record.id, data: { isApproved: true } },
        {
            onSuccess: (data) => {
                redirect('/comments');
                notify('Comment approved');
            },
            onError: (error) => {
                notify(`Comment approval error: ${error.message}`, { type: 'error' });
            },
        }
    );
    
    return <Button label="Approve" onClick={() => approve()} disabled={isLoading} />;
};
```

しかし、ほとんどの場合、サーバーは成功応答を返すため、ユーザーはこの応答を待つ必要がありません。

これは**悲観的レンダリング**と呼ばれ、すべてのユーザーがサーバーの障害の可能性があるために待たされることになります。

ミューテーションのもう一つのモードは**楽観的レンダリング**です。このアイデアは、クライアント側で最初に`dataProvider`への呼び出しを処理し（つまり、react-queryキャッシュ内のエンティティを更新）、画面を即座に再レンダリングすることです。ユーザーは遅延なしでアクションの効果を見ることができます。その後、react-adminは成功の副作用を適用し、その後のみ`dataProvider`への呼び出しをトリガーします。フェッチが成功すると、react-adminはサーバーから最新のデータを取得するためにリフレッシュするだけです。ほとんどの場合、ユーザーは違いを感じません（react-queryキャッシュ内のデータと`dataProvider`からのデータは同じです）。フェッチが失敗すると、react-adminはエラー通知を表示し、ミューテーションを元に戻します。

三つ目のミューテーションモードは**アンドゥ可能**です。これは楽観的レンダリングと似ていますが、追加の機能があります。ローカルで変更と副作用を適用した後、react-adminは`dataProvider`への呼び出しをトリガーする前に数秒間待ちます。この遅延中、エンドユーザーはアクションをキャンセルするための「アンドゥ」ボタンを見ることができ、クリックすると`dataProvider`への呼び出しがキャンセルされ、画面がリフレッシュされます。

以下は三つのミューテーションモードのクイックリキャップです:

|悲観的|楽観的|アンドゥ可能|
|---|---|---|---|
|dataProvider呼び出し|即座に|即座に|遅延|
|ローカル変更|dataProviderが返るとき|即座に|即座に|
|副作用|dataProviderが返るとき|即座に|即座に|
|キャンセル可能|いいえ|いいえ|はい|

デフォルトでは、react-adminはEditビューに`undoable`モードを使用します。データプロバイダーメソッドフックの場合、デフォルトは`悲観的`モードです。

**ヒント**: Createビューでは、リソースのIDを知ってリダイレクトする必要があるため、ミューテーションモードは悲観的です。

`useUpdate`フックを呼び出すときにも楽観的およびアンドゥ可能なモードを活用できます。`mutationMode`オプションを渡すだけです:

```diff
import * as React from 'react';
import { useUpdate, useNotify, useRedirect, useRecordContext, Button } from 'react-admin';

const ApproveButton = () => {
    const record = useRecordContext();
    const notify = useNotify();
    const redirect = useRedirect();
    const [approve, { isLoading }] = useUpdate(
        'comments',
        { id: record.id, data: { isApproved: true } }
        {
+           mutationMode: 'undoable',
-           onSuccess: (data) => {
+           onSuccess: () => {
                redirect('/comments');
-               notify('Comment approved');
+               notify('Comment approved', { undoable: true });
            },
            onError: (error) => notify(`Error: ${error.message}`, { type: 'error' }),
        }
    );
    return <Button label="Approve" onClick={() => approve()} disabled={isLoading} />;
};
```

この例に示すように、アンドゥ可能な呼び出しの場合、通知を微調整する必要があります。`undo: true`を渡すと、通知に「アンドゥ」ボタンが表示されます。また、副作用は即座に実行されるため、成功応答に依存することはできません。

次のフックは`mutationMode`オプションを受け入れます:

* [`useUpdate`](./useUpdate.md)
* [`useUpdateMany`](./useUpdateMany.md)
* [`useDelete`](./useDelete.md)
* [`useDeleteMany`](./useDeleteMany.md)

## `fetch`を使用したAPIクエリの実行

データプロバイダーメソッドフックはAPIにクエリを実行するための「react-adminの方法」です。しかし、必要に応じて`fetch`を使用することも可能です。例えば、APIのRPCメソッドに対してルーティングロジックを追加したくない場合は、その方が理にかなっています。

その場合、特別なreact-adminのソースはありません。コンポーネント内で`fetch`を呼び出す例の実装を示します:

```jsx
// src/comments/ApproveButton.js
import * as React from 'react';
import { useState } from 'react';
import { useNotify, useRedirect, useRecordContext, Button } from 'react-admin';

const ApproveButton = () => {
    const record = useRecordContext();
    const redirect = useRedirect();
    const notify = useNotify();
    const [loading, setLoading] = useState(false);
    const handleClick = () => {
        setLoading(true);
        const updatedRecord = { ...record, is_approved: true };
        fetch(`/comments/${record.id}`, { method: 'PUT', body: updatedRecord })
            .then(() => {
                notify('Comment approved');
                redirect('/comments');
            })
            .catch((e) => {
                notify('Error: comment not approved', { type: 'error' })
            })
            .finally(() => {
                setLoading(false);
            });
    };
    return <Button label="Approve" onClick={handleClick} disabled={loading} />;
};

export default ApproveButton;
```

**ヒント**: APIは認証、クエリパラメータ、エンコーディング、ヘッダーなどに対処するためのHTTPのプラミングを必要とすることがよくあります。RESTリクエストをHTTPリクエストにマッピングする関数をすでに持っている可能性があります。それがあなたの[Data Provider](./DataProviders.md)です。そのため、`fetch`の代わりに`useDataProvider`を使用する方が良いことがよくあります。


