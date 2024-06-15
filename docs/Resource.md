---
layout: default
title: "リソースコンポーネント"
---

# `<Resource>`コンポーネント

`<Resource>`コンポーネントは、react-adminアプリケーションのCRUDルートを定義します。

react-adminの用語では、*resource*はエンティティタイプ（例えば「products」、「subscribers」、「tags」など）を指す文字列です。*records*は`id`フィールドを持つオブジェクトで、同じ*resource*に属する2つのrecordは同じフィールド構造を持ちます（例えば、全てのpostレコードはタイトル、公開日などを持つ）。

`<Resource>`コンポーネントには3つの責任があります：

- 指定されたリソースのCRUDルートを定義します（レコードのリストを表示、レコードの詳細を表示、新しいレコードを作成）。
- すべての子孫コンポーネントが現在のリソース名を知ることができるコンテキストを作成します（このコンテキストは`ResourceContext`と呼ばれます）。
- リソース定義（名前、アイコン、ラベル）を共有コンテキストに保存します（このコンテキストは`ResourceDefinitionContext`と呼ばれます）。

`<Resource>`コンポーネントは[<Admin>コンポーネント](./Admin.md)の子としてのみ使用できます。

## 使用方法

例えば、以下のadminアプリケーションはJSONPlaceholder APIが提供するリソース（[posts](https://jsonplaceholder.typicode.com/posts)、[users](https://jsonplaceholder.typicode.com/users)、[comments](https://jsonplaceholder.typicode.com/comments)、および[tags](https://jsonplaceholder.typicode.com/tags)）に対するインターフェースを提供します：

```jsx
    import * as React from "react";
    import { Admin, Resource } from 'react-admin';
    import jsonServerProvider from 'ra-data-json-server';

    import { PostList, PostCreate, PostEdit, PostShow, PostIcon } from './posts';
    import { UserList } from './posts';
    import { CommentList, CommentEdit, CommentCreate, CommentIcon } from './comments';

    const App = () => (
        <Admin dataProvider={jsonServerProvider('https://jsonplaceholder.typicode.com')}>
            {/* postsの完全なCRUDページ */}
            <Resource name="posts" list={PostList} create={PostCreate} edit={PostEdit} show={PostShow} />
            {/* 読み取り専用のuserリスト */}
            <Resource name="users" list={UserList} />
            {/* commentsリソースのshowページはなし */}
            <Resource name="comments" list={CommentList} create={CommentCreate} edit={CommentEdit} icon={CommentIcon} />
        </Admin>
    );
```

これらのルートは以下の`dataProvider`メソッドを呼び出します：

* `list`はマウント時に`getList()`を呼び出します
* `show`はマウント時に`getOne()`を呼び出します
* `edit`はマウント時に`getOne()`を呼び出し、送信時に`update()`または`delete()`を呼び出します
* `create`は送信時に`create()`を呼び出します

**ヒント**: リソースがどのAPIエンドポイントに依存しているかは、`<Resource>`コンポーネントには分かりません。それは[`dataProvider`](./DataProviders.md)の役割です。

## `name`

`name`は`<Resource>`に必要な唯一のpropです。React-adminは`name` propを使用してAPIエンドポイント（`dataProvider`に渡される）を決定し、リソースのURLを形成します。

```jsx
<Resource name="posts" list={PostList} create={PostCreate} edit={PostEdit} show={PostShow} />
```

このリソースの場合、react-adminはデータを取得するために`https://jsonplaceholder.typicode.com/posts`エンドポイントにアクセスします。

ルーティングは以下のようにコンポーネントにマップされます：

* `/posts/`は`PostList`にマップ
* `/posts/create`は`PostCreate`にマップ
* `/posts/:id`は`PostEdit`にマップ
* `/posts/:id/show`は`PostShow`にマップ

**ヒント**: 特殊なAPIエンドポイント（例：'[https://jsonplaceholder.typicode.com/my-custom-posts-endpoint'）を使用しつつ、react-adminアプリケーション内のURL（例：\`/posts\`）を変更しない場合、リソース\`name\`（\`posts\`）とAPIエンドポイント（\`my-custom-posts-endpoint\`）のマッピングを独自の\[\`dataProvider\`]()\](./Admin.md#dataprovider)に記述します。

## `list`, `create`, `edit`, `show`

`<Resource>`は各CRUD操作のためのコンポーネントを以下のprop名で定義できます：

* `list`（通常[<List>コンポーネント](./List.md)を使用）（定義されている場合、リソースはメニューに表示されます）
* `create`（通常[<Create>コンポーネント](./Create.md)を使用）
* `edit`（通常[<Edit>コンポーネント](./Edit.md)を使用）
* `show`（通常[<Show>コンポーネント](./Show.md)を使用）

**ヒント**: 内部的には、`<Resource>`コンポーネントは[react-router](https://reactrouter.com/web/guides/quick-start)を使用して複数のルートを作成します：

* `/`は`list`コンポーネントにマップ
* `/create`は`create`コンポーネントにマップ
* `/:id`は`edit`コンポーネントにマップ
* `/:id/show`は`show`コンポーネントにマップ

`<Resource>`は追加のpropsも受け付けます：

* [`name`](#name)
* [`icon`](#icon)
* [`options`](#options)
* [`recordRepresentation`](#recordrepresentation)

## `children`

`<Resource>`はアプリケーションのCRUDルートを定義します。したがって、`<Resource name="posts">`は`/posts`から始まる一連のルートを定義します。

`<Resource>`は`children`として`<Route>`コンポーネントを受け入れ、リソースのサブルートを定義できます。

例えば、以下のコードは`authors`リソースを作成し、指定された著者の本を表示する`/authors/:authorId/books`ルートを追加します：

```jsx
// src/App.jsx内
import { Admin, Resource } from 'react-admin';
import { Route } from 'react-router-dom';

import { AuthorList } from './AuthorList';
import { BookList } from './BookList';

export const App = () => (
    <Admin dataProvider={dataProvider}>
        <Resource name="authors" list={AuthorList}>
            <Route path=":authorId/books" element={<BookList />} />
        </Resource>
    </Admin>
);
```

`BookList`コンポーネントは、`useParams`フックを使用してURLから`authorId`パラメータを取得し、指定された著者の本のリストを表示するために`<List filter>`パラメータとして渡すことができます：

{% raw %}

```jsx
// src/BookList.jsx内
import { List, Datagrid, TextField } from 'react-admin';
import { useParams } from 'react-router-dom';

export const BookList = () => {
    const { authorId } = useParams();
    return (
        <List resource="books" filter={{ authorId }}>
            <Datagrid rowClick="edit">
                <TextField source="id" />
                <TextField source="title" />
                <TextField source="year" />
            </Datagrid>
        </List>
    );
};
```

{% endraw %}

**ヒント**: 上記の例では、`<List>`内で`resource="books"`プロパティが必要なのは、`<Resource name="authors">`内では`ResourceContext`が`authors`にデフォルト設定されているためです。

`/authors/:id/books`ルートへのルーティングは、例えば各`AuthorList`コンポーネントの行から行います：

```jsx
// src/AuthorList.jsx内
const BooksButton = () => {
    const record = useRecordContext();
    return (
        <Button
            component={Link}
            to={`/authors/${record.id}/books`}
            color="primary"
        >
            Books
        </Button>
    );
};

export const AuthorList = () => (
    <List>
        <Datagrid>
            <TextField source="id" />
            <TextField source="firstName" />
            <TextField source="lastName" />
            <BooksButton />
        </Datagrid>
    </List>
);
```

**ヒント**: `/authors/:authorId/books`ルートは`/authors`ルートのサブルートであるため、アクティブなメニューアイテムは「Authors」となります。

## `icon`

React-adminはメニューに`icon`プロパティのコンポーネントを表示します：

```jsx
// src/App.js内
import * as React from "react";
import PostIcon from '@mui/icons-material/Book';
import UserIcon from '@mui/icons-material/People';
import { Admin, Resource } from 'react-admin';
import jsonServerProvider from 'ra-data-json-server';

import { PostList } from './posts';

const App = () => (
    <Admin dataProvider={jsonServerProvider('https://jsonplaceholder.typicode.com')}>
        <Resource name="posts" list={PostList} icon={PostIcon} />
        <Resource name="users" list={UserList} icon={UserIcon} />
    </Admin>
);
```

## `options`

`options.label`は、メニュー内の特定のリソースの表示名をカスタマイズするために使用されます。

{% raw %}

```jsx
<Resource name="v2/posts" options={{ label: 'Posts' }} list={PostList} />
```

{% endraw %}

## `recordRepresentation`

react-adminがレコードをレンダリングする必要がある場合（例えば、編集ビューのタイトルや`<ReferenceField>`内など）、`recordRepresentation`を使用します。デフォルトでは、レコードの表現はその`id`フィールドです。しかし、希望する表現を指定してカスタマイズできます。

例えば、「users」レコードのデフォルト表現をidではなくフルネームに変更する場合：

```jsx
<Resource
    name="users"
    list={UserList}
    recordRepresentation={(record) => `${record.first_name} ${record.last_name}`}
/>
```

`recordRepresentation`は3種類の値を取ることができます：

* 文字列（例：`'title'`）で、表現に使用するフィールドを指定
* 関数（例：`(record) => record.title`）で、カスタム文字列表現を指定
* Reactコンポーネント（例：`<MyCustomRecordRepresentation />`）。このようなコンポーネントでは[`useRecordContext`](./useRecordContext.md)を使用してレコードにアクセスします。

このレコード表現をどこかで表示したい場合、[`useGetRecordRepresentation`](./useGetRecordRepresentation.md)フックや[`<RecordRepresentation>`](./RecordRepresentation.md)コンポーネントを活用できます。

## `hasCreate`, `hasEdit`, `hasShow`

一部のコンポーネント（例：[`<CreateDialog>`](./CreateDialog.md)、[`<EditDialog>`](./EditDialog.md)、[`<ShowDialog>`](./ShowDialog.md)）は、`<Resource>`コンポーネントの外でCRUDコンポーネントを宣言する必要があります。その場合、特定のリソースにどのCRUDコンポーネントが利用可能かをreact-adminに伝えるために`hasCreate`、`hasEdit`、`hasShow`プロパティを使用できます。

例えば、`<ReferenceField>`コンポーネントが参照されるレコードの編集または表示ビューへのリンクを表示するようにします。

```jsx
// src/App.js内
import { Admin, Resource } from 'react-admin';
import { dataProvider } from './dataProvider';

import { PostList } from './posts';
import { CommentEdit } from './commentEdit';

const App = () => (
    <Admin dataProvider={dataProvider}>
        <Resource name="posts" list={PostList} hasEdit />
        <Resource name="comment" edit={CommentEdit} />
    </Admin>
);

// src/commentEdit.js内
import { Edit, SimpleForm, ReferenceField } from 'react-admin';

const CommentEdit = () => (
    <Edit>
        <SimpleForm>
            {/* `hasEdit`が`<Resource>`で設定されているため、編集ビューへのリンクが表示されます */}
            <ReferenceField source="post_id" reference="posts" />
        </SimpleForm>
    </Edit>
);
```

## Resource Context

`<Resource>`はまた、現在のリソース名にアクセスできる`ResourceContext`を作成し、メインページコンポーネント（`list`、`create`、`edit`、`show`）のすべての子孫に提供します。

現在のリソース名を読み取るには、`useResourceContext()`フックを使用します。

例えば、以下のコンポーネントは現在のリソース名を表示します：

```jsx
import * as React from 'react';
import { Datagrid, DateField, TextField, List, useResourceContext } from 'react-admin';

const ResourceName = () => {
    const resource = useResourceContext();
    return <>{resource}</>;
}

const PostList = () => (
    <List>
        <>
            <ResourceName /> {/* 'posts'をレンダリング */}
            <Datagrid>
                <TextField source="title" />
                <DateField source="published_at" />
            </Datagrid>
        </>
    </List>
)
```

**ヒント**: 現在のリソースコンテキストを*変更*することができます。例えば、関連リソースのコンポーネントを使用する場合。これには、`<ResourceContextProvider>`コンポーネントを使用します：

```jsx
const MyComponent = () => (
    <ResourceContextProvider value="comments">
        <ResourceName /> {/* 'comments'をレンダリング */}
        ...
    </ResourceContextProvider>
);
```

## ネストされたリソース

React-adminはネストされたリソースをサポートしていませんが、[`children`プロパティ](#children)を使用して、特定のサブルート用のカスタムコンポーネントをレンダリングできます。例えば、特定のアーティストの曲のリストを表示するには：

```jsx
import { Admin, Resource } from 'react-admin';
import { Route } from 'react-router-dom';

export const App = () => (
    <Admin dataProvider={dataProvider}>
        <Resource name="artists" list={ArtistList} edit={ArtistDetail}>
            <Route path=":id/songs" element={<SongList />} />
            <Route path=":id/songs/:songId" element={<SongDetail />} />
        </Resource>
    </Admin>
);
```

この設定は4つのルートを作成します：

* `/artists`は`<ArtistList>`要素をレンダリング
* `/artists/:id`は`<ArtistDetail>`要素をレンダリング
* `/artists/:id/songs`は`<SongList>`要素をレンダリング
* `/artists/:id/songs/:songId`は`<SongDetail>`要素をレンダリング

選択されたアーティストの曲のリストを表示するために、`<SongList>`は`id`パラメータで曲をフィルタリングする必要があります。これを行うには、`react-router-dom`の`useParams`フックを使用します：

{% raw %}

```jsx
// src/SongList.jsx内
import { List, Datagrid, TextField, useRecordContext } from 'react-admin';
import { useParams } from 'react-router-dom';
import { Button } from '@mui/material';

export const SongList = () => {
    const { id } = useParams();
    return (
        <List resource="songs" filter={{ artistId: id }}>
            <Datagrid rowClick="edit">
                <TextField source="title" />
                <DateField source="released" />
                <TextField source="writer" />
                <TextField source="producer" />
                <TextField source="recordCompany" label="Label" />
                <EditSongButton />
            </Datagrid>
        </List>
    );
};

const EditSongButton = () => {
    const song = useRecordContext();
    return (
        <Button
            component={Link}
            to={`/artists/${song?.artist_id}/songs/${song?.id}`}
            startIcon={<EditIcon />}
        >
            Edit
        </Button>
    );
};
```

{% endraw %}

`<SongDetail>`コンポーネント内では、`useParams`フックを使用して`songId`パラメータを取得し、対応する`id`の曲を表示する必要があります：

{% raw %}

```jsx
// src/SongDetail.jsx内
import { Edit, SimpleForm, TextInput } from 'react-admin';
import { useParams } from 'react-router-dom';

export const SongDetail = () => {
    const { id, songId } = useParams();
    return (
        <Edit resource="posts" id={songId} redirect={`/artists/${id}/songs`}>
            <SimpleForm>
                <TextInput source="title" />
                <DateInput source="released" />
                <TextInput source="writer" />
                <TextInput source="producer" />
                <TextInput source="recordCompany" label="Label" />
            </SimpleForm>
        </Edit>
    );
};
```

{% endraw %}

**ヒント**: 上記のスクリーンキャストで示されているように、ネストされたリソースに移動するとき、ユーザーは画面にパンくずリストが表示されていないと迷子になりやすくなります。このナビゲーション要素の設定については、[<Breadcrumb>コンポーネント](./Breadcrumb.md#nested-resources)をご確認ください。

## レイジーローディング

アプリケーションの初期読み込みを高速化する必要がある場合、[`React.lazy()`](https://react.dev/reference/react/lazy#suspense-for-code-splitting)を使用してコード分割を有効にしたいかもしれません。デフォルトのreact-adminレイアウトはSuspenseを使用しているため、`<Resource>`内でレイジーローディングされたコンポーネントを使用するための特別な設定は必要ありません。

```jsx
// src/App.js内
import * as React from 'react';
import { Admin, Resource } from 'react-admin';

import { dataProvider } from './dataProvider';
import { users } from './users';

const PostList = React.lazy(() => import('./posts/PostList'));
const PostEdit = React.lazy(() => import('./posts/PostEdit'));

const App = () => (
    <Admin dataProvider={dataProvider}>
        <Resource name="users" {...users} />
        <Resource name="posts" list={PostList} edit={PostEdit} />
    </Admin>
);
```

ユーザーが`/posts`ルートに移動すると、react-adminは`PostList`コンポーネントの読み込み中にロードインジケーターを表示します。

![Loading indicator](./img/lazy-resource.png)

```

```

