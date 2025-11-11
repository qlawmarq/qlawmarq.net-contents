---
title: "AI時代のiPhoneからmacOSへのリモート開発：Tailscale + Shellfish/Blink + Mosh + tmux"
description: "Claude Code などの AI ツールに最適化された、外出先の iPhone から macOS に安全にリモートアクセスして快適な開発作業を可能にする環境構築ガイド（2025年11月更新版）"
tags: ["Software Development", "AI", "Remote Development", "Claude Code"]
publishedAt: "2025-11-01T12:00:00.000Z"
updatedAt: "2025-11-11T12:00:00.000Z"
---

AI/LLM の普及に伴い、AI/LLM との会話だけで作業できることが増えつつあります。
ソフトウェア開発という文脈でも例外ではなく、Claude Code や Codex CLI などのツールのおかげで、AI/LLM と会話しているだけでソフトウェア開発ができるという世界が現実になりつつあります。

以前のように PC の画面いっぱいにコードエディターを開く必要も薄くなりつつあり、依然としてソフトウェアの実行環境は必要ですがスマホからリモートで開発するということも選択肢に入るようになってきています。
つまり、机に向かって作業するだけでなく、スマートフォンから自宅の開発マシンにリモート接続し、場所を問わず開発できる時代になったのです。

この記事では、電車の中で iPhone からコーディングしたい、カフェで始めた作業を帰宅後に iPhone で続けたい——そんな要望を実現するリモートアクセス環境の構築方法を紹介します。

## 概要

このガイドでは、Tailscale と Mosh を組み合わせることで、以下を実現します：

- ルーターのポート開放不要で外出先から自宅 Mac に安全アクセス
- Wi-Fi↔LTE 切替時も接続が途切れない
- 接続が切れても作業が継続され、再接続時に続きから再開可能

## 必要なもの

### macOS 側

- [Homebrew](https://brew.sh/) - macOS 向けパッケージ管理ツール
- 以下のツール（Homebrew でインストール）:
  - [Tailscale](https://tailscale.com/) - VPN
  - [Mosh](https://mosh.org/) - モバイルシェル
  - [tmux](https://github.com/tmux/tmux/wiki) - ターミナルマルチプレクサ

### iPhone/iOS デバイス

- App Store へのアクセス
- 以下のアプリ（App Store でインストール）:
  - [Tailscale](https://tailscale.com/) - VPN
  - SSH/Mosh クライアント（後述の推奨アプリから選択）

### 使用するツール

| ツール                | 役割                  | 解決する課題                                        |
| --------------------- | --------------------- | --------------------------------------------------- |
| **Tailscale**         | ピアツーピア VPN      | ポート開放不要で外出先から自宅 Mac に安全アクセス   |
| **Shellfish / Blink** | SSH/Mosh クライアント | iPhone で GUI 操作、タップで接続、Mosh 標準サポート |
| **Mosh**              | モバイルシェル        | Wi-Fi↔LTE 切替時も接続維持、遅延があっても快適入力 |
| **tmux**              | セッション管理        | 接続切断後も作業継続、複数デバイスで画面共有        |

#### 接続の仕組み

```markdown
iPhone (Shellfish または Blink Shell)
↓
Tailscale VPN (ピアツーピア接続)
↓
macOS（開発マシン）
├─ SSH または Mosh サーバー
└─ tmux セッション（永続化、マルチデバイス共有）
```

各レイヤーが連携して、モバイル環境でも安定したリモート開発を実現します：

| レイヤー         | 技術              | 役割                          |
| ---------------- | ----------------- | ----------------------------- |
| **クライアント** | Shellfish / Blink | タップで接続、GUI 管理        |
| **ネットワーク** | Tailscale         | 安全な接続、NAT 越え、固定 IP |
| **プロトコル**   | Mosh              | 接続維持、ローカルエコー      |
| **セッション**   | tmux              | 切断後も継続、画面共有        |

## セットアップ手順

### 1. macOS 側の設定

#### 1.1 必要なパッケージのインストール

Homebrew を使って macOS に必要なツールをインストールします：

```bash
# Tailscale (VPN) のインストール
brew install --cask tailscale-app

# Mosh (モバイルシェル) のインストール
brew install mosh

# tmux (ターミナルマルチプレクサ) のインストール
brew install tmux
```

#### 1.2 tmux の設定（推奨）

tmux の設定ファイルを作成します。以下は Claude Code や AI ツールとの使用に最適化された設定です。

`~/.tmux.conf` に以下の内容を保存してください：

```bash
# ====================================================================================
# 基本設定 - 2025年推奨構成
# ====================================================================================

# Terminal Type: tmux-256color（現代的な推奨、tmux 2.6+）
# - screen-256color より優れた点：
#   - Italics サポート（screen は非対応）
#   - より正確な色再現
#   - CJK（日本語）文字の改善された処理
set -g default-terminal "tmux-256color"

# True Color（24-bit RGB）サポート
# Modern tmux 3.2+ の推奨方法
set -as terminal-features ",*:RGB"
set-option -sa terminal-overrides ",tmux*:Tc"

# マウスサポート（iOS でのタッチ操作に必須）
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

**この設定の利点**：

- **Claude Code 対応**[^6]: 500000 行のスクロールバックで長い出力を完全に確認可能
- **True Color**[^5]: 最新のカラースキームが正しく表示される（tmux-256color を使用）
- **CJK 文字最適化**: 日本語文字の表示品質向上
- **iOS 最適化**: タッチ操作、キーボード位置を考慮した設計

#### 1.3 リモートログインの有効化

macOS のリモートログイン（SSH サーバー）を有効化します：

1. システム設定を開く
2. **一般** > **共有** を選択
3. **リモートログイン** をオンにする
4. アクセスを許可するユーザーを選択（推奨: 管理者のみ）

#### 1.4 Tailscale のセットアップ

```bash
# Tailscale アプリを起動
open -a Tailscale
```

1. Tailscale アプリが起動したら、指示に従ってログイン
2. SSO プロバイダー（Google、GitHub など）で認証
3. 接続が完了したら、**macOS に割り当てられた Tailscale IP アドレス**を確認:

```bash
tailscale ip -4
```

**出力例**: `100.64.1.2`

**重要**: この IP アドレスは macOS（接続先サーバー）の Tailscale IP です。後の手順で使用するため、**必ずメモしておいてください**。

**補足**: Tailscale は各デバイスに `100.x.y.z` 形式の固定 IP を自動割り当てします。この IP は、デバイスが Tailscale ネットワークに参加している限り変わりません。

#### 1.5 Tailscale の 2FA 有効化（推奨）

1. [Tailscale 管理画面](https://login.tailscale.com/admin) にアクセス
2. **Settings** > **Users** から自分のアカウントを選択
3. **Two-factor authentication** を有効化

### 2. iPhone 側の設定

#### 2.1 SSH クライアントの選択

iPhone/iPad 用の SSH クライアントは複数ありますが、2025 年 11 月時点での推奨アプリは以下の通りです。

##### **推奨: Shellfish (Secure ShellFish)**[^1]

**特徴**：

- ✅ **優れた tmux サポート**: セッションのサムネイルプレビュー、2025 年も継続的に改善
- ✅ **Mosh ネイティブサポート**: 接続の安定性が高い
- ✅ **Files アプリ統合**: SSH サーバーを iOS の Files アプリに直接統合
- ✅ **Handoff 対応**: デバイス間でセッション移行が可能（iPhone ↔ iPad ↔ Mac）
- ✅ **iCloud Keychain 同期**: サーバー設定を自動同期
- ✅ **買い切り可能**: $29.99 で永久使用可能（月額 $2.99、年額 $14.99 も選択可）

**推奨理由**：

- 長期的なコストパフォーマンスが高い（買い切り）
- tmux との統合が優秀で、Claude Code の長い出力も問題なく扱える
- 開発者のサポートが早い

##### **代替案: Blink Shell**[^2]

**特徴**：

- ✅ **Mosh の完全サポート**: デバイス再起動後も接続維持
- ✅ **オープンソース**: コミュニティによる継続的な改善
- ✅ **Blink Code**: ブラウザ版 VSCode 統合
- ✅ **高度なカスタマイズ**: テーマ、フォント、レイアウトの自由度が高い
- ✅ **iPad マルチタスク最適化**: Split Screen / Slide Over に優れた対応

**価格**: 年額 $19.99（買い切りオプションなし）

**推奨理由**：

- VSCode をモバイルで使いたい場合に最適

#### 2.2 必要なアプリのインストール

App Store から以下をインストール:

1. **Tailscale** - VPN 接続用
2. **Shellfish** または **Blink Shell** - SSH/Mosh クライアント（上記の推奨を参照）

#### 2.3 Tailscale のセットアップ

1. Tailscale アプリを起動
2. macOS と同じアカウントでログイン
3. 接続が完了すると、macOS と同じ VPN ネットワークに参加

#### 2.4 SSH クライアントのセットアップ

選択した SSH クライアント（Shellfish または Blink Shell）に SSH 接続を設定します。

##### **Shellfish の場合**

1. Shellfish アプリを起動
2. **+** ボタンをタップして新規ホストを追加
3. 以下を入力:
   - **Label**: `mac`（任意の接続名）
   - **Hostname**: `<macOS の Tailscale IP>`（例: `100.64.1.2`）
     - これは手順 1.4 でメモした macOS 側の IP アドレスです
   - **User**: `<macOS のユーザー名>`
     - macOS で `whoami` コマンドを実行すると確認できます
   - **Port**: `22`（SSH のデフォルトポート）
4. **Save** をタップして保存

Mosh を使用する場合:

1. 保存したホストをタップして **Edit** を選択
2. **Advanced** セクションで **Use Mosh** をオンにする
3. **Save** をタップ

##### **Blink Shell の場合**

1. Blink Shell アプリを起動
2. `config` コマンドを入力してホストを追加
3. **Hosts** タブで **+** をタップ
4. 以下を入力:
   - **Host**: `mac`（任意のホスト名）
   - **HostName**: `<macOS の Tailscale IP>`（例: `100.64.1.2`）
   - **User**: `<macOS のユーザー名>`
   - **Port**: `22`
5. **Save** をタップ

Mosh を使用する場合:

- Blink Shell は Mosh をネイティブサポートしているため、`mosh mac` と入力するだけで Mosh 接続できます

これで設定が完了しました。ホスト名をタップするだけで接続できるようになりました。

### 3. 接続テスト

#### 3.1 SSH 接続のテスト

SSH クライアントで macOS への接続をテストします。

**接続手順:**

1. SSH クライアント（Shellfish または Blink Shell）を開く
2. 先ほど作成した `mac` ホストをタップ
3. 初回接続時は、ホストキーの確認ダイアログが表示されるので **Continue** または **Trust** をタップ
4. パスワード入力が求められた場合は、macOS のユーザーパスワードを入力
5. 接続に成功すると、macOS のターミナル画面が表示されます

**確認:**
接続後、以下のコマンドでホスト名を確認できます:

```bash
hostname
```

macOS のホスト名が表示されれば成功です。

#### 3.2 Mosh 接続の利点

Mosh は SSH よりもモバイル環境に適した接続方式です。すでに設定済み（手順 2.4 参照）であれば、以下の利点を体験できます：

**Mosh の利点:**

- 📱 **接続の安定性**: iPhone をスリープして数分後に復帰しても接続が維持されている
- 🔄 **ネットワーク切り替え**: Wi-Fi から LTE に切り替えても接続が継続される
- ⚡ **ローカルエコー**: 文字を入力すると即座にローカルでエコーされる（遅延のある回線でも快適）
- 🔌 **デバイス再起動**: Blink Shell の場合、デバイス再起動後も接続を維持（Mosh の完全実装）

**Mosh vs SSH の使い分け:**

- **Mosh 推奨**: 移動中、不安定な回線、長時間セッション
- **SSH でも OK**: 自宅の安定した Wi-Fi、短時間の作業

#### 3.3 tmux セッションの開始

macOS に接続できたら、tmux セッションを開始します。または既存のセッションに参加することもできます：

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

tmux を使うと、iPhone と Mac で同じターミナルセッションを共有できます。

1. **セッションを開始する（どちらかのデバイスで）:**

   macOS または iPhone から macOS に接続して tmux セッションを作成:

   ```bash
   tmux new -s dev
   ```

2. **もう一方のデバイスから参加する:**
   - iPhone の場合: SSH クライアント（Shellfish/Blink Shell）で `mac` ホストをタップして接続
   - Mac の場合: ターミナルで `ssh <Tailscale IP>` または `mosh <Tailscale IP>` で接続

   接続後、以下のコマンドで同じセッションに参加:

   ```bash
   tmux attach -t dev
   ```

3. **同期された画面:**

   どちらのデバイスで入力しても、両方の画面に即座に反映されます。
   これにより、iPhone で作業を開始し、Mac で続きを行う、といった使い方が可能です。

### tmux の便利な機能

上記の設定により、以下の機能が有効になります：

- **マウスサポート**: タッチ操作でペインを選択可能
- **vi キーバインド**: スクロールモードで vim のキーが使える
- **自動番号振り直し**: ウィンドウを閉じても番号が詰まる
- **ESC 遅延なし**: vim 使用時の遅延を解消

必要に応じて、`~/.tmux.conf` に設定を追加してカスタマイズできます。

## トラブルシューティング

### Tailscale で macOS が見つからない

**原因**: 両デバイスで異なるアカウントを使用している、または接続が確立されていない

**解決方法**:

1. 両デバイスで同じ Tailscale アカウントにログインしているか確認
2. macOS で Tailscale の接続状態を確認:
   ```bash
   tailscale status
   ```
3. macOS でリモートログインが有効になっていることを確認
4. iPhone の Tailscale アプリで接続状態を確認

### Mosh で接続できない（`command not found: mosh-server`）

**原因**: SSH 経由で実行されるコマンドで PATH が正しく設定されていない

SSH クライアントが Mosh 接続を試みる際、以下のエラーが表示される場合：

```
Command executed with error: zsh:1: command not found: mosh-server
No response from Mosh server
```

これは、SSH の non-interactive shell で実行される際に、`.zprofile` や `.zshrc` が読み込まれず、Homebrew の PATH が設定されていないことが原因です。

**解決方法**:

1. `~/.zshenv` を作成して Homebrew の PATH を追加:

   ```bash
   # Homebrew
   eval "$(/opt/homebrew/bin/brew shellenv)"
   ```

2. 設定後、新しいシェルで確認:

   ```bash
   zsh -c 'which mosh-server'
   ```

   `/opt/homebrew/bin/mosh-server` と表示されれば成功です。

3. SSH クライアントから再度 Mosh 接続を試してください。

### その他の Mosh 接続の問題

**原因**: Mosh がインストールされていない、またはファイアウォールで UDP ポートがブロックされている

**解決方法**:

1. macOS 側で Mosh がインストールされているか確認:
   ```bash
   which mosh-server
   ```
2. Tailscale 経由の接続なので通常ファイアウォールは問題ないが、念のため確認:
   - システム設定 > ネットワーク > ファイアウォール
3. SSH で接続できるか確認（Mosh は SSH 経由で初期接続を行う）

### SSH で "Permission denied" エラー

**原因**: リモートログインが有効になっていない、またはユーザー権限の問題

**解決方法**:

1. macOS でリモートログインが有効か確認:
   - システム設定 > 一般 > 共有 > リモートログイン
2. 正しいユーザー名を使用しているか確認:
   ```bash
   # macOS で現在のユーザー名を確認
   whoami
   ```

### tmux セッションが見つからない

**原因**: セッションがまだ作成されていない、またはセッション名が間違っている

**解決方法**:

1. 既存のセッション一覧を確認:
   ```bash
   tmux ls
   ```
2. セッションが存在しない場合は新規作成:
   ```bash
   tmux new -s dev
   ```

## 応用編：VS Code on Browser

### code-server のセットアップ

VS Code を iPhone の Safari で使用したい場合:

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

お疲れさまでした。これで iPhone から macOS に安全にリモートアクセスし、開発作業ができる環境が整いました。

ここまでで構築した環境は、まさに AI 時代の開発スタイル――モバイルで、常時接続で、途切れないという開発環境を実現できるはずです。
Tailscale の安全な VPN 接続、Mosh の安定したセッション維持、tmux の共有機能を組み合わせることで、AI ツールとどこからでも共同作業が可能になります。

AI アシスト開発とモバイル中心のワークフローは、生産性の新しい形を提示しています。
通勤中のデバッグ、カフェでのテスト実行、外出先からの即時デプロイ——どの場面でも、途切れずに「開発の流れ」を保つことができます。

セキュリティ設定（Tailscale の 2FA 有効化など）を忘れずに行い、快適なモバイル開発ライフをお楽しみください。

---

**免責事項**: この記事の情報は 2025 年 11 月時点のものです。ソフトウェアのバージョンアップや仕様変更により、内容が古くなる可能性があります。最新の情報は各ツールの公式ドキュメントをご確認ください。

---

[^1]: **Shellfish (Secure ShellFish)**: [App Store](https://apps.apple.com/app/ssh-client-secure-shellfish/id1336634154) | [公式サイト](https://secureshellfish.app/) | MacStories "Secure ShellFish Review" (2019) | 2025 年アップデート: DECSLRM 対応による tmux 改善

[^2]: **Blink Shell**: [App Store](https://apps.apple.com/app/blink-shell-build-code/id1594898306) | [公式サイト](https://blink.sh/) | [GitHub](https://github.com/blinksh/blink) | オープンソース、5 年以上 AppStore でトップの開発者ツール

[^3]: [GitHub Issue: google-gemini/gemini-cli #10349](https://github.com/google-gemini/gemini-cli/issues/10349) - "Automatic scrolling to input on iOS Termius prevents viewing previous content" (2025 年 10 月)

[^4]: [Termius - iOS バックグラウンド制限](https://support.termius.com/hc/en-us/articles/900006226306)

[^5]: **Terminal Type 設定**: tmux-256color vs screen-256color については、コミュニティで広く議論されています。[Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/1045/getting-256-colors-to-work-in-tmux)、[Stack Overflow](https://stackoverflow.com/questions/10158508/lose-vim-colorscheme-in-tmux) など複数のフォーラムで tmux-256color が推奨されています。

[^6]: [Brian P. Hogan "Working with Claude Code"](https://bphogan.com/2025/06/19/2025-06-19-claude-code-tips/) (2025 年 6 月) - 推奨: "big scrollback buffer" for Claude Code | 関連: [pchalasani/claude-code-tools](https://github.com/pchalasani/claude-code-tools) - Claude Code + tmux の統合ツール | [ooloth/dotfiles](https://github.com/ooloth/dotfiles) - Claude Code 対応の実践的な dotfiles

[^7]: **tmux 公式**: [tmux Wiki](https://github.com/tmux/tmux/wiki) | [tmux 3.5 リリースノート](https://github.com/tmux/tmux/blob/master/CHANGES) (2024 年 10 月) | [tmux FAQ](https://github.com/tmux/tmux/wiki/FAQ)

[^8]: **Mosh 公式**: [公式サイト](https://mosh.org/) | [GitHub リポジトリ](https://github.com/mobile-shell/mosh) | [GitHub Issue #122](https://github.com/mobile-shell/mosh/issues/122) - スクロールバック制限（解決策: tmux との併用を推奨）

[^9]: **Tailscale 公式**: [公式サイト](https://tailscale.com/) | [セキュリティベストプラクティス](https://tailscale.com/kb/) | [管理コンソール](https://login.tailscale.com/admin)
