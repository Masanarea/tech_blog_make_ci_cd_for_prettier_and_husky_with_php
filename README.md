
# はじめに

個人開発を進める際にPrettierとHuskyをPHPプロジェクトに導入して自動でコード整形してくれるように実装しました。
今回はその時の導入手順をシンプルにまとめました！😆

![iugserrs](https://github.com/Masanarea/tech_blog_make_ci_cd_for_prettier_and_husky_with_php/assets/93495976/20b63f0b-1659-4e92-809d-6032f5bfcb4e)


フロントエンド界隈ならまだしも、バックエンド言語での記事をあまり見かけなかったり、見かけたとしても『動かない...(あるある)』みたいなことを体験したので
* イメージを掴みやすい！
* しっかりと動く！(エラーが出ない)

の部分にも気をつけて解説してみました！✨
よければご参考ください！😊

# URL一覧

完成Gitリポジトリ

https://github.com/Masanarea/tech_blog_make_ci_cd_for_prettier_and_husky_with_php


# 最終的に使用するファイル一覧
```:構成
laravel_app
├─ .husky
│    ├─ pre-commit
│    └─ pre-oush
├─ app
│ 
├─ config
│ 
├─ database
│
├─ その他(省略)
│
│─ package.json  
│─ .prettierrc     // Prettier設定
│─ my_test0.php    // Prettier用(ローカルテスト用)
│─ my_test.php    // Prettier用(commit用)
└─ my_test2.php    // Prettier用(push用)

```



# 対象者
* PHP(Laravel)使用者
* Prettier や Husky に少しでも興味のある方
* PHPプロジェクトで、楽にソースコードの整形できたら便利だよな~と感じる人<br>(PSR-2に則ってコード整形してくれる優れもの！)


# 記事を読むメリット
* ハンズオンでも Git クローンでもどちらでも試すことができる
* どのサイトよりもわかりやすく速く実装できる<br>(Laravel10に対応 + 参考リンク・リポジトリ付き)
* Prettier や Husky が何者なのかわかる

## 使用技術・プラットフォーム
* Git
* GitHub

## そもそも Prettier、Husky とは何なのか?🤔

ざっくり解説します！

### Prettierとは？
ソースコードを良い感じに整えてくれるツールです。
下のキャプチャーでいうと、『コード整形前』から『コード整形後』のようにきれいに書き換えてくれる機能が Prettier の役割です。

![iugserrs](https://github.com/Masanarea/tech_blog_make_ci_cd_for_prettier_and_husky_with_php/assets/93495976/5a8c8bf5-be3b-4480-be0c-9dec61d37240)

### Husky とは？

自動でコマンドを実行してくれるツールです。
今回の例で言うと、Git で commit や push した時に、HuskyのおかげでPritter(コード整形)を実行してくれます。逆にHuskyがない場合、自動でコード整形はしてもらえず、毎回Pritterをcommitやpushした時に手動で実行させなければいけません。



# 導入方法

## 1.Github リポジトリを作成
まずはGithub リポジトリを作成しましょう


![スクリーンショット 2023-05-20 8.54.16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2980785/e6d80d38-2562-50a6-b8e5-7fbc6d3f4f7e.png)

![スクリーンショット 2023-05-20 8.54.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2980785/5a9e7282-4449-8ccf-0353-b6a243598b81.png)


## 2.ローカルでLaravelをインストール
※今回はComposerを使用してLaravelをインストールします。

まずはローカル(自身のPC)でフォルダを作成し、以下のコマンドを実行して
Composer が使用できるかどうかを確認します。

```
composer -V 
// Composer version 2.5.4 2023-02-15 13:10:06
```

composer が使用できることを確認できたら、composer でLaravelをインストールします。
```php:ターミナル
//例 composer create-project laravel/laravel <プロジェクト名>
composer create-project laravel/laravel make_ci_cd_project
```

作成したLaravel プロジェクトの階層部分に移動します。
```php:ターミナル
// cd <プロジェクト名>
cd make_ci_cd_project
```
『ls』コマンドや『php artisan -V』コマンドで、
LaravelがインストールできていたらOKです。
```php:ターミナル
ls
//README.md       artisan         composer.json   config          package.json    public          routes          tests           vite.config.js
app             bootstrap       composer.lock   database        phpunit.xml     resources       storage         vendor

php artisan -V
//Laravel Framework 10.x.x
```


## 3. Git 準備
Gitローカルリポジトリを作成し、
先ほど作成した自身のGitリモートリポジトリにpushできるように設定しましょう。
```php:ターミナル
git init
```
```php:ターミナル
//この下の『https://github.com/Masana....』の部分は適宜変更して下さい
git remote add origin https://github.com/Masanarea/test1.git

//しっかりリモートリポジトリの設定ができているかを確認する
git remote -v
//origin  https://github.com/Masanarea/test1.git (fetch)
//origin  https://github.com/Masanarea/test1.git (push)
```

この辺りの記述を参照することになると思います
![スクリーンショット 2023-05-20 8.57.35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2980785/3c7d0837-0057-8272-eb85-480d258b86b5.png)


#### 参考リンク
これで完璧! git remoteでリポジトリを【追加,削除,確認,変更】
https://www.sejuku.net/blog/71492

## 4 Prettier をインストール

Prettierを導入します。

先ほどインストールした Laravel アプリのルートディレクトリで、
下記コマンドを実行します。
```:ターミナル
npm install --save-dev prettier @prettier/plugin-php
```

## 5 .prettierrc.jsonの作成
プロジェクトのルートディレクリに 『.prettierrc』ファイルを作成します。
実際に個人開発でよく使用している無難な設定です。
```:構成
make_ci_cd_project
│
├─ その他(省略)
│
│─ .prettierrc     // Prettier設定
│
```

```javascript:.prettierrc.json
{
    "phpVersion": "8.0",
    "singleQuote": true,
    "tabWidth": 4,
    "printWidth": 120,
    "useTabs": false
}
```

軽く解説すると、
```javascript:.prettierrc.json
{
    "phpVersion": "8.0", // PHP バージョン(必須)
    "singleQuote": true, // 文字列をシングルクォートで囲うか
    "tabWidth": 4, // タブをスペースに変換する際のスペース文字数
    "printWidth": 120, // コードに関して、折り返す行の長さを指定します。
    "useTabs": false // タブを許容するかどうか
}
```
というような意味になります。

詳細は公式ドキュメントをご覧ください！

公式

https://github.com/prettier/plugin-php

#### 参考リンク
PHPの開発でPrettierを使用してフォーマットする方法
https://maasaablog.com/integrated-development-environment/visual-studio-code/2095/


## 6 『prettier』をスクリプトとして追加

```:構成
make_ci_cd_project
│
├─ その他(省略)
│
│─ .prettierrc     // Prettier設定
│─ package.json
```
package.jsonに次のように 『prettier』をスクリプトとして追加して下さい
※『"prettier": "prettier"』 の部分を追記
```javascript:package.json
"scripts": {
    "dev": "vite",
    "build": "vite build",
    "prettier": "prettier"
},
```
これで準備は整いました！


## 7 Prettier を試してみる！



試しにプロジェクトのルートディレクリに 『my_test0.php』ファイルを作成してください。

```:構成
make_ci_cd_project
│
├─ その他(省略)
│
│─ composer.json  
│─ .prettierrc     // Prettier設定
└─ my_test0.php    // Prettier用(ローカルテスト用)

```

そして今回のために用意した、
『世界一汚いPHPのソースコード』
をそのまま貼り付けてみてください。笑

```php:my_test0.php
<!-- 世界一汚いPHPのソースコード -->
<?php
echo "Hello World!";
if     (true)


{
       echo "Hello World!";
}



echo 'Hello World!';

$num = 1;
if
($num>=10)      { echo "The variable num is a number greater than or equal to 10.";}else {        echo 'The variable num is a number less than 10.';}
?>

```

その後、ターミナルで
次のコマンドを打ち込みます。
```php:ターミナル
npm run prettier -- my_test0.php --write
```


打ち込んだ後、再度my_test0.phpを覗いてみると
```php:my_test0.php
<!-- 世界一汚いPHPのソースコード -->
<?php
echo 'Hello World!';
if (true) {
    echo 'Hello World!';
}

echo 'Hello World!';

$num = 1;
if ($num >= 10) {
    echo 'The variable num is a number greater than or equal to 10.';
} else {
    echo 'The variable num is a number less than 10.';
}


?>
```
のように綺麗にソースコードが形成されているとおもいます。
これが『Prettier』の醍醐味です！

変更点も確認してみます。
#### 変更点一覧
* ダブルクォーテーション『""』が、全てシングルクォーテーション『''』に変換
* if 文あたりでいい感じにスペース空けてくれる(PSR-2準拠の記述)
* スペース4つ分の間隔を自動で開けてくれる<br>『.prettierrc』ファイルの "tabWidth": 2 にすればスペース2つ分になります!

![iugserrs](https://github.com/Masanarea/tech_blog_make_ci_cd_for_prettier_and_husky_with_php/assets/93495976/08ad2ef5-c8c2-4211-9bbc-2fb1c6dc282b)


これが Preiiter というソースコード整形ツールを使用するメリットになります。
これだけでも十分有益なツールだと思いますが、
今回はHuskyというツールを利用することで、Git Commit/Push された際に実行で該当するソースコードを整形してくれるように設定していこうと思います。


## 8 Husky をインストール


Husky をインストールします。

 Laravel アプリのルートディレクトリで、
下記コマンドを実行して下さい。
```php:ターミナル
npm install husky --save-dev
```

この後、Huskyの初期設定も行います。
```php:ターミナル
npx husky install
```



## 9 Husky の設定(Git Commit)

ルートディレクリ直下で次のコマンドを打ってください
```
// commit したときに実行されるファイルを用意する
npx husky add .husky/pre-commit "php artisan foo"
```

その後、
『.husky/pre-commit』ファイルを以下のように修正してください。

```php:.husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"


// ここから下を編集
npm run prettier -- my_test.php --write
git add .
```



その後、
ルートディレクトリ直下に、『my_test.php』を作成してください。
```php:my_test.php
<!-- 世界一汚いPHPのソースコード -->
<?php
echo "Hello World!";
if     (true)


{
       echo "Hello World!";
}



echo 'Hello World!';

$num = 1;
if
($num>=10)      { echo "The variable num is a number greater than or equal to 10.";}else {        echo 'The variable num is a number less than 10.';}
?>
```

これで準備が整いました。

試しに commit してみましょう。
```php:ターミナル
//ステージング
git add -A

//コミット (my_test.phpファイルのコードを整形)
git commit -m "commit時: Huskyをmy_test.phpファイルに対して実行"
```

すると『my_test.php』ファイルのソースコードが整形されたのが確認できると思います。
以上が Husky で自動でソースコートを整形してくれるように設定する流れになります。







## 10 Husky の設定(Git Push)

同じく pushされたときにも特定の処理を実行することができます。




```:ターミナル
npx husky add .husky/pre-push "php artisan foo"
```
その後、
『.husky/pre-push』 ファイルを以下のように修正してください。

```php:.husky/pre-push
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"


// ここから下を編集
npm run prettier -- my_test2.php --write
git add .

```



その後、
ルートディレクトリに、『my_test2.php』を作成してください。
```php:my_test2.php
<!-- 世界一汚いPHPのソースコードを追記！ -->
<?php
echo "Hello World!";
if     (true)


{
       echo "Hello World!";
}



echo 'Hello World!';

$num = 1;
if
($num>=10)      { echo "The variable num is a number greater than or equal to 10.";}else {        echo 'The variable num is a number less than 10.';}
?>
```

これで準備が整いました。

試しにcommitからのpushをしてみましょう。

```php:ターミナル
//ステージング
git add -A

//コミット (my_test.phpファイルのコードを整形)
git commit -m "push時に起動: Huskyをmy_test2.phpファイルに対して実行"
//プッシュ　:この時にmy_test2.phpがコード整形
git push origin master
```

すると commit時ではなく、push 時に『my_test.php2ファイルのコードを整形』されたのが確認できると思います。ただ、コードを整形した内容をPushの内容に含めることはどうもできなさそうでした💦


# 『Prettier × Husky』 応用編
ここまで見てきた人の中でも、
『じゃあこれをどう日頃の開発で利用してけばいいんだよ！』
っていう意見もあると思います。
これをある程度解決するためにサンプルを作成しました！

例えば、
* 『app/Http/Controllers/Controller.php』 ファイルのみ
* 『config』　フォルダ内全て
* 『database』フォルダ内全て

のソースコードを commit時に全て整形したいという場合どうすれば良いんでしょうか？
結論、『.husky/pre-commit』を次のように記述すると解決できます。

```php:.husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"


# Huskyのカスタマイズサンプル

# 具体例１:
# フォルダ内の
# 『app/Http/Controllers/Controller.php,config,database』
# フォルダやファイルのソースコードを全て整形する場合

npm run prettier -- app/Http/Controllers/Controller.php --write
npm run prettier -- config/* --write
npm run prettier -- database/* --write

git add .

```


commit してみましょう。
```php:ターミナル
//ステージング
git add -A

//コミット (my_test.phpファイルのコードを整形)
git commit -m "commit時: 応用"
```

※今回、『app/Http/Controllers/Controller.php』もソースコードの形成をする対象だったと思いますが、元々綺麗だったため特に変更がされませんでした。(↓ 差分)



![スクリーンショット 2023-05-19 14.18.03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2980785/0b65b05b-da31-ad6b-4668-b5efdfc11ba3.png)






#### 参考
```php:app/Http/Controllers/Controller.php
<?php

namespace App\Http\Controllers;

use Illuminate\Foundation\Auth\Access\AuthorizesRequests;
use Illuminate\Foundation\Validation\ValidatesRequests;
use Illuminate\Routing\Controller as BaseController;

class Controller extends BaseController
{
    use AuthorizesRequests, ValidatesRequests;
}
```

# 終わりに
今回は Prettier と Husky をフル活用して、コード整形する方法を解説していきました。
個人的に『限られた時間の中で比較的自動化しやすい部分は自動化してしまうべきである』と考えているので、同じような考え方を持つ人にとっては、特に便利で有益な技術なのではないかと考えています。
これを機に、ぜひ『Prettier』や『Husky』を自身の開発へ導入してみてはいかがでしょうか？
今回は以上です。👋

