---
layout: default
title: "リストのフィルタリング"
---

# リストのフィルタリング

リストページの最も重要な機能の一つは、結果をフィルタリングできることです。React-adminは強力なフィルタコンポーネントを提供しており、さらにカスタマイズしたい場合にも対応できます。

## 概要

<table><tbody>
<tr style="border:none">
<td style="width:50%;border:none;text-align:center">
            <a title="フィルターボタン/フォームコンボ" href="./img/list_filter.webm">
                <video controls autoplay playsinline muted loop>
                    <source src="./img/list_filter.webm" type="video/webm"/>
                    <source src="./img/list_filter.mp4" type="video/mp4"/>
                    Your browser does not support the video tag.
                </video>
            </a>
            <a href="#the-filter-buttonform-combo" style="display: block;transform: translateY(-10px);">フィルターボタン/フォームコンボ</a>
        </td>
        <td style="width:50%;border:none;text-align:center">
            <a title="<FilterList> サイドバー" href="./img/filter-sidebar.webm">
                <video controls autoplay playsinline muted loop>
                    <source src="./img/filter-sidebar.webm" type="video/webm"/>
                    <source src="./img/filter-sidebar.mp4" type="video/mp4"/>
                    Your browser does not support the video tag.
                </video>
            </a>
            <a href="#the-filterlist-sidebar" style="display: block;transform: translateY(-10px);"><code>&lt;FilterList&gt;</code> サイドバー</a>
        </td>
</tr>
<tr style="border:none;background-color:#fff;">
        <td style="width:50%;border:none;text-align:center">
            <a title="積み重ねフィルター" href="https://marmelab.com/ra-enterprise/modules/assets/ra-form-layout/latest/stackedfilters-overview.webm">
                <video controls autoplay playsinline muted loop width="90%" style="margin:1rem;box-shadow:0px 4px 4px 0px rgb(0 0 0 / 24%);">
                    <source src="https://marmelab.com/ra-enterprise/modules/assets/ra-form-layout/latest/stackedfilters-overview.webm" type="video/mp4" />
                        Your browser does not support the video tag.
                </video>
            </a>
            <a href="#the-stackedfilters-component" style="display: block;transform: translateY(-10px);"><code>&lt;StackedFilters&gt;</code> ダイアログ</a>
        </td>
        <td style="width:50%;border:none;text-align:center;vertical-align:top;">
            <a title="<Search> 入力" href="https://marmelab.com/ra-enterprise/modules/assets/ra-search-overview.gif"><img src="https://marmelab.com/ra-enterprise/modules/assets/ra-search-overview.gif" /></a>
            <a href="#global-search" style="display: block;transform: translateY(-10px);">グローバル <code>&lt;Search&gt;</code></a>
        </td>
</tr>
</tbody></table>

React-adminはリストをフィルタリングするための4つの異なる方法を提供します。表示するデータの種類、フィルターの種類や数、ユーザーが使用するデバイスに応じて、適切な方法を選択できます。

## フィルターボタン/フォームコンボ

<video controls autoplay playsinline muted loop>
      <source src="./img/list_filter.webm" type="video/webm"/>
      <source src="./img/list_filter.mp4" type="video/mp4"/>
      Your browser does not support the video tag.
</video>

デフォルトのフィルター表示は、リストの上部に表示されるインラインフォームです。ユーザーはそのフォームに入力を追加するドロップダウンボタンも見ることができます。この機能は`<List filters>`プロパティに依存しています：

```jsx
    import { TextInput } from 'react-admin';

    const postFilters = [
        <TextInput label="検索" source="q" alwaysOn />,
        <TextInput label="タイトル" source="title" defaultValue="こんにちは、世界!" />,
    ];

    export const PostList = () => (
        <List filters={postFilters}>
            ...
        </List>
    );
```

`filters`として渡された要素は通常の入力です。つまり、参照や配列値などに基づく高度なフィルターを構築することができます。`<List>`は、`alwaysOn`プロパティを持つもの以外のすべての入力をデフォルトで[`Filter Form`](./FilterForm.md)に隠します。

**ヒント**: 技術的な理由から、react-adminは`defaultValue`と`alwaysOn`の両方を持つフィルター入力を受け入れません。常にオンのフィルターにデフォルト値を設定するには、`<List>`コンポーネントの[`filterDefaultValues`](./List.md#filterdefaultvalues)プロパティを使用してください。

`<List>`は`filters`として渡された要素を2回使用します：

* 一度はフィルターフォームをレンダリングするため
* 一度はフィルターボタンをレンダリングするため（各要素の`label`を使用し、`source`の人間化された形式にフォールバック）

### `<SearchInput>`
<video controls autoplay playsinline muted loop> <source src="./img/search\_input.webm" type="video/webm"/> <source src="./img/search\_input.mp4" type="video/mp4"/> Your browser does not support the video tag. </video>

[通常の入力タイプ](./Inputs.md)（`<TextInput>`, `<SelectInput>`, `<ReferenceInput>`など）に加えて、`filters`配列に`<SearchInput>`を使用できます。この入力は特に[`Filter Form`](./FilterForm.md)用に設計されています。これは、虫眼鏡アイコン付きの`<TextInput resettable>`のようなもので、ユーザーが全文検索を行いたいときに探すタイプの入力です。

```jsx
import { SearchInput, TextInput } from 'react-admin';

const postFilters = [
    <SearchInput source="q" alwaysOn />
];
```

上記の例では、`q`フィルターがすべてのフィールドで全文検索をトリガーします。全文フィルタリング機能を`dataProvider`やAPIで実装するのはあなたの責任です。

詳細については、[`<SearchInput>`コンポーネントの章](./SearchInput.md)を参照してください。

### クイックフィルター
<video controls autoplay playsinline muted loop> <source src="./img/quick\_filters.webm" type="video/webm"/> <source src="./img/quick\_filters.mp4" type="video/mp4"/> Your browser does not support the video tag. </video>

ユーザーは通常、リストをフィルタリングするためにキーボードを使用することを嫌います（特にモバイルで）。このユーザーの要件を満たす良い方法は、フィルターを*クイックフィルター*に変えることです。クイックフィルターは、編集不可の`defaultValue`を持つフィルターです。ユーザーはそれらを有効または無効にするだけです。

汎用の`<QuickFilter>`コンポーネントを実装する方法は次のとおりです：

{% raw %}

```jsx
import { SearchInput } from 'react-admin';
import { Chip } from '@mui/material';

const QuickFilter = ({ label }) => {
    const translate = useTranslate();
    return <Chip sx={{ marginBottom: 1 }} label={translate(label)} />;
};

const postFilters = [
    <SearchInput source="q" alwaysOn />,
    <QuickFilter source="commentable" label="コメント可能" defaultValue={true} />,
    <QuickFilter source="views_lte" label="低視聴数" defaultValue={150} />,
    <QuickFilter source="tags" label="タグ付きコード" defaultValue={[3]} />,
];
```

{% endraw %}

**ヒント**: 同じソースに対して2つのクイックフィルターを使用することは現在できません。

## `<FilterList>` サイドバー
<video controls autoplay playsinline muted loop> <source src="./img/filter-sidebar.webm" type="video/webm"/> <source src="./img/filter-sidebar.mp4" type="video/mp4"/> Your browser does not support the video tag. </video>

フィルターボタン/フォームコンボの代替UIは、FilterListサイドバーです。ユーザーが通常eコマースサイトで見るものと似ています。これは、多くのシンプルなフィルターをマウスで有効にし、組み合わせることができるパネルです。ユーザー体験はボタン/フォームコンボよりも優れています。なぜなら、フィルターの値が明示的で、フォームに入力する必要がないからです。しかし、有限の値（または区間）を持つフィルターのみを使用できるため、少し強力ではありません。

以下はFilterListサイドバーの例です：

{% raw %}

```jsx
import { SavedQueriesList, FilterLiveSearch, FilterList, FilterListItem } from 'react-admin';
import { Card, CardContent } from '@mui/material';
import MailIcon from '@mui/icons-material/MailOutline';
import CategoryIcon from '@mui/icons-material/LocalOffer';

export const PostFilterSidebar = () => (
    <Card sx={{ order: -1, mr: 2, mt: 9, width: 200 }}>
        <CardContent>
            <SavedQueriesList />
            <FilterLiveSearch />
            <FilterList label="ニュースレターに登録済み" icon={<MailIcon />}>
                <FilterListItem label="はい" value={{ has_newsletter: true }} />
                <FilterListItem label="いいえ" value={{ has_newsletter: false }} />
            </FilterList>
            <FilterList label="カテゴリー" icon={<CategoryIcon />}>
                <FilterListItem label="テスト" value={{ category: 'tests' }} />
                <FilterListItem label="ニュース" value={{ category: 'news' }} />
                <FilterListItem label="取引" value={{ category: 'deals' }} />
                <FilterListItem label="チュートリアル" value={{ category: 'tutorials' }} />
            </FilterList>
        </CardContent>
    </Card>
);
```

{% endraw %}

`<List aside>`プロパティを使用してリストビューに追加します：

```jsx
import { PostFilterSidebar } from './PostFilterSidebar';

export const PostList = () => (
    <List aside={<PostFilterSidebar />}>
        ...
    </List>
);
```

**ヒント**: 上記の`PostFilterSidebar`コンポーネントの`<Card sx>`プロパティは、デフォルトの右側ではなく、サイドバーを画面の左側に配置するために使用されています。

詳細については、[`<FilterList>`のドキュメント](./FilterList.md)を参照してください。

FilterListを使用する場合、検索入力が必要になることが多いでしょう。FilterListサイドバーはフォームではないため、少し余分な作業が必要です。幸いなことに、react-adminはそのための専用検索入力コンポーネントを提供しています。詳細については、[`<FilterLiveSearch>`のドキュメント](./FilterLiveSearch.md)を参照してください。

<video controls autoplay playsinline muted loop> <source src="./img/filter-live-search.webm" type="video/webm"/> <source src="./img/filter-live-search.mp4" type="video/mp4"/> Your browser does not support the video tag. </video>

最後に、フィルターサイドバーはユーザーのお気に入りのフィルターを表示するのに理想的な場所です。詳細については、[`<SavedQueriesList>`のドキュメント](./SavedQueriesList.md)を参照してください。

<video controls autoplay playsinline muted loop> <source src="./img/SavedQueriesList.webm" type="video/webm"/> <source src="./img/SavedQueriesList.mp4" type="video/mp4"/> Your browser does not support the video tag. </video>
## `<StackedFilters>` コンポーネント
<video controls autoplay playsinline muted loop width="100%"> <source src="https://marmelab.com/ra-enterprise/modules/assets/ra-form-layout/latest/stackedfilters-overview.webm" type="video/mp4" /> Your browser does not support the video tag. </video>

もう一つの代替フィルターUIは、[エンタープライズ版](https://marmelab.com/ra-enterprise)<img class="icon" src="./img/premium.svg" />限定のStacked Filtersダイアログです。これは、フィールド、オペレーター、値を組み合わせて複雑なフィルターを構築することができます。フィルターボタン/フォームコンボよりも強力ですが、データプロバイダーの設定に多くの手間がかかります。

以下はStackedFiltersの設定例です：

```jsx
import {
    BooleanField,
    CreateButton,
    Datagrid,
    List,
    NumberField,
    ReferenceArrayField,
    TextField,
    TopToolbar,
} from 'react-admin';
import {
    textFilter,
    dateFilter,
    booleanFilter,
    referenceFilter,
    StackedFilters,
} from '@react-admin/ra-form-layout';

const postListFilters = {
    id: textFilter({ operators: ['eq', 'neq'] }),
    title: textFilter(),
    published_at: dateFilter(),
    is_public: booleanFilter(),
    tags: referenceFilter({ reference: 'tags' }),
};

const PostListToolbar = () => (
    <TopToolbar>
        <CreateButton />
        <StackedFilters config={postListFilters} />
    </TopToolbar>
);

const PostList = () => (
    <List actions={<PostListToolbar />}>
        <Datagrid>
            <TextField source="title" />
            <NumberField source="views" />
            <ReferenceArrayField tags="tags" source="tag_ids" />
            <BooleanField source="published" />
        </Datagrid>
    </List>
)
```

詳細については、[`<StackedFilters>`のドキュメント](./StackedFilters.md)を参照してください。

## グローバル検索
<video controls autoplay playsinline muted loop> <source src="https://marmelab.com/ra-enterprise/modules/assets/ra-search-overview.webm" type="video/webm" /> <source src="https://marmelab.com/ra-enterprise/modules/assets/ra-search-overview.mp4" type="video/mp4" /> Your browser does not support the video tag. </video>

リストフィルターはフィールドごとの条件を使って正確なクエリを作成できますが、ユーザーはしばしば全文検索のようなシンプルなインターフェースを好みます。結局、それは彼らが毎日検索エンジン、メールクライアント、ファイルエクスプローラーで使用するものです。

管理画面で単一のフォーム入力を使用して任意のレコードを検索する全文検索を表示したい場合は、[<Search>コンポーネント](./Search.md)を確認してください。これは[エンタープライズ版](https://marmelab.com/ra-enterprise)<img class="icon" src="./img/premium.svg" />限定です。

`<Search>`は任意の既存の検索エンジン（ElasticSearch、Lucene、またはカスタム検索エンジン）に接続でき、検索結果をカスタマイズして関連項目への迅速なナビゲーションを提供し、検索エンジンを「Omnibox」に変えることができます：

<video controls autoplay playsinline muted loop> <source src="https://marmelab.com/ra-enterprise/modules/assets/ra-search-demo.webm" type="video/webm" /> <source src="https://marmelab.com/ra-enterprise/modules/assets/ra-search-demo.mp4" type="video/mp4" /> Your browser does not support the video tag. </video>

グローバル検索の詳細については、[`<Search>`のドキュメント](./Search.md)を参照してください。

## フィルタークエリパラメータ

React-adminはURLから`filter`クエリパラメータを使用して、リストに適用するフィルターを決定します。

React-adminアプリケーションの典型的なリストページURLは次のとおりです：

> [https://myadmin.dev/#/posts?displayedFilters=%7B%22commentable%22%3Atrue%7D&filter=%7B%22commentable%22%3Atrue%2C%22q%22%3A%22lorem%20%22%7D&order=DESC&page=1&perPage=10&sort=published\_at](https://myadmin.dev/#/posts?displayedFilters=%7B%22commentable%22%3Atrue%7D&filter=%7B%22commentable%22%3Atrue%2C%22q%22%3A%22lorem%20%22%7D&order=DESC&page=1&perPage=10&sort=published_at)

デコードすると、`filter`クエリパラメータはJSON値として表示されます：

```sql
filter={"commentable":true,"q":"lorem "}
```

これにより、次のデータプロバイダー呼び出しが行われます：

```js
dataProvider.getList('posts', {
    filter: { commentable: true, q: 'lorem ' },
    pagination: { page: 1, perPage: 10 },
    sort: { field: 'published_at', order: 'DESC' },
});
```

ユーザーがフィルターを追加または削除すると、react-adminはURLの`filter`クエリパラメータを変更し、`<List>`コンポーネントが新しいフィルターで再度`dataProvider.getList()`をフェッチします。

**ヒント**: ユーザーがフィルターを設定すると、react-adminはフィルター値をアプリケーションの状態に保持し、ユーザーがリストに戻ったときにフィルター付きリストを表示します。これは設計上の選択です。

**ヒント**: `<Link>`コンポーネントや`react-router-dom`の`useNavigate()`フックを使用して、クエリパラメータを更新することでプログラム的にフィルターを変更できます。

## 事前フィルターされたリストへのリンク

フィルター値はURLから取得されるため、`filter`クエリパラメータを設定することで、事前フィルターされたリストへのリンクを作成できます。

たとえば、タグのリストがある場合、そのタグでフィルターされた投稿のリストにリンクするボタンを表示できます：

{% raw %}

```jsx
import { useTranslate, useRecordContext } from 'react-admin';
import Button from '@mui/material/Button';
import { Link } from 'react-router-dom';

const LinkToRelatedProducts = () => {
    const record = useRecordContext();
    const translate = useTranslate();
    return record ? (
        <Button
            color="primary"
            component={Link}
            to={{
                pathname: '/posts',
                search: `filter=${JSON.stringify({ category_id: record.id })}`,
            }}
        >
            {record.name}のカテゴリーのすべての投稿
        </Button>
    ) : null;
};
```

{% endraw %}

このボタンを`<Datagrid>`の子として使用できます。この技術を使用して、フィルター値を`{}`に設定することで、フィルターされていないリストにリンクするカスタムメニューボタンを作成することもできます。

## フィルターオペレーター

フィルターを内部的に格納し、データプロバイダーに送信する形式はオブジェクトです。例えば：

```js
{ commentable: true, q: "lorem " }
```

これは等値フィルターには問題ありませんが、「間」、「含む」、「始まる」、「より大きい」などのより複雑なフィルターをどのように行うか？

このような複雑なフィルターをAPIに渡す標準的な方法はないため、react-adminはその点について何も決定しません。それをフィルターオブジェクトに格納する方法を決めるのはあなた次第です。

デモでは一つの方法を示しています：フィルター名にオペレーターをサフィックスとして付けます。例えば、"greater than or equal to"のために"\_gte"を使用します。

```jsx
const postFilters = [
    <DateInput source="released_gte" label="これ以降にリリース" />,
    <DateInput source="released_lte" label="これ以前にリリース" />
];
```

一部のAPIバックエンド（例：JSON Server）はこの構文を理解します。あなたのAPIがこれらの「仮想フィールド」を理解しない場合は、それらをデータプロバイダーで期待される構文に変換する必要があります。

```jsx
// dataProvider.js内
export default {
    getList: (resource, params) => {
        // フィルターオブジェクトをオペレーター付きのフィルター配列に変換する
        // フィルターは{ commentable: true, released_gte: '2018-01-01' }のようになります
        const filter = params.filter;
        const operators = { '_gte': '>=', '_lte': '<=', '_neq': '!=' };
        // フィルターは次のようになります：
        //    { field: "commentable", operator: "=", value: true},
        //    { field: "released", operator: ">=", value: '2018-01-01'}
        const filters = Object.keys(filter).map(key => {
            const operator = operators[key.slice(-4)];
            return operator
                ? { field: key.slice(0, -4), operator, value: filter[key] }
                : { field: key, operator: '=', value: filter[key] };
        });
        const query = {
            pagination: params.pagination,
            sort: params.sort,
            filter: filters,
        };
        const url = `${apiUrl}/${resource}?${stringify(query)}`;
        return httpClient(url).then(({ json }) => ({
            data: json,
            total: parseInt(headers.get('content-range').split('/').pop(),10),
        }));
    },
    // ...
}
```

## 保存されたクエリ：ユーザーにフィルターとソートを保存させる
<video controls autoplay playsinline muted loop> <source src="./img/SavedQueriesList.webm" type="video/webm"/> <source src="./img/SavedQueriesList.mp4" type="video/mp4"/> Your browser does not support the video tag. </video>

保存されたクエリにより、ユーザーはフィルターとソートの組み合わせを新しい個人用フィルターとして保存できます。保存されたクエリはセッション間で持続するため、ユーザーは管理画面を閉じたり再度開いたりしてもカスタムクエリを見つけることができます。保存されたクエリは、フィルターボタン/フォームコンボと<FilterList>サイドバーの両方で利用できます。フィルターボタン/フォームコンボにはデフォルトで有効ですが、<FilterList>サイドバーには自分で追加する必要があります。

`<SavedQueriesList>`はフィルターサイドバーの<FilterList>セクションを補完します

```diff
import { FilterList, FilterListItem, List, Datagrid } from 'react-admin';
import { Card, CardContent } from '@mui/material';

+import { SavedQueriesList } from 'react-admin';

const SongFilterSidebar = () => (
    <Card>
        <CardContent>
+           <SavedQueriesList />
            <FilterList label="レコード会社" icon={<BusinessIcon />}>
                ...
            </FilterList>
            <FilterList label="リリース済み" icon={<DateRangeeIcon />}>
               ...
            </FilterList>
        </CardContent>
    </Card>
);

const SongList = () => (
    <List aside={<SongFilterSidebar />}>
        <Datagrid>
            ...
        </Datagrid>
    </List>
);
```

## カスタムフィルターの構築
<video controls autoplay playsinline muted loop> <source src="./img/filter\_with\_submit.webm" type="video/webm"/> <source src="./img/filter\_with\_submit.mp4" type="video/mp4"/> Your browser does not support the video tag. </video>

フィルターボタン/フォームコンボや<FilterList>サイドバーがニーズに合わない場合は、独自のものを作成できます。React-adminはカスタムフィルターの開発を容易にするショートカットを提供します。

例えば、デフォルトではフィルターボタン/フォームコンボは送信ボタンを提供せず、ユーザーがフォームとの対話を終了した後に自動的に送信されます。これはスムーズなユーザー体験を提供しますが、一部のAPIにとっては呼び出しが多すぎることがあります。

その場合、ユーザーがフォーム入力に値を入力するのではなく、送信ボタンをクリックしたときにフィルターを処理するのが解決策です。React-adminはそのためのコンポーネントを提供していませんが、フィルタ機能の内部を説明する良い機会です。実際には、フィルターボタン/フォームコンボの代替実装を提供します。

カスタムフィルターUIを作成するには、デフォルトのリストツールバーコンポーネントを上書きする必要があります。これには、リストフィルターとやり取りするフィルターボタンと[`Filter Form`](./FilterForm.md)の両方が含まれます。

### フィルターコールバック

新しい要素は[useListContextフック](./useListContext.md)を使用して、リストフィルターとより簡単にやり取りできます。このフックは以下の定数を返します：

* `filterValues`: URIに基づくフィルターの値、例：`{ "commentable": true, "q": "lorem" }`
* `setFilters()`: フィルター値を設定するコールバック、例：`setFilters({ "commentable":true })`
* `displayedFilters`: フォームに表示されるフィルターの名前、例：`['commentable', 'title']`
* `showFilter()`: フォームに追加のフィルターを表示するコールバック、例：`showFilter('views')`
* `hideFilter()`: フォームにフィルターを隠すコールバック、例：`hideFilter('title')`

この知識を活用して、送信時にフィルターを設定するカスタム`<List>`コンポーネントを書きます。

### カスタムフィルターボタン

次のコンポーネントはクリック時にフィルターフォームを表示します。`showFilter`関数を利用します：

```jsx
import { useListContext } from 'react-admin';
import { Button } from '@mui/material';
import ContentFilter from '@mui/icons-material/FilterList';

const PostFilterButton = () => {
    const { showFilter } = useListContext();
    return (
        <Button
            size="small"
            color="primary"
            onClick={() => showFilter("main")}
            startIcon={<ContentFilter />}
        >
            フィルター
        </Button>
    );
};
```

通常、`showFilter()`は`displayedFilters`リストに1つの入力を追加します。フィルターフォーム全体を隠すか表示するかなので、仮想の「main」入力を使用してフィルター全体を表します。

### カスタムフィルターフォーム

次に、ユーザーがフィルターボタンをクリックしたときに表示されるフィルターフォームコンポーネントです。フォーム入力は直接フォーム内に表示され、フォーム送信はパラメータとして渡された`setFilters()`コールバックをトリガーします。フォーム状態を処理するために`react-hook-form`を使用します：

{% raw %}

```jsx
import * as React from 'react';
import { useForm, FormProvider } from 'react-hook-form';
import { Box, Button, InputAdornment } from '@mui/material';
import SearchIcon from '@mui/icons-material/Search';
import { TextInput, NullableBooleanInput, useListContext } from 'react-admin';

const PostFilterForm = () => {
    const {
        displayedFilters,
        filterValues,
        setFilters,
        hideFilter
    } = useListContext();

    const form = useForm({
        defaultValues: filterValues,
    });

    if (!displayedFilters.main) return null;

    const onSubmit = (values) => {
        if (Object.keys(values).length > 0) {
            setFilters(values);
        } else {
            hideFilter("main");
        }
    };

    const resetFilter = () => {
        setFilters({}, []);
    };

    return (
        <FormProvider {...form}>
            <form onSubmit={form.handleSubmit(onSubmit)}>
                <Box display="flex" alignItems="flex-end" mb={1}>
                    <Box component="span" mr={2}>
                        {/* Full-text search filter. We don't use <SearchFilter> to force a large form input */}
                        <TextInput
                            resettable
                            helperText={false}
                            source="q"
                            label="検索"
                            InputProps={{
                                endAdornment: (
                                    <InputAdornment>
                                        <SearchIcon color="disabled" />
                                    </InputAdornment>
                                )
                            }}
                        />
                    </Box>
                    <Box component="span" mr={2}>
                        {/* Commentable filter */}
                        <NullableBooleanInput
                            helperText={false}
                            source="commentable"
                        />
                    </Box>
                    <Box component="span" mr={2} mb={1.5}>
                        <Button variant="outlined" color="primary" type="submit">
                            フィルター
                        </Button>
                    </Box>
                    <Box component="span" mb={1.5}>
                        <Button variant="outlined" onClick={resetFilter}>
                            閉じる
                        </Button>
                    </Box>
                </Box>
            </form>
        </FormProvider>
    );
};
```

{% endraw %}

### カスタムフィルターをリストアクションで使用する

最後に、`<ListAction>`コンポーネントを作成し、`actions`プロパティを使用して`<List>`コンポーネントに渡します：

```jsx
import { TopToolbar, ExportButton } from 'react-admin';
import { Box } from '@mui/material';

const ListActions = () => (
    <Box width="100%">
        <TopToolbar>
            <PostFilterButton />
            <ExportButton />
        </TopToolbar>
        <PostFilterForm />
    </Box>
);

export const PostList = () => (
    <List actions={<ListActions />}>
        ...
    </List>
);
```

**ヒント**: フィルターをリストに渡す必要はもうありません。`<PostFilterForm>`コンポーネントがそれらを表示します。

同様のアプローチを使用して、データフィルタリングのための代替ユーザーエクスペリエンスを提供することもできます。例えば、フィルターをデータグリッドヘッダーの行として表示することができます。


