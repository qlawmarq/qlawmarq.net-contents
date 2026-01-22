---
title: "AI時代のiPhoneからmacOSへのリモート開発：Tailscale + Shellfish + tmux"
description: "Claude CodeなどのAIツールに最適化された、外出先の iPhone から macOS に安全にリモートアクセスして快適な開発作業を可能にする環境構築ガイド（2025年11月更新版）"
tags: ["Software Development", "AI", "Remote Development", "Claude Code"]
publishedAt: "2025-11-01T12:00:00.000Z"
updatedAt: "2025-12-03T12:00:00.000Z"
---

AI/LLMの普及に伴い、AI/LLMとの会話だけで作業できることが増えつつあります。
ソフトウェア開発という文脈でも例外ではなく、Claude CodeやCodex CLIなどのツールのおかげで、AI/LLMと会話しているだけでソフトウェア開発ができるという世界が現実になりつつあります。

以前のようにPCの画面いっぱいにコードエディターを開く必要も薄くなりつつあり、依然としてソフトウェアの実行環境は必要ですがスマホからリモートで開発するということも選択肢に入るようになってきています。
つまり、机に向かって作業するだけでなく、スマートフォンから自宅の開発マシンにリモート接続し、場所を問わず開発できる時代になったのです。

この記事では、電車の中でiPhoneからコーディングしたい、風呂に入りながらiPhoneからコードを更新したい——そんな要望を実現するリモートアクセス環境の構築方法を紹介します。

## 必要なもの

### 前提

この記事ではClaude CodeやGemini CLIなどのAIターミナルツールを使用することを前提としています。
それぞれのターミナルツールのインストールと基本的な使い方については、本記事では扱いませんので、各ツールの公式ドキュメントをそれぞれ参照してください。

### デバイス

- macOS
  - [Tailscale](https://tailscale.com/) - VPN
  - [tmux](https://github.com/tmux/tmux/wiki) - ターミナルマルチプレクサ
- iPhone / iPad
  - [Tailscale](https://tailscale.com/) - VPN
  - [Secure ShellFish](https://secureshellfish.app/) - iOS用SSHクライアント

### 接続の仕組み

```markdown
iPhone (Shellfish)
↓
Tailscale VPN (ピアツーピア接続)
↓
macOS（開発マシン）
├─ SSH サーバー
└─ tmux セッション（永続化、マルチデバイス共有）
```

各レイヤーが連携して、モバイル環境でも安定したリモート開発を実現します：

| レイヤー         | 技術      | 役割                          |
| ---------------- | --------- | ----------------------------- |
| **クライアント** | Shellfish | タップで接続、GUI 管理        |
| **ネットワーク** | Tailscale | 安全な接続、NAT 越え、固定 IP |
| **プロトコル**   | SSH       | 暗号化された安全な接続        |
| **セッション**   | tmux      | 切断後も継続、画面共有        |

## セットアップ手順

### 1. macOS 側の設定

#### 1.1 必要なパッケージのインストール

Homebrewを使ってmacOSに必要なツールをインストールします：

```bash
# Tailscale (VPN) のインストール
brew install --cask tailscale-app

# tmux (ターミナルマルチプレクサ) のインストール
brew install tmux
```

Homebrewを使わない場合は、各公式サイトからインストーラーをダウンロードしてインストールしてください。

#### 1.2 tmux の設定（推奨）

tmuxの設定ファイルを作成します。

既にtmuxを使っている場合は、それをそのまま使っても構いませんが、以下の設定はAIターミナルツールに最適化されていますので、参考にしてください。

**この設定の利点**：

- **Claude Code 対応**[^5]: 500000行のスクロールバックで長い出力を完全に確認可能
- **True Color**[^4]: 最新のカラースキームが正しく表示される（tmux-256colorを使用）
- **CJK 文字最適化**: 日本語文字の表示品質向上
- **iOS 最適化**: タッチ操作、キーボード位置を考慮した設計

`~/.tmux.conf` に以下の内容を保存してください：

```bash
# ====================================================================================
# 基本設定 - 2025年推奨構成
# ====================================================================================

# Terminal Type: tmux-256color（現代的な推奨、tmux 2.6+）
set -g default-terminal "tmux-256color"

# マウスサポート（iOS でのタッチ操作に）
set -g mouse on

# ウィンドウ/ペイン番号を 1 から開始（人間に優しい）
set -g base-index 1
setw -g pane-base-index 1

# ウィンドウを閉じたら番号を詰める
set -g renumber-windows on

# ====================================================================================
# Claude Code / AI ツール最適化
# ====================================================================================

# スクロールバックバッファを大幅に増加
# Claude Code は長い出力を生成するため、大きなバッファが必要
# 推奨: 500000行（約250MB）
# 参考: デフォルトは2000行、一般的な使用で50000行
set -g history-limit 500000

# ESC キー応答時間を最小化（vim/neovim 使用時の快適性）
# tmux 3.5 デフォルト: 10ms
set -sg escape-time 0

# Focus events を有効化（vim/neovim 統合）
set -g focus-events on

# メッセージ表示時間を延長（Claude Code の出力確認用）
set -g display-time 3000

# ペイン番号表示時間を延長（小さい画面での操作性向上）
set -g display-panes-time 4000

# ====================================================================================
# キーバインド
# ====================================================================================

# vi モードキーバインド（コピーモードで vim のキーが使える）
setw -g mode-keys vi

# ステータスバーは Emacs スタイル（zsh デフォルトと一致）
set -g status-keys emacs

# ====================================================================================
# ビジュアル設定
# ====================================================================================

# ステータスバーを上部に配置（iOS キーボードで隠れない）
set -g status-position top

# ステータスバーの色設定（シンプルで読みやすい）
set -g status-style bg=black,fg=white

# 現在のウィンドウを強調表示
setw -g window-status-current-style bg=blue,fg=white,bold
```

#### 1.3 リモートログインの有効化

macOSのリモートログイン（SSHサーバー）を有効化します：

1. システム設定を開く
2. **一般** > **共有** を選択
3. **リモートログイン** をオンにする
4. アクセスを許可するユーザーを選択（推奨： 管理者のみ）

#### 1.4 Tailscale のセットアップ

```bash
# Tailscale アプリを起動
open -a Tailscale
```

1. Tailscaleアプリが起動したら、指示に従ってログイン
2. SSOプロバイダー（Google、GitHubなど）で認証
3. 接続が完了したら、**macOS に割り当てられた Tailscale IP アドレス**を確認：

```bash
tailscale ip -4
```

**出力例**: `100.64.1.2`

**重要**: このIPアドレスはmacOS（接続先サーバー）のTailscale IPです。後の手順で使用するため、**必ずメモしておいてください**。

**補足**: Tailscaleは各デバイスに `100.x.y.z` 形式の固定IPを自動割り当てします。このIPは、デバイスがTailscaleネットワークに参加している限り変わりません。

#### 1.5 Tailscale の 2FA 有効化（推奨）

1. [Tailscale 管理画面](https://login.tailscale.com/admin) にアクセス
2. **Settings** > **Users** から自分のアカウントを選択
3. **Two-factor authentication** を有効化

### 2. iPhone 側の設定

#### 2.1 SSH クライアントの選択

iPhone/iPad用のSSHクライアントは複数あります。筆者はいくつかのアプリを試した結果、**Secure ShellFish** が最も使いやすいと感じました。2025年11月時点での各アプリの評価は以下の通りです。

##### **推奨: Shellfish (Secure ShellFish)**[^1]

**特徴**：

- **優れた tmux サポート**: セッションのサムネイルプレビュー、Handoffでデバイス間のセッション移行が可能
- **バックグラウンド SSH 維持**: アプリがバックグラウンドでもSSH接続を維持する機能
- **Files アプリ統合**: SSHサーバーをiOSのFilesアプリに直接統合
- **iCloud Keychain 同期**: サーバー設定を自動同期
- **買い切り可能**: $29.99で永久使用可能（月額 $2.99、年額 $14.99も選択可、無料版もあり）

**推奨理由**：

- tmuxとの統合が優秀で、Claude Codeの長い出力も問題なく扱える
- 長期的なコストパフォーマンスが高い（買い切り）
- 使い心地が良い

##### **代替案 1: Termius**[^2]

**特徴**：

- **クロスプラットフォーム**: Windows、macOS、Linux、iOS、Androidで同期
- **Mosh サポート**: 接続の安定性向上
- **SFTP 統合**: ファイル転送機能内蔵

**注意点**：

- **Claude Code との互換性問題**: 2025年11月時点で、AIターミナルツール（Gemini CLI等）使用時にスクロールバックの問題が[報告されています](https://github.com/google-gemini/gemini-cli/issues/10349)。長い出力の後に自動的に入力欄にスクロールされ、前の出力を確認できなくなる場合があります。（筆者はこの問題によりTermiusの使用を中止しました）

**価格**: 無料版あり、Proプランは月額 $10（年間請求時）、月額 $15（月次請求時）

##### **代替案 2: Blink Shell**[^3]

**特徴**：

- **Mosh の完全サポート**: ネットワーク切り替え時も接続維持、デバイス再起動後も接続維持
- **オープンソース**: コミュニティによる継続的な改善
- **Blink Code**: ブラウザ版VSCode統合
- **高度なカスタマイズ**: テーマ、フォント、レイアウトの自由度が高い
- **iPad マルチタスク最適化**: Split Screen / Slide Overに優れた対応

**価格**: 年額 $19.99（買い切りオプションなし）

#### 2.2 必要なアプリのインストール

App Storeから以下をインストール：

1. **Tailscale** - VPN接続用
2. **Shellfish** - SSHクライアント（上記の推奨を参照）

#### 2.3 Tailscale のセットアップ

1. Tailscaleアプリを起動
2. macOSと同じアカウントでログイン
3. 接続が完了すると、macOSと同じVPNネットワークに参加

#### 2.4 Shellfish のセットアップ

ShellfishにSSH接続を設定します。

1. Shellfishアプリを起動
2. **+** ボタンをタップして新規ホストを追加
3. 以下を入力：
   - **Label**: `mac`（任意の接続名）
   - **Hostname**: `<macOS の Tailscale IP>`（例： `100.64.1.2`）
     - これは手順1.4でメモしたmacOS側のIPアドレスです
   - **User**: `<macOS のユーザー名>`
     - macOSで `whoami` コマンドを実行すると確認できます
   - **Port**: `22`（SSHのデフォルトポート）
4. **Save** をタップして保存

これで設定が完了しました。ホスト名をタップするだけで接続できるようになります。

### 3. 接続テスト

#### 3.1 SSH 接続のテスト

ShellfishでmacOSへの接続をテストします。

**接続手順:**

1. Shellfishを開く
2. 先ほど作成した `mac` ホストをタップ
3. 初回接続時は、ホストキーの確認ダイアログが表示されるので **Continue** または **Trust** をタップ
4. パスワード入力が求められた場合は、macOSのユーザーパスワードを入力
5. 接続に成功すると、macOSのターミナル画面が表示されます

**確認:**
接続後、以下のコマンドでホスト名を確認できます：

```bash
hostname
```

macOSのホスト名が表示されれば成功です。

#### 3.2 tmux セッションの開始

macOSに接続できたら、tmuxセッションを開始します。または既存のセッションに参加することもできます：

```bash
# 新しいセッションを作成
tmux new -s dev

# または既存のセッションに参加
tmux attach -t dev
```

## 基本的な使い方

### tmux の基本コマンド

#### セッション管理

```bash
# 新しいセッションを作成
tmux new -s <セッション名>

# 既存のセッションに参加
tmux attach -t <セッション名>

# セッション一覧を表示
tmux ls

# セッションから一時的に離脱（セッションは継続）
# Ctrl+B → d

# セッションを終了
exit
```

### 同一セッションを iPhone と Mac で共有

tmuxを使うと、iPhoneとMacで同じターミナルセッションを共有できます。

1. **セッションを開始する（どちらかのデバイスで）:**

   macOSまたはiPhoneからmacOSに接続してtmuxセッションを作成：

   ```bash
   tmux new -s dev
   ```

2. **もう一方のデバイスから参加する:**
   - iPhoneの場合： SSHクライアントで `mac` ホストをタップして接続
   - Macの場合： ターミナルで `ssh <Tailscale IP>` または `mosh <Tailscale IP>` で接続

   接続後、以下のコマンドで同じセッションに参加：

   ```bash
   tmux attach -t dev
   ```

3. **同期された画面:**

   どちらのデバイスで入力しても、両方の画面に即座に反映されます。
   これにより、iPhoneで作業を開始し、Macで続きを行う、といった使い方が可能です。

## トラブルシューティング

### Tailscale で macOS が見つからない

**原因**: 両デバイスで異なるアカウントを使用している、または接続が確立されていない

**解決方法**:

1. 両デバイスで同じTailscaleアカウントにログインしているか確認
2. macOSでTailscaleの接続状態を確認：
   ```bash
   tailscale status
   ```
3. macOSでリモートログインが有効になっていることを確認
4. iPhoneのTailscaleアプリで接続状態を確認

### SSH で "Permission denied" エラー

**原因**: リモートログインが有効になっていない、またはユーザー権限の問題

**解決方法**:

1. macOSでリモートログインが有効か確認：
   - システム設定 > 一般 > 共有 > リモートログイン
2. 正しいユーザー名を使用しているか確認：
   ```bash
   # macOS で現在のユーザー名を確認
   whoami
   ```

### tmux セッションが見つからない

**原因**: セッションがまだ作成されていない、またはセッション名が間違っている

**解決方法**:

1. 既存のセッション一覧を確認：
   ```bash
   tmux ls
   ```
2. セッションが存在しない場合は新規作成：
   ```bash
   tmux new -s dev
   ```

## 応用編：VS Code on Browser

### code-server のセットアップ

VS CodeをiPhoneのSafariで使用したい場合：

```bash
# macOS 側で code-server をインストール
brew install code-server

# code-server を起動
code-server

# iPhone の Safari で以下にアクセス
# http://<Tailscale IP>:8080
```

パスワードは `~/.config/code-server/config.yaml` に記載されています。

---

## まとめ

お疲れさまでした。これでiPhoneからmacOSに安全にリモートアクセスし、開発作業ができる環境が整いました。

ここまでで構築した環境は、まさにAI時代の開発スタイル――モバイルで、常時接続で、途切れないという開発環境を実現できるはずです。
Tailscaleの安全なVPN接続、ShellfishのバックグラウンドSSH維持機能、tmuxの共有機能を組み合わせることで、AIツールとどこからでも共同作業が可能になります。

AIアシスト開発とモバイル中心のワークフローは、生産性の新しい形を提示しています。
通勤中のデバッグ、カフェでのテスト実行、外出先からの即時デプロイ——どの場面でも、途切れずに「開発の流れ」を保つことができます。

セキュリティ設定（Tailscaleの2FA有効化など）を忘れずに行い、快適なモバイル開発ライフをお楽しみください。

---

**免責事項**: この記事の情報は2025年11月時点のものです。ソフトウェアのバージョンアップや仕様変更により、内容が古くなる可能性があります。最新の情報は各ツールの公式ドキュメントをご確認ください。

---

[^1]: **Shellfish (Secure ShellFish)**: [App Store](https://apps.apple.com/app/ssh-client-secure-shellfish/id1336634154) | [公式サイト](https://secureshellfish.app/) | MacStories "Secure ShellFish Review" (2019) | 2025年アップデート： DECSLRM対応によるtmux改善

[^2]: **Termius**: [App Store](https://apps.apple.com/app/termius-terminal-ssh-client/id549039908) | [公式サイト](https://termius.com/) | 2025年11月時点での注意： AIターミナルツール使用時にスクロールバックの問題あり - [GitHub Issue: google-gemini/gemini-cli #10349](https://github.com/google-gemini/gemini-cli/issues/10349)

[^3]: **Blink Shell**: [App Store](https://apps.apple.com/app/blink-shell-build-code/id1594898306) | [公式サイト](https://blink.sh/) | [GitHub](https://github.com/blinksh/blink) | オープンソース、5年以上AppStoreでトップの開発者ツール

[^4]: **Terminal Type 設定**: tmux-256color vs screen-256colorについては、コミュニティで広く議論されています。[Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/1045/getting-256-colors-to-work-in-tmux)、[Stack Overflow](https://stackoverflow.com/questions/10158508/lose-vim-colorscheme-in-tmux) など複数のフォーラムでtmux-256colorが推奨されています。

[^5]: [Brian P. Hogan "Working with Claude Code"](https://bphogan.com/2025/06/19/2025-06-19-claude-code-tips/)（2025年6月）- 推奨： "big scrollback buffer" for Claude Code | 関連： [pchalasani/claude-code-tools](https://github.com/pchalasani/claude-code-tools) - Claude Code + tmuxの統合ツール | [ooloth/dotfiles](https://github.com/ooloth/dotfiles) - Claude Code対応の実践的なdotfiles

[^6]: **tmux 公式**: [tmux Wiki](https://github.com/tmux/tmux/wiki) | [tmux 3.5 リリースノート](https://github.com/tmux/tmux/blob/master/CHANGES)（2024年9月27日、3.5aは2024年10月5日）| [tmux FAQ](https://github.com/tmux/tmux/wiki/FAQ) | **注**: 最新版はtmux 3.6

[^7]: **Mosh 公式**: [公式サイト](https://mosh.org/) | [GitHub リポジトリ](https://github.com/mobile-shell/mosh) | Blink ShellなどでMoshを使用する場合の参考情報

[^8]: **Tailscale 公式**: [公式サイト](https://tailscale.com/) | [セキュリティベストプラクティス](https://tailscale.com/kb/) | [管理コンソール](https://login.tailscale.com/admin)
