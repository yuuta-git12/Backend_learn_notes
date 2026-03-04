#PHP 

- PHP の SPL（Standard PHP Library）に含まれるクラスで、ディレクトリを**再帰的に（サブフォルダ含めて）走査**するためのイテレータ
- `DirectoryIterator` は指定ディレクトリの直下のみを対象とするが、`RecursiveDirectoryIterator` はネストされたサブディレクトリも辿れる
- 単体では再帰は行われない。`RecursiveIteratorIterator` と組み合わせることで実際の再帰走査が機能する

## クラス継承関係

```
Iterator
  └── FilesystemIterator
        └── RecursiveDirectoryIterator  ← このクラス
```

- `FilesystemIterator` → `DirectoryIterator` → `SplFileInfo` のメソッドも継承している
- `RecursiveIterator` インターフェースを実装しており、`RecursiveIteratorIterator` から呼び出せる

---

## 基本的な使い方

`RecursiveDirectoryIterator` だけでは**再帰走査は行われない**。
`RecursiveIteratorIterator` でラップすることで、サブディレクトリを含めた全エントリを順番に処理できる。

```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator('/path/to/dir', FilesystemIterator::SKIP_DOTS)
);

foreach ($iterator as $file) {
    echo $file->getPathname() . PHP_EOL;
}
```

---

## フラグ（定数）

コンストラクタの第2引数にフラグを指定することで動作を変更できる。

```php
new RecursiveDirectoryIterator($path, $flags);
```

| フラグ                                    | 説明                                             |
|-------------------------------------------|--------------------------------------------------|
| `FilesystemIterator::SKIP_DOTS`           | `.` と `..` をスキップする（ほぼ必須）           |
| `FilesystemIterator::CURRENT_AS_FILEINFO` | 現在のエントリを `SplFileInfo` として返す（デフォルト） |
| `FilesystemIterator::CURRENT_AS_PATHNAME` | 現在のエントリをパス文字列として返す             |
| `FilesystemIterator::KEY_AS_FILENAME`     | キーをファイル名にする                           |
| `FilesystemIterator::KEY_AS_PATHNAME`     | キーをフルパスにする（デフォルト）               |
| `FilesystemIterator::FOLLOW_SYMLINKS`     | シンボリックリンクを辿って再帰する               |

```php
// 複数フラグは | で組み合わせる
$flags = FilesystemIterator::SKIP_DOTS | FilesystemIterator::FOLLOW_SYMLINKS;
$dir = new RecursiveDirectoryIterator('/path/to/dir', $flags);
```

---

## RecursiveIteratorIterator のモード

`RecursiveIteratorIterator` の第2引数でサブディレクトリの処理順序を指定できる。

| モード定数                                         | 説明                                               |
|----------------------------------------------------|----------------------------------------------------|
| `RecursiveIteratorIterator::LEAVES_ONLY`           | ファイルのみ（ディレクトリは含まない）。**デフォルト** |
| `RecursiveIteratorIterator::SELF_FIRST`            | 親ディレクトリ → 子を順番に処理                    |
| `RecursiveIteratorIterator::CHILD_FIRST`           | 子 → 親ディレクトリの順番に処理（削除処理に向く）  |

```php
// ファイルのみ走査（デフォルト）
$it = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator('/path', FilesystemIterator::SKIP_DOTS),
    RecursiveIteratorIterator::LEAVES_ONLY
);

// 親ディレクトリ → 子の順
$it = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator('/path', FilesystemIterator::SKIP_DOTS),
    RecursiveIteratorIterator::SELF_FIRST
);

// 子 → 親ディレクトリの順（ディレクトリ削除時に使う）
$it = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator('/path', FilesystemIterator::SKIP_DOTS),
    RecursiveIteratorIterator::CHILD_FIRST
);
```

---

## メソッド

`RecursiveDirectoryIterator` 自身が持つメソッドに加え、`SplFileInfo` のメソッドもすべて使える。

### `hasChildren(): bool`
- 現在のエントリがサブディレクトリ（子要素を持つ）かどうかを返す

```php
$dir = new RecursiveDirectoryIterator('/path/to/dir', FilesystemIterator::SKIP_DOTS);

foreach ($dir as $entry) {
    if ($dir->hasChildren()) {
        echo $entry->getFilename() . " はディレクトリです\n";
    }
}
```

---

### `getChildren(): RecursiveDirectoryIterator`
- 現在のサブディレクトリのイテレータを返す
- `RecursiveIteratorIterator` が内部的に使用するメソッド

---

### `getSubPath(): string`
- 走査のルートからの**相対ディレクトリパス**を返す（ファイル名は含まない）

```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator('/var/www', FilesystemIterator::SKIP_DOTS)
);

foreach ($iterator as $file) {
    echo $iterator->getSubPath() . PHP_EOL;
    // 例: html/assets/css
}
```

---

### `getSubPathname(): string`
- 走査のルートからの**相対パス（ファイル名含む）**を返す

```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator('/var/www', FilesystemIterator::SKIP_DOTS)
);

foreach ($iterator as $file) {
    echo $iterator->getSubPathname() . PHP_EOL;
    // 例: html/assets/css/style.css
}
```

> `getSubPath()` vs `getSubPathname()` vs `getPathname()`
>
> | メソッド              | 返り値の例                        |
> |-----------------------|-----------------------------------|
> | `getSubPath()`        | `html/assets/css`                 |
> | `getSubPathname()`    | `html/assets/css/style.css`       |
> | `getPathname()`       | `/var/www/html/assets/css/style.css` |

---

### `SplFileInfo` から継承している主なメソッド

| メソッド            | 説明                          |
|---------------------|-------------------------------|
| `getFilename()`     | ファイル名のみ                |
| `getExtension()`    | 拡張子                        |
| `getPathname()`     | フルパス                      |
| `isFile()`          | 通常ファイルか               |
| `isDir()`           | ディレクトリか               |
| `getSize()`         | ファイルサイズ（バイト）      |
| `getMTime()`        | 最終更新日時（タイムスタンプ）|

---

## 用途と実装例

### ディレクトリ内ファイル一括処理（拡張子フィルタ）

```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator('/path/to/dir', FilesystemIterator::SKIP_DOTS)
);

foreach ($iterator as $file) {
    if ($file->isFile() && $file->getExtension() === 'php') {
        echo $file->getPathname() . PHP_EOL;
    }
}
```

---

### 解凍されたファイルの削除

子から先に処理する `CHILD_FIRST` モードにより、ファイル → サブディレクトリ → 親ディレクトリの順に削除できる。

```php
function deleteDirectory(string $path): void
{
    $iterator = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($path, FilesystemIterator::SKIP_DOTS),
        RecursiveIteratorIterator::CHILD_FIRST  // 子から先に処理
    );

    foreach ($iterator as $entry) {
        if ($entry->isDir()) {
            rmdir($entry->getPathname());
        } else {
            unlink($entry->getPathname());
        }
    }

    rmdir($path); // 最後にルートディレクトリ自体を削除
}

deleteDirectory('/path/to/extracted');
```

---

### CSV 一括処理

サブディレクトリを含む複数の CSV ファイルをまとめて読み込む。

```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator('/path/to/csvdir', FilesystemIterator::SKIP_DOTS)
);

foreach ($iterator as $file) {
    if ($file->getExtension() !== 'csv') continue;

    $handle = fopen($file->getPathname(), 'r');
    while (($row = fgetcsv($handle)) !== false) {
        // $row を処理
        var_dump($row);
    }
    fclose($handle);
}
```

---

### ログ解析

特定期間のログファイルを再帰的に集めて解析する。

```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator('/var/log/app', FilesystemIterator::SKIP_DOTS)
);

$threshold = strtotime('-7 days'); // 7日前のタイムスタンプ

foreach ($iterator as $file) {
    if (!$file->isFile() || $file->getExtension() !== 'log') continue;
    if ($file->getMTime() < $threshold) continue; // 7日以内のみ対象

    $lines = file($file->getPathname(), FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    foreach ($lines as $line) {
        if (str_contains($line, 'ERROR')) {
            echo '[' . $file->getSubPathname() . '] ' . $line . PHP_EOL;
        }
    }
}
```

---

### S3 バッチ処理（ローカルファイルを S3 にアップロード）

```php
// AWS SDK for PHP が Composer でインストール済みの前提
$s3 = new Aws\S3\S3Client([
    'region'  => 'ap-northeast-1',
    'version' => 'latest',
]);

$localDir = '/path/to/files';
$bucket   = 'my-bucket';

$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator($localDir, FilesystemIterator::SKIP_DOTS)
);

foreach ($iterator as $file) {
    if (!$file->isFile()) continue;

    $key = $iterator->getSubPathname(); // ローカルの相対パスをS3キーに使う

    $s3->putObject([
        'Bucket'     => $bucket,
        'Key'        => $key,
        'SourceFile' => $file->getPathname(),
    ]);

    echo "アップロード完了: {$key}\n";
}
```

---

## DirectoryIterator との比較

| 比較項目             | `DirectoryIterator`        | `RecursiveDirectoryIterator`          |
|----------------------|----------------------------|---------------------------------------|
| 走査範囲             | 指定ディレクトリの直下のみ | サブディレクトリも含めて再帰的に走査 |
| 使い方               | 単体で `foreach` 可能      | `RecursiveIteratorIterator` が必要    |
| `getSubPath()`       | なし                       | あり（相対パスの取得）               |
| `getSubPathname()`   | なし                       | あり（相対パス＋ファイル名）          |
