# PHP Composer

- PHP依存関係管理ツール
- PHPやLaravelのアプリケーションに必要なライブラリがある場合、Composerを通してインストールすると、依存関係を解決してくれる
  - ライブラリAを動作させるために必要なライブラリを合わせてインストールしてくれる

---

## composer.json

- プロジェクトの設定、ライブラリの依存関係が記述されたファイル
- `composer require` コマンドを実行すると自動的に更新される
- 手動で編集することもできる

### 主なフィールド

```json
{
    "name": "vendor/project-name",
    "description": "プロジェクトの説明",
    "require": {
        "php": "^8.1",
        "laravel/framework": "^10.0"
    },
    "require-dev": {
        "phpunit/phpunit": "^10.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "app/"
        }
    },
    "scripts": {
        "test": "phpunit"
    }
}
```

| フィールド      | 説明                                               |
|-----------------|----------------------------------------------------|
| `name`          | パッケージ名（`ベンダー名/パッケージ名` の形式）   |
| `description`   | パッケージの説明                                   |
| `require`       | 本番環境で必要なライブラリと対応バージョン         |
| `require-dev`   | 開発環境のみで必要なライブラリ（テストツールなど） |
| `autoload`      | オートローディングのクラスマッピング設定           |
| `scripts`       | カスタムコマンドの定義                             |

---

## composer.lock

- 実際にインストールしたライブラリとその**正確なバージョン**を記録しているファイル
- プロジェクトのメンバーと共有することで、全員が同じバージョンのライブラリをインストールできる
- `composer install` 実行時はこのファイルを参照してインストールする
- **Git管理に含める**（チーム開発では必須）

> `composer.json` が「こんなバージョンを使いたい」という意図の宣言なのに対して、
> `composer.lock` は「実際にこのバージョンをインストールした」という記録。

---

## ライブラリのインストール方法

### コマンド

```bash
# ライブラリをインストール（composer.json と composer.lock を更新）
composer require [ライブラリ名]

# 例: Guzzle HTTP クライアントをインストール
composer require guzzlehttp/guzzle

# バージョンを指定してインストール
composer require guzzlehttp/guzzle:^7.0

# 開発環境専用でインストール（--dev フラグ）
composer require --dev phpunit/phpunit

# composer.lock に基づいて全ライブラリをインストール（チーム開発・デプロイ時）
composer install

# 本番環境では開発用ライブラリを除外してインストール
composer install --no-dev
```

### バージョン表記

| 表記           | 意味                                              | 例                        |
|----------------|---------------------------------------------------|---------------------------|
| `1.2.3`        | 完全一致（厳密指定）                              | 必ず 1.2.3                |
| `^1.2.3`       | メジャーバージョンを固定（後方互換を許可）        | 1.2.3 以上 2.0.0 未満     |
| `~1.2.3`       | パッチバージョンのみ更新を許可                    | 1.2.3 以上 1.3.0 未満     |
| `>=1.2.3`      | 指定バージョン以上                                | 1.2.3 以上ならなんでも    |
| `1.2.*`        | ワイルドカード                                    | 1.2.x すべて              |
| `*`            | すべてのバージョン（非推奨）                      | 制約なし                  |

> **よく使うのは `^`（キャレット）**。
> Composer 公式が推奨しており、`composer require` のデフォルトも `^` が付く。

### セマンティックバージョン（SemVer）

バージョン番号を `MAJOR.MINOR.PATCH` の3つの数字で管理するルール。

```
1  .  2  .  3
↑     ↑     ↑
MAJOR MINOR PATCH
```

| 種別      | 上がるタイミング                         | 例                         |
|-----------|------------------------------------------|----------------------------|
| `MAJOR`   | 後方互換性のない変更（破壊的変更）       | API の仕様変更、削除        |
| `MINOR`   | 後方互換性を保ちつつ機能を追加           | 新しいメソッドの追加        |
| `PATCH`   | バグ修正（機能の変化なし）               | バグ修正、セキュリティパッチ |

`^1.2.3` が「1.x.x なら安全にアップデートできる」と判断できるのは、セマンティックバージョンに従っていれば MAJOR が上がらない限り後方互換性が保たれるため。

---

## ライブラリのアップデート方法

```bash
# 特定のライブラリをアップデート（composer.json の制約範囲内で最新に）
composer update [ライブラリ名]

# 全ライブラリをアップデート
composer update

# 本番用ライブラリのみアップデート（開発用を除く）
composer update --no-dev
```

バージョン制約を超えてアップデートしたい場合は `composer.json` の `require` を手動で変更してから実行する。

```json
// composer.json の制約を変更
"require": {
    "guzzlehttp/guzzle": "^8.0"  // ^7.0 から変更
}
```

```bash
# 変更後に実行
composer update guzzlehttp/guzzle
```

### ライブラリのアンインストール

```bash
composer remove [ライブラリ名]

# 例
composer remove guzzlehttp/guzzle
```

---

## メリット

### 1. ライブラリのインストール / アップグレード / アンインストールをコマンドで自動化できる
- 依存性の自動解決
- パッケージのまとめてインストール

### 2. アプリ単位にパッケージを管理

### 3. ライブラリの自動ローディング機能
- クラスライブラリの個別インポートが不要、自動で読み出してくれる
- `autoload.php` をインクルードするだけで必要なファイルを自動でロードしてくれる

```php
require_once __DIR__ . '/vendor/autoload.php';

// これだけで composer でインストールしたすべてのライブラリが使える
use GuzzleHttp\Client;
$client = new Client();
```

### 4. 対応ライブラリが豊富
- [Packagist](https://packagist.org/) で公開されている数十万のパッケージを利用可能

---

## Composerのインストール

### コマンドでのローカル環境へのインストール

```bash
# インストーラをダウンロードして実行
curl -sS https://getcomposer.org/installer | php

# グローバルにインストール（どこでも composer コマンドが使えるようになる）
mv composer.phar /usr/local/bin/composer

# インストール確認
composer --version
```

または公式インストーラを使う方法:

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

### Dockerでのインストール

**マルチステージビルドを使う方法（推奨）**

```dockerfile
# Composer の公式イメージからバイナリだけコピー
FROM composer:latest AS composer

FROM php:8.2-fpm
COPY --from=composer /usr/bin/composer /usr/bin/composer
```

**RUN コマンドでインストールする方法**

```dockerfile
FROM php:8.2-fpm

RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer
```

**docker-compose での使い方例**

```yaml
services:
  app:
    build: .
    volumes:
      - .:/var/www/html
    working_dir: /var/www/html
```

```bash
# コンテナ内で composer コマンドを実行
docker compose exec app composer install
docker compose exec app composer require [ライブラリ名]
```

---

## PECL

- **PHP Extension Community Library**
- PHP の**拡張モジュール（Extension）**を管理するツール
- Composer がPHPライブラリ（クラスファイル）を管理するのに対し、PECL は PHP 本体に組み込む C言語製の拡張機能を管理する
- インストールした拡張機能は `php.ini` に `extension=拡張名` と追記して有効化する

> Composer vs PECL
>
> | ツール   | 対象                        | 言語 |
> |----------|-----------------------------|------|
> | Composer | PHPライブラリ（クラス群）   | PHP  |
> | PECL     | PHP拡張機能（エンジン拡張） | C    |

---

### PECL 自体のインストール

PECL は `pear` コマンドに含まれているため、PEAR をインストールすることで使用できる。

```bash
# Ubuntu / Debian 系
apt-get install -y php-pear php-dev

# CentOS / RHEL 系
yum install -y php-pear php-devel

# インストール確認
pecl version
```

> `php-dev`（または `php-devel`）は拡張機能のビルドに必要なヘッダファイルを含むパッケージ。
> これがないとインストール時にコンパイルエラーになる。

---

### 拡張機能のインストール手順

#### 基本的な流れ

```bash
# 1. 拡張機能をインストール（コンパイル・ビルドが走る）
pecl install [拡張名]

# 2. php.ini に追記して有効化
echo "extension=[拡張名].so" >> /usr/local/etc/php/conf.d/[拡張名].ini

# 3. PHP を再起動して反映（Web サーバー経由の場合）
service php-fpm restart
# または Apache の場合
service apache2 restart

# 4. 有効化されているか確認
php -m | grep [拡張名]
```

#### バージョンを指定してインストール

```bash
# バージョンを指定
pecl install redis-5.3.7

# ベータ版をインストール
pecl install [拡張名]-beta
```

---

### php.ini の設定

PECLでインストールした拡張は自動では有効にならないため、手動で設定が必要。

```bash
# php.ini の場所を確認
php --ini

# 設定ファイルに追記する方法（例: redis）
echo "extension=redis.so" >> /usr/local/etc/php/conf.d/redis.ini

# 専用の .ini ファイルを作成する方法（推奨）
echo "extension=redis" > /usr/local/etc/php/conf.d/20-redis.ini
```

```ini
; /usr/local/etc/php/conf.d/20-redis.ini の内容例
extension=redis

; xdebug のように zend_extension を使う拡張もある
zend_extension=xdebug
```

> `extension` と `zend_extension` の違い
>
> | 設定キー         | 用途                                   |
> |------------------|----------------------------------------|
> | `extension`      | 通常の拡張（redis, imagick など）      |
> | `zend_extension` | Zend エンジンレベルの拡張（xdebug など）|

---

### Docker でのインストール

Docker イメージで PECL 拡張を使う場合は Dockerfile に記述する。

```dockerfile
FROM php:8.2-fpm

# ビルドに必要な依存ライブラリをインストール
RUN apt-get update && apt-get install -y \
    libredis-dev \
    libmagickwand-dev \
    --no-install-recommends

# PECL で拡張をインストールして有効化
RUN pecl install redis \
    && docker-php-ext-enable redis

# imagick のインストール例
RUN pecl install imagick \
    && docker-php-ext-enable imagick
```

> PHP 公式イメージには `docker-php-ext-enable` というヘルパースクリプトが用意されており、
> `php.ini` への追記を自動でやってくれる。

```dockerfile
# 公式イメージ組み込みの拡張（PECL 不要）は docker-php-ext-install で入れる
RUN docker-php-ext-install pdo pdo_mysql
```

---

### よく使うPECL拡張

| 拡張名      | 用途                               | インストールコマンド       |
|-------------|------------------------------------|----------------------------|
| `redis`     | Redis クライアント                 | `pecl install redis`       |
| `imagick`   | 画像処理（ImageMagick）            | `pecl install imagick`     |
| `xdebug`    | デバッグ・カバレッジ計測           | `pecl install xdebug`      |
| `mongodb`   | MongoDB ドライバ                   | `pecl install mongodb`     |
| `apcu`      | PHP プロセス内キャッシュ           | `pecl install apcu`        |
| `swoole`    | 非同期処理・コルーチン             | `pecl install swoole`      |

---

### 主要コマンド一覧

```bash
pecl install [拡張名]        # インストール
pecl install [拡張名]-[ver]  # バージョン指定インストール
pecl uninstall [拡張名]      # アンインストール
pecl upgrade [拡張名]        # アップグレード
pecl upgrade-all             # 全拡張をアップグレード
pecl list                    # インストール済み拡張を一覧表示
pecl search [キーワード]     # 拡張を検索
pecl info [拡張名]           # 拡張の詳細情報を表示
```

---

## PEAR

- **PHP Extension and Application Repository**
- Composer が登場する以前（〜2010年代初頭）に広く使われていた PHP のパッケージ管理ツール
- 現在は**ほぼ使われておらず、Composer に取って代わられた**
- PEAR でインストールしたライブラリはグローバルにインストールされる（アプリ単位の管理が難しかった）
- レガシーなプロジェクトのメンテナンスで稀に遭遇することがある

```bash
# PEAR でのインストール例（参考）
pear install Mail
```

> 新規プロジェクトでは PEAR ではなく **Composer を使う**のが現在の標準。
