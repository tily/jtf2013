---
layout: page
title: JFT ハンズオン資料
tagline: 2013/07/14
---
{% include JB/setup %}

## 1. 準備

### 1.1. SSH でサーバーへログイン

 1. TeraTerm を起動
 2. 「ホスト」に当日配布する IP アドレスを入力
 3. 「TCP ポート」に "22" を入力
 4. 「OK」ボタンをクリック (次の画面が表示されます)
 5. 「ユーザ名」に "root" を入力
 6. 「パスフレーズ」に当日配布するパスフレーズを入力
 7. 「RSA/DSA 鍵を使う」を選択し「秘密鍵」ボタンから当日配布する秘密鍵を指定
 8. 「OK」ボタンをクリック

### 1.2. Chef Solo のインストール

下記コマンドで一発でインストールすることができます。

    # curl -L https://www.opscode.com/chef/install.sh | bash

ちなみにバージョンを固定したい場合は -v で指定したり RPM から直接インストールすることも可能です。

    # bash install.sh -v 11.4.4.-2
    # rpm -ivh https://opscode-omnibus-packages.s3.amazonaws.com/el/6/x86_64/chef-11.4.4-2.el6.x86_64.rpm

正常にインストールが完了したかどうか chef-solo コマンドで確認してみましょう。

    # chef-solo
    [2013-07-09T17:15:39+09:00] WARN: *****************************************
    [2013-07-09T17:15:39+09:00] WARN: Did not find config file: /etc/chef/solo.rb, using command line options.
    [2013-07-09T17:15:39+09:00] WARN: *****************************************
    Starting Chef Client, version 11.4.0
    Compiling Cookbooks...
    Converging 0 resources
    Chef Client finished, 0 resources updated
    # which chef-solo
    /usr/bin/chef-solo

Chef 関連のモジュールは /opt/chef 配下にインストールされるので興味のある人は ls で閲覧してみてください。

    # ls /opt/chef/bin/
    chef-apply  chef-client  chef-shell  chef-solo  erubis  knife  ohai  restclient  shef

### 1.3. Chef Apply を試してみる

chef-apply というコマンドもインストールされます。
chef-solo よりも手軽に Chef の「べき等性」が試せるツールなので、試してみましょう。

まずはかんたんなレシピを作成します。
ファイルを作成して中に "hello world" と書き込むだけのものです。
(chef-apply なので 1 つのファイルから Chef 実行できますが、実際には COOKBOOK_NAME/recipes/default.rb のように cookbook の中のレシピに書くような内容です。)

    # vi recipe.rb
    ## 下記内容を書き込みます
    file '/var/tmp/hello.txt' do
      content "hello world!\n"
    end

実行してみます。

    # chef-apply recipe.rb
    Recipe: (chef-apply cookbook)::(chef-apply recipe)
      * file[/var/tmp/test.txt] action create
        - create new file /var/tmp/test.txt with content checksum 853ff9
            --- /tmp/chef-tempfile20130709-856-1n2lyfc      2013-07-09 17:19:24.658237382 +0900
            +++ /tmp/chef-diff20130709-856-1qjl5ez  2013-07-09 17:19:24.658237382 +0900
            @@ -0,0 +1 @@
            +hello, world

ファイルが作成されたようです。実際に確認してみます。

    # ls /var/tmp/test.txt
    /var/tmp/test.txt
    # cat /var/tmp/test.txt
    hello, world

もう一度実行してみます。

    # chef-apply recipe.rb
    Recipe: (chef-apply cookbook)::(chef-apply recipe)
      * file[/var/tmp/test.txt] action create (up to date)

今度は up to date と表示され何も起きません。これはサーバーがレシピに書かれた内容通りの状態になっていることを Chef が認識し、ファイル作成をスキップしたためです。
このような性質を構成管理ツールの世界では idemponent (べき等) と呼んでいます。

次にわざと作成されたファイルを書き換えた上で実行してみましょう。

    # echo hello nifty cloud > /var/tmp/test.txt
    # cat /var/tmp/test.txt
    hello nifty cloud
    # chef-apply recipe.rb
    Recipe: (chef-apply cookbook)::(chef-apply recipe)
      * file[/var/tmp/test.txt] action create
        - update content in file /var/tmp/test.txt from bac308 to 853ff9
            --- /var/tmp/test.txt   2013-07-09 17:28:37.003242663 +0900
            +++ /tmp/chef-diff20130709-2021-umd0np  2013-07-09 17:28:54.504237642 +0900
            @@ -1 +1 @@
            -hello nifty cloud
            +hello, world

ファイルが書き換わったことを認識して Chef がレシピ通りの状態に復元してくれたことが確認できたと思います。
今度は recipe.rb 自体のほうを書き換えた上で Chef 実行してみます。


    # vi recipe.rb
    ## world を chef に書き換えた
    file '/var/tmp/test.txt' do
      content "hello, chef\n"
    end
    # chef-apply recipe.rb
    Recipe: (chef-apply cookbook)::(chef-apply recipe)
      * file[/var/tmp/test.txt] action create
        - update content in file /var/tmp/test.txt from 853ff9 to 5b8079
            --- /var/tmp/test.txt   2013-07-09 17:28:54.557365556 +0900
            +++ /tmp/chef-diff20130709-2432-3waga3  2013-07-09 17:31:54.521237988 +0900
            @@ -1 +1 @@
            -hello, world
            +hello, chef

この場合にもサーバーの状態がレシピに記述された状態とは異なるので変更が適用されます。

このように、サーバーに変更が反映されるのは、下記の 2 つの場合のみです。

<ul>
  <li>サーバーの状態がレシピと異なってしまった場合</li>
  <li>レシピ自体が更新された場合</li>
</ul>

### 1.4. Chef Solo 用の設定ファイル配置

    /etc/chef/solo.rb
    /etc/chef/dna.json
    mkdir -p /var/chef/cookbooks

## 2. レシピ開発

(スキルのある人は自由に開発？)

### 2.1. 

### 2.2.

## 3. レシピのテストを書く

(serverspec を使ってレシピのテストを書いてみる)

## 4. Chef サーバーを使ってみる (時間のある人向け)

