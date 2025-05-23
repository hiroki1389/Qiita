# Gitはじめてガイド：Mac/Windows/Ubuntu 対応 初心者向けセットアップ＆基本操作まとめ

# 🚀 はじめに

この記事では，これからGitを使って開発を始めたい方向けに，

- Mac / Windows / Ubuntu での Git 環境構築
- GitHubとのSSH接続設定
- よく使うGitコマンドの具体例（コミット，プッシュ等）
- 間違えたときのリセット方法や履歴の見方

などを**一気通貫で解説**します．  
最初はこれを覚えておけば問題ないです．

# 目次
1. [🛠 Gitのインストール](#anchor1)
1. [⚙️ Git 初期セットアップ](#anchor2)
1. [🔰 基本操作](#anchor3)
1. [🧑‍💻 複数端末での開発](#anchor4)
1. [🙈 .gitignoreの作成と適用](#anchor5)
1. [🔄 コミット履歴の確認・取り消し](#anchor6)
1. [✅ まとめ](#anchor7)

<a id="anchor1"></a>
#  1. 🛠 Gitのインストール

## 🍎 Mac
Homebrewが未インストールの場合は以下を実行：
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
Gitのインストール：
```bash
brew install git
```
Gitのバージョン確認（インストール成功の確認）：
```bash
git --version
```

## 🪟 Windows
- [Git for Windows](https://git-scm.com/download/win) をダウンロードし，Git Bash をインストール．
- Git Bash で以下を実行：
```bash
git --version
```

## 🐧 Ubuntu
Gitのインストール：
```bash
sudo apt update
sudo apt install git -y
git --version
```

<a id="anchor2"></a>
# 2. ⚙️ Git 初期セットアップ

## ユーザー名・メールの設定
```bash
git config --global user.name "あなたのユーザー名"
git config --global user.email "あなたのメールアドレス"
```

## SSHキーの生成・GitHub登録

### 🔧 ローカル環境用（GitHub CLIを利用）
```bash
brew install gh
gh auth login
```
選択肢：GitHub.com → SSH → Y → Webブラウザ → 指示に従う．

### 🐳 Docker環境用（手動設定）
```bash
ssh-keygen -t ed25519 -C "あなたのメールアドレス"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```
ブラウザで GitHub にログインし，[SSHキー設定ページ](https://github.com/settings/keys) に公開鍵を登録．

## 接続テスト
```bash
ssh -T git@github.com
```
`Hi ユーザー名! You've successfully authenticated...` のような出力があれば成功です（標準エラーに出ることもありますが正常です）．


<a id="anchor3"></a>
# 3. 🔰 基本操作

## リポジトリの初期化
```bash
git init
```

## 状態確認（変更されたファイルなど）
```bash
git status
```

## ファイル追加・コミット
ステージング（コミットするファイルの追加）
```bash
git add ファイル名
git add .  # すべての変更ファイルを追加
```

コミット（ステージングされたファイルを履歴に記録）
```bash
git commit -m "コメント"
```


## GitHub CLI（gh）でリモートリポジトリ作成およびプッシュ（GitHubにアップロード）
```bash
gh repo create {リポジトリ名} --private --source=. --remote={リモート名} --push
```
- {リポジトリ名}：GitHubに作成するリポジトリの名前．プロジェクト名などを付けると分かりやすいです．
- {リモート名}：ローカルからこのリポジトリにアクセスする際の「あだ名」です．詳しくは応用編で説明しますが，基本的には origin とするのが一般的です．
- --push：これつけると，リモート作成後にそのまま初回 push まで実行され，今後は"git push"だけでプッシュできます．

### 例
```bash
gh repo create testRepository --private --source=. --remote=origin --push
```

## 二回目以降のプッシュ
```bash
git push
```

## 初回〜2回目以降における，ステージング〜プッシュ例

### 初回
```bash
git add .
git commit -m "Initial Commit"
gh repo create testRepository --private --source=. --remote=origin --push
```

### 2回目以降
```bash
git add .
git commit -m "通信機能の実装"
git push
```

<a id="anchor4"></a>
# 4. 🧑‍💻 複数端末での開発

## 自分のリポジトリをclone（別端末用）
```bash
git clone https://github.com/{ユーザー名}/{リポジトリ名}.git
```
### 例
```bash
git clone https://github.com/hiroki1389/testRepository.git
```

## 他端末の変更を取り込む
```bash
git pull
```

<a id="anchor5"></a>
# 5. 🙈 .gitignoreの作成と適用
Gitにあげたくないファイルを".gitignore"に記述し，プロジェクトのルートに置くことでGitの管理対象外に設定できる．  
参考テンプレート：[GitHub gitignore リポジトリ](https://github.com/github/gitignore)

## `.gitignore` に後から追加したファイルをGit管理外にする
一度 Git に push したファイルは，後から `.gitignore` に追加しても自動で Git 管理外にはなりません．  
そのため，以下の操作で一度追跡対象から外し，`.gitignore` に従って再追跡するようにします．

```bash
git rm -r --cached .
git add .
git commit -m "Apply .gitignore and untrack ignored files"
```

- `--cached` オプションはファイルをローカルから削除せず，**Git の追跡対象からのみ削除**します．
- `.gitignore` に記載されたファイルは，`git add .` をしても再び Git に追加されません．

> ⚠️ 注意：この操作はファイルを Git の管理から外すだけであり，**これまでに push された履歴からは削除されません**．完全な削除方法は応用編で説明します．

<a id="anchor6"></a>
# 6. 🔄 コミット履歴の確認・取り消し

履歴を確認：
```bash
git log --oneline
```

直前のコミットを1つ戻す（内容は残す）
```bash
git reset --soft HEAD^
```

履歴も内容も完全に戻す
```bash
git reset --hard HEAD^
```

特定の履歴まで戻す
```bash
git reset --hard <コミットID>
```

強制プッシュ（GitHub上の履歴も上書きされる）
```bash
git push origin main --force
```

<a id="anchor7"></a>
# 7. ✅ まとめ

これだけ覚えれば，Gitの基本操作がひと通りこなせます．  
次はブランチ，マージ，プルリクなどチーム開発向けの操作を学びましょう．
