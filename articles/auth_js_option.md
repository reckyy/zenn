---
title: Auth.js
emoji: "🫡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs","認証", "authjs"]
published: true
---
## はじめに
現在、Rails API と Next.js (App Router) を組み合わせて Web アプリを作成しています。認証機能を実装するにあたって、Auth.js を使用しました。この記事では、その際に使用したオプションを備忘録として記録します。

:::message
本記事にもし間違いがあれば、優しく教えていただけると嬉しいです。🙂‍↕️
:::

## 認証チェック

認証状態によってアクセス制限を設けたいと思い、middleware で制限しようとしました。調べてみると多くの記事が見つかりましたが、うまくいきませんでした。そこで公式ドキュメントを見直してみると、便利なオプションが見つかりました。（最初からもっとちゃんと見ておけばよかった…）

https://authjs.dev/reference/nextjs#authorized

```ts
optional authorized: (params) => Awaitable<undefined | boolean | Response | NextResponse<unknown>>;
```

例えば以下のように設定すると、ログインが必要なページに未ログイン状態でアクセスした場合、ルートにリダイレクトされます。
```ts: src/auth.ts
authorized: async ({ request, auth }) => {
  if (auth) {
    return true;
  } else {
    return NextResponse.redirect(new URL('/', request.nextUrl));
  }
},
```

ログインが必要ないページは、middleware内のmatcherオプションであらかじめ除外しておきます。**この際ルートを含めるのを忘れずに！！**
無限ループになってしまうので。

> If you are returning a redirect response, make sure that the page you are redirecting to is not protected by this callback, otherwise you could end up in an infinite redirect loop.


自分はルートのページを、認証状態によって異なるコンポーネントを表示するようにしています。
```ts: src/app/page.tsx
export default async function Page() {
  const session = await auth();
  if (session?.user) {
    return <AuthenticatedTopPage />;
  } else {
    return <TopPage />;
  }
}
```



## Sessionの拡張
最初は Rails API に email を送っていましたが、毎回メールアドレスを送るのは手間だと感じたので、Session に id を追加しました。ここでは、Google の ID を使用しています。


```ts: src/auth.ts
callbacks: {
  async jwt({ token, profile }) {
    if (profile) {
      token.sub = profile.sub || undefined;
    }
    return token;
  },
  async session({ session, token }) {
    if (token.sub) {
      session.user.id = token.sub;
    }
    return session;
  },
}
```

公式ドキュメントにわかりやすく書かれています。
https://authjs.dev/guides/extending-the-session

## 終わりに
アウトプットももっと頻繁にやっていかないと思い、まずは簡単な備忘録を記しました。
読んでいただき、ありがとうございました。
