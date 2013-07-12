---
layout: page
title: JFT ハンズオン資料
tagline: 2013/07/14
---
{% include JB/setup %}

## 0. 目次

<ul>
  <li>
    <a href="#1_">1. 準備</a>
    <ul>
      <li><a href="#11_ssh_">1.1. SSH でサーバーへログイン</a></li>
      <li><a href="#12_chef_solo_">1.2. Chef Solo のインストール</a></li>
    </ul>
  </li>
  <li>
    <a href="#2_chef_apply_">2. Chef Apply を試す</a>
    <ul>
      <li><a href="#21_chef_apply_">2.1. Chef Apply とは</a></li>
      <li><a href="#22_">2.2. レシピの作成と実行</a></li>
      <li><a href="#23_">2.3. べき等性の確認</a></li>
    </ul>
  </li>
  <li>
    <a href="#3_chef_solo__wordpress_">3. Chef Solo で WordPress レシピ開発</a>
    <ul>
      <li><a href="#31_chef_solo_">3.1. Chef Solo 用の設定ファイル配置</a></li>
      <li><a href="#32_wordpress_">3.2. WordPress レシピのダウンロード・実行</a></li>
    </ul>
  </li>
  <li>
    <a href="#4_">4. レシピのテストを書く</a>
    <ul>
      <li><a href="#41_serverspec">4.1. serverspec のインストール</a></li>
      <li><a href="#42_">4.2. テストを書く</a></li>
      <li><a href="#43_">4.3. テストの実行</a></li>
    </ul>
  </li>
  <li>
    <a href="#5_cloudautomation_">5. CloudAutomation で自動化！</a>
    <ul>
      <li><a href="#erverspec_">4.1. serverspec のインストール</a></li>
      <li><a href="#">4.2. テストを書く</a></li>
      <li><a href="#">4.3. テストの実行</a></li>
    </ul>
  </li>
</ul>

## 1. 準備

### 1.1. SSH でサーバーへログイン

<a href="#0_">目次へ</a>

<ol>
  <li> TeraTerm を起動</li>
  <li> 「ホスト」に当日配布の IP アドレスを入力</li>
  <li> 「TCP ポート」に "22" を入力</li>
  <li> 「OK」ボタンをクリック (次の画面が表示されます)</li>
  <li> 「ユーザ名」に "root" を入力</li>
  <li> 「パスフレーズ」に当日配布のパスフレーズを入力</li>
  <li> 「RSA/DSA 鍵を使う」を選択し「秘密鍵」ボタンから当日配布する秘密鍵を指定</li>
  <li> 「OK」ボタンをクリック</li>
</ol>

下記のようなプロンプトが表示されればログイン成功です。

    [root@devopschf001 ~]#

### 1.2. Chef Solo のインストール

<a href="#0_">目次へ</a>

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

Chef 関連のモジュールは /opt/chef 配下にインストールされるので ls で閲覧してみてください。

    # ls /opt/chef/bin/
    chef-apply  chef-client  chef-shell  chef-solo  erubis  knife  ohai  restclient  shef

## 2. Chef Apply を試す

### 2.1. Chef Apply とは

<a href="#0_">目次へ</a>

`chef` gem をインストールすると、`chef-apply` というコマンドもインストールされます。
`chef-solo` よりも手軽に Chef の「べき等性」が試せるので、少し触ってみましょう。

### 2.2. レシピの作成と実行

<a href="#0_">目次へ</a>

まずはかんたんなレシピを作成します。
ファイルを作成して中に "hello world" と書き込むだけのものです。

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

### 2.3. べき等性の確認

<a href="#0_">目次へ</a>

もう一度実行してみます。

    # chef-apply recipe.rb
    Recipe: (chef-apply cookbook)::(chef-apply recipe)
      * file[/var/tmp/test.txt] action create (up to date)

今度は up to date と表示され何も起きません。これはサーバーがレシピに書かれた内容通りの状態になっていることを Chef が認識し、ファイル作成をスキップしたためです。

このような性質を Chef (や構成管理ツール) の世界では idemponent (べき等) と呼んでいます。

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

## 3. Chef Solo で WordPress レシピ開発

### 3.1. Chef Solo 用の設定ファイル配置

<a href="#0_">目次へ</a>

まずは準備として Chef Solo 用の設定ファイルを作成しましょう。

    /etc/chef/solo.rb
    /etc/chef/dna.json
    mkdir -p /var/chef/cookbooks

### 3.2. WordPress レシピのダウンロード・実行

<a href="#0_">目次へ</a>

    # knife cookbook site install wordpress

    # ls /var/chef/cookbooks

## 4. レシピのテストを書く

### 4.1. serverspec のインストール

<a href="#0_">目次へ</a>

(serverspec を使ってレシピのテストを書いてみる)

### 4.2. テストを書く

<a href="#0_">目次へ</a>


### 4.3. テストの実行

<a href="#0_">目次へ</a>

## 5. CloudAutomation β で自動化！

### 5.1. レシピのアップロード

<a href="#0_">目次へ</a>

### 5.2. JSON ファイル作成

<a href="#0_">目次へ</a>

### 5.3. コントロールパネルから実行

<a href="#0_">目次へ</a>


