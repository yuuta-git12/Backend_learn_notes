#PHP
# PHP ZIPArchive の使い方

- PHP 標準の ZIP ファイル操作クラス（SPL）
- ZIP ファイルの**作成・展開・追加・削除・読み込み**ができる
- 外部ライブラリ不要（`zip` 拡張が有効であれば使用可能）

---

## 基本的な使い方の流れ

```php
$zip = new ZipArchive();

// 1. ZIPファイルを開く or 作成する
$zip->open('archive.zip', ZipArchive::CREATE);

// 2. ファイルを追加・操作する
$zip->addFile('/path/to/file.txt', 'file.txt');

// 3. 閉じる（ここで実際にファイルが書き込まれる）
$zip->close();
```

---

## open() のフラグ

```php
$zip->open(string $filename, int $flags = 0): bool|int
```

| フラグ定数              | 意味                                         |
|-------------------------|----------------------------------------------|
| `ZipArchive::CREATE`    | ファイルが存在しなければ新規作成する         |
| `ZipArchive::OVERWRITE` | 既存ファイルを上書きする                     |
| `ZipArchive::EXCL`      | ファイルが既に存在する場合エラーにする       |
| `ZipArchive::RDONLY`    | 読み取り専用で開く                           |

```php
// 新規作成（存在すれば上書き）
$zip->open('archive.zip', ZipArchive::CREATE | ZipArchive::OVERWRITE);

// 既存のZIPを開く（読み書き）
$zip->open('archive.zip');
```

---

## メソッド

### `addFile()` — ファイルを追加する

```php
$zip->addFile(string $filepath, string $entryname = ''): bool
```

- `$filepath`：追加するローカルファイルのパス
- `$entryname`：ZIP 内でのファイル名（省略すると `$filepath` がそのまま使われる）

```php
$zip = new ZipArchive();
$zip->open('archive.zip', ZipArchive::CREATE);

// ZIP内では 'data/sample.csv' として格納される
$zip->addFile('/var/www/html/files/sample.csv', 'data/sample.csv');

$zip->close();
```

---

### `addFromString()` — 文字列からファイルを追加する

```php
$zip->addFromString(string $name, string $content): bool
```

- ファイルを作成せずに文字列を直接 ZIP 内のファイルとして追加できる

```php
$zip = new ZipArchive();
$zip->open('archive.zip', ZipArchive::CREATE);

$csvContent = "id,name\n1,田中\n2,佐藤";
$zip->addFromString('export/users.csv', $csvContent);

$zip->close();
```

---

### `addEmptyDir()` — 空のディレクトリを追加する

```php
$zip->addEmptyDir(string $dirname): bool
```

```php
$zip->addEmptyDir('logs/');
```

---

### `extractTo()` — ZIPを展開する

```php
$zip->extractTo(string $pathto, array|string|null $files = null): bool
```

- `$pathto`：展開先のディレクトリ
- `$files`：展開するファイル名を指定（省略すると全て展開）

```php
$zip = new ZipArchive();
$zip->open('archive.zip');

// 全て展開
$zip->extractTo('/path/to/destination/');

// 特定のファイルのみ展開
$zip->extractTo('/path/to/destination/', ['data/sample.csv']);

$zip->close();
```

---

### `getFromName()` — ファイルの中身を文字列で取得する

```php
$zip->getFromName(string $name): string|false
```

```php
$zip = new ZipArchive();
$zip->open('archive.zip');

$content = $zip->getFromName('data/sample.csv');
// → CSVの中身が文字列で取得できる

$zip->close();
```

---

### `deleteName()` — ZIP内のファイルを削除する

```php
$zip->deleteName(string $name): bool
```

```php
$zip = new ZipArchive();
$zip->open('archive.zip');

$zip->deleteName('old/unnecessary.txt');

$zip->close();
```

---

### `count()` — ZIP内のファイル数を取得する

```php
$zip = new ZipArchive();
$zip->open('archive.zip');

echo $zip->count(); // ZIP内のファイル数

$zip->close();
```

---

### `getNameIndex()` — インデックスでファイル名を取得する

```php
$zip = new ZipArchive();
$zip->open('archive.zip');

for ($i = 0; $i < $zip->count(); $i++) {
    echo $zip->getNameIndex($i) . PHP_EOL;
}

$zip->close();
```

---

## よく使うパターン

### 複数ファイルを一括でZIPに追加する

```php
$zip = new ZipArchive();
$zip->open('archive.zip', ZipArchive::CREATE | ZipArchive::OVERWRITE);

$files = glob('/path/to/csv/*.csv');

foreach ($files as $file) {
    $zip->addFile($file, basename($file)); // ZIPの中ではファイル名のみ
}

$zip->close();
```

---

### ディレクトリ全体をZIPに圧縮する（RecursiveDirectoryIterator と組み合わせ）

```php
function zipDirectory(string $sourceDir, string $outZipPath): void
{
    $zip = new ZipArchive();
    $zip->open($outZipPath, ZipArchive::CREATE | ZipArchive::OVERWRITE);

    $iterator = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($sourceDir, FilesystemIterator::SKIP_DOTS),
        RecursiveIteratorIterator::LEAVES_ONLY
    );

    foreach ($iterator as $file) {
        if (!$file->isFile()) continue;

        // ZIP内の相対パスを計算
        $relativePath = substr($file->getPathname(), strlen($sourceDir) + 1);
        $zip->addFile($file->getPathname(), $relativePath);
    }

    $zip->close();
}

zipDirectory('/var/www/html/exports', '/tmp/exports.zip');
```

---

### ZIPを展開してファイルを処理する（解凍 → 処理 → 削除）

```php
$zipPath     = '/tmp/uploaded.zip';
$extractPath = '/tmp/extracted/';

$zip = new ZipArchive();

if ($zip->open($zipPath) !== true) {
    throw new RuntimeException('ZIPファイルを開けません');
}

$zip->extractTo($extractPath);
$zip->close();

// 展開されたファイルを処理
foreach (glob($extractPath . '*.csv') as $csv) {
    // CSVの処理...
    echo basename($csv) . " を処理しました\n";
}

// 処理後に展開先ディレクトリを削除（解凍後の後片付け）
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator($extractPath, FilesystemIterator::SKIP_DOTS),
    RecursiveIteratorIterator::CHILD_FIRST
);
foreach ($iterator as $entry) {
    $entry->isDir() ? rmdir($entry->getPathname()) : unlink($entry->getPathname());
}
rmdir($extractPath);
unlink($zipPath); // ZIPファイル自体も削除
```

---

### ブラウザに直接ZIPをダウンロードさせる

```php
$zip = new ZipArchive();
$tmpPath = tempnam(sys_get_temp_dir(), 'zip');

$zip->open($tmpPath, ZipArchive::CREATE);
$zip->addFromString('report.csv', "id,name\n1,田中\n2,佐藤");
$zip->close();

// ダウンロードレスポンスを返す
header('Content-Type: application/zip');
header('Content-Disposition: attachment; filename="download.zip"');
header('Content-Length: ' . filesize($tmpPath));

readfile($tmpPath);
unlink($tmpPath); // 一時ファイルを削除
exit;
```

---

## エラーハンドリング

`open()` は成功時に `true`、失敗時にエラーコード（int）を返す。

| エラーコード定数           | 意味                           |
|----------------------------|--------------------------------|
| `ZipArchive::ER_OK`        | エラーなし（成功）             |
| `ZipArchive::ER_NOZIP`     | ZIPファイルではない            |
| `ZipArchive::ER_INCONS`    | ZIPファイルが壊れている        |
| `ZipArchive::ER_NOENT`     | ファイルが存在しない           |
| `ZipArchive::ER_EXISTS`    | ファイルが既に存在する         |

```php
$zip = new ZipArchive();
$result = $zip->open('archive.zip');

if ($result !== true) {
    match ($result) {
        ZipArchive::ER_NOZIP  => throw new RuntimeException('ZIPファイルではありません'),
        ZipArchive::ER_INCONS => throw new RuntimeException('ZIPファイルが破損しています'),
        ZipArchive::ER_NOENT  => throw new RuntimeException('ファイルが見つかりません'),
        default               => throw new RuntimeException("ZIPエラー: {$result}"),
    };
}
```
