---
title: "AI時代のiPhoneからmacOSへのリモート開発：Tailscale + Termius + Mosh + tmux"
description: "外出先のiPhoneからmacOSに安全にリモートアクセスし、快適な開発作業を可能にする環境構築ガイドです。"
tags: ["Software Development"]
publishedAt: "2025-11-01T12:00:00.000Z"
updatedAt: "2025-11-01T12:00:00.000Z"
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

- [Homebrew（macOS向けのパッケージ管理ツール）](https://brew.sh/)
  - [Tailscale](https://tailscale.com/)
  - [Mosh](https://mosh.org/)
  - [tmux](https://github.com/tmux/tmux/wiki)

### iPhone/iOS デバイス

- App Store へのアクセス
  - [Tailscale](https://tailscale.com/)
  - [Termius](https://termius.com/)

### 使用するツール

| ツール        | 役割                  | 解決する課題                                        |
| ------------- | --------------------- | --------------------------------------------------- |
| **Tailscale** | ピアツーピア VPN      | ポート開放不要で外出先から自宅 Mac に安全アクセス   |
| **Termius**   | SSH/Mosh クライアント | iPhone で GUI 操作、タップで接続、Mosh 標準サポート |
| **Mosh**      | モバイルシェル        | Wi-Fi↔LTE 切替時も接続維持、遅延があっても快適入力 |
| **tmux**      | セッション管理        | 接続切断後も作業継続、複数デバイスで画面共有        |

#### 接続の仕組み

```
iPhone (Termius)
    ↓
 Tailscale VPN (ピアツーピア接続)
    ↓
macOS（開発マシン）
    ├─ SSH または Mosh サーバー（Termius で選択）
    └─ tmux セッション（永続化、マルチデバイス共有）
```

各レイヤーが連携して、モバイル環境でも安定したリモート開発を実現します：

| レイヤー         | 技術      | 役割                          |
| ---------------- | --------- | ----------------------------- |
| **クライアント** | Termius   | タップで接続、GUI 管理        |
| **ネットワーク** | Tailscale | 安全な接続、NAT 越え、固定 IP |
| **プロトコル**   | Mosh      | 接続維持、ローカルエコー      |
| **セッション**   | tmux      | 切断後も継続、画面共有        |

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

#### 1.2 tmux の設定（オプショナル）

tmux の設定ファイルを作成します。以下の内容を `~/.tmux.conf` に保存してください：

```bash
# マウスサポートを有効化（タッチ操作でペイン選択可能）
set -g mouse on

# viモードキーバインド
setw -g mode-keys vi

# ウィンドウ番号を自動で詰める
set -g renumber-windows on

# ESC入力時の遅延をなくす（vim使用時の快適性向上）
set -s escape-time 0

# ステータスバーの色設定
set -g status-style bg=black,fg=white

# 256色ターミナルを有効化
set -g default-terminal "screen-256color"
```

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

#### 2.1 必要なアプリのインストール

App Store から以下をインストール:

1. **Tailscale** - VPN 接続用
2. **Termius** - SSH/Mosh クライアント

#### 2.2 Tailscale のセットアップ

1. Tailscale アプリを起動
2. macOS と同じアカウントでログイン
3. 接続が完了すると、macOS と同じ VPN ネットワークに参加

#### 2.3 Termius のセットアップ

Termius は GUI ベースのアプリなので、すべての設定を画面上で行います。

**ホストの追加手順:**

1. Termius アプリを起動
2. 下部の **Hosts** タブをタップ
3. 右上の **+** ボタンをタップして新規ホストを追加
4. **New Host** 画面で以下を入力:
   - **Alias**: `mac`（任意の接続名、わかりやすい名前を付ける）
   - **Hostname**: `<macOS の Tailscale IP>`
     - 例: `100.64.1.2`
     - **これは手順 1.3 でメモした macOS 側の IP アドレスです**
   - **Username**: `<macOS のユーザー名>`
     - macOS で `whoami` コマンドを実行すると確認できます
   - **Port**: `22`（SSH のデフォルトポート、変更不要）
   - **Protocol**: `SSH`（初期設定、後で Mosh に変更可能）
5. 右上の **Save** をタップして保存

これで Hosts 一覧に `mac` が表示されます。タップするだけで接続できるようになりました。

### 3. 接続テスト

#### 3.1 SSH 接続のテスト

Termius アプリで macOS への接続をテストします。

**接続手順:**

1. Termius アプリを開く
2. **Hosts** タブから先ほど作成した `mac` をタップ
3. 初回接続時は、ホストキーの確認ダイアログが表示されるので **Continue** または **Trust** をタップ
4. パスワード入力が求められた場合は、macOS のユーザーパスワードを入力
5. 接続に成功すると、macOS のターミナル画面が表示されます

**確認:**
接続後、以下のコマンドでホスト名を確認できます:

```bash
hostname
```

macOS のホスト名が表示されれば成功です。

#### 3.2 Mosh 接続のセットアップ

Mosh は SSH よりもモバイル環境に適した接続方式です。通常の SSH 接続でも問題ない場合はこのステップはスキップ可能です。

**Termius での Mosh 設定手順:**

Termius は Mosh を標準サポートしているため、GUI で簡単に設定できます。

1. Termius アプリで **Hosts** タブを開く
2. `mac` ホストを **長押し**（または左スワイプ）して **Edit** を選択
3. **Connection** セクションで以下を設定:
   - **Protocol**: `Mosh` を選択（デフォルトは `SSH`）
4. **Save** をタップして保存

**初回接続時の動作:**

1. `mac` ホストをタップして接続
2. Termius が自動的に SSH 経由で macOS に接続
3. パスワードまたは SSH キーで認証
4. mosh-server が自動的にインストールされる（macOS 側、root 権限不要）
5. UDP 接続（ポート 60000-61000）に切り替わる
6. 接続完了

**2 回目以降の接続:**

`mac` ホストをタップするだけで、自動的に Mosh 経由で接続されます。

**Mosh の利点を確認:**

接続後、以下の操作で Mosh の特徴を体験できます:

- iPhone をスリープして数分後に復帰 → 接続が維持されている
- Wi-Fi から LTE に切り替え → 接続が継続される
- 文字を入力 → 即座にローカルエコーされる（遅延のある回線でも快適）

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
   - iPhone の場合: Termius で `mac` ホストをタップして接続
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

Termius が Mosh 接続を試みる際、以下のエラーが表示される場合：

```
Command executed with error: zsh:1: command not found: mosh-server
No response from Mosh server
```

これは、SSH の non-interactive shell で実行される際に、`.zprofile` や `.zshrc` が読み込まれず、Homebrew の PATH が設定されていないことが原因です。

**解決方法**:

1. 手動で設定する場合:

   `~/.zshenv` を作成して Homebrew の PATH を追加:

   ```bash
   # Homebrew
   eval "$(/opt/homebrew/bin/brew shellenv)"
   ```

2. 設定後、新しいシェルで確認:

   ```bash
   zsh -c 'which mosh-server'
   ```

   `/opt/homebrew/bin/mosh-server` と表示されれば成功です。

3. Termius から再度 Mosh 接続を試してください。

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

ここまでで構築した環境は、まさにAI時代の開発スタイル――モバイルで、常時接続で、途切れないという開発環境を実現できるはずです。
Tailscaleの安全なVPN接続、Moshの安定したセッション維持、tmuxの共有機能を組み合わせることで、AIツールとどこからでも共同作業が可能になります。

AIアシスト開発とモバイル中心のワークフローは、生産性の新しい形を提示しています。
通勤中のデバッグ、カフェでのテスト実行、外出先からの即時デプロイ——どの場面でも、途切れずに「開発の流れ」を保つことができます。

セキュリティ設定（Tailscale の 2FA 有効化など）を忘れずに行い、快適なモバイル開発ライフをお楽しみください。
