---
layout: page
title: JFT ニフティクラウド ハンズオン
tagline: Supporting tagline
---
{% include JB/setup %}

## 1. 準備

### 1.1. SSH でサーバーへログイン

### 1.2. Chef Solo のインストール

下記コマンドで一発でインストールすることができます。

    # curl -L https://www.opscode.com/chef/install.sh | bash

ちなみにバージョンを固定したい場合は下記のように RPM からインストールすることも可能です。

    # rpm -ivh https://opscode-omnibus-packages.s3.amazonaws.com/el/6/x86_64/chef-11.4.4-2.el6.x86_64.rpm

正常にインストールが完了したかどうかは chef-solo コマンドで確認しましょう。

    # chef-solo
    # which chef-solo
    /usr/bin/chef-solo

Chef 関連のモジュールは /opt/chef 配下にインストールされるので興味のある人は ls で閲覧してみてください。

### 1.3. Chef Apply を試してみる

chef-apply というコマンドもインストールされます。
chef-solo よりも手軽に Chef の「べき等性」が試せるツールなので、試してみましょう。

    vi recipe.rb
    file '/var/tmp/hello.txt' do
      content "hello world!\n"
    end

### 1.4. Chef Solo 用の設定ファイル配置

    /etc/chef/solo.rb
    /etc/chef/dna.json
    mkdir -p /var/chef/cookbooks

## 2. レシピ開発

### 2.1. 

スキルのある人は自由に開発？

### 2.2.


## 3. Chef サーバーを使ってみる (時間のある人向け)

