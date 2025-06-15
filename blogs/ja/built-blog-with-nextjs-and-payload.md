---
title: "Next.js App RouterとPayload CMS を使ってブログを構築した"
description: "Next.jsのApp RouterとPayload CMSを使ってどのように本ブログを構築しているかについて紹介したいと思います。"
tags: ["Next.js", "Payload CMS", "Software Development"]
publishedAt: "2024-01-25T15:00:00.000Z"
---

Web やフロントエンド開発の技術についてある程度分かると自分で Web サイトを作りたくなるものです。かく言う自分もアプリ開発の仕事をしながら、実際に自分で Web サイトも運営してきました。

そんな中で、年末や正月の空いた時間を使ってブログサイトで使っている技術構成を一新しましたので、どのように技術選定や実装を行なったのかについてシェアしたいと思います。

当初は3日で終わるつもりでしたが、色々と変更を加えているうちに1週間ほど必要になってしまいました。

## 以前使っていたブログの技術構成

本題に入る前に、以前使っていたブログの技術構成について紹介しておきます。

- [Next.js Page Router](https://nextjs.org/docs/pages/building-your-application/routing)
- CSS-in-JS (Emotion) によるスタイリング
- Markdown ファイルでコンテンツを管理
- Vercel に本番環境をデプロイ

2019年頃に作った構成ですが、シンプルにマークダウンファイルを更新するだけでブログを運用できるので、手間をかけずに一人で運用する分には申し分は無かったです。

上記の技術構成でブログを運用していて、Webサイトや表示速度などのパフォーマンスには問題は無かったのですが、運用していく中で以下のような問題を抱えていることも分かってきました。

- コンテンツ管理システム(CMS) が無いのでコンテンツの量が増えるにつれてコンテンツの管理が煩雑になる。
- コンテンツの量が増えるにつれてコンテンツを更新するたびに必要な静的サイト生成に時間が増える。

## 今回構築したブログの技術構成

それでは、今回ブログの技術構成について紹介します。

- [Next.js App Router](https://nextjs.org/docs/app/building-your-application/routing)
- CSS Modules によるスタイリング
- Payload CMS によるコンテンツ管理
- GraphQL を使い動的にコンテンツを表示

まず、コンテンツ管理システム(CMS) が無かったので、今回は Payload という CMS を導入しました。  
これで Web 上の UI を通してコンテンツを管理できるようにしました。

また、技術スタックも最新のトレンドに沿ったものに更新しました。例えば、2023年にリリースされた Next.js App Router に乗り換えるなどです。採用した技術については以下で詳しく説明します。

## フロントエンド

### Next.js App Router

2023年にリリースされた Next.js App Router で今回はフロントエンドの構築を行いました。

現段階で、Page Router から完全に乗り換える必要性はまだ高くありませんが、今後使うことが増えるのは確実なので勉強も兼ねています。

App Router に関しては、[Layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts) 専用のコンポーネントが備わりレンダリングの最適化が簡単に行えるようになるなど、実用的なアップデートも多数備わっています。その一方で、新規導入された Server Components などノウハウがまだ完全に出ていない部分も多く、安定して高速に開発という段階にはまだ達することができていません。

App Router 自体、まだリリースされて間も無いので手探りの部分も多いですが、運用していくにつれてより洗練されてくることでしょう。

### スタイリング

UI のデザインの細かいスタイリングは自分で行いたかったので、枠組みが用意されている Tailwind CSS はまず候補から外れました。

CSS Modules と CSS-in-JS で悩んだのですが、最終的に CSS Modules を選びました。

CSS-in-JS 自体は数年前から使っており使用経験があったのですが、近頃 React 開発チームが力を入れている Server Components と CSS-in-JS の相性の悪さに関する懸念が拭えなかったので、今回で CSS Modules に乗り換える判断をしました。

### GraphQL Client

GraphQL でコンテンツのデータを取得するための GraphQL Client がこのプロジェクトでは必要になりましたが、今回は URQL を選びました。

- Apollo Client:
- - ユーザー数が多くドキュメントが整っている
- Relay:
- - クライアントサイドの React で使うのに向いている
  - サーバーサイドレンダリングや Node.js 上で使う際の例が少ない
- URQL
- - バンドルサイズが一番軽量
  - サーバーサイドレンダリングや Node.js 上での動作も保証されている

今回はサーバーサイドレンダリングの際にデータを取得する必要があったので、その中で一番軽量な URQL を選びました。

### 国際化・多言語化対応 (i18n)

Next.js Pages Router ではビルドインでサポートされていた 国際化・多言語化対応 (i18n) ですが、App Router では[ガイド](https://nextjs.org/docs/app/building-your-application/routing/internationalization)は用意されてますが、対応自体は自分で行う必要があります。

今回は [`next-international`](https://next-international.vercel.app/) というライブラリを使って対応を行いました。

このライブラリを選択した理由は、App Router に対応していたことと、軽量かつ型安全であることが理由です。

### パフォーマンス

Next.js App Router の [Layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts) 機能や CSS Modules 移行によるスタイリング用のコード削減でパフォーマンスは多少改善されています。

250ms ほど早くなるというデータが計測できるほどには改善されていますが、正直な話をいうと体感で違いは分かりません。

## CMS・GraphQL サーバー

### フレームワーク

当初は Contentful などのサービスとして展開している Headless CMS を使うことも検討していました。

ただ、長期的に使うことを考慮し、自前でサーバーを立てて管理することができるオープンソースの Headless CMS を使うことにしました。

ただ、オープンソースで使うことのできそうな Headless CMS 自体の数が少なかったので、以下の２つから選ぶことになりました。

- [Payload](https://payloadcms.com/)
- [Strapi](https://strapi.io/)

どちらの候補も、JavaScript/TypeScript で書かれたオープンソースの CMS フレームワークで、GraphQL や REST API をサポートしているなどの共通の特徴を持ち合わせています。

Strapi は使ったことがありましたが、Payload と比較して煩雑なコードを書く必要があったり、型安全が不十分なところから開発体験があまり良くなく、結果的に Payload 一択のような状況でした。

Payload 自体は、使うことができるデータベースが Mongo か Postgre （ベータ版）に限られるなど、まだまだ開発段階である印象は否めませんが、それでも Strapi と比較して少ないコードの量で CMS を構築でき、TypeScript の型安全にも配慮されてあるので開発体験は悪くありません。

## まとめ

以前使っていた技術構成で実現できなかった機能を実現できたので、総合的には満足しています。

運用していると分かってくる改善点もあると思うので、継続的にアップデートしていきたいですね。
