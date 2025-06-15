---
title: "Claude Desktopで調査した内容をシームレスにGitHubに保存するMCPサーバーをn8nで構築した話"
description: "Claude Desktopでの効率的な調査が可能になったものの、チャット履歴からの知識再利用が課題だったので、MCPとn8nを活用してシームレスな知識保存ワークフローを構築し、GitHubに構造化された形で調査結果を管理できるシステムを実装しました。"
tags: ["AI", "MCP", "n8n", "Claude Desktop"]
publishedAt: "2025-06-15T12:00:00.000Z"
---

## はじめに

2025年6月現在、私の情報収集方法は大きく変化しました。以前はGoogle検索が主流でしたが、今では**Claude Desktop**を毎日のように使って情報を調べています。これは現在まだ広く普及していない利用方法かもしれませんが、私にとってAI/LLMを使った情報検索が日常となっています。

この変化の背景には、Claude 4以降に導入された**拡張思考（Extended thinking）機能**[^1][^2]があります。この機能は、Claudeが応答前により多くの時間をかけて問題を分解・分析し、複数のアプローチを検討することを可能にします。これにより、Claude DesktopでWeb検索を組み合わせた応答の精度が飛躍的に向上しました。さらに、ファクトチェックを実施するようにシステムプロンプトを活用することで、AI/LLMに情報検索からファクトチェックまで一貫して任せることができるようになっています。

実際に、React NativeとFlutterの比較調査やn8nワークフロー実装の技術調査などをClaude Desktopを使って最近したのですが、複数のソースから情報を収集し、整理・分析まで行ってくれるため、従来のGoogle検索と比較して格段に効率的でした。

しかし、この便利さには副作用がありました。効率的な調査ができる一方で、その結果をどう管理・活用するかという新たな課題が生まれたのです。

## チャット履歴に蓄積された知識の活用困難さ

効率的に調査タスクを行えるようになった一方で、Claude Desktopの**チャット履歴に大量の知識（ナレッジ）が蓄積**されるようになりました。調べた内容は分析結果として残るものの、後でその知識を再利用しようとすると、チャット履歴から目当ての情報を探すのが非常に煩わしい作業になってしまいました。

具体的には、調査した数週間後に「あの時調べたReact Nativeの情報はどこだったかな？」と思っても、大量のチャット履歴から目当ての情報を見つけるのは困難でした。Claude Desktopの検索機能はあるものの、調査内容の関連性や分類による整理が効きません。

### 構造化された知識管理の必要性

調査結果をただ履歴として残すのではなく、以下の要素を含む構造化された形で管理したいと考えていました：

- **分類とタグ付け**：技術領域や用途による分類
- **参考文献の紐付け**：情報の信頼性を担保するための出典管理
- **検索とフィルタリング**：後で効率的に情報を発見できる仕組み
- **関連性の可視化**：異なる調査内容間の関連性の把握

### 普段活用しているツールとの統合

私は普段、**Obsidian**というマークダウンエディターを使ってメモやノートを管理しています。Obsidianは、マークダウン形式でAI/LLMが理解しやすく、リンク機能による知識の関連性表現、豊富なプラグインエコシステムなど、AI/LLMとの親和性が高いツールです[^9]。

理想的には、Claude Desktopでの調査結果をそのままObsidianで管理できるワークフローを構築したいと思いました。

## 解決策

これらの課題を解決するため、以下のアプローチを用いました。

### AI/LLMとの対話で得た知識をシームレスに活用するためのMCPサーバー

具体的には、AIとのチャットを通じて調査した内容をマークダウン形式で外部に保存できるようにしました。AIとの会話で得られた知見を、そのままシームレスに同じチャットを使って、調査で見つけた参考文献付きの情報を保存可能になりました。

また、保存した知識をAIが検索できるようにもしました。そのため、チャットセッションが異なる場合でも、AIに指示を出すことで過去に調査した知識を活用できるようになっています。

以下は使った技術に関する簡単な説明です。

### Model Context Protocol（MCP）

2024年にAnthropicが発表したMCP[^3][^4]は、AI/LLMと外部ツールの連携を標準化するためのプロトコルです。Claude Desktopが公式にMCPをサポートしていることも選定理由の一つです。

簡単に説明すると、AI/LLMから外部へのアクセスを簡単に行うための規格です。今回のケースだとClaude Desktopとのチャットを通して、チャットの内容をそのままGitHubなどの外部へ保存するための窓口として使っています。

### n8n

n8nを簡単に説明すると、AIを簡単に組み込めるワークフロー自動化ツールです。

私は既にn8nを使って、ニュース収集や領収書処理などの自動化を行っていたため、既存の運用ノウハウも活用できました。

このn8nでは、「MCP Server Trigger node」という専用ノードを提供しています[^5]。この機能により、n8nをMCPサーバーとして動作させ、ワークフローをAI/LLMから利用できるようになります。

つまり、Claude DesktopなどのAI/LLMとのチャットからワークフローを実行し、GitHubやGoogle Cloudなどの外部サービスと簡単に連携できます。

### GitHub

今回、データの保存先にはGitHubを使っています。理由は単純でAPI経由でファイル検索が実現できるからです。[^7][^8]

最初はクラウドストレージなども検討したのですが、検索するためのインデックスをどうするのかという問題がありました。結局、デフォルトでファイルやコンテンツを検索できるGitHubを使うことで落ち着きました。

元からGitHubを毎日使うようなヘビーユーザーだったので、簡単に組み込めてよかったです。

## 実装詳細とワークフロー解説

上述の技術を使い、実際のMCPサーバーを構築しました。

### 作成した機能

**1. save_knowledge（保存）** Claude Desktopから直接、調査結果をマークダウン形式でGitHubに保存する機能。参考文献の必須化による信頼性担保、カテゴリによる自動分類、タグシステム、YAMLフロントマターによるメタデータ自動付与を実装。

**2. search_knowledge（検索）** GitHub Search APIを活用した検索機能[^7]。ファイル名、タイトル、タグでの柔軟な検索と、関連度順の結果表示を提供。

**3. read_knowledge（読み取り）** 保存された知識ドキュメントの内容を取得する機能。GitHub Contents API[^8]を使用し、日本語ファイル名に対応。

**4. get_available_categories（カテゴリ一覧）** 利用可能なカテゴリを確認し、適切な分類選択をサポートする機能。

### 工夫した点・苦労した点

#### エラーハンドリングの工夫

MCPサーバーでは実際の使い方をAIに伝えるための説明文のような要素があります。ただ、実際にAIが使用中にエラーになった場合、適切にエラー内容を伝えてあげないとAIを混乱させてしまいます。

実際のデバッグでもAIにMCPサーバーを使ってもらうことでデバッグを進めたのですが、最初は適切なエラー内容をAIに伝えておらず、何度もリトライを繰り返すというような挙動をさせてしまいました。

そのため、AIが理解し、ユーザーに適切に伝えられるエラーメッセージ形式を採用し、技術的なエラーではなく、AIが解釈しやすい形式でのフィードバックを実装しました。

#### mcp-remoteによる接続ブリッジ

Claude DesktopのようなMCPクライアントは、元々ローカルのMCPサーバーとの接続を前提として設計されています。しかし、n8nのMCPサーバーはリモートで動作するため、**mcp-remote**[^6]というアダプターツールが必要になりました。

MCP自体がまだ発表されてから半年程度であり、Claude Desktopからリモートで動作するMCPサーバーへの接続のための知識はあまり出回っておらず、少し苦労しました。

MCPの仕様は急速に発展しており、現在のHTTP+SSE方式から、より効率的なStreamable HTTP方式への移行が予定されています[^6]。この変化により、将来的にはmcp-remoteのようなアダプターツールが不要になる可能性があります。

設定例（claude_desktop_config.json）：

```json
{
  "mcpServers": {
    "n8n-knowledge-server": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://n8n.your-server.net/mcp/knowledge-mcp-server/sse",
        "--header",
        "Authorization: Bearer ${AUTH_TOKEN}"
      ],
      "env": {
        "AUTH_TOKEN": "your-authentication-token"
      }
    }
  }
}
```

## 実際の運用と効果

実は、**この記事自体が、構築したMCPサーバーの実証実験を兼ねています**。記事執筆にあたって、事前にClaude Desktopで調査し保存していた以下のナレッジを活用しました：

- n8nのMCPサーバー機能に関する技術文書
- MCPプロトコルの仕様調査結果
- GitHub API統合の実装知見

記事執筆時には、Claude Desktopから「n8n-knowledge-server MCPを使って知識を調べて」と指示することで、関連する保存済みナレッジを即座に活用できました。そのまま流れでLLMにデータを渡せるので記事執筆も楽にできています。

## おわりに

Claude Desktopから直接「この調査結果を保存して」と依頼するだけで、適切なメタデータ付きで構造化された知識ベースに保存される**シームレスな知識保存ワークフロー**を実現できました。

n8nを使っているので、「保存前にGeminiに文章をレビューさせる」や「保存先を変更する」、「メールで通知する」などの機能追加も比較的簡単に行えます。

このような形で、MCPやn8nを活用することで知識の収集と集約を効率的に行うことができます。

AIによりできることの範囲が拡大したおかげで最近は時間が足りなくなってきていたので、このようにAIで効率化ができることには助かっています。

---

[^1]: [Introducing Claude 4 | Anthropic](https://www.anthropic.com/news/claude-4)

[^2]: [Claude's extended thinking | Anthropic](https://www.anthropic.com/news/visible-extended-thinking)

[^3]: [Introducing the Model Context Protocol | Anthropic](https://www.anthropic.com/news/model-context-protocol)

[^4]: [Model Context Protocol Documentation](https://modelcontextprotocol.io/introduction)

[^5]: [MCP Server Trigger node documentation | n8n Docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-langchain.mcptrigger/)

[^6]: [Build and deploy Remote Model Context Protocol (MCP) servers to Cloudflare](https://blog.cloudflare.com/remote-model-context-protocol-servers-mcp/)

[^7]: [GitHub Search API Documentation](https://docs.github.com/en/rest/search)

[^8]: [GitHub Repository Contents API](https://docs.github.com/en/rest/repos/contents)

[^9]: [Get Plugged In: How to Use Generative AI Tools in Obsidian | NVIDIA Blog](https://blogs.nvidia.com/blog/ai-decoded-obsidian/)
