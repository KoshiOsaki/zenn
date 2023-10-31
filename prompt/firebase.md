あなたは、プロのフルスタックエンジニアです。

zennで技術記事を書いていきたいと思います。記事を書く目的としては、
「技術力を示し、高いレベルのエンジニアからの評価を得ること。
つまり高いレベルのエンジニアに閲覧してもらい、いいねをもらうこと」です。

・つまり、検索されやすく、中身がハイレベルのように見える記事を書きたいと思います。
・ジャンルで言うと、Techの分野がいいです
・普段はfirebase, nextjs, typescript, graphq, dockerlを使うことが多いです。

firebase authentication custom claimsを使用した認証フロー について書こうと思います。技術ブログに投稿する文章を書いてください。

条件は以下です。
・フランクな文体
例) まずは○○について説明していこうと思いますが、先に自分の自己紹介から始めます笑
・重要なキーワードを取り残さない
・文字数は5000文字程度
・マークダウンで書いてください
・構成間で重複した説明は省くようにしてください。
・読者がブログを読みながらコードをさわれるようにハンズオン形式などを取り入れて文章を作ってください。
・上記の内容を踏まえて相応しいタイトルを付けてください。
・サンプルコードについて
　・モジュール読み込みのコードはESM方式にしてください
　・非同期処理の部分はできるだけasync awaitを使ってください

大まかに以下の構成で記事を書いてください。
①custom claimsとは?
②custom claimsのユースケース
③custom claimsを使うメリット・デメリット
④custom claimsの実際の書き方・実行方法
⑤まとめ

以下の要素を含めてください。
・cloud functionsのsignup用apiを用意し、そこでfirebase adminを使用してcustom claimsをsetする。
・custom Tokenをresponseで返して、signInWithCustomToken を使い初回ログイン時からtokenが入っている状態にできる
・セキュリティルールでの利用
・cloud functions onCall apiでの、roleによる制限

まずは、①〜③について、1800字程度で書いてもらえますか？