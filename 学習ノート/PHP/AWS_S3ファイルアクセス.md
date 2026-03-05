#PHP 
#AWS

# AWS S3 へのファイルアクセス（PHP）

---

## 使用するライブラリ

`getObject()` / `putObject()` などのメソッドは **AWS SDK for PHP** を読み込むことで使用できる。

```bash
# Composer でインストール
composer require aws/aws-sdk-php
```

```php
// autoload を読み込む
require_once __DIR__ . '/vendor/autoload.php';

use Aws\S3\S3Client;
use Aws\Exception\AwsException;
```

---

## S3Client のインスタンス生成

```php
$s3Client = new S3Client([
    'version'     => 'latest',
    'region'      => 'ap-northeast-1', // 東京リージョン
    'credentials' => [
        'key'    => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
    ],
]);
```

> IAM ロール（EC2・ECS・Lambda 上）を使う場合は `credentials` の指定を省略できる（自動で認証情報を取得する）。

---

## ファイル取得（getObject）

```php
$res = $s3Client->getObject([
    'Bucket' => $バケット,
    'Key'    => $S3キー,
    'SaveAs' => $path      // 指定したローカルパスに保存する
]);
```

### 主なオプション

| オプション | 説明                                                       |
|------------|------------------------------------------------------------|
| `Bucket`   | S3 バケット名                                              |
| `Key`      | S3 上のオブジェクトキー（ファイルパス）                    |
| `SaveAs`   | ローカルに保存するパス（省略するとレスポンスボディで取得） |

### ファイルをローカルに保存する場合

```php
$localPath = '/tmp/downloaded.csv';

$res = $s3Client->getObject([
    'Bucket' => 'my-system-bucket',
    'Key'    => 'uploads/sample.csv',
    'SaveAs' => $localPath,
]);

echo "ダウンロード完了: " . $localPath;
```

### ファイルの中身を直接取得する場合（保存せず）

```php
$res = $s3Client->getObject([
    'Bucket' => 'my-system-bucket',
    'Key'    => 'uploads/sample.csv',
]);

$body = (string) $res['Body']; // ファイルの中身を文字列で取得
```

---

## ファイル保存（putObject）

```php
$s3Client->putObject([
    'Bucket' => 'my-system-bucket',
    'Key'    => 'upload/sample.csv',
    'Body'   => fopen($file, 'r'),  // ファイルストリームで渡す
]);
```

### 主なオプション

| オプション      | 説明                                                       |
|-----------------|------------------------------------------------------------|
| `Bucket`        | S3 バケット名                                              |
| `Key`           | アップロード先のオブジェクトキー                           |
| `Body`          | アップロードするデータ（文字列・ストリーム・リソース）     |
| `SourceFile`    | アップロードするローカルファイルのパス（`Body` の代替）    |
| `ContentType`   | MIMEタイプ（例: `text/csv`、`image/png`）                  |
| `ACL`           | アクセス制御（例: `private`、`public-read`）               |

### ローカルファイルをアップロードする場合

```php
// Body にファイルストリームを渡す
$s3Client->putObject([
    'Bucket' => 'my-system-bucket',
    'Key'    => 'upload/sample.csv',
    'Body'   => fopen('/path/to/local/sample.csv', 'r'),
]);

// SourceFile を使う場合（SDK が内部でストリームを開く）
$s3Client->putObject([
    'Bucket'     => 'my-system-bucket',
    'Key'        => 'upload/sample.csv',
    'SourceFile' => '/path/to/local/sample.csv',
    'ContentType'=> 'text/csv',
]);
```

### 文字列を直接アップロードする場合

```php
$csvContent = "id,name\n1,田中\n2,佐藤";

$s3Client->putObject([
    'Bucket'      => 'my-system-bucket',
    'Key'         => 'export/output.csv',
    'Body'        => $csvContent,
    'ContentType' => 'text/csv; charset=UTF-8',
]);
```

---

## エラーハンドリング

```php
use Aws\Exception\AwsException;

try {
    $res = $s3Client->getObject([
        'Bucket' => 'my-system-bucket',
        'Key'    => 'uploads/sample.csv',
    ]);
} catch (AwsException $e) {
    // S3 のエラーコードを取得
    $errorCode = $e->getAwsErrorCode();   // 例: NoSuchKey
    $message   = $e->getMessage();

    if ($errorCode === 'NoSuchKey') {
        echo 'ファイルが存在しません';
    } else {
        echo 'S3エラー: ' . $message;
    }
}
```

---

## その他のよく使うメソッド

### ファイルの存在確認

```php
use Aws\S3\Exception\S3Exception;

try {
    $s3Client->headObject([
        'Bucket' => 'my-system-bucket',
        'Key'    => 'uploads/sample.csv',
    ]);
    echo 'ファイルが存在します';
} catch (S3Exception $e) {
    if ($e->getStatusCode() === 404) {
        echo 'ファイルが存在しません';
    }
}
```

### ファイル削除

```php
$s3Client->deleteObject([
    'Bucket' => 'my-system-bucket',
    'Key'    => 'uploads/sample.csv',
]);
```

### ファイル一覧取得

```php
$result = $s3Client->listObjectsV2([
    'Bucket' => 'my-system-bucket',
    'Prefix' => 'uploads/', // 特定のフォルダ以下を絞り込む
]);

foreach ($result['Contents'] as $object) {
    echo $object['Key'] . PHP_EOL;
}
```

### 署名付きURL（一時的なダウンロードリンク）の生成

```php
$cmd = $s3Client->getCommand('GetObject', [
    'Bucket' => 'my-system-bucket',
    'Key'    => 'private/document.pdf',
]);

// 1時間有効な署名付きURLを生成
$request = $s3Client->createPresignedRequest($cmd, '+1 hour');
$url = (string) $request->getUri();

echo $url; // このURLを使えば認証不要でダウンロードできる
```
