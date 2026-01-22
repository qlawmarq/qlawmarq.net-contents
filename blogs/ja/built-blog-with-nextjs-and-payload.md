---
title: "Next.js App RouterとPayload CMS を使ってブログを構築した"
description: "Next.jsのApp RouterとPayload CMSを使ってどのように本ブログを構築しているかについて"
tags: ["Next.js", "Payload CMS", "Software Development"]
publishedAt: "2024-01-25T15:00:00.000Z"
updatedAt: "2024-01-25T15:00:00.000Z"
---

Webやフロントエンド開発の技術についてある程度分かると自分でWebサイトを作りたくなるものです。かく言う自分もアプリ開発の仕事をしながら、実際に自分でWebサイトも運営してきました。

そんな中で、年末や正月の空いた時間を使ってブログサイトで使っている技術構成を一新しましたので、どのように技術選定や実装を行なったのかについてシェアしたいと思います。

当初は3日で終わるつもりでしたが、色々と変更を加えているうちに1週間ほど必要になってしまいました。

## 以前使っていたブログの技術構成

本題に入る前に、以前使っていたブログの技術構成について紹介しておきます。

- [Next.js Page Router](https://nextjs.org/docs/pages/building-your-application/routing)
- CSS-in-JS (Emotion) によるスタイリング
- Markdownファイルでコンテンツを管理
- Vercelに本番環境をデプロイ

2019年頃に作った構成ですが、シンプルにマークダウンファイルを更新するだけでブログを運用できるので、手間をかけずに一人で運用する分には申し分は無かったです。

上記の技術構成でブログを運用していて、Webサイトや表示速度などのパフォーマンスには問題は無かったのですが、運用していく中で以下のような問題を抱えていることも分かってきました。

- コンテンツ管理システム(CMS) が無いのでコンテンツの量が増えるにつれてコンテンツの管理が煩雑になる。
- コンテンツの量が増えるにつれてコンテンツを更新するたびに必要な静的サイト生成に時間が増える。

## 今回構築したブログの技術構成

それでは、今回ブログの技術構成について紹介します。

- [Next.js App Router](https://nextjs.org/docs/app/building-your-application/routing)
- CSS Modulesによるスタイリング
- Payload CMSによるコンテンツ管理
- GraphQLを使い動的にコンテンツを表示

まず、コンテンツ管理システム(CMS) が無かったので、今回はPayloadというCMSを導入しました。  
これでWeb上のUIを通してコンテンツを管理できるようにしました。

また、技術スタックも最新のトレンドに沿ったものに更新しました。例えば、2023年にリリースされたNext.js App Routerに乗り換えるなどです。採用した技術については以下で詳しく説明します。

## フロントエンド

### Next.js App Router

2023年にリリースされたNext.js App Routerで今回はフロントエンドの構築を行いました。

現段階で、Page Routerから完全に乗り換える必要性はまだ高くありませんが、今後使うことが増えるのは確実なので勉強も兼ねています。

App Routerに関しては、[Layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts) 専用のコンポーネントが備わりレンダリングの最適化が簡単に行えるようになるなど、実用的なアップデートも多数備わっています。その一方で、新規導入されたServer Componentsなどノウハウがまだ完全に出ていない部分も多く、安定して高速に開発という段階にはまだ達することができていません。

App Router自体、まだリリースされて間も無いので手探りの部分も多いですが、運用していくにつれてより洗練されてくることでしょう。

### スタイリング

UIのデザインの細かいスタイリングは自分で行いたかったので、枠組みが用意されているTailwind CSSはまず候補から外れました。

CSS ModulesとCSS-in-JSで悩んだのですが、最終的にCSS Modulesを選びました。

CSS-in-JS自体は数年前から使っており使用経験があったのですが、近頃React開発チームが力を入れているServer ComponentsとCSS-in-JSの相性の悪さに関する懸念が拭えなかったので、今回でCSS Modulesに乗り換える判断をしました。

### GraphQL Client

GraphQLでコンテンツのデータを取得するためのGraphQL Clientがこのプロジェクトでは必要になりましたが、今回はURQLを選びました。

- Apollo Client:
- - ユーザー数が多くドキュメントが整っている
- Relay:
- - クライアントサイドのReactで使うのに向いている
  - サーバーサイドレンダリングやNode.js上で使う際の例が少ない
- URQL
- - バンドルサイズが一番軽量
  - サーバーサイドレンダリングやNode.js上での動作も保証されている

今回はサーバーサイドレンダリングの際にデータを取得する必要があったので、その中で一番軽量なURQLを選びました。

### 国際化・多言語化対応 (i18n)

Next.js Pages Routerではビルドインでサポートされていた国際化・多言語化対応 (i18n) ですが、App Routerでは[ガイド](https://nextjs.org/docs/app/building-your-application/routing/internationalization)は用意されてますが、対応自体は自分で行う必要があります。

今回は [`next-international`](https://next-international.vercel.app/) というライブラリを使って対応を行いました。

このライブラリを選択した理由は、App Routerに対応していたことと、軽量かつ型安全であることが理由です。

### パフォーマンス

Next.js App Routerの [Layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts) 機能やCSS Modules移行によるスタイリング用のコード削減でパフォーマンスは多少改善されています。

250msほど早くなるというデータが計測できるほどには改善されていますが、正直な話をいうと体感で違いは分かりません。

## CMS・GraphQL サーバー

### フレームワーク

当初はContentfulなどのサービスとして展開しているHeadless CMSを使うことも検討していました。

ただ、長期的に使うことを考慮し、自前でサーバーを立てて管理することができるオープンソースのHeadless CMSを使うことにしました。

ただ、オープンソースで使うことのできそうなHeadless CMS自体の数が少なかったので、以下の2つから選ぶことになりました。

- [Payload](https://payloadcms.com/)
- [Strapi](https://strapi.io/)

どちらの候補も、JavaScript/TypeScriptで書かれたオープンソースのCMSフレームワークで、GraphQLやREST APIをサポートしているなどの共通の特徴を持ち合わせています。

Strapiは使ったことがありましたが、Payloadと比較して煩雑なコードを書く必要があったり、型安全が不十分なところから開発体験があまり良くなく、結果的にPayload一択のような状況でした。

Payload自体は、使うことができるデータベースがMongoかPostgre（ベータ版）に限られるなど、まだまだ開発段階である印象は否めませんが、それでもStrapiと比較して少ないコードの量でCMSを構築でき、TypeScriptの型安全にも配慮されてあるので開発体験は悪くありません。

## まとめ

以前使っていた技術構成で実現できなかった機能を実現できたので、総合的には満足しています。

運用していると分かってくる改善点もあると思うので、継続的にアップデートしていきたいですね。
