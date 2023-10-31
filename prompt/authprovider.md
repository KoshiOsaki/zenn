あなたは、プロのフルスタックエンジニアです。

zenn で技術記事を書いていきたいと思います。記事を書く目的としては、
「技術力を示し、高いレベルのエンジニアからの評価を得ること。
つまり高いレベルのエンジニアに閲覧してもらい、いいねをもらうこと」です。

・つまり、検索されやすく、中身がハイレベルのように見える記事を書きたいと思います。
・ジャンルで言うと、Tech の分野がいいです
・普段は firebase, nextjs, typescript, graphql, docker, go, postgresql を使うことが多いです。

Next.js, FirebaseAuth, Graphql を用いた AuthProvider について書こうと思います。技術ブログに投稿する文章を書いてください。

条件は以下です。
・フランクな文体
・重要なキーワードを取り残さない
・文字数は 3600 文字程度
・マークダウンで書いてください
・構成間で重複した説明は省くようにしてください。
・読者がブログを読みながらコードをさわれるようにハンズオン形式などを取り入れて文章を作ってください。
・上記の内容を踏まえて相応しいタイトルを付けてください。
・ハンズオン部分について
　・簡単な環境構築や initialize、ライブラリの install は日本語で書き、別の記事を参照するように記載するだけで大丈夫です。コードやコマンドを記載する必要はありません。
・TS のサンプルコードについて
　・モジュール読み込みのコードは ESM 方式にしてください
　・非同期処理の部分はできるだけ async await を使ってください

大まかに以下の構成で記事を書いてください。

1. AuthProvider とは？
2. FirebaseAuth×Graphql のユースケース
3. FirebaseAuth×Graphql を使うメリット・デメリット
4. AuthProvider の実際の書き方・実行方法
5. まとめ

以下の要素を含めてください。
・以下のような hooks を作成

```
const useCurrentUser = () => {
  const { firebaseUser, initialized: isFirebaseInitialized } = useFirebaseUser();

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
    setCurrentUser(data?.user.mobile.setting.fetchUserMobileSettingUserInfo ?? null);
  }, [data]);
  useEffect(() => {
    if (!firebaseUser) {
      setCurrentUser(null);
      return;
    }
    void refetch();
  }, [firebaseUser]);

  const [loading, setLoading] = useState<boolean>(true);
  useEffect(() => setLoading(!isFirebaseInitialized || isQueryLoading), [isFirebaseInitialized, isQueryLoading]);

  const [deserializedError, setDeserializedError] = useState<Error | null>(null);
  useEffect(() => {
    if (!error) {
      setDeserializedError(null);
      return;
    }

    // user not found は graphQLErrorとなるが、webapp的には想定しているエラー
    if (error.graphQLErrors[0]?.message === 'user not found') {
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

```
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

・以下のような authProvider を実装

```
const AuthProvider = (props: Props) => {
  const router = useRouter();
  const pathname = usePathname();
  const { loading, error, currentUser } = useCurrentUser();

  const canAccess = (path: string): boolean => {
    const isNotAuthorized = !currentUser;
    if (isNotAuthorized) {
      return UNAUTHORIZED_USER_AVAILABLE_PATH_LIST.some((item) => path.startsWith(item));
    }
    return true;
  };

  useEffect(() => {
    if (loading) return;
    if (!canAccess(pathname)) {
      router.replace('/signin');
    }
  }, [loading, pathname]);

  return <>{loading ? <LoadingScreen /> : error ? <ErrorScreen /> : <>{props.children}</>}</>;
};
```

・実際のコード部分は下記では省略して構いませんが、どこに載せるべきか明示してください。

まずは、①〜③ について、1800 字程度で書いてもらえますか？
