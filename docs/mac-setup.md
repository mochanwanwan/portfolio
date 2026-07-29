# Mac リモート開発セットアップガイド

外出先から自宅の Mac に遠隔接続してコード開発を行い、GitHub 経由で成果を同期するための手順書。

---

## 0. 全体像

```
[外出先のスマホ / PC]
        │
        │ ① 遠隔接続（SSH / 画面共有）
        ▼
   [自宅の Mac]  ←── ここで開発する
        │
        │ ② GitHub 連携（clone / commit / push / pull）
        ▼
    [GitHub]  ←── mochanwanwan アカウントに集約
        │
        │ ③ すでに連携済み
        ▼
[Claude Code on the web]
```

①がネットワークの話、②③が GitHub の話。**②を正しく設定すれば、Mac で書いたコードもクラウドで書いたコードも同じ場所に集まる。**

---

## 1. アカウントを1つに集約する

### 現状の問題

3つの GitHub アカウントが混在しており、これが「push したはずのリポジトリが見つからない」原因になっていた。

| アカウント | リポジトリ数 | 最終更新 | Claude 連携 |
| --- | --- | --- | --- |
| `MICHANWANWAN` | 14 | **数十分前**（VTT） | ❌ 未連携 |
| `mochanwanwan` | 15 | 2週間前（lp-1） | ✅ 連携済み |
| `primeprojecta-beep` | （コミット作者設定のみ） | — | — |

`MICHANWANWAN` と `mochanwanwan` は **i と o が1文字違うだけの別アカウント**。見間違えやすいので注意。

⚠️ **実際に開発が動いているのは `MICHANWANWAN` 側**（AI-kenshu、DOUGA-JIDOU-TOUKOU-2、PSB2 など直近1ヶ月で6件更新）。`mochanwanwan` は `portfolio` を除くとほぼ休眠状態。

### ⚠️ 名前衝突リポジトリ

以下は**両方のアカウントに同名で存在する**。同一アカウントに同名リポジトリは共存できないため、これらを移管する際は事前にどちらかをリネームする必要がある。

- `lp-1`
- `training-service-lp`

`VTT` は `mochanwanwan` 側に存在しないため、そのまま移管できる。

### 手順 1-1: `VTT` を `mochanwanwan` へ移管する

#### 方法A: コマンドで実行する（推奨）

Mac の `gh` は `MICHANWANWAN` として認証済み（=`VTT` の管理権限あり）なので、ターミナルから直接実行できる。ブラウザでのログインし直しが不要。

```bash
# 念のためアクティブアカウントを確認（MICHANWANWAN であること）
gh auth status

# 移管を実行する
gh api -X POST repos/MICHANWANWAN/VTT/transfer -f new_owner=mochanwanwan
```

#### 方法B: ブラウザで実行する

1. ブラウザで **`MICHANWANWAN` としてログイン**する
   - プロフィール画面に **「Follow」** ボタンが出ていたら、それは**別アカウントで他人として見ている状態**。自分のページなら「Edit profile」が出る
2. `https://github.com/MICHANWANWAN/VTT/settings` を開く
3. ページ最下部の **Danger Zone** → **Transfer ownership**
4. New owner に `mochanwanwan` と入力
5. 確認のためリポジトリ名 `VTT` を入力して実行

#### 移管後: 受け取り側で承認する

個人アカウント間の移管は、**受け取り側（`mochanwanwan`）の承認**が必要。招待メールが届くか GitHub 上に通知が出るので承認する。

承認が完了すると URL は `https://github.com/mochanwanwan/VTT` になる。

### 手順 1-2: Claude から見えることを確認する

移管が完了すると、`mochanwanwan` には Claude の GitHub App が導入済みのため、原則そのまま見えるようになる。

見えない場合は `https://github.com/settings/installations`（`mochanwanwan` でログインした状態）を開き、Claude の **Repository access** に `VTT` が含まれているか確認する。`Only select repositories` になっている場合は `VTT` を選択に追加する。

---

## 2. Mac 側のアカウント整理

### ⚠️ アカウントは「切り替えて使う」ことになる

`VTT` だけを移管したため、Mac から扱うリポジトリは2つのアカウントに分かれている。

| 作業対象 | 必要なアクティブアカウント |
| --- | --- |
| `VTT`、`portfolio` | `mochanwanwan` |
| `AI-kenshu`、`DOUGA-JIDOU-TOUKOU-2`、`PSB2` ほか12件 | `MICHANWANWAN` |

**`gh auth switch` を一度実行して放置すると、もう一方のアカウントのリポジトリへ push できなくなる。** `gh` は「アクティブなアカウント」のトークンを git に渡すため。

`Permission denied` や `403` が出たら、まず今どちらがアクティブか疑うこと。

```bash
# 今どちらがアクティブか確認する
gh auth status

# VTT / portfolio を触る前
gh auth switch --user mochanwanwan

# MICHANWANWAN 側のリポジトリを触る前
gh auth switch --user MICHANWANWAN
```

> **切り替えが面倒になったら**
> `MICHANWANWAN` にも Claude の GitHub App を導入すれば、リポジトリを動かさずに両アカウントを
> 扱えるようになる。その場合でも Mac 側の `gh` 切り替えは必要だが、リポジトリの移管作業は不要。

### コミットの署名を修正する

現状 `primeprojecta@gmail.com` になっており、コミットが GitHub 上のアカウントへ紐付いていない。**普段いちばん使うアカウントに合わせて**グローバル設定する。

```bash
git config --global user.name "mochanwanwan"
git config --global user.email "174401109+mochanwanwan@users.noreply.github.com"

# 確認
git config --global --get user.name
git config --global --get user.email
```

> **なぜこのメールアドレスなのか**
> `174401109+mochanwanwan@users.noreply.github.com` は GitHub が発行する非公開メール。
> 実際のアドレスを公開リポジトリに晒さずに、コミットを GitHub アカウントへ正しく紐付けられる。

`MICHANWANWAN` 側のリポジトリでは、そのフォルダ内だけ設定を上書きできる（`--global` を付けない）。

```bash
cd ~/path/to/AI-kenshu
git config user.name "MICHANWANWAN"
git config user.email "MICHANWANWAN の非公開メール"
```

> `MICHANWANWAN` の非公開メールは `https://github.com/settings/emails`（当該アカウントでログイン）
> の "Keep my email addresses private" 欄で確認できる。

なお、過去のコミットの作者は後から変えられない。今後のコミットから正しくなる。

---

## 3. Mac 側のリモート URL を張り替える

移管によりリポジトリの所在が変わったため、Mac のローカルリポジトリに教え直す。

**`VTT` の場所: `/Users/mit/projects/VTT`**

以下を**1行ずつ**実行する。

```bash
cd /Users/mit/projects/VTT
```

```bash
git remote -v
```

```bash
git remote set-url origin https://github.com/mochanwanwan/VTT.git
```

```bash
gh auth switch --user mochanwanwan
```

```bash
git fetch origin
```

| コマンド | 目的 |
| --- | --- |
| `git remote -v` | 張り替え前の送り先を確認 |
| `git remote set-url` | 移管後の新しい場所を教える |
| `gh auth switch` | `VTT` の新しい持ち主 `mochanwanwan` の権限に切り替える |
| `git fetch origin` | 実際に通信できるか検証 |

`git fetch origin` がエラーなく完了すれば成功。

> GitHub は旧 URL から自動リダイレクトしてくれるが、それに頼ると後で混乱するため明示的に張り替える。

### リポジトリの場所

| リポジトリ | Mac 上のパス | GitHub |
| --- | --- | --- |
| `VTT` | `/Users/mit/projects/VTT` | `mochanwanwan/VTT` |

> 他のリポジトリの場所が分からなくなったら、以下で探せる。
>
> ```bash
> find ~ -maxdepth 4 -type d -name ".git" -exec dirname {} \; 2>/dev/null
> ```

---

## 4. Mac へのリモート接続

### 4-1: Mac 側で共有を有効にする

| 機能 | 用途 |
| --- | --- |
| **リモートログイン（SSH）** | ターミナル操作。Claude Code を動かすならこれ |
| **画面共有** | Mac のデスクトップをそのまま操作したい場合 |

#### GUI で有効にする（推奨）

| 手順 | 操作 |
| --- | --- |
| 1 |  → **システム設定** |
| 2 | サイドバーの **「一般」** |
| 3 | **「共有」** |
| 4 | **「リモートログイン」のトグルをオン** |

「リモートログイン」右の ⓘ ボタンから、接続を許可するユーザーを確認できる（既定は「すべてのユーザ」）。

有効になったか、ターミナルで確認する。

```bash
nc -z localhost 22 && echo "SSH 有効" || echo "SSH 無効"
```

#### コマンドで有効にする場合

```bash
sudo systemsetup -setremotelogin on
```

⚠️ **現在の macOS では、これは既定で失敗する。**

```
setremotelogin: Turning Remote Login on or off requires Full Disk Access privileges.
```

このコマンドを通すには、**システム設定 → プライバシーとセキュリティ → フルディスクアクセス** でターミナルを許可する必要がある。ターミナルに強い権限を与えることになるため、**上記の GUI トグルで済ませるほうが安全**。

#### 接続に必要な情報を控える

```bash
whoami
```

```bash
scutil --get LocalHostName
```

ここで表示されるユーザー名が SSH 接続時の `ユーザー名@` の部分になる。

### 4-2: Mac がスリープしないようにする

**スリープすると遠隔接続できなくなる。** 外出先から繋ぐ前に必ず設定しておく。

#### 方法A: コマンドで設定する（確実・推奨）

ターミナルで以下を実行する。`sudo` のためログインパスワードの入力を求められる（入力しても画面には表示されないが正常）。

```bash
sudo pmset -c sleep 0 disksleep 0 displaysleep 10 womp 1
```

| 設定 | 効果 |
| --- | --- |
| `sleep 0` | 本体をスリープさせない ← **これが本命** |
| `disksleep 0` | ディスクを停止させない |
| `displaysleep 10` | 画面は10分で消す（本体は起きたまま。省電力・焼き付き対策） |
| `womp 1` | ネットワーク経由での復帰を許可 |

> ### ⚠️ ターミナルへの貼り付け時の注意
>
> **`#` で始まる説明行を一緒に貼り付けてはいけない。**
> zsh の対話プロンプトでは `#` がコメントとして扱われず（`interactive_comments` が既定で無効）、
> `zsh: command not found: #` というエラーになる。
>
> さらに危険なのが、`sudo` の `Password:` 表示中に**貼り付けた次の行がパスワードとして吸い込まれる**こと。
> 複数行をまとめて貼ると、成功したのか失敗したのか判別できなくなる。
>
> **`sudo` を含むコマンドは1行ずつ実行し、パスワードを入力し終えてから次に進むこと。**

設定できたか確認する。

```bash
pmset -g custom
```

`AC Power` の欄が `sleep 0`、`disksleep 0`、`womp 1` になっていれば成功。

> **`-c` はどういう意味か**
> `-c` は「電源アダプタ接続時のみ」という指定。バッテリー駆動時の設定（`-b`）は変更されないため、
> 持ち歩く際のバッテリー消費は今まで通りに保たれる。`-a` にすると両方に適用されるので使わない。

#### MacBook を閉じたまま使う場合

上記だけでは、**画面を閉じるとスリープする**。閉じたまま運用したい場合のみ追加で実行する。

```bash
sudo pmset -c disablesleep 1
```

⚠️ これは電源アダプタ接続時のスリープを完全に無効化する強い設定。**必ず電源アダプタに接続し、通気の良い場所に置くこと**（閉じたまま高負荷が続くと熱がこもる）。

元に戻す場合は以下。

```bash
sudo pmset -c disablesleep 0
```

#### 方法B: GUI で設定する

**システム設定 → バッテリー**（デスクトップ機では **省エネルギー**）を開き、

- **「ディスプレイがオフのときに自動スリープさせない」をオン**
- ノートの場合は電源アダプタに接続しておく

#### 一時的に起こしておくだけでよい場合

設定を変更せず、その場限りで眠らせたくない場合は `caffeinate` を使う。

```bash
# このコマンドを実行している間だけスリープしない（Ctrl+C で解除）
caffeinate -dims

# 特定の処理が終わるまで起こしておく
caffeinate -dims ./長時間かかる処理.sh
```

### 4-3: 外出先から繋ぐ（Tailscale）

⚠️ **ルーターのポート開放（ポートフォワーディング）で SSH を直接インターネットに晒してはいけない。** 22番ポートは常時スキャンされており、総当たり攻撃の標的になる。

代わりに **Tailscale** を使う。Mac とスマホを同じ「仮想 LAN」に入れる仕組みで、ルーター設定の変更が一切不要、通信も暗号化される。個人利用は無料。

#### ステップ1: Mac に Tailscale を入れる

https://tailscale.com/download/macos を開き、**Standalone 版**（`.pkg` 形式のダウンロード）を選ぶ。

> ### ⚠️ App Store 版を選ばないこと
>
> Tailscale には2種類ある。
>
> | 版 | 動作 | 常時稼働サーバー用途 |
> | --- | --- | --- |
> | **Standalone（.pkg）** | システムデーモンとして動作。**ログアウト後も起動時も接続を維持** | ✅ 適する |
> | App Store 版 | ログイン中のユーザーセッション内でのみ動作 | ❌ 不適 |
>
> Mac mini を遠隔開発サーバーとして使う場合、再起動やログアウトで接続が切れると
> 外出先から復旧できなくなる。**必ず Standalone 版を選ぶ。**

#### ステップ2: ログインする

インストール後 Tailscale を起動し、**Log in** からアカウントを作成する（Google / GitHub / Microsoft アカウントが使える）。

**このアカウント情報は必ず控えておくこと。** 外出先の端末でも同じアカウントでログインする必要がある。

#### ステップ3: Mac の Tailscale IP を確認する

メニューバーの Tailscale アイコンをクリックすると、`100.x.x.x` 形式の IP アドレスが表示される。これを控える。

コマンドでも確認できる。

```bash
/Applications/Tailscale.app/Contents/MacOS/Tailscale ip -4
```

#### ステップ4: 外出先の端末に入れる

スマホ（iOS / Android）や持ち出し用 PC にも Tailscale を入れ、**ステップ2と同じアカウント**でログインする。

これで両者が同じ仮想ネットワークに入る。

#### ステップ5: 接続する

外出先の端末から、Mac の Tailscale IP へ SSH する。

```bash
ssh mit@100.x.x.x
```

初回は `Are you sure you want to continue connecting?` と聞かれるので `yes` と入力する。次に Mac のログインパスワードを入力すれば接続完了。

> **前提**: §4-1 の「リモートログイン」がオンになっていること。オフのままだと
> `Connection refused` になる。

#### 接続できないときの切り分け

| 症状 | 原因 | 対処 |
| --- | --- | --- |
| `Connection refused` | Mac 側の SSH が無効 | §4-1 のトグルをオン |
| 応答なし / タイムアウト | Tailscale が繋がっていない | 両端のメニューバーで接続状態を確認 |
| Mac が一覧に出ない | 別アカウントでログインしている | 同一アカウントか確認 |
| 数時間後に繋がらなくなる | Mac がスリープした | §4-2 の `pmset` を確認 |

---

## 5. 日々の作業の流れ

**この順番を守ることが最も重要。**

| 順 | コマンド | 意味 |
| --- | --- | --- |
| ① | `cd /Users/mit/projects/VTT` | 作業フォルダへ移動 |
| ② | `git pull` | **作業前に必ず**最新を取り込む |
| ③ | （編集する） | Claude Code でもエディタでも |
| ④ | `git add -A` | 変更をまとめる |
| ⑤ | `git commit -m "変更内容"` | 記録する |
| ⑥ | `git push` | GitHub へ送る |

```bash
cd /Users/mit/projects/VTT
```

```bash
git pull
```

```bash
git add -A
```

```bash
git commit -m "何を変えたかを簡潔に書く"
```

```bash
git push
```

### 鉄則

- **作業前に `git pull`、作業後に `git push`**
- これを守らないと、Mac とクラウドで別々の変更が育ち、衝突（コンフリクト）が起きる
- 迷ったらまず `git status` — 今どういう状態かを教えてくれる

---

## 6. トラブルシューティング

### `git push` したのに GitHub に無い

まず送り先と送り主を確認する。

```bash
git remote -v      # どこへ送る設定か
gh auth status     # 誰としてログインしているか
git log --oneline -3   # そもそもコミットされているか
```

**アカウントの取り違え**が最も多い原因。`MICHANWANWAN` と `mochanwanwan` は特に紛らわしい。

### `Permission denied` / `403` が出る

アクティブなアカウントに、そのリポジトリへの権限がない。

```bash
gh auth switch --user mochanwanwan
```

### `not a git repository` と出る

そのフォルダはまだ Git 管理されていない。新規に GitHub へ上げる場合は以下。

```bash
cd ~/path/to/フォルダ
git init
git add -A
git commit -m "最初のコミット"

# GitHub 上にリポジトリを作って push まで一気に行う
gh repo create mochanwanwan/リポジトリ名 --private --source=. --push
```

### コミットが GitHub 上で自分の名前にならない

`git config --global user.email` が GitHub に登録されたメールと一致していない。→ **第2章**を再実行する。

なお、過去のコミットの作者は後から変えられない。今後のコミットから正しくなる。

---

## 参考リンク

- [GitHub: リポジトリの移管](https://docs.github.com/ja/repositories/creating-and-managing-repositories/transferring-a-repository)
- [GitHub: コミットメールアドレスの設定](https://docs.github.com/ja/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address)
- [Tailscale](https://tailscale.com/)
- [Claude Code on the web ドキュメント](https://code.claude.com/docs/en/claude-code-on-the-web)
