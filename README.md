# Laravel8のセッションをファイルセッションからredisに変更する
## 手順
### laravel8インストール
```
mkdir test
composer create-project --prefer-dist laravel/laravel .
# 下記コマンドでも可
# composer create-project --prefer-dist laravel/laravel test
# cd test
```
### ログイン認証用パッケージインストール
```
composer require laravel/jetstream
```
### ログイン認証追加コマンド実行
```
php artisan jetstream:install livewire
```
### パッケージのインストール&&コンパイル
```
npm install && npm run dev
```
### /.env, /config/database.phpを変更
- /config/database.phpは.envの正しく変更すれば特に修正の必要はありませんので、各自必要な場合は修正してください
```env:/.env
DB_CONNECTION=mysql
DB_HOST=10.10.xx.yy # 各自環境に合わせてください
DB_PORT=3306 
DB_DATABASE=test # データベース名は各自決めてください
DB_USERNAME=root # 各自環境に合わせてください
DB_PASSWORD=root # 各自環境に合わせてください
```
### データベースの作成
- 前の手順で適宜したデータベース名と同じ値にしてください
```
mysql -u root -p
mysql> create database test;
```
### マイグレーションの実行
```sh
php artisan migrate
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (34.54ms)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (36.11ms)
Migrating: 2014_10_12_200000_add_two_factor_columns_to_users_table
Migrated:  2014_10_12_200000_add_two_factor_columns_to_users_table (35.28ms)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (36.09ms)
Migrating: 2019_12_14_000001_create_personal_access_tokens_table
Migrated:  2019_12_14_000001_create_personal_access_tokens_table (58.10ms)
Migrating: 2021_01_20_003347_create_sessions_table
Migrated:  2021_01_20_003347_create_sessions_table (80.96ms)
```
- mysqlから確認
```mysql
mysql> show tables;
+------------------------+
| Tables_in_test         |
+------------------------+
| failed_jobs            |
| migrations             |
| password_resets        |
| personal_access_tokens |
| sessions               |
| users                  |
+------------------------+
6 rows in set (0.01 sec)
```

### Factoryを使ってユーザを追加する
#### Factoryを実行する前にLaravelの設定が日本語が対応しているか確認
- 下記変更
```php:/config/app.php
<?php

return [
    // 省略
    'timezone' => 'Asia/Tokyo', // タイムゾーン
    'locale' => 'ja', // 利用する言語
    'fallback_locale' => 'en', // 日本語がない場合の予備言語（そのままでOK）
    'faker_locale' => 'ja_JP', // ここがFactoryに影響します。ここがen_USのままだと英語名になります
    // 省略
];
```
#### Factoriesを使ってユーザを追加する
  - すでにUserテーブル用のFactoriesが存在するので、そちらを使う
  - /database/factories/UserFactory.php
- /database/seeders/DatabaseSeeder.phpでユーザFactoriesを実行する用に修正
```php:/database/seeders/DatabaseSeeder.php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        \App\Models\User::factory(10)->create(); // 初期はコメントアウトしてあるのでコメントアウトを外す
        # notes
        # factory(10)の引数を変更することで何件作成するか変更することができます(ここでは初期値10のまま実行します)
        # その他のテーブルに対してのFactoryを実行する場合にはここに追加することでできます
        # ここでは1度しか書かないので\App\Models~~と書いていますが、use App\Models\User;としUser::factory(10)->create();でも実行可能です
    }
}
```
- Seederの実行
```sh
php artisan db:seed
```
### データベースからユーザを確認
```
SELECT * FROM users LIMIT 1 \G
*************************** 1. row ***************************
                       id: 1
                     name: 若松 充
                    email: sayuri46@example.com
        email_verified_at: 2021-01-20 10:01:26
                 password: $2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi # password
        two_factor_secret: NULL
two_factor_recovery_codes: NULL
           remember_token: ffK945rXbN
          current_team_id: NULL
       profile_photo_path: NULL
               created_at: 2021-01-20 10:01:26
               updated_at: 2021-01-20 10:01:26
1 row in set (0.00 sec)
```

### URLをhttps固定にする
- 利用している環境の問題ですが、https通信をするのでLaravelのリンクURLをhttpsに強制します
```/app/Providers/AppServiceProvider.php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{

    // 省略

    public function boot()
    {
        //
        \URL::forceScheme('https'); // ここを追加
    }
}
```

### ログインする
- ログイン画面http://example.con/login # URLは各自の環境に合わせてください
  - 前の手順で確認したemail, passwordを入力
- ダッシュボードが開くことを確認


# まとめ
今回はLaravel8で追加されたjetstreamでログイン機能を作成しました。  
記事では触れていない機能がまだまだあるので、興味ある方は調べてみてください。
触れてない機能の例
- 新規登録ページ
- ログイン認証失敗時のエラー文言
- パスワードを忘れた時の処理
- ユーザ情報更新ページ
