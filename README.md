# 起動手順
## 前提
環境を構築するため、下記のアプリケーションをインストールしてください。  
インストール方法はリンク先をご参照ください。
- [Visual Studio Code](https://azure.microsoft.com/ja-jp/products/visual-studio-code/)
  - 拡張機能 [Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [WSL2](https://learn.microsoft.com/ja-jp/windows/wsl/install) (Windows ユーザのみ)

## 初回起動時

1. 任意のディレクトリにこのリポジトリを `git clone` します。
```bash
git clone <このリポジトリの URL>
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
  ├ .github
  ├ app
  │  ├ Console
  │  ├ Exceptions
  │  ├ Http
  │  │  ├ <strong>Controllers</strong> ← URL アクセス時の挙動を定義
  │  │  ├ Middleware
  │  │  └ Kernel.php
  │  ├ <strong>Models</strong> ← データベースのテーブル構造を Laravel で使用するための定義
  │  └ Providers
  ├ bootstrap
  ├ config
  ├ database
  │  ├ factories
  │  ├ <strong>migrations</strong> ← データベースの構成を定義
  │  └ <strong>seeders</strong> ← データベースに初期登録しておくデータを定義
  ├ lang
  ├ <del>node_modules</del> ← npm パッケージマネージャが自動生成するディレクトリ（編集禁止）
  ├ public
  ├ resources
  │  ├ css
  │  │  └ app.css
  │  ├ js
  │  │  ├ <strong>components</strong> ← Vue.js で画面のコントロールを作成
  │  │  ├ <strong>app.js</strong> ← components に作成したモジュールの読み込み
  │  │  └ bootstrap.js
  │  ├ <strong>sass</strong> ← UI のスタイルを定義
  │  │  ├ app.scss
  │  │  └ _variables.scss
  │  └ <strong>views</strong> ← UI を HTML(Blade) で作成して配置
  │      ├ auth
  │      └ <mark>layouts</mark> ← 画面の共通レイアウト（ヘッダ・フッタ等）の定義
  ├ routes
  │  ├ api.php
  │  ├ channels.php
  │  ├ console.php
  │  └ <strong>web.php</strong> ← ルーティングの定義
  ├ storage
  ├ tests
  └ <del>vendor</del> ← composer パッケージマネージャが自動生成するディレクトリ（編集禁止）
</pre>

## データベース関係
マイグレーション、シーダー等の ORM (Object-Relational Mapping) の機能概要について説明します。  
これらは Laravel (Eloquent ORM) だけでなく、 ORM の普遍的な概念ですので、覚えておくと今後の役に立つかと思います。

### マイグレーション
詳細は[Laravel 公式ドキュメント - マイグレーション](https://readouble.com/laravel/9.x/ja/migrations.html)の項を参照いただくほうが良いと思いますが、概要だけ説明します。

#### マイグレーションファイルの生成
新しいテーブルを作成したい、既存のテーブルの構造を変更したいといった場合、以下のコマンドで作成用・変更用ファイルのテンプレートを生成することができます。
```bash
php artisan make:migrate <マイグレーション名>
```
`<マイグレーション名>` の部分は適切に命名することで、 Laravel がいい感じのテンプレートを作ってくれます。  
例えば `create_hoge_table` とした場合、 `hoge` という名前の `table` を `create` するのに適切なテンプレートが選択されます。  
ファイルは `database/migrations` 配下に `<タイムスタンプ>_<マイグレーション名>.php` の形で生成されます。  
後述するマイグレーションの実行時は `<タイムスタンプ>` の昇順に実行されるため、依存関係を意識してファイルの作成を行うとよいでしょう。

#### 実行と取り消し
マイグレーションを実行する場合は以下のコマンドを実行します。
```bash
php artisan migrate
```
このコマンドは環境作成時にも実行していますが、 Laravel はどのマイグレーションファイルを実行したかを覚えているため、基本的に同じマイグレーションファイルは二度と実行されません。  
もちろん同じマイグレーションを再実行する方法はあり、ローカルでの開発時は期待通りに動くまで修正と実行と取り消しを繰り返すことになると思います。  
マイグレーションの実行を取り消す場合は以下のコマンドを実行します。
```bash
php artisan migrate:rollback
```
また、以下のコマンドではすべてのマイグレーションの再実行が行えます。  
環境をきれいな状態に戻したい場合などに有用でしょう。
```bash
php artisan migrate:fresh
```

#### 注意事項
期待通りに動くようになったところで `git commit` 、 `git push` することになると思います。  
レビューを経てマージされた後はマイグレーションファイルに変更を加えてはいけません。  
なぜならマージ後はチームメンバーがマイグレーションを実行してしまっているため、マイグレーションファイルを修正したとしてもマイグレーションの取り消しを行わない限りは再実行されないためです。  
こういった場合は新たなマイグレーションファイルを生成し、変更したい内容を記載するとよいでしょう。

### シーダー
テーブルに初期データを入れておきたいシチュエーションはしばしばあります。  
例えばユーザ登録の際に性別や居住地をプルダウンから選択するケースで、ユーザテーブルは次のようなレイアウトになると思います。

ユーザテーブル
|id|name|gender|location|
|-:|-|-|-|
|1|山田|男性|東京都|
|2|田中|女性|大阪府|
|3|鈴木|未回答|海外|

通常こういったデータの場合、 `gender` や `location` はマスタテーブルを用意して、ユーザテーブルには結合キーを登録するように設計することが一般的だと思います。

性別マスタ
|id|code|name|
|-:|-|-|
|1|1|男性|
|2|2|女性|
|3|9|未回答|

居住地マスタ  
※余談ですが、 `code` には[都道府県番号](https://www.mhlw.go.jp/topics/2007/07/dl/tp0727-1d.pdf)が設定される想定です
|id|code|name|longitude|latitude|
|-:|-|-|-:|-:|
|1|13|東京都|139.69167|35.68944|
|2|27|大阪府|135.52|34.68639|
|3|99|海外|null|null|

上記のようなマスタを用意した場合は、ユーザテーブルは次のように設計できます。  
※ `code` 列を使用してマスタと結合する想定です。
トラン同士の結合を行う場合は `id` 列を用いたほうが良い結果を生むことが多いと思います。
このあたりの設計の是非については `ナチュラルキー` や `サロゲートキー` 等をキーワードに検索してみてください。
さらに余談ですが ORM やそれを用いたフレームワークでは `id` 列を主キーとした設計を前提としていることが多いため、脳死で `id` 作っとけって考え方もありです。

ユーザテーブル
|id|name|gender_code|location_code|
|-:|-|-|-|
|1|山田|1|13|
|2|田中|2|27|
|3|鈴木|9|99|

このような設計を行いマスタテーブルに初期値を投入する場合にシーダーを定義するとよいでしょう。  
例によって詳細は[Laravel 公式ドキュメント - シーディング](https://readouble.com/laravel/9.x/ja/seeding.html)の項をご覧いただき、ここでは概要の説明に留めます。

#### シーダーファイルの生成
以下のコマンドでファイルを生成できます。
```bash
php artisan make:seeder <シーダー名>
```
ファイルは `database/seeders` 配下に `<シーダー名>.php` の形で生成されます。  
生成されたファイルの `run` メソッド中に投入したい初期値の定義を行いましょう。  
最後に `database/seeders/DatabaseSeeder.php` の `run` メソッドから作成したシーダーを呼び出すように処理を記述しましょう。
```php
public function run() {
    $this->call([
        // 生成したシーダーファイル呼び出しの定義
        <シーダー名>::class,
        // ここは配列なので、呼び出したいシーダーをカンマ区切りで複数定義可能
    ]);
}
```

#### 実行
以下のコマンドで `database/seeders/DatabaseSeeder.php` を実行することができます。
```bash
php artisan db:seed
```
特定のシーダーファイルのみを実行したい場合は `--class` パラメータを付与します。
```bash
php artisan db:seed --class=<シーダー名>
```
マイグレーションのコマンドと合わせて実行することもできます。  
この場合はトランに蓄積された（画面を操作して登録された）データは削除されてしまうためご注意ください。
```bash
php artisan migrate:fresh --seed
```

#### 注意事項
シーダーはマイグレーションと異なり、実行するたびにデータの投入が行われます。  
重複データが発生する可能性がありますが、テーブルにユニーク制約を設定するなどで防ぐことができます。  
例えば前述のマスタテーブルは `code` 列にユニークインデックスを張ることを想定しています。

### テーブルをプログラム上で扱う
これまではデータベースにテーブルを作成し、初期データの登録までを行いました。  
これらをプログラム上から扱う方法についてを簡単に説明します。  
繰り返しになりますが、[Laravel 公式ドキュメント - Eloquentの準備](https://readouble.com/laravel/9.x/ja/eloquent.html)の項が詳しくわかりやすく記載されているため、ここでは概要に留めます。

#### モデルクラスの生成
プログラム上でテーブルの操作を行うにはモデルクラスを作成する必要があります。  
モデルクラスは以下のコマンドで生成することができます。
```bash
php artisan make:model <モデルクラス名>
```
また `--migration` または `-m` パラメータを付与することで同時にマイグレーションファイルの生成も行うことができます。  
テーブルを新規に作成する場合はこちらのコマンドを実行したほうがよいでしょう。
```bash
php artisan make:model <モデルクラス名> --migration
# または
php artisan make:model <モデルクラス名> -m
```
この時生成されるファイルはそれぞれ以下に格納されます。
- モデルクラス: `app/Models/<モデルクラス名>.php`
- マイグレーション: `database/migrations/<タイムスタンプ>_create_<モデルクラス名:小文字/複数形>_table.php`

ちなみに `<モデルクラス名>` は単数形で命名することを推奨します。  
テーブルは1行のモデルの集合と考えられるため、 Laravel はテーブル名を複数形で生成するからです。  
例えば `User` というモデルを格納するテーブルの場合 Laravel は `users` という名前でテーブルを生成します。  
※またまた余談となりますが `Country` の場合は `countries` 、 `Person` の場合は `people` といったように特殊な変化にも対応しています。単純に末尾に `s` をつけているわけではなく、単語ごとに単数形と複数形の辞書を持っているようです。

#### CRUD の方法
CRUD とは Create Read Update Delete の頭文字を取った操作のことです。  
データベースではそれぞれ `insert` `select` `update` `delete` に該当します。  
※システム開発に携わる場合、データベース関係にかかわらずよく耳にするキーワードなので覚えておくとよいでしょう。

以下は `insert` `select` `update` `delete` を連続して行う例です。  
実際にこういったシチュエーションはないと思いますが、コードサンプルとしてご確認ください。
```php
// User はシーダーで説明に使用したユーザテーブルを想定
User::create([
    // id は一般的にオートインクリメントの型が採用されるため、 insert 時に指定しない
    'name' => '山田',
    'gender_code' => '1',
    'location_code' => '13',
]);

// 作成したユーザ「山田」を name 列でフィルタして1行目を取得
// 他の山田さんがヒットする可能性があるため、ユニークとなる列(メールアドレス等)を追加してそっちで絞ったほうがいい
$user = User::where('name', '山田')->first();
// 主キー (id) が分かっている場合はこれで取れる
// $user = User::find(1);

// 東京から大阪住まいに変更
$user->update(['location_code' => '27']);

// 「山田」のレコード削除
$user->delete();
```
`insert` と `select` はモデルクラスが持つ静的メソッドを使用して登録・読込を行い、 `update` と `delete` は取得結果のインスタンスメソッドを実行することで実現しています。  
これら以外にもモデルクラスが持つメソッドは多数あるため、ドキュメントを参照して適切な実装を行ってください。

#### リレーションについて
テーブル同士の関係のことを `リレーション` といいます。  
例えばユーザテーブルは `gender_code` で性別マスタと結びつき、 `location_code` で居住地マスタと結びつきます。  
モデルクラスではこういったテーブル間の関係性を定義することができます。
詳細は[Laravel 公式ドキュメント - リレーション](https://readouble.com/laravel/9.x/ja/eloquent-relationships.html)の項をご参照ください。

ユーザテーブルと性別マスタの関係は、性別マスタが親でユーザテーブルが子の関係といえます。  
なぜなら、ユーザの性別は性別マスタに定義されたものから選択されるべきで、すなわち外部キーであるためです。  
そのためリレーションを考える場合の主観は性別マスタであるべきでしょう。  
性別マスタから見たユーザテーブルは、一つの性別（レコード）に対して複数のユーザが存在しうる関係だといえます。  
というのも、男性も女性もそれぞれ世界の人口の約半分ずつ存在しており、一人で男性・女性の両方の属性を持つことはあり得ないからです。

こういう場合、テーブルのリレーションを表す言葉として 1対多 (より正確には 1:0..n 等) と表現します。  
本来人類は男女どちらかの性を持っていますが、今回の性別マスタには「未回答」というレコードを定義しており、すべてのユーザが「未回答」を選択する、あるいはすべてのユーザが「未回答」以外を選択するといったケースが考えられます。  
また、ユーザの登録件数が性別マスタのレコード件数より少ない場合、必然的に選択されなかったレコードが存在することになります。  
そのため、リレーションとしては**性別マスタ1件に対し、ユーザテーブルは0件以上のn件存在する**という風に定義できます。  
※長々と書きましたが、通常はマスタとトランの関係は9割方 1:0..n の関係なので、ここまで考慮する必要はありません。

ではこれをモデルクラスで表現してみましょう。  
```php
// 性別マスタ (app/Models/Gender.php)
class Gender extends Model {
    public function users() {
        // id を結合キーに使用している場合は第2引数、第3引数の指定は不要
        // 今回はこだわり()の code 結合を採用しているため指定が必須
        return $this->hasMany(User::class, 'gender_code', 'code');
    }
}

// ユーザテーブル (app/Models/User.php)
class User extends Model {
    public function gender() {
        // Gender クラスに同じ
        // ちなみに第2引数が外部キー(子テーブルの結合キー)で、第3引数が親テーブルの結合キーです
        return $this->belongsTo(Gender::class, 'gender_code', 'code');
    }
}
```
実際に使用する場合は以下のように記述します。
```php
// users の id = 1 のユーザの性別名称が代入される
$userGenderName = User::find(1)->gender->name;

// 登録されている男性をすべて取得
$men = Gender::where('code', '1')->first()->users();
```
リレーションの定義としては以上となります。  
このほかにも 1対1 や 多対多 のリレーションが存在しますので、必要に応じて調査してみてください。

## ルーティング関係
ルーティングとは URL アクセスが起こった際、どういった処理を行うのかを定義することです。  
Laravel ではドメインからの相対パスと HTTP メソッドの組み合わせに対し何を行うのかを `routes/web.php` に記述します。  
※バックエンドとフロントエンドが完全に分離している (Laravel を API サーバとして運用する) 場合は `routes/api.php` に記述するのがお作法のようです。
両者の違いについては[こちら](https://qiita.com/pyon141/items/b3a2d2865c70b6a73ae2)が参考になります。

### ルーティング
例によって[Laravel 公式ドキュメント - ルーティング](https://readouble.com/laravel/9.x/ja/routing.html)の項が非常に詳細に記載されていますのでこちらをご参照いただくことを強く推奨します。  
ここでは簡単に概要と注意事項を記載します。

`web.php` には `Route::<任意の HTTP メソッド>(<ドメインからの相対パス>, <行う処理>);` という形でルーティングを定義できます。  
例えば環境構築時点では次の定義があります。
```php
Route::get('/', function () {
    return view('welcome');
});
```
これは `/` (index) に GET リクエストがあった場合に `resources/views/welcome.blade.php` を表示する、という意味になります。  
静的なページを表示する場合は特に問題ないのですが、サーバ日時を表示したいといった要望に対応した場合は次のように修正を加える必要があります。
```php
Route::get('/', function () {
    // 現在時刻を取得
    $currentTimestamp = date("Y-m-d H:i:s");
    // compact は引数のキー名と同じ変数名の値をレスポンスに含めることができる関数
    return view('welcome', compact('currentTimestamp'));
});
```
では今度はサーバ日時に加えてアクセスポイントの情報と周辺の天気・気温・湿度も表示したいと要望が出たとします。  
要望に応えているうちに `web.php` は際限なく肥大化していきます。
```php
Route::get('/', function () {
    // 現在時刻を取得
    $currentTimestamp = date("Y-m-d H:i:s");
    // アクセスポイントを取得
    $accessPoint = getAccessPoint();
    // アクセスポイント周辺の天気を取得
    $weather = getWeather($accessPoint);
    // アクセスポイント周辺の気温を取得
    $temperature = getTemperature($accessPoint);
    // アクセスポイント周辺の湿度を取得
    $humidity = getHumidity($accessPoint);
    // 返さないといけない情報が増えてきた…
    return view('welcome', compact('currentTimestamp', 'accessPoint', 'weather', 'temperature', 'humidity'));
});
```

こういう場合はコントローラクラスを定義し処理を委譲するようにしましょう。  
コントローラは `app/Http/Controllers` 配下に作成します。  
今回の場合は `WelcomeController.php` を作成し、以下のように修正するとよいでしょう。
```php
// app/Http/Controllers/WelcomeController.php
<?php

namespace App\Http\Controllers;

class WelcomeController extends Controller {
    public function index() {
        $currentTimestamp = date("Y-m-d H:i:s");
        $accessPoint = getAccessPoint();
        $weather = getWeather($accessPoint);
        $temperature = getTemperature($accessPoint);
        $humidity = getHumidity($accessPoint);
        return view('welcome', compact('currentTimestamp', 'accessPoint', 'weather', 'temperature', 'humidity'));
    }
}

// routes/web.php
use App\Http\Controllers\WelcomeController;

// / (index) に GET リクエストがあった場合に WelcomeController というクラスが持つ index メソッドが実行される、という定義
Route::get('/', [WelcomeController::class, 'index']);
```

## フロント関係
Vue.js と Laravel の連携方法について説明します。

### Vue コンポーネントの作成方法
`resources/js/components` 配下に vue ファイルを作成します。  
作成方法についてはサンプルで配置しているファイルや、[Vue.js 公式ドキュメント](https://v3.ja.vuejs.org/guide/introduction.html)等を参考にしてください。  
基本的には `<template>` タグの中に HTML でレイアウトを、 `<script>` タグの中に JavaScript でイベントやデータ定義等のフロント側ロジックを、 `<style>` タグの中に CSS で HTML の装飾を行うと覚えておけばだいたい問題ないです。  
ライフサイクル（ボタン押下などではなくバックグラウンドで自然と発生するイベント）については[こちら](https://v3.ja.vuejs.org/guide/instance.html#%E3%83%A9%E3%82%A4%E3%83%95%E3%82%B5%E3%82%A4%E3%82%AF%E3%83%AB%E3%82%BF%E3%82%99%E3%82%A4%E3%82%A2%E3%82%AF%E3%82%99%E3%83%A9%E3%83%A0)が参考になると思います。

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
`resources/views/welcome.blade.php` にはサンプルの Vue コンポーネント (LaravelTop.vue) を配置してあります。  
コンポーネントへ PHP の値を受け渡しているため、そのあたりも参考にしてください。  
ポイントとしては、
- .vue ファイルに直接 PHP を埋め込むことはできない
- Vue コンポーネントは `props` を定義することで親から値を受け取れる
- Blade ファイルは `{{}}` 内に PHP の値を出力できる

といったところです。  
これらを組み合わせることでバックエンドから受け取った値を Vue 上に表示することができると思います。  
Blade そのものの記述方法については[Laravel 公式ドキュメント - Blade テンプレート](https://readouble.com/laravel/9.x/ja/blade.html)の項をご参照ください。

### 注意事項
Vue コンポーネントを使用する場合 `resources/js/app.js` を Blade 側で読み込んであげる必要があります。  
環境構築時点での `resources/views/welcome.blade.php` は `resources/views/layouts/app.blade.php` をレイアウトとして使用しており、その中で `resources/js/app.js` を読み込むようになっています。  
新しいレイアウトを定義する場合はこの点にご注意ください。
