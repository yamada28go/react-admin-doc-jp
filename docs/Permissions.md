---
layout: default
title: "Permissions"
---

# パーミッション

デフォルトでは、react-admin アプリはユーザーのパーミッションをチェックしません。`authProvider`が存在する場合、リスト、作成、編集、および表示ページにはログインが必要です。

しかし、アプリケーションによっては、特定の機能へのアクセスを有効または無効にするために細かいパーミッションが必要になることがあります。多くの戦略が考えられるため（シングルロール、マルチロール、ACLなど）、react-adminはパーミッションロジックを`authProvider`に委任します。

ユーザーのパーミッションに応じてページをカスタマイズする必要がある場合は、[`usePermissions()`](./usePermissions.md)フックを使用して`authProvider`からパーミッションを取得できます。

## `authProvider`のパーミッション

`authProvider.getPermissions()`はユーザーのパーミッションを返す責任があります。これらのパーミッションは以下のような形式を取ることができます：

- 文字列（例：`'admin'`）
- ロールの配列（例：`['post_editor', 'comment_moderator', 'super_admin']`）
- 細かいパーミッションのオブジェクト（例：`{ postList: { read: true, write: false, delete: false } }`）
- または関数

パーミッションの形式は自由です。なぜなら、react-admin自体はパーミッションを実際に使用しないためです。コード内でコンテンツを非表示または表示したり、ユーザーを別のページにリダイレクトしたり、警告を表示するためにそれらを使用するのはあなた次第です。

react-adminはパーミッションの形式には依存しませんが、最も一般的なパーミッション形式である[ロールベースアクセス制御（RBAC）](./AuthRBAC.md)の実装を提供します。RBACを使用する場合、`authProvider.getPermissions()`メソッドはパーミッションオブジェクトの配列を返す必要があります。

```tsx
    const authProvider = {
        // ...
        getPermissions: () => Promise.resolve([
            { action: ["read", "create", "edit", "export"], resource: "companies" },
            { action: ["read", "create", "edit"], resource: "people" },
            { action: ["read", "create", "edit", "export"], resource: "deals" },
            { action: ["read", "create"], resource: "comments" },
            { action: ["read", "create"], resource: "tasks" },
            { action: ["write"], resource: "tasks.completed" },
        ])
    };
```

ロールベースアクセス制御の使用方法の詳細については、[RBACの章](./AuthRBAC.md)を参照してください。

以下は、`authProvider`が認証時にユーザーのパーミッションを`localStorage`に保存し、`getPermissions`でこれらのパーミッションを返す例です：

{% raw %}

```jsx
// in src/authProvider.js
import decodeJwt from 'jwt-decode';

export default {
    login: ({ username, password }) => {
        const request = new Request('https://mydomain.com/authenticate', {
            method: 'POST',
            body: JSON.stringify({ username, password }),
            headers: new Headers({ 'Content-Type': 'application/json' }),
        });
        return fetch(request)
            .then(response => {
                if (response.status < 200 || response.status >= 300) {
                    throw new Error(response.statusText);
                }
                return response.json();
            })
            .then(({ token }) => {
                const decodedToken = decodeJwt(token);
                localStorage.setItem('token', token);
                localStorage.setItem('permissions', decodedToken.permissions);
            });
    },
    checkError: (error) => { /* ... */ },
    checkAuth: () => {
        return localStorage.getItem('token') ? Promise.resolve() : Promise.reject();
    },
    logout: () => {
        localStorage.removeItem('token');
        localStorage.removeItem('permissions');
        return Promise.resolve();
    },
    getIdentity: () => { /* ... */ },
    getPermissions: () => {
        const role = localStorage.getItem('permissions');
        return role ? Promise.resolve(role) : Promise.reject();
    }
};
```

{% endraw %}

## ユーザーパーミッションの取得

デフォルトのreact-adminビューやカスタムページでパーミッションを確認する必要がある場合は、[`usePermissions()`](./usePermissions.md)フックを使用できます：

以下は、パーミッションに基づいて条件付きの入力を行う`Create`ビューの例です：

{% raw %}

```jsx
export const UserCreate = () => {
    const { permissions } = usePermissions();
    return (
        <Create>
            <SimpleForm
                defaultValue={{ role: 'user' }}
            >
                <TextInput source="name" validate={[required()]} />
                {permissions === 'admin' &&
                    <TextInput source="role" validate={[required()]} />}
            </SimpleForm>
        </Create>
    )
}
```

{% endraw %}

カスタムページでも同様に機能します：

```jsx
// in src/MyPage.js
import * as React from "react";
import { Card } from '@mui/material';
import CardContent from '@mui/material/CardContent';
import { usePermissions } from 'react-admin';

const MyPage = () => {
    const { permissions } = usePermissions();
    return (
        <Card>
            <CardContent>Lorem ipsum sic dolor amet...</CardContent>
            {permissions === 'admin' &&
                <CardContent>Sensitive data</CardContent>
            }
        </Card>
    );
}
```

**ヒント**: RBACを使用する場合は、[`<IfCanAccess>`コンポーネント](./IfCanAccess.md)を使用してパーミッションを取得し、ユーザーが必要なパーミッションを持っている場合にのみ子コンテンツをレンダリングします。

```jsx
import { IfCanAccess } from '@react-admin/ra-rbac';

const MyPage = () => (
    <Card>
        <CardContent>Lorem ipsum sic dolor amet...</CardContent>
        <IfCanAccess action="read" resource="sensitive_data">
            <CardContent>Sensitive data</CardContent>
        </IfCanAccess>
    </Card>
);
```

## リソースやビューへのアクセスの制限

パーミッションはリソースやそのビューへのアクセスを制限するのに役立ちます。これを行うには、`<Admin>`コンポーネントの子として関数を渡す必要があります。react-adminは`authProvider`が返すパーミッションでこの関数を呼び出します。このような関数子を提供できるのは一つだけです。

```jsx
export const App = () => (
    <Admin dataProvider={dataProvider} authProvider={authProvider}>
        {permissions => (
            <>
                {/* 編集ビューへのアクセスを管理者にのみ制限 */}
                <Resource
                    name="customers"
                    list={VisitorList}
                    edit={permissions === 'admin' ? VisitorEdit : null}
                    icon={VisitorIcon}
                />
                {/* カテゴリリソースを管理者ユーザーのみに含める */}
                {permissions === 'admin'
                    ? <Resource name="categories" list={CategoryList} edit={CategoryEdit} icon={CategoryIcon} />
                    : null}
            </>
        )}
    </Admin>
);
```

関数は必要なだけ多くのフラグメントを返すことができます。

**ヒント**: RBACを使用する場合、上記のように手動でパーミッションをチェックする必要はありません。ただ[`ra-rbac`の`<Resource>`コンポーネント](./AuthRBAC.md#resource)を使用するだけです。これにより自動的にパーミッションがチェックされます。

```jsx
import { Admin } from 'react-admin';
import { Resource } from '@react-admin/ra-rbac';

export const App = () => (
    <Admin dataProvider={dataProvider} authProvider={authProvider}>
        <Resource name="customers" list={VisitorList} edit={VisitorEdit} icon={VisitorIcon} />
        <Resource name="categories" list={CategoryList} edit={CategoryEdit} icon={CategoryIcon} />
    </Admin>
);
```

## フォーム入力へのアクセス制限

特定のパーミッションを持つユーザーにのみいくつかの入力を表示したい場合があります。そのためには`usePermissions`フックを使用できます。

以下は、パーミッションに基づいて条件付きの入力を行う`Create`ビューの例です：

{% raw %}

```jsx
import { usePermissions, Create, SimpleForm, TextInput } from 'react-admin';

export const UserCreate = () => {
    const { permissions } = usePermissions();
    return (
        <Create>
            <SimpleForm>
                <TextInput source="name" />
                {permissions === 'admin' &&
                    <TextInput source="role" />}
            </SimpleForm>
        </Create>
    );
}
```

{% endraw %}

**注意**: `usePermissions`は非同期です。つまり、`permissions`はマウント時に常に`undefined`になります。`authProvider.getPermissions()`のプロミスが解決されると、`permissions`はプロミスが返す値に設定され、コンポーネントは再レンダリングされます。これにより、`defaultValue`などのリアクティブでないプロップスで`permissions`を使用する際に予期しないことが起こる可能性があります：

```jsx
import { usePermissions, Create, SimpleForm, TextInput } from 'react-admin';

export const UserCreate = () => {
    const { permissions } = usePermissions();
    return (
        <Create>
            <SimpleForm>
                <TextInput source="name" defaultValue={
                    // この方法は機能しません：defaultValueは常に'user'になります 
                    permissions === 'admin' ? 'admin' : 'user'
                } />
            </SimpleForm>
        </Create>
    );
}
```

`react-hook-form`では、`defaultValue`はマウント時にのみ使用され、初期レンダー後にその値が変更されてもデフォルト値は変わりません。解決策は、パーミッションが解決されるまで入力のレンダリングを遅らせることです：

```jsx
import { usePermissions, Create, SimpleForm, TextInput } from 'react-admin';

export const UserCreate = () => {
    const { isLoading, permissions } = usePermissions();
    return (
        <Create>
            <SimpleForm>
                {!isLoading && <TextInput source="name" defaultValue={
                    permissions === 'admin' ? 'admin' : 'user'
                } />}
            </SimpleForm>
        </Create>
    );
}
```

**ヒント**: RBACを使用する場合は、[`ra-rbac`の`<SimpleForm>`コンポーネント](./AuthRBAC.md#simpleform)を使用してください。これにより、ユーザーが必要なパーミッションを持っている場合にのみ子コンテンツがレンダリングされます。

```jsx
import { Create, TextInput } from 'react-admin';
import { SimpleForm } from '@react-admin/ra-rbac';

export const UserCreate = () => (
    <Create>
        <SimpleForm>
            <TextInput source="name" />
            <TextInput source="role" />
        </SimpleForm>
    </Create>
);
```

## リストの列へのアクセス制限

いくつかの列を非表示にするために`usePermissions`を使用できます。

```jsx
import * as React from 'react';
import { usePermissions, List, Datagrid, ShowButton, TextField }  from 'react-admin';

export const UserList = () => {
    const { permissions } = usePermissions();
    return (
        <List>
            <Datagrid rowClick="edit">
                <TextField source="id" />
                <TextField source="name" />
                {permissions === 'admin' && <TextField source="role" />}
            </Datagrid>
        </List>
    );
};
```

**ヒント**: RBACを使用する場合は、[`ra-rbac`の`<Datagrid>`コンポーネント](./AuthRBAC.md#datagrid)を使用してください。これにより、ユーザーがその列のパーミッションを持っている場合にのみ列がレンダリングされます。

```jsx
import * as React from 'react';
import { List, ShowButton, TextField, TextInput }  from 'react-admin';
import { Datagrid } from '@react-admin/ra-rbac';

export const UserList = () => (
    <List>
        <Datagrid rowClick="edit">
            <TextField source="id" />
            <TextField source="name" />
            <TextField source="role" />
        </Datagrid>
    </List>
);
```

## メニューへのアクセス制限

[カスタムメニュー](./Admin.md#menu)内でパーミッションを確認したい場合はどうしますか？カスタムページ内でパーミッションを取得するのと同様に、`usePermissions`フックを使用する必要があります：

```jsx
// in src/myMenu.js
import * as React from "react";
import { Menu, usePermissions } from 'react-admin';

const MyMenu = ({ onMenuClick }) => {
    const { permissions } = usePermissions();
    return (
        <Menu>
            <Menu.Item to="/posts" primaryText="Posts" onClick={onMenuClick} />
            <Menu.Item to="/comments" primaryText="Comments" onClick={onMenuClick} />
            {permissions === 'admin' &&
                <Menu.Item to="/custom-route" primaryText="Miscellaneous" onClick={onMenuClick} />
            }
        </Menu>
    );
}
```

**ヒント**: RBACを使用する場合は、[`ra-rbac`の`<Menu>`コンポーネント](./AuthRBAC.md#menu)を使用してください。これにより、ユーザーがそのメニュー項目のパーミッションを持っている場合にのみメニュー項目がレンダリングされます。

## ロールベースアクセス制御

役割やグループを使用したより複雑なパーミッション、最小特権の原則、レコードレベルのパーミッション、明示的な拒否などが必要な場合は、次のセクションで[ロールベースアクセス制御](./AuthRBAC.md)の詳細を確認してください。


