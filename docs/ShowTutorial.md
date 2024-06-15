---
layout: default
title: "The Show Page"
---

# The Show Page

Showビューは単一のレコードの詳細を表示します。

![post show view](./img/show-view.png)

## 純粋なReactからReact-Adminへ

Showビューは管理画面の中で最もシンプルなビューです。単一のレコードのフィールドを取得して表示します。これまでにこうしたページを何度も開発してきたかもしれませんが、データ取得ロジックと表示コードを書くのは長くて退屈な作業であり、ビジネスロジックが隠れてしまうことがあります。だからこそ、こんなシンプルなページでもreact-adminは大いに役立ちます。

react-adminのShowビュー専用フックやコンポーネントの使い方をより理解するために、まずは手動でShowビューを構築するところから始めましょう。

### 手動で構築するShowビュー

以下に、react-adminの[データ取得フック](./DataProviders.md)を活用して、シンプルな本のShowビューを書く方法を示します：

```jsx
    import { useParams } from 'react-router-dom';
    import { useGetOne, useRedirect, Title } from 'react-admin';
    import { Card, Stack, Typography } from '@mui/material';

    /**
     * APIから本を取得して表示する
     */
    const BookShow = () => {
        const { id } = useParams(); // このコンポーネントは /books/:id パスでレンダリングされます
        const redirect = useRedirect();
        const { data, isLoading } = useGetOne(
            'books',
            { id },
            // 本が見つからない場合はリストにリダイレクト
            { onError: () => redirect('/books') }
        );
        if (isLoading) { return <Loading />; }
        return (
            <div>
                <Title title="Book Show"/>
                <Card>
                    <Stack spacing={1}>
                        <div>
                            <Typography variant="caption" display="block">タイトル</Typography>
                            <Typography variant="body2">{data.title}</Typography>
                        </div>
                        <div>
                            <Typography variant="caption" display="block">出版日</Typography>
                            <Typography variant="body2">{new Date(data.published_at).toDateString()}</Typography>
                        </div>
                    </Stack>
                </Card>
            </div>
        );
    };
```

この `BookShow` コンポーネントを `<Resource name="books" />` の `show` プロップとして渡すと、react-adminはこれを `/books/:id/show` パスにレンダリングします。

この例では、認証とリクエスト状態ロジックを既に含んでいる `useGetOne` フックを使用していますが、`fetch` を使ってShowビューを書くことも可能です。

### `<Labeled>`でフィールドにラベルを表示

上記のようなShowビューを構築する際、各フィールドごとにかなりのコードを繰り返す必要があります。React-adminのFieldコンポーネントを利用することで、この繰り返しを避けることができます。次の例では、その目的で `<Labeled>`, `<TextField>`, および `<DateField>` コンポーネントを活用しています：

```diff
import { useParams } from 'react-router-dom';
-import { useGetOne, useRedirect, Title } from 'react-admin';
+import { useGetOne, useRedirect, Title, Labeled, TextField, DateField } from 'react-admin';
-import { Card, Stack, Typography } from '@mui/material';
+import { Card, Stack } from '@mui/material';

const BookShow = () => {
    const { id } = useParams();
    const redirect = useRedirect();
    const { data, isLoading } = useGetOne(
        'books',
        { id },
        { onError: () => redirect('/books') }
    );
    if (isLoading) { return <Loading />; }
    return (
        <div>
            <Title title="Book Show"/>
            <Card>
                <Stack spacing={1}>
-                   <div>
-                       <Typography variant="caption" display="block">タイトル</Typography>
-                       <Typography variant="body2">{data.title}</Typography>
-                   </div>
+                   <Labeled label="タイトル">
+                       <TextField source="title" record={data} />
+                   </Labeled>
-                   <div>
-                       <Typography variant="caption" display="block">出版日</Typography>
-                       <Typography variant="body2">{new Date(data.published_at).toDateString()}</Typography>
-                   </div>
+                   <Labeled label="出版日">
+                       <DateField source="published_at" record={data} />
+                   </Labeled>
                </Stack>
            </Card>
        </div>
    );
};
```

### `<RecordContext>` で `record` を公開

フィールドコンポーネントはレンダリングのために `record` が必要ですが、`record` プロップの代わりに `RecordContext` から取得することができます。 `<RecordContextProvider>` を使用してこのようなコンテキストを作成することで、各フィールドに必要なコードの量をさらに減らすことができます。

```diff
import { useParams } from 'react-router-dom';
-import { useGetOne, useRedirect, Title, Labeled, TextField, DateField } from 'react-admin';
+import { useGetOne, useRedirect, RecordContextProvider, Title, Labeled, TextField, DateField } from 'react-admin';
import { Card, Stack } from '@mui/material';

const BookShow = () => {
    const { id } = useParams();
    const redirect = useRedirect();
    const { data, isLoading } = useGetOne(
        'books',
        { id },
        { onError: () => redirect('/books') }
    );
    if (isLoading) { return <Loading />; }
    return (
+       <RecordContextProvider value={data}>
            <div>
                <Title title="Book Show"/>
                <Card>
                    <Stack spacing={1}>
                        <Labeled label="タイトル">
-                           <TextField source="title" record={data} />
+                           <TextField source="title" />
                        </Labeled>
                        <Labeled label="出版日">
-                           <DateField source="published_at" record={data} />
+                           <DateField source="published_at" />
                        </Labeled>
                    </Stack>
                </Card>
            </div>
+       </RecordContextProvider>
    );
};
```

### `<SimpleShowLayout>` でフィールドをスタックに表示

ラベル付きのフィールドをスタック表示することは非常に一般的なタスクであるため、react-adminはそのためのヘルパーコンポーネントを提供しています。それが[`<SimpleShowLayout>`](./SimpleShowLayout.md)です：

```diff
import { useParams } from 'react-router-dom';
-import { useGetOne, useRedirect, RecordContextProvider, Title, Labeled, TextField, DateField } from 'react-admin';
+import { useGetOne, useRedirect, RecordContextProvider, SimpleShowLayout, Title, TextField, DateField } from 'react-admin';
-import { Card, Stack } from '@mui/material';
+import { Card } from '@mui/material';

const BookShow = () => {
    const { id } = useParams();
    const redirect = useRedirect();
    const { data, isLoading } = useGetOne(
        'books',
        { id },
        { onError: () => redirect('/books') }
    );
    return (
        <RecordContextProvider value={data}>
            <div>
                <Title title="Book Show" />
                <Card>
-                   <Stack spacing={1}>
+                   <SimpleShowLayout>
-                       <Labeled label="タイトル">
                            <TextField label="タイトル" source="title" />
-                       </Labeled>
-                       <Labeled label="出版日">
                            <DateField label="出版日" source="published_at" />
-                       </Labeled>
+                   </SimpleShowLayout>
-                   </Stack>
                </Card>
            </div>
        </RecordContextProvider>
    );
};
```

`<SimpleShowLayout>` は `data` が読み込まれていない間は何もレンダリングしないので、`isLoading` 変数はもう必要ありません。

### `useShowController`: コントローラロジック

ロケーションからIDを取得し、APIからレコードを取得する初期ロジックも一般的で、react-adminはこれを行うための[`useShowController`フック](./useShowController.md)を公開しています：

```diff
-import { useParams } from 'react-router-dom';
-import { useGetOne, useRedirect, RecordContextProvider, SimpleShowLayout, Title, TextField, DateField } from 'react-admin';
+import { useShowController, RecordContextProvider, SimpleShowLayout, Title, TextField, DateField } from 'react-admin';
import { Card } from '@mui/material';

const BookShow = () => {
-   const { id } = useParams();
-   const redirect = useRedirect();
-   const { data, isLoading } = useGetOne(
-       'books',
-       { id },
-       { onError: () => redirect('/books') }
-   );
+   const { data } = useShowController();
    return (
        <RecordContextProvider value={data}>
            <div>
                <Title title="Book Show" />
                <Card>
                    <SimpleShowLayout>
                        <TextField label="タイトル" source="title" />
                        <DateField label="出版日" source="published_at" />
                    </SimpleShowLayout>
                </Card>
            </div>
        </RecordContextProvider>
    );
};
```

`useShowController` は 'books' リソース名を必要としません。これは、`<Resource>` コンポーネントによって設定された `ResourceContext` に依存してそれを推測するためです。

### `<ShowBase>`: コントローラのコンポーネント版

Showコントローラを呼び出してその結果をコンテキストに入れることも一般的であるため、react-adminはこれを行うための[`<ShowBase>`コンポーネント](./ShowBase.md)を提供しています。このため、例をさらに以下のように簡素化できます：

```diff
-import { useShowController, RecordContextProvider, SimpleShowLayout, Title, TextField, DateField } from 'react-admin';
+import { ShowBase, SimpleShowLayout, Title, TextField, DateField } from 'react-admin';
import { Card } from '@mui/material;

const BookShow = () => {
-   const { data } = useShowController();
    return (
-       <RecordContextProvider value={data}>
+       <ShowBase>
            <div>
                <Title title="Book Show" />
                <Card>
                    <SimpleShowLayout>
                        <TextField label="タイトル" source="title" />
                        <DateField label="出版日" source="published_at" />
                    </SimpleShowLayout>
                </Card>
            </div>
+       </ShowBase>
-      </RecordContextProvider>
    );
};
```

### `<Show>`: タイトル、フィールド、アクションをレンダリング

`<ShowBase>` はヘッドレスコンポーネントで、子要素のみをレンダリングします。しかし、ほとんどのShowビューにはラッピングする `<div>`、リソース名から構築されたタイトル、そして `<Card>` が必要です。そのためreact-adminは、`<ShowBase>` コンポーネント、リソース名から構築されたタイトル、そしてリソースに編集コンポーネントがある場合には「編集」ボタンさえも含む[`<Show>`コンポーネント](./Show.md)を提供しています：

```diff
-import { ShowBase, SimpleShowLayout, Title, TextField, DateField } from 'react-admin';
+import { Show, SimpleShowLayout, TextField, DateField } from 'react-admin';
-import { Card } from '@mui/material';

const BookShow = () => (
-   <ShowBase>
-       <div>
-           <Title title="Book Show" />
-           <Card>
+   <Show>
        <SimpleShowLayout>
            <TextField label="タイトル" source="title" />
            <DateField label="出版日" source="published_at" />
        </SimpleShowLayout>
+   </Show>
-           </Card>
-       </div>
-   </ShowBase>
);
```

**Tip**: 実際には、`<Show>` は前の例のコードが置き換えるもの以上のことを行います。`useGetOne` の呼び出しがエラーを返した場合にリストビューにリダイレクトし、ページタイトルを設定し、準備したすべてのデータを `<ShowContext>` に保存します。

**Tip**: `RecordContext`（例：`{ id: '1', title: '指輪物語' }`）と`<ResourceContext>`（例：`'book'`）を混同しないでください。

### 典型的なReact-AdminのShowビュー

今やコードはビジネスロジックのみを表現しています。React-adminを使用すると、React単独で必要だった26行のコードが、わずか6行で済みます：

```jsx
import { Show, SimpleShowLayout, TextField, DateField } from 'react-admin';

const BookShow = () => (
    <Show>
        <SimpleShowLayout>
            <TextField label="タイトル" source="title" />
            <DateField label="出版日" source="published_at" />
        </SimpleShowLayout>
    </Show>
);
```

React-adminコンポーネントは魔法ではなく、ビジネスロジックに集中し、繰り返しの作業を避けるために設計されたReactコンポーネントです。

## レコードにアクセスする

`useGetOne` を手動で呼び出す代わりに `<Show>` コンポーネントを使用することには、一つの欠点があります。それは、取得されたレコードを含む `data` オブジェクトがなくなることです。代わりに、`<RecordContext>` からレコードにアクセスする必要があります。[`useRecordContext` フック](./useRecordContext.md)を使用してこれを行います。

次の例は、本の評価に応じて星を表示するカスタムフィールドコンポーネントを使用する場合の、このフックの使い方を示しています：

```jsx
import { Show, SimpleShowLayout, TextField, DateField, useRecordContext } from 'react-admin';
import StarIcon from '@mui/icons-material/Star';

const NbStarsField = () => {
    const record = useRecordContext();
    return <>
        {[...Array(record.rating)].map((_, index) => <StarIcon key={index} />)}
    </>;
};

const BookShow = () => (
    <Show>
        <SimpleShowLayout>
            <TextField label="タイトル" source="title" />
            <DateField label="出版日" source="published_at" />
            <NbStarsField label="評価" />
        </SimpleShowLayout>
    </Show>
);
```

新しいコンポーネントを作成せずに `useRecordContext` フックを使用したい場合もあります。そのような場合、フックのレンダープロップバージョンである[`<WithRecord>`コンポーネント](./WithRecord.md)を使用できます：

```jsx
import { Show, SimpleShowLayout, TextField, DateField, WithRecord } from 'react-admin';
import StarIcon from '@mui/icons-material/Star';

const BookShow = () => (
    <Show>
        <SimpleShowLayout>
            <TextField label="タイトル" source="title" />
            <DateField label="出版日" source="published_at" />
            <WithRecord label="評価" render={record => <>
                {[...Array(record.rating)].map((_, index) => <StarIcon key={index} />)}
            </>} />
        </SimpleShowLayout>
    </Show>
);
```

## 別のレイアウトを使用する

Showビューが多数のフィールドを表示する必要がある場合、`<SimpleShowLayout>` コンポーネントは非常に長いページになり、ユーザーフレンドリーではありません。代わりに、フィールドをタブで表示する `<TabbedShowLayout>` コンポーネントを使用できます：

```jsx
import { Show, TabbedShowLayout, TextField, DateField, WithRecord } from 'react-admin';
import StarIcon from '@mui/icons-material/Star';
import FavoriteIcon from '@mui/icons-material/Favorite';
import PersonPinIcon from '@mui/icons-material/PersonPin';

const BookShow = () => (
    <Show>
        <TabbedShowLayout>
            <TabbedShowLayout.Tab label="説明" icon={<FavoriteIcon />}>
                <TextField label="タイトル" source="title" />
                <ReferenceField label="著者" source="author_id">
                    <TextField source="name" />
                </ReferenceField>
                <DateField label="出版日" source="published_at" />
            </TabbedShowLayout.Tab>
            <TabbedShowLayout.Tab label="ユーザー評価" icon={<PersonPinIcon />}>
                <WithRecord label="評価" render={record => <>
                    {[...Array(record.rating)].map((_, index) => <StarIcon key={index} />)}
                </>} />
                <DateField label="最終評価日" source="last_rated_at" />
            </TabbedShowLayout.Tab>
        </TabbedShowLayout>
    </Show>
);
```

## カスタムレイアウトの構築

多くの場合、`<SimpleShowLayout>` や `<TabbedShowLayout>` コンポーネントだけでは表示したいフィールドを十分に表示できないことがあります。そのような場合は、レイアウトコンポーネントを `<Show>` コンポーネントの子要素として直接渡します。 `<Show>` はレコードを取得して `<RecordContextProvider>` に入れるため、フィールドコンポーネントを直接使用できます。

たとえば、複数のフィールドを1行に表示するには、Material UIの `<Grid>` コンポーネントを使用できます：

{% raw %}

```jsx
import { Show, TextField, DateField, ReferenceField, WithRecord } from 'react-admin';
import { Grid } from '@mui/material';
import StarIcon from '@mui/icons-material/Star';

const BookShow = () => (
    <Show emptyWhileLoading>
        <Grid container spacing={2} sx={{ margin: 2 }}>
            <Grid item xs={12} sm={6}>
                <TextField label="タイトル" source="title" />
            </Grid>
            <Grid item xs={12} sm={6}>
                <ReferenceField label="著者" source="author_id" reference="authors">
                    <TextField source="name" />
                </ReferenceField>
            </Grid>
            <Grid item xs={12} sm={6}>
                <DateField label="出版日" source="published_at" />
            </Grid>
            <Grid item xs={12} sm={6}>
                <WithRecord label="評価" render={record => <>
                    {[...Array(record.rating)].map((_, index) => <StarIcon key={index} />)}
                </>} />
            </Grid>
        </Grid>
    </Show>
);
```

{% endraw %}

**Tip**: `emptyWhileLoading` をオンにすると、`<Show>` コンポーネントはレコードが利用可能になるまで子コンポーネントをレンダリングしません。このフラグがないと、フィールドコンポーネントはローディングフェーズ中にもレンダリングされ、空のレコードコンテキストで動作するように計画されていない場合は壊れる可能性があります。代わりに `ShowContext` から `isLoading` 状態を取得することもできますが、それでは `<BookShow>` コンポーネントを二つに分割する必要があります。

リストのフィールドを二つのスタックに分割し、メインパネルに `<SimpleShowLayout>` を使用することもできます：

{% raw %}

```jsx
import { Show, SimpleShowLayout, TextField, DateField, WithRecord } from 'react-admin';
import StarIcon from '@mui/icons-material/Star';

const BookShow = () => (
    <Show emptyWhileLoading>
        <Grid container spacing={2} sx={{ margin: 2 }}>
            <Grid item xs={12} sm={8}>
                <SimpleShowLayout>
                    <TextField label="タイトル" source="title" />
                    <DateField label="出版日" source="published_at" />
                    <WithRecord label="評価" render={record => <>
                        {[...Array(record.rating)].map((_, index) => <StarIcon key={index} />)}
                    </>} />
                </SimpleShowLayout>
            </Grid>
            <Grid item xs={12} sm={4}>
                <Typography>詳細</Typography>
                <Stack spacing={1}>
                    <Labeled label="ISBN"><TextField source="isbn" /></Labeled>
                    <Labeled label="最終評価日"><DateField source="last_rated_at" /></Labeled>
                </Stack>
            </Grid>
        </Grid>
    </Show>
);
```

{% endraw %}

## サードパーティコンポーネントの使用

react-adminのコンポーネントはサードパーティリポジトリで見つけることができます。

* [ra-compact-ui](https://github.com/ValentinnDimitroff/ra-compact-ui#layouts): カスタムスタイルのShowレイアウトを持つことができるプラグイン。


