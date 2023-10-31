---
title: "実戦！Next.js × FirebaseAuth × Graphqlで実現するAuthProvider"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["firebase", "Next.js"]
published: true
---

どうも！フルスタックエンジニアのおおさきです！今回は、Next.js、FirebaseAuth、Graphql を組み合わせた AuthProvider の作成について解説していきます。これを通して、みんなも高いレベルのエンジニアリング力を身につけられることを願っています！

## 1. AuthProvider とは？

**AuthProvider**は、アプリケーションの認証状態を管理し、適切なコンテンツやリダイレクトを行うためのコンポーネントのことを指します。これにより、ログインしていないユーザーが特定のページにアクセスしようとすると、ログインページにリダイレクトされるなどの動作を実現できます。

例えば、E-コマースサイトでは、ログインせずに購入ページにアクセスしようとすると、ログインページにリダイレクトされることが一般的ですよね。これを簡単に実現するのが、**AuthProvider**の役目です！

## 2. FirebaseAuth×Graphql のユースケース

FirebaseAuth と Graphql を組み合わせることで、どのようなことができるのか。具体的なユースケースを考えてみましょう。

1. **リアルタイム認証情報の取得**  
   FirebaseAuth は、リアルタイムにユーザーの認証状態を取得することができます。これを Graphql と組み合わせることで、認証状態に基づいたデータをリアルタイムに取得することが可能となります。

2. **セキュアな API の実装**  
   FirebaseAuth で取得したトークンを Graphql のヘッダーに付与することで、セキュアな API 通信が実現できます。この組み合わせにより、認証されていないユーザーは必要なデータを取得できなくなり、セキュリティが向上します。

3. **フレキシブルなデータフェッチ**  
   Graphql は、必要なデータのみを取得することができるため、ユーザーの認証状態に応じてデータを動的にフェッチすることが可能となります。

これらのユースケースを元に、私たちは FirebaseAuth と Graphql を組み合わせることで、効率的で高性能な認証システムを実現できるのです！

## 3. FirebaseAuth×Graphql を使うメリット・デメリット

### メリット

1. **効率的なデータ取得**  
   Graphql の特性を生かし、必要なデータだけを取得することができます。
2. **高セキュリティ**  
   FirebaseAuth のトークンを利用して API を呼び出すことで、セキュリティを強化できます。
3. **リアルタイムな認証状態の反映**  
   FirebaseAuth のリアルタイム性を利用し、即座に認証状態の変化を検知できます。

### デメリット

1. **学習コスト**  
   FirebaseAuth や Graphql の仕組みを理解するための学習コストがかかります。
2. **複雑なセットアップ**  
   両者を組み合わせるための初期セットアップが複雑になる場合があります。

3. **コードの管理**  
   複数の技術を組み合わせるため、コードの管理や構造に気を付ける必要があります。

次回、実際の AuthProvider の書き方や実行方法について詳しく解説していきます！お楽しみに！

---

以上で、AuthProvider の基本的な概念や FirebaseAuth と Graphql の組み合わせのユースケース、メリット・デメリットについて解説しました。次に、実際のコードを見ながら、どのように AuthProvider を実装していくのかを解説していきます 🚀

---

# 実戦！Next.js × FirebaseAuth × Graphql で実現する高機能 AuthProvider

前回は、AuthProvider の基本と FirebaseAuth×Graphql のユースケース、メリット・デメリットについて触れましたね。今回は、いよいよ具体的な実装方法について解説します。最後には、実装のまとめもありますので、最後までぜひお付き合いください！

## 4. AuthProvider の実際の書き方・実行方法

まずは環境のセットアップから始めましょう。Next.js のプロジェクトが既にあるという前提で話を進めますが、まだの方は[こちらの記事](#)で環境構築の方法をご確認ください。（注：リンクはダミーです。適切な記事のリンクを指定してください。）

### 環境構築

1. **Firebase プロジェクトのセットアップ**  
   Firebase のコンソールから新しいプロジェクトを作成し、アプリを登録します。

2. **Graphql サーバーの準備**  
   Graphql サーバーがまだない場合は、適切なものをセットアップしてください。

3. **必要なライブラリのインストール**  
   必要なライブラリは以下の通りです。

   ```
   npm install firebase @apollo/client graphql
   ```

   これで、環境構築は完了です！

### AuthProvider の実装

では、AuthProvider を実装していきましょう。まずは、Firebase の認証状態を管理する`useFirebaseUser`フックを作成します。

```tsx
import { useState, useEffect } from "react";
import { User } from "firebase/auth";
import { onAuthStateChanged } from "firebase/auth";
import { auth, setAuthToken } from "path/to/your/firebase/config";

const useFirebaseUser = () => {
  const [initialized, setInitialized] = useState(false);
  const [firebaseUser, setFirebaseUser] = useState<User | null>(null);
  const onChangeFirebaseUser = async (user: User | null) => {
    setFirebaseUser(user);
    try {
      const token = await user?.getIdToken();
      setAuthToken(token);
    } catch (e) {
      console.error(e);
    }
  };

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (user) => {
      setInitialized(true);
      void onChangeFirebaseUser(user);
    });
    return () => {
      unsubscribe();
    };
  }, []);

  return {
    initialized,
    firebaseUser,
  };
};
```

次に、Graphql を使用して現在のユーザー情報を取得する`useCurrentUser`フックを作成します。

```tsx
import { useState, useEffect } from "react";
import { useQuery } from "@apollo/client";
import { AuthProviderDocument } from "path/to/your/graphql/documents";
import { useFirebaseUser } from "./useFirebaseUser";

const useCurrentUser = () => {
  const { firebaseUser, initialized: isFirebaseInitialized } =
    useFirebaseUser();

  const {
    data,
    loading: isQueryLoading,
    error,
    refetch,
  } = useQuery(AuthProviderDocument, {
    skip: !firebaseUser,
  });

  // apiのrequest結果をglobalStateで共有
  const [currentUser, setCurrentUser] = useAtom(currentUserAtom);
  useEffect(() => {
    setCurrentUser(
      data?.user.mobile.setting.fetchUserMobileSettingUserInfo ?? null
    );
  }, [data]);
  useEffect(() => {
    if (!firebaseUser) {
      setCurrentUser(null);
      return;
    }
    void refetch();
  }, [firebaseUser]);

  const [loading, setLoading] = useState<boolean>(true);
  useEffect(
    () => setLoading(!isFirebaseInitialized || isQueryLoading),
    [isFirebaseInitialized, isQueryLoading]
  );

  const [deserializedError, setDeserializedError] = useState<Error | null>(
    null
  );
  useEffect(() => {
    if (!error) {
      setDeserializedError(null);
      return;
    }

    // user not found は graphQLErrorとなるが、webapp的には想定しているエラー
    if (error.graphQLErrors[0]?.message === "user not found") {
      setDeserializedError(null);
      return;
    }

    setDeserializedError(new Error());
  }, [error]);

  return {
    loading,
    error: deserializedError,
    currentUser,
  };
};
```

最後に、上記のフックを使用して AuthProvider コンポーネントを実装します。

```tsx
import { useEffect } from "react";
import { useRouter } from "next/router";
import { useCurrentUser } from "./useCurrentUser";
import { LoadingScreen, ErrorScreen } from "path/to/your/components";

const AuthProvider = (props: Props) => {
  const router = useRouter();
  const pathname = usePathname();
  const { loading, error, currentUser } = useCurrentUser();

  const canAccess = (path: string): boolean => {
    const isNotAuthorized = !currentUser;
    if (isNotAuthorized) {
      return UNAUTHORIZED_USER_AVAILABLE_PATH_LIST.some((item) =>
        path.startsWith(item)
      );
    }
    return true;
  };

  useEffect(() => {
    if (loading) return;
    if (!canAccess(pathname)) {
      router.replace("/signin");
    }
  }, [loading, pathname]);

  return (
    <>
      {loading ? (
        <LoadingScreen />
      ) : error ? (
        <ErrorScreen />
      ) : (
        <>{props.children}</>
      )}
    </>
  );
};
```

このように、`useFirebaseUser`と`useCurrentUser`を使用して、AuthProvider コンポーネントを構築することができます。

## 5. まとめ

今回は、Next.js、FirebaseAuth、Graphql を組み合わせた AuthProvider の実装について詳しく見てきました。この実装を通じて、セキュアかつ効率的な認証システムを構築することができます。

### 実装のポイント

- Firebase の認証状態と Graphql のクエリを組み合わせることで、リアルタイムかつセキュアな認証システムを構築できる。
- フック（useFirebaseUser、useCurrentUser）を利用して、コードの再利用性と可読性を高める。
- AuthProvider コンポーネントを利用して、認証状態に応じた UI の制御を行う。

これで、Next.js × FirebaseAuth × Graphql で実現する高機能 AuthProvider の実装についての解説は終了です。これを実践すれば、セキュアで使い勝手の良い認証システムを構築できることでしょう。興味を持った方はぜひ挑戦してみてくださいね！🚀
次回も、みなさんに役立つ情報をお届けします！お楽しみに！👋

---
