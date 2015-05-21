# Recker
docker command line helper

![Recker](https://raw.githubusercontent.com/wiki/hilotech/recker/img/recker.png)

---

## Recker とは

Recker は [Docker](https://www.docker.com/) のコマンドラインヘルパーツールです。

* 作業対象イメージ・コンテナを一覧画面からカーソルキーで選択する機能
* `TAB` キーでのイメージ・コンテナ名・ID補完機能
* `docker build` の゛土台“（scaffold）作成機能
* `docker run` の簡便化、ポート・リンク・ボリュームの設定サポート機能

をもっています。でも、中身はただの bash スクリプトです。

以下に、なんとなくビルド例集もあります：

* [Reckerfiles](https://github.com/hilotech/Reckerfiles)

## 利用例

### SLを走らせる

````console
$ recker scaffold my_sl_container
$ cd ~/Docker/Dockerfiles/my_sl_container
$ cat <<'_EOF_' >> container_build.sh
yum -y install epel-release
yum -y install sl
_EOF_
$ recker build ./
$ recker run ./
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
$ 
````

### Apache族をたくさん呼ぶホホホ

````
$ recker scaffold call_apache
$ cd ~/Docker/Dockerfiles/call_apache
$ cat <<'_EOF_' >> container_build.sh
yum -y install httpd
_EOF_
$ cat <<'_EOF_' >> container_services.sh
service httpd start
_EOF_
$ sed -i \
    -e 's|^\(RUN_AS_SERVICE\)=.\{0,\}$|\1=1|' \
    -e 's|^\(AUTOMAP_PORT_HTTP\)=.\{0,\}$|\1=1|' \
    config.sh
$ recker build ./
$ recker run ./
HTTP port auto-mapped to: 8000
Changed mode to background because of declaration RUN_AS_SERVICE
8214acd40c4c5afeae9746a08ff9b32722709b05d18fd5783f11d929b7f76b00
$ recker run ./
HTTP port auto-mapped to: 8001
Changed mode to background because of declaration RUN_AS_SERVICE
0d7a3a762cffe07b0522841b9f71b4bdea29fe9f24b647e962a9ba5ce505eb3f
$ recker run ./
HTTP port auto-mapped to: 8002
Changed mode to background because of declaration RUN_AS_SERVICE
cd0a73733580ad604263d1fa4c140d41367fde3a7314fb567495826089342ce0
    :
````

### コンテナ名をTAB補完 or カーソルで選択する

```
$ recker ps [TAB]
0d7a3a762cff   9763f9c4bed2   call_apache    call_apache-2  lamp
8214acd40c4c   c5eb911def0f   call_apache-1  cd0a73733580   mean

$ recker ps [Enter]
    ↓画面
```

![カーソル選択画面](https://raw.githubusercontent.com/wiki/hilotech/recker/img/recker-selection.png)

## 実行環境・必要プロダクト

* Docker 1.6.0 以上  
  たぶん1以上なら動く気がする
* bash 4以上
* [Percol](https://github.com/mooz/percol)
* CentOS 7.x  
  でしか動作確認していません

なお、コンテナのデフォルトは CentOS 6 になっています（変更可能）。
