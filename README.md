# 起動手順
## 前提
環境を構築するため、下記のアプリケーションをインストールしてください。  
インストール方法はリンク先をご参照ください。
- Visual Studio Code
  - 拡張機能 Remote - Containers
- Docker Desktop
- WSL2 (Windows ユーザのみ)

## 初回起動時

1. 任意のディレクトリにこのリポジトリを `git clone` します。
```bash
git clone https://xxxx
```
2. `git clone` してきたフォルダに Slack で配布された `.env` ファイルを配置します。
3. `git clone` してきたフォルダを VSCode で開きます。
4. VSCodeのターミナル上で下記のコマンドを実行します。  
※実行完了まで数分かかります。  
```bash
docker-compose -f docker-compose.init.yml run --rm composer
```
5. 上記のコマンドが完了したら、 VSCode のウィンドウ左下の「><」を押下して  
![1-1](https://user-images.githubusercontent.com/105618751/191493624-a6ff6248-491e-4f3f-97a3-372594f66a09.png)  
`Reopen in Container` を選択します。  
![1-2](https://user-images.githubusercontent.com/105618751/191493629-b51da169-af21-400b-9218-c689347fb74d.png)  
コンテナで開くことができた場合、 VSCode の表示が下記のように変更されます。  
![1-3](https://user-images.githubusercontent.com/105618751/191495808-daf37c0f-2948-433b-8877-85691df4b7aa.png)  
※コンテナで開くのに初回は数分かかります。  
　途中で動かなくなった場合、 VSCode を起動しなおしてもう一度「Reopen in Container」を選択してみてください。  
6. VSCode のターミナル(bash)で以下のコマンドを実行します。  
```bash
php artisan migrate
```
7. VSCode のターミナル(bash)で以下のコマンドを実行します。
```bash
npm install && npm run dev
```
8. [localhost](http://localhost) に接続し、下記の画面が表示されれば環境構築は完了です。  
![image](https://user-images.githubusercontent.com/105618751/191496833-fa40fdd6-34bf-4f6b-b5f2-b43f4651c93b.png)

## 初回起動完了後の起動手順
1. `git clone` してきたフォルダを VSCode で開きます。
2. VSCode のウィンドウ左下の「><」を押下して `Reopen in Container` を選択します。  
3. VSCode のターミナル(bash)で以下のコマンドを実行します。  
```bash
npm run dev
```
※開発状況に応じて下記のコマンドも実行が必要になる可能性があります。  
　チームメンバーと相談・確認を行い、必要に応じて実行してください。
```bash
# データベース変更時
php artisan migrate
# Laravel パッケージ変更時
composer install
# npm パッケージ変更時
npm install
```
4. [localhost](http://localhost) に接続します。  

# 開発手順
## ディレクトリ構成
<pre>
/var/www/html
  ├ .devcontainer
  │  └ devcontainer.json ← VSCode の環境設定
  ├ app
  │  ├ Console
  │  ├ Exceptions
  │  ├ Http
  │  │  ├ <mark>Controllers</mark> ← URL アクセス時の挙動を定義
  │  │  ├ Middleware
  │  │  └ Kernel.php
  │  ├ <mark>Models</mark> ← データベースのテーブル構造を Laravel で使用するための定義
  │  └ Providers
  ├ bootstrap
  ├ config
  ├ database
  │  ├ factories
  │  ├ <mark>migrations</mark> ← データベースの構成を定義
  │  └ <mark>seeders</mark> ← データベースに初期登録しておくデータを定義
  ├ lang
  ├ <del>node_modules</del> ← npm パッケージマネージャが自動生成するディレクトリ（編集禁止）
  ├ public
  ├ resources
  │  ├ css
  │  │  └ app.css
  │  ├ js
  │  │  ├ <mark>components</mark> ← Vue.js で画面のコントロールを作成
  │  │  ├ <mark>app.js</mark> ← components に作成したモジュールの読み込み
  │  │  └ bootstrap.js
  │  ├ sass
  │  │  ├ app.scss
  │  │  └ _variables.scss
  │  └ <mark>views</mark> ← UI を HTML(Blade) で作成して配置
  │    ├ auth
  │    └ <mark>layouts</mark> ← 画面の共通レイアウト（ヘッダ・フッタ等）の定義
  ├ routes
  │  ├ api.php
  │  ├ channels.php
  │  ├ console.php
  │  └ <mark>web.php</mark> ← ルーティングの定義
  ├ storage
  ├ tests
  └ <del>vendor</del> ← composer パッケージマネージャが自動生成するディレクトリ（編集禁止）
</pre>

## データベース関係
todo: マイグレーションとかについて書く

## ルーティング関係
todo: web.php と Controllers について書く

## フロント関係
### Vue コンポーネントの作成方法
`resources/js/components` 配下に vue ファイルを作成します。  
作成方法についてはサンプルで配置しているファイルや、[公式ドキュメント](https://v3.ja.vuejs.org/guide/introduction.html)等を参考にしてください。  
基本的には `<template>` タグの中に HTML でレイアウトを、 `<script>` タグの中に JavaScript でイベントやデータ定義を、 `<style>` タグの中に CSS で HTML の装飾を行うと覚えておけばだいたい問題ないです。  
ライフサイクルイベント（ボタン押下などではなく画面表示で自然と発生するイベント）については[こちら](https://v3.ja.vuejs.org/guide/instance.html#%E3%83%A9%E3%82%A4%E3%83%95%E3%82%B5%E3%82%A4%E3%82%AF%E3%83%AB%E3%82%BF%E3%82%99%E3%82%A4%E3%82%A2%E3%82%AF%E3%82%99%E3%83%A9%E3%83%A0)が参考になると思います。

### 作成したコンポーネントの登録
`resources/js/app.js` に作成したコンポーネントを登録することで、
Blade 内で使用することが可能になります。
```diff
import ExampleComponent from './components/ExampleComponent.vue';
import LaravelTop from './components/LaravelTop.vue';
+ // 作成したコンポーネントを import する
+ import HogeComponent from './components/HogeComponent.vue';
app.component('example-component', ExampleComponent);
app.component('laravel-top', LaravelTop);
+ // Vue インスタンス (app) に登録する
+ app.component('hoge-component', HogeComponent);
```

### 作成したコンポーネントを Blade に配置する
`resources/vies/welcome.blade.php` にはサンプルの Vue コンポーネント (LaravelTop.vue) を配置してあります。  
コンポーネントへ PHP の値を受け渡しているため、そのあたりも参考にしてください。  
ポイントとしては、
- .vue ファイルに直接 PHP を埋め込むことはできない
- Vue コンポーネントは `props` を定義することで親から値を受け取れる
- Blade ファイルは `{{}}` 内に PHP の値を出力できる

といったところです。  
これらを組み合わせることでバックエンドから受け取った値を Vue 上に表示することができると思います。  
Blade そのものの記載方法については[公式ドキュメント](https://readouble.com/laravel/9.x/ja/blade.html)をご参照ください。
