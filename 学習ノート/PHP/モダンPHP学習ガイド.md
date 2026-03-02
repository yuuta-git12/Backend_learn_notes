#PHP 

# モダンPHP 完全学習ガイド

> 作成日: 2026年3月1日

---

## はじめに

PHPは現在もWebバックエンドの主要言語のひとつです。かつての「古くて汚いPHP」というイメージとは異なり、PHP 8.x系は型システムの強化・パフォーマンス向上・モダンな構文が揃った、非常に実用的な言語に進化しています。このガイドでは「モダンなPHP開発者」になるための具体的なロードマップを示します。

---

## ステップ 1：環境を整える

### 推奨環境

|ツール|推奨バージョン|用途|
|---|---|---|
|PHP|8.3以上|本体|
|Composer|最新版|パッケージ管理|
|Docker|最新版|開発環境の統一|
|VS Code / PhpStorm|最新版|エディタ|

### セットアップ方法

**Docker を使う（推奨）**

bash

```bash
# docker-compose.yml を使ってPHP+Nginx+MySQL環境を立ち上げる
docker compose up -d
```

Dockerを使うことで「自分のPCに直接インストールしない」ため、環境を壊しにくく、チームでの共有も容易です。

**ローカルにインストールする場合（Mac）**

bash

```bash
brew install php
brew install composer
```

---

## ステップ 2：PHP 8.x の新機能を理解する

モダンPHPの核心はPHP 8系の新機能にあります。順番に習得しましょう。

### 2-1. 型宣言（Type Declarations）

php

```php
// 引数・戻り値・プロパティに型を明示する
function greet(string $name): string {
    return "Hello, " . $name;
}

// Union型（PHP 8.0〜）
function formatId(int|string $id): string {
    return (string) $id;
}

// Intersection型（PHP 8.1〜）
function process(Countable&Iterator $data): void {
    // ...
}
```

**なぜ重要か**：型を明示することでバグを早期発見でき、IDEの補完も強力になります。

### 2-2. Enum（PHP 8.1〜）

php

```php
enum Status: string {
    case Active   = 'active';
    case Inactive = 'inactive';
    case Pending  = 'pending';
}

$status = Status::Active;
echo $status->value; // "active"
```

### 2-3. Fibers（PHP 8.1〜）

php

```php
$fiber = new Fiber(function(): void {
    $value = Fiber::suspend('first');
    echo "Got: " . $value;
});

$value = $fiber->start();
$fiber->resume('hello');
```

非同期処理・コルーチンの基礎となる機能です。

### 2-4. readonly プロパティ（PHP 8.1〜）

php

```php
class User {
    public function __construct(
        public readonly int    $id,
        public readonly string $name,
    ) {}
}

$user = new User(1, 'Alice');
// $user->id = 2; // エラー：変更不可
```

### 2-5. First-class callable syntax（PHP 8.1〜）

php

```php
$fn = strlen(...);
echo $fn('hello'); // 5
```

### 2-6. その他の重要機能

- **Named arguments**（PHP 8.0〜）：引数名を指定して渡せる
- **Match 式**（PHP 8.0〜）：switch の強化版
- **Nullsafe operator** `?->`（PHP 8.0〜）：null チェックの簡略化
- **Constructor Promotion**（PHP 8.0〜）：コンストラクタでのプロパティ定義省略

---

## ステップ 3：オブジェクト指向設計を深める

### 3-1. SOLID原則を学ぶ

|原則|概要|
|---|---|
|**S** ingle Responsibility|クラスは1つの責任だけ持つ|
|**O** pen/Closed|拡張に開き、修正に閉じる|
|**L** iskov Substitution|サブクラスは親クラスと置換可能にする|
|**I** nterface Segregation|インターフェースは小さく分割する|
|**D** ependency Inversion|具体クラスでなく抽象に依存する|

### 3-2. デザインパターンの実践

まずは以下3つを習得するのが効果的です。

1. **Repository パターン**：データアクセスロジックを分離する
2. **Service クラス**：ビジネスロジックをコントローラから切り出す
3. **DTO（Data Transfer Object）**：レイヤー間のデータ受け渡し用クラス

php

```php
// Repository パターン例
interface UserRepositoryInterface {
    public function findById(int $id): ?User;
    public function save(User $user): void;
}

class EloquentUserRepository implements UserRepositoryInterface {
    public function findById(int $id): ?User {
        return User::find($id);
    }
    // ...
}
```

---

## ステップ 4：フレームワークを習得する

### Laravel（最優先で学ぶべき）

PHPフレームワークの中で最も人気があり、学習リソースも豊富です。

**学ぶべき機能（優先順）**

1. ルーティング・コントローラ
2. Eloquent ORM（データベース操作）
3. Blade テンプレートエンジン
4. マイグレーション・シーディング
5. バリデーション
6. 認証（Breeze / Jetstream）
7. キュー・ジョブ
8. イベント・リスナ

**公式リソース**

- 公式ドキュメント: [https://laravel.com/docs](https://laravel.com/docs)
- Laracasts（動画チュートリアル）: [https://laracasts.com](https://laracasts.com)

### Symfony（深く学ぶ場合）

LaravelはSymfonyのコンポーネントを多数利用しています。大規模・エンタープライズ案件ではSymfony自体を使うことも多いです。

---

## ステップ 5：テストを書く習慣をつける

### PHPUnit

php

```php
use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase {
    public function test_add_two_numbers(): void {
        $calc = new Calculator();
        $this->assertSame(5, $calc->add(2, 3));
    }
}
```

### Pest（モダンな選択肢）

PestはPHPUnitをベースにした、よりシンプルな記法のテストフレームワークです。

php

```php
it('adds two numbers', function () {
    $calc = new Calculator();
    expect($calc->add(2, 3))->toBe(5);
});
```

**テストの種類**

|種類|概要|
|---|---|
|Unit Test|1つのクラス・メソッド単体のテスト|
|Feature Test|複数クラスを組み合わせた機能テスト|
|Integration Test|データベースなど外部システムを含むテスト|

---

## ステップ 6：コード品質ツールを使う

### 静的解析

bash

```bash
# PHPStan（最も普及）
composer require --dev phpstan/phpstan
vendor/bin/phpstan analyse src --level=8

# Psalm
composer require --dev vimeo/psalm
vendor/bin/psalm
```

**レベルを上げるほど厳密な型チェック**が行われます。新プロジェクトはレベル8以上を目標にしましょう。

### コードスタイル

bash

```bash
# PHP CS Fixer（PSR-12準拠のフォーマッタ）
composer require --dev friendsofphp/php-cs-fixer
vendor/bin/php-cs-fixer fix src
```

---

## ステップ 7：Composerとパッケージ管理

bash

```bash
# パッケージの追加
composer require guzzlehttp/guzzle

# 開発用パッケージの追加
composer require --dev phpunit/phpunit

# オートローダの更新
composer dump-autoload
```

**押さえておくべき主要パッケージ**

|パッケージ|用途|
|---|---|
|`guzzlehttp/guzzle`|HTTP クライアント|
|`vlucas/phpdotenv`|.env ファイルの読み込み|
|`symfony/var-dumper`|デバッグ用ダンプ|
|`ramsey/uuid`|UUID生成|
|`nesbot/carbon`|日付・時間操作|

---

## ステップ 8：非同期・高パフォーマンスPHP

### Swoole / OpenSwoole

PHPをイベントループで動かし、Node.jsに近いパフォーマンスを実現します。

### ReactPHP

非同期・イベント駆動のPHPライブラリ。

### RoadRunner

Go製のアプリケーションサーバーでPHPを動かし、高スループットを実現します。LaravelをRoadRunnerで動かすことも可能です。

---

## 学習ロードマップ（目安スケジュール）

```
Month 1：環境構築 + PHP 8.x の新機能 + OOP基礎
Month 2：Laravel 基礎（ルーティング・DB・認証）
Month 3：テスト（PHPUnit/Pest）+ 静的解析（PHPStan）
Month 4：設計パターン（Repository/Service/DTO）
Month 5：パフォーマンス最適化・キャッシュ・キュー
Month 6：実プロジェクト構築・OSS貢献
```

---

## 推奨学習リソース

### 書籍

- **「Modern PHP」** – Josh Lockhart 著（英語）
- **「PHPの教科書」** – 日本語入門書として定評あり

### オンライン

- [Laracasts](https://laracasts.com) – Laravel/PHP の動画学習サイト（英語）
- [PHP The Right Way](https://phptherightway.com) – PHPベストプラクティス集（英語・日本語訳あり）
- [Zenn.dev](https://zenn.dev) – 日本語のPHP技術記事が豊富

### 公式ドキュメント

- [PHP マニュアル（日本語版）](https://www.php.net/manual/ja/)
- [Laravel ドキュメント](https://laravel.com/docs)

---

## まとめ：モダンPHP開発者になるための3原則

1. **型を常に書く** — `declare(strict_types=1)` を全ファイルに記述し、型宣言を徹底する
2. **テストを書く** — コードを書いたら必ずテストを書く習慣をつける
3. **静的解析を通す** — PHPStan をCI/CDに組み込み、型エラーを機械的に検出する

この3つを守るだけで、コードの品質は劇的に向上します。

---

_Happy Coding! 🐘_