#Laravel 

- Laravelに用意されている認証のスターターキット
- ユーザーの登録、ログイン、ログアウトなどの基本的な認証機能を提供してくれる

## インストール方法
- composerにLaravel Breezeを追加
```php
// 先にcomposerの更新が必要かも
composer self-update

//  collisionも必要
composer require nunomaduro/collision --dev

composer require laravel/breeze --dev
```
- Laravel Breezeをインストール
```
php artisan breeze:install
```
- 認証関連の画面を何で作成するか聞かれる
```
┌ Which Breeze stack would you like to install? ───────────────┐
 │ › ● Blade with Alpine                                        │
 │   ○ Livewire (Volt Class API) with Alpine                    │
 │   ○ Livewire (Volt Functional API) with Alpine               │
 │   ○ React with Inertia                                       │
 │   ○ Vue with Inertia                                         │
 │   ○ API only                                                 │
 └──────────────────────────────────────────────────────────────┘
 
 // ダークモードのサポートを行うか
  ┌ Would you like dark mode support? ───────────────────────────┐
 │ Yes                                                          │
 └──────────────────────────────────────────────────────────────┘

// PHP UnitのテストをPestに置き換えるか
 ┌ Which testing framework do you prefer? ──────────────────────┐
 │   ○ Pest                                                     │
 │ › ● PHPUnit                                                  │
 └──────────────────────────────────────────────────────────────┘

```
- 認証画面に必要なファイルが作成される
	- コントローラー関係：www/app/Http/Controllers/Auth/
	- リクエスト：www/app/Http/Requests/Auth/
	- ビュー関係：
		- www/app/View/Components/
		- www/resources/views/auth/
	- レイアウト関係
		- www/resources/views/layouts/
	- ルート関係
		- www/routes/auth.php
---
## 認証機能の全体像
### ガード
- 認証方法の種類を指定
- セッションによる認証、トークンによる認証などが指定できる
- 設定は`config/auth.php`に記載
### プロパイダ
- ユーザー情報の永続化方法の種類の指定
- Eloquent、クエリビルダなどが指定できる
- 設定は`config/auth.php`に記載

- 例：usersプロパイダーは`User`モデルを利用して、永続化方法はEloquentを指定
	- 「`User`モデルを利用して、セッションによる認証をする」という設定になっている
```php

'guards' => [
'web' => [ //ガードの名前
'driver' => 'session', //セッションを用いた認証が指定されている
'provider' => 'users', //プロパイダーを指定（usersテーブルを使用）
],
],

'providers' => [
'users' => [ //プロバイダーの名前
'driver' => 'eloquent', //Eloquentを用いた認証が指定されている
'model' => App\Models\User::class, //ユーザーモデルを指定（Userクラスを使用）
],
],
```

- 認証用に利用するモデルは`Authenticatable`を継承したモデルである必要がある
```php
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
}
```

## パスワードのバリデーション
- `Auth/`は以下のコントローラーでパスワードや登録内容のバリデーションを行う
- パスワードのバリデーションは`Rules\Password`で定義する
	- `Rules\Password::defaults()`：デフォルトのパスワードルール
	- `Rules\Password::min(8)->mixedCase()`:8文字以上、1文字以上の大文字、小文字を含むこと
		- `letters()`:1文字以上のアルファベットを含む
- 参考ページ：
	- https://laravel.com/docs/12.x/validation#validating-passwords
- 自分で設定する場合は、採用文字数の設定が必須なので、==必ず`min`メソッドから始める==

## Authクラス
- 認証に関するクラス
	- 以下の機能を有する
		- 認証済みとしてユーザー情報を保持する
		- 入力パスワードとDBパスワードの照合
		- 認証済みの情報を返す
- `only`メソッド
	- リクエストの中から引数に指定した内容だけを取り出す
	- 例：`this->only('email', 'password')`
		- `email`と`password`だけを取り出している