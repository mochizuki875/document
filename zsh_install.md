入れると便利になるっぽいのでメモ

### zshの導入
https://qiita.com/nenokido2000/items/763a4af5c161ff5ede68

### zshのフレームワークpreztoの導入
https://qiita.com/rspepe/items/9e30e698c4fc3e90891d
https://andooown.com/blog/post/2018/02/prezto/

### zshのプラグイン管理ツールzplug
http://post.simplie.jp/posts/59
https://sanoto-nittc.hatenablog.com/entry/2017/12/16/213735


~/.zshrcに以下を追記 
~~~
#
# Executes commands at the start of an interactive session.
#
# Authors:
#   Sorin Ionescu <sorin.ionescu@gmail.com>
#

# Source Prezto.
if [[ -s "${ZDOTDIR:-$HOME}/.zprezto/init.zsh" ]]; then
  source "${ZDOTDIR:-$HOME}/.zprezto/init.zsh"
fi

# Customize to your needs...


# zplugが無ければgitからclone
if [[ ! -d ~/.zplug ]];then
  git clone https://github.com/zplug/zplug ~/.zplug
fi

# zplugを使う
source ~/.zplug/init.zsh

# zsh の補完を有効化
autoload -Uz compinit && compinit

# インストールするプラグインを定義する

# 入力中のコマンドをコマンド履歴から推測し、候補として表示するプラグイン
# Zshの候補選択を拡張するプラグイン
# プロンプトのコマンドを色づけするプラグイン
zplug 'zsh-users/zsh-autosuggestions'
zplug 'zsh-users/zsh-completions'
zplug 'zsh-users/zsh-syntax-highlighting'

# git の補完を効かせる
# 補完＆エイリアスが追加される
zplug "plugins/git",   from:oh-my-zsh
zplug "peterhurford/git-aliases.zsh"

# インストールされていないプラグインはインストールする
if ! zplug check --verbose; then
  printf 'Install? [y/N]: '
  if read -q; then
    echo; zplug install
  fi
fi

# コマンドをリンクして、PATH に追加し、プラグインは読み込む
zplug load --verbose
~~~


