Reckerリファレンス
==================

<!-- toc -->

* [Recker とは](#recker-とは)
* [環境・導入方法](#環境導入方法)
  * [実行環境・必要プロダクト](#実行環境必要プロダクト)
  * [導入方法](#導入方法)
* [サブコマンド群](#サブコマンド群)
  * [`images` | `images-a`](#images-images-a)
  * [`ps` | `ps-a`](#ps-ps-a)
  * [`rmi` | `rmi-a` [image ids...]](#rmi-rmi-a-image-ids)
  * [`stop` | `stop-a` [container ids...]](#stop-stop-a-container-ids)
  * [`kill` | `kill-a` [container ids...]](#kill-kill-a-container-ids)
  * [`rm` [container ids...]](#rm-container-ids)
  * [`stop-rm` | `stop-rm-a` [container ids...]](#stop-rm-stop-rm-a-container-ids)
  * [`kill-rm` | `kill-rm-a` [container ids...]](#kill-rm-kill-rm-a-container-ids)
  * [`get-container-ip` [container id]](#get-container-ip-container-id)
  * [`get-container-ports` [container id]](#get-container-ports-container-id)
  * [`get-container-port-http` | `get-container-port-ssh` [container id]](#get-container-port-http-get-container-port-ssh-container-id)
  * [`get-containers-names`](#get-containers-names)
  * [`top` [container id]](#top-container-id)
  * [`run` [context directory]](#run-context-directory)
  * [`scaffold` [context name]](#scaffold-context-name)
  * [`build` [context directory]](#build-context-directory)
  * [`bashcompletion`](#bashcompletion)
* [Recker でのイメージビルドのしかた](#recker-でのイメージビルドのしかた)
  * [ひながたからのイメージ定義](#ひながたからのイメージ定義)
  * [ビルドする](#ビルドする)
  * [イメージからコンテナを起動する](#イメージからコンテナを起動する)
* [ビルドから起動までの詳細](#ビルドから起動までの詳細)
  * [ひながたづくり＝`scaffold`](#ひながたづくりscaffold)
  * [ビルド＝`build`](#ビルドbuild)
  * [コンテナ起動＝`run`](#コンテナ起動run)

<!-- toc stop -->



Recker とは
-----------

Recker は [Docker](https://www.docker.com/) のコマンドラインヘルパーツールです。

* 作業対象イメージ・コンテナを一覧画面からカーソルキーで選択する機能
* `TAB` キーでのイメージ・コンテナ名・ID補完機能
* `docker build` の゛土台“（scaffold）作成機能
* `docker run` の簡便化、ポート・リンク・ボリュームの設定サポート機能

をもっています。

用途としては、

* Docker を試したいが `Dockerfile` などが煩雑
* ある程度 Docker を使いこなしているが、コマンドライン作業が煩雑
  * `Dockerfile` にまともなシェルスクリプトが書けない
  * supervisord などを使うのがめんどくさい
  * ポートの割り当て管理がめんどくさい

とお悩みの方の支援を主としています。Recker を使っていてもいつでも Docker に移行できるよう配慮されています。

環境・導入方法
--------------

### 実行環境・必要プロダクト

* Docker 1.6.0 以上  
  たぶん1以上なら動く気がする
* bash 4以上
* [Percol](https://github.com/mooz/percol)
* CentOS 7.x  
  でしか動作確認していません

なお、コンテナのデフォルトベースイメージは CentOS 6 になっています（変更可能）。

### 導入方法

root 権限でインストールする場合は、

```console
# wget -O /usr/local/bin/recker \
  https://raw.githubusercontent.com/hilotech/recker/master/recker
# chmod +x /usr/local/bin/recker
```

で完了です。スクリプトを好きなところに置くだけで通常ユーザー権限でも利用できます。ただし、実際に使用する場合は該当ユーザーが `docker` グループに所属していたほうがいいでしょう。

TAB補完機能は、全ユーザーで永続的に利用したい場合は、

```console
# echo '[[ "${PS1-}" ]] && source <(recker bashcompletion)' \
    >> /etc/bashrc
```

とします。一時的に利用するだけなら、

```console
$ source <(recker bashcompletion)
```

でOKです。

メニュー形式の選択画面を利用する場合は、Python製のツール [Percol](https://github.com/mooz/percol) が必要です。以下の手順に準じて `easy_install`, `pip` を導入後に `percol` をインストールしてください。

```console
# yum -y install python-setuptools
# easy_install pip
# pip install percol
```

サブコマンド群
--------------

Recker は基本的に、

```console
$ recker サブコマンド [オプション...]
```

として実行します。 `recker` がサブコマンドとして認識しないサブコマンドについては、そのまま、

```console
$ docker サブコマンド [オプション...]
```

として `docker` に引き渡されます。ただしすべてのサブコマンドが正常に引き渡されるとは限りません。

以下、サブコマンドを列挙・説明します。

### `images` | `images-a`

Docker がローカルに管理しているイメージの一覧を表示します。`docker` の `images` とは以下の点で動作が異なります。

* 表示がターミナル横文字数に合わせて切り捨てられます。  
  これは以下すべての情報表示系サブコマンドで同様です
* `images-a` を使用した場合、`docker` の `-a` オプションを使用したのと同じ結果になります。これは以降 `*-a` で終わるすべてのサブコマンドで同様です

使用例：
```console
$ recker images
REPOSITORY                      TAG                 IMAGE ID            CREATED
hilotech/phab                   latest              746f0e81cc47        12 days
hilotech/multipostfix           latest              cf9fea7ae06e        3 weeks
```

### `ps` | `ps-a`

Docker のコンテナ一覧を表示します。

使用例：
```console
$ recker ps
CONTAINER ID        IMAGE                          COMMAND                CREAT
a6a8c4dcc7eb        hilotech/multipostfix:latest   "/etc/container_init   3 wee
28ff19eb5cc0        hilotech/submin:latest         "/etc/container_init   5 wee
```

### `rmi` | `rmi-a` [image ids...]

指定した ID の Docker イメージを削除します。

イメージIDを指定すべき部分で `TAB` キーを押すと、補完機能によりイメージ名候補一覧が表示されます。これは以下、イメージID・コンテナ名を引数として取るサブコマンドに共通の動作です。

イメージIDを指定せずに実行した場合、 候補イメージの一覧が表示されカーソルの上下・`Enter` で選択することができます。また `CTRL+SPACE` で複数指定も可能です。これは以下、イメージID・コンテナ名を引数として取るサブコマンドに共通の動作です。なお `rmi` と `rmi-a` の違いは「使用されているイメージだけが表示されるか」「中間イメージも表示されるか」です。以下、同種のコマンドは同様の動作をします。

使用例：
```console
$ recker rmi [TAB]
hilotech/base:latest       hilotech/base_ssh:latest
hilotech/gitbucket:latest  hilotech/multipostfix:latest
   :
$ recker rmi hilotech/base:latest
```

```console
$ recker rmi [Enter]
Recker rmi                                                          (1/43) [1/1]
:tqf1b10cd84249 Virtual Size: 0 B
:x mqb9aeeaeb5e17 Virtual Size: 202.6 MB Tags: docker.io/centos:centos6
:x   x  mq746f0e81cc47 Virtual Size: 548 MB Tags: hilotech/phab:latest
               :
```

### `stop` | `stop-a` [container ids...]

指定した Docker コンテナの動作を停止させます。`docker stop` とおなじく、コンテナは停止されるだけで削除されません。

### `kill` | `kill-a` [container ids...]

指定した Docker コンテナの動作を強制停止させます。`docker stop` とおなじく、コンテナは停止されるだけで削除されません。

### `rm` [container ids...]

指定した Docker コンテナを削除します。

### `stop-rm` | `stop-rm-a` [container ids...]

指定した Docker コンテナを停止させたのち、削除します。

### `kill-rm` | `kill-rm-a` [container ids...]

指定した Docker コンテナを強制停止させたのち、削除します。

### `get-container-ip` [container id] 

指定した Docker コンテナに割り振られている IP アドレスを取得・表示します。

使用例：
```console
$ recker get-container-ip openldap
172.17.0.5
```

### `get-container-ports` [container id]

指定した Docker コンテナで `-p` オプションを利用してマップされているポートの一覧を表示します。 

使用例：
```console
$ recker get-container-ports openldap
389/tcp:389 636/tcp:636 80/tcp:8804 8000
```

### `get-container-port-http` | `get-container-port-ssh` [container id]

`get-container-ports` と同じですが、それぞれ対象を「HTTP」「SSH」の各ポートに限定します。

### `get-containers-names`

現在 Docker で動いているコンテナの名前一覧を表示します。

使用例：
```console
$ recker get-containers-names
multipostfix submin wp_diary gitbucket redmine phpmyadmin openldap wp_www
```

### `top` [container id]

指定されたコンテナで動いているプロセスの一覧を表示します。

使用例：
```console
$ recker top wp_diary
UID       PID       PPID      CMD
499       6982      39246     php-fpm: pool hilotech.jp
499       6984      39246     php-fpm: pool hilotech.jp
499       10882     39246     php-fpm: pool hilotech.jp
root      31685     39027     sleep 10
root      39027     807       /bin/bash /etc/container_init.sh
         :
```

### `run` [context directory]

指定されたディレクトリ内にある設定ファイル等に従って、Docker のコンテナを起動します。詳細は後述します。

使用例：
```console
$ recker run openldap
```

### `scaffold` [context name]

Docker イメージビルド／コンテナコンテキスト作成のためのひながたを `context name` の名でつくります。場所は `root` ユーザーならデフォルトで `/var/docker/Dockerfiles/[context name]`、非 `root` ユーザーなら `~/Docker/Dockerfiles/[context name]` です。詳細は後述します。

使用例：
```console
$ recker scaffold wordpress
```

### `build` [context directory]

`scaffold` や手作業で準備された Docker コンテキストディレクトリを対象に、`build` を実行してイメージを作成します。詳細は後述します。

使用例：
```console
$ recker build ./ 
```

### `bashcompletion`

`recker` 用の bash 自動補完設定を出力します。`source` コマンドに読み込ませると自動保管を利用できるようになります。なお、各Linuxディストリビューションが配布している bashcompletion 用のパッケージを事前導入する必要はありません。

使用例：
```console
$ source <(recker bashcompletion) 
```

Recker でのイメージビルドのしかた
---------------------------------

ヘルパースクリプトにより、直接 Dockerfile を書く必要はほぼありません。ただし、逆に Recker でイメージビルドのため作成した設定ファイル群は、そのまま Docker でも `docker build` するだけで使えるように配慮しています。

### ひながたからのイメージ定義

Recker でイメージを一から作成する際には、まず `scaffold` コマンドで「ひながた」をつくるところからはじめます。以下 `root` ユーザーであることを前提に話を進めます。

まずイメージの名前を決めて `scaffold` を実行します。ここでは `my_sl_container` とします。

```console
# recker scaffold my_sl_container 
```

デフォルトでは `/var/docker/Dockerfiles/my_sl_container` にひながたが作成されます。作業のために該当ディレクトリに移動します。

```console
# cd /var/docker/Dockerfiles/my_sl_container
```

用意されたファイル群を確認します。

```console
# ls
config.sh           container_services.sh  post_build.sh  _upload
container_build.sh  Dockerfile             pre_build.sh
```

それぞれの意味は後述しますが、イメージの作成手順は `Dockerfile` ではなく `container_build.sh` に追記します。ここでは以下のようにします。

```console
# cat <<'_EOF_' >> container_build.sh
yum -y install epel-release
yum -y install sl
_EOF_
```

また、イメージを起動した際に呼び出される実行ファイルの指定は `config.sh` ファイル中の `EXEC=` フィールドでおこないます。ここでは `/usr/bin/sl` が呼び出されるよう書き換えます。

```console
# sed -i -e 's|^\(EXEC\)=$|\1=/usr/bin/sl|' config.sh
```

これでイメージ作成の準備はすべて終わりました。

### ビルドする

Recker でのイメージビルドには `build` コマンドを使います。引数として先ほど作成した設定ファイル群のあるディレクトリパス（＝コンテキスト）を指定します。相対パスでもかまいません。

```console
# recker build ./
```

これでイメージ作成は終わりました。

### イメージからコンテナを起動する

Recker でのコンテナ起動には `run` コマンドを使います。引数としてディレクトリパス（＝コンテキスト）を指定します。相対パスでもかまいません。`run` コマンドのそのほかの実行時条件指定は、コンテキストにある `config.sh` でおこないます。詳細は後述します。

ここでは、先ほど `build` したイメージを `run` してみます。

```console
# recker run ./
                      (@@) (  ) (@)  ( )  @@    ()    @
                 (   )
             (@@@@)
          (    )

        (@@@)
     ====        ________                ___________
 _D _|  |_______/        \__I_I_____===__|_________|
  |(_)---  |   H\________/ |   |        =|___ ___|
  /     |  |   H  |  |     |   |         ||_| |_||     _
 |      |  |   H  |__--------------------| [___] |   =|
 | ________|___H__/__|_____/[][]~\_______|       |   -|
 |/ |   |-----------I_____I [][] []  D   |=======|____|_
__/ =| o |=-~~\  /~~\  /~~\  /~~\ ____Y___________|__|___
|/-=|___|=   O=====O=====O=====O|_____/~\___/          |
 \_/      \__/  \__/  \__/  \__/      \_/
# 
```

`sl` コマンドが起動してコンテナが終了します。


ビルドから起動までの詳細
------------------------

ここでは `scaffold`, `build`, `run` の各コマンドおよび関連ファイルの詳細を説明します。

### ひながたづくり＝`scaffold`

`scaffold` はビルドに必要な `Dockerfile` の基盤、および `run` で必要な設定ファイルの準備をおこないます。ひながたを作るよう指示されたコンテキスト（ディレクトリ）に生成されるファイルは以下のとおりです：

* `Dockerfile`  
  本来のDocker用のイメージ定義ファイルです。が、Recker では内部で `container_build.sh` を呼び出すため *基本的に編集する必要はありません*
* `container_build.sh`  
  Reckerでのイメージ定義ファイルです。書式は bash です。シェルスクリプトでインストール手順をそのまま書いてください
* `_upload`  
  ビルド時に必要となるファイル群を保持しておくためのディレクトリです。このディレクトリ内にあるファイルはビルド開始とともに `/tmp/_upload` にコピーされます
* `pre_build.sh`  
  ビルド開始前に Docker ホスト上で必要な処理をシェルスクリプトで記述します
* `post_build.sh`  
  ビルド完了後に Docker ホスト上で実行させたい処理をシェルスクリプトで記述します
* `container_services.sh`  
  起動したコンテナで、いわば `rc.local` のように実行されるシェルスクリプトです。実行したままにしておきたいデーモンなどは、ここに `service postfix start` などと書いておけば通常どおり動作します。supervisord などを使用する必要はありません
* `config.sh`  
  以下のような変数を疑似シェルスクリプト形式で列挙定義します：
    * ビルド用
        * `EXEC`  
          コンテナが起動したときに呼び出される実行ファイルパス。
          指定されていない場合は `/bin/bash` が呼び出されます
        * `BASE_IMAGE`
          ビルドに使用するベースイメージ（FROM）を指定します。デフォルトでは `centos:centos6` です
    * コンテナ向け
        * RUN_AS_SERVICE  
          コンテナをサービス（デーモン）として起動したままにするか（=1）どうかを指定します
        * AUTOMAP_PORT_HTTP  
          1の場合、コンテナの80番ポートを Docker ホストの8000番以上で空いているポートにマップします
        * AUTOMAP_PORT_SSH  
          1の場合、コンテナの25番ポートを Docker ホストの2500番以上で空いているポートにマップします
        * VOLUMES  
          Docker ホスト上に永続化したいコンテナのディレクトリパスを列記します。区切りはスペースです。たとえば `"/var/www"` と書くとコンテナの `/var/www` が Docker ホストの `/var/docker/volumes/{コンテナ名}/var/www` に永続化されます
        * PORTS  
          Docker ホスト側のポートとコンテナのポートをマップします。スペース区切りで列挙できます。たとえば `"8080:80"` と書くとコンテナの 80 番ポートが Docker ホストの 8080 番ポートにマップされます
        * LINKS  
          ほかのコンテナの名前を列記することで、コンテナ同士の通信を可能にします。スペース区切りで列挙できます。たとえば `"openldap"` と書くとコンテナは `openldap` というホスト名で `openldap` コンテナと通信できます。なお、Docker ホストはデフォルトで `dockerhost` として通信できるようになっています
        * SET SOMENAME="VALUE..."  
          コンテナに対して `SOMENAME` という名称で `VALUE` という値をもった環境変数を受け渡します。ただし、VALUE 中の ` `（半角スペース）は `_` （アンダースコア）に置き換えて定義してください
  
これらの中でももっと編集する必要性が高いのは：

* container_build.sh
* container_services.sh
* config.sh

でしょう。

### ビルド＝`build`

前項の説明に従えば、基本的にビルド作業は、

```console
# recker build ./
```

で終わりです。

いくつかの注意点があります：

* ビルド時には `VOLUMES`, `LINKS` の設定は無効です。この制限を避けるにはいささかトリッキーな方法を採用する必要があります
* `recker build` は通常の `docker build` よりも遅いです。これは、ビルド手順をシェルスクリプトに追い出している関係上、キャッシュをオフにしているためです

### コンテナ起動＝`run`

`run` も基本的には、

```console
# recker run ./
```

で動作します。その動作は前出の `config.sh` に大きく左右されます。
`config.sh` の `run` 用項目についていくつか、ユースケースを説明するため再掲します。   

* RUN_AS_SERVICE  
  `container_services.sh` に記載したサービス用のコンテナとして常駐させたい場合に使います
* AUTOMAP_PORT_HTTP, AUTOMAP_PORT_SSH  
  おなじポートを使っていたり、おなじイメージを元にしたコンテナが動作していても、自動的に空いているポートを割り当ててコンテナを起動させたい場合に使います
* SET SOMENAME="VALUE..."  
  コンテナ起動時に `container_services.sh` で環境変数からSSH公開鍵を受け取り、所定の位置に保存するなどすると便利です
