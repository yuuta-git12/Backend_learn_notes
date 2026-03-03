# PHP DirectoryIterator クラス

- 指定されたディレクトリ以下のファイル情報にアクセスするためのクラス（SPL 標準ライブラリ）
- `SplFileInfo` を継承しており、`Iterator` インターフェースを実装している
- インスタンス化でフォルダを開く
  - `new DirectoryIterator($path)` パスにアクセスできない場合は例外（`UnexpectedValueException`）を投げる
- `foreach` でフォルダの内容を配列のように順番に処理できる

## 基本的な使い方

```php
$dir = new DirectoryIterator('/path/to/dir');

foreach ($dir as $entry) {
    // . と .. をスキップするのが定番
    if ($entry->isDot()) {
        continue;
    }
    echo $entry->getFilename() . PHP_EOL;
}
```

---

## メソッド

### `isDot(): bool`
- エントリが `.`（カレントディレクトリ）または `..`（親ディレクトリ）かどうかを返す
- `foreach` 内で最初にスキップするのが定番の書き方

```php
foreach (new DirectoryIterator('/path/to/dir') as $entry) {
    if ($entry->isDot()) continue; // . と .. をスキップ
    echo $entry->getFilename() . PHP_EOL;
}
```

---

### `isFile(): bool`
- エントリが通常ファイルであれば `true` を返す
- ディレクトリやシンボリックリンクは `false`

```php
foreach (new DirectoryIterator('/path/to/dir') as $entry) {
    if ($entry->isDot()) continue;
    if ($entry->isFile()) {
        echo $entry->getFilename() . ' はファイルです' . PHP_EOL;
    }
}
```

---

### `isDir(): bool`
- エントリがディレクトリであれば `true` を返す
- `.` と `..` も `true` になるため、`isDot()` と組み合わせる

```php
foreach (new DirectoryIterator('/path/to/dir') as $entry) {
    if ($entry->isDot()) continue;
    if ($entry->isDir()) {
        echo $entry->getFilename() . ' はディレクトリです' . PHP_EOL;
    }
}
```

---

### `isLink(): bool`
- エントリがシンボリックリンクであれば `true` を返す
- **シンボリックリンク**とは：別のファイルやディレクトリを指し示す「ショートカット」のようなもの

```php
foreach (new DirectoryIterator('/path/to/dir') as $entry) {
    if ($entry->isDot()) continue;
    if ($entry->isLink()) {
        echo $entry->getFilename() . ' はシンボリックリンクです' . PHP_EOL;
    }
}
```

---

### `getFilename(): string`
- エントリのファイル名（またはディレクトリ名）のみを返す
- パス情報は含まない

```php
// /var/www/html/index.php の場合
echo $entry->getFilename(); // index.php
```

---

### `getBasename(string $suffix = ''): string`
- ファイル名のベース名を返す
- `$suffix` を指定すると、末尾から一致する文字列を除いた名前を返す

> **ベース名とは？**
> パス区切り（`/` や `\`）を除いた、ファイル名部分のこと。
> `getFilename()` と同じに見えるが、`$suffix` で拡張子を取り除けるのが違い。

```php
// /var/www/html/index.php の場合
echo $entry->getBasename();        // index.php
echo $entry->getBasename('.php');  // index（拡張子 .php を除いた名前）
echo $entry->getBasename('.html'); // index.php（一致しない場合は変化なし）
```

---

### `getPath(): string`
- エントリが存在する**ディレクトリのパス**を返す（ファイル名は含まない）

```php
// /var/www/html/index.php の場合
echo $entry->getPath(); // /var/www/html
```

---

### `getPathname(): string`
- エントリの**フルパス**（ディレクトリ + ファイル名）を返す

```php
// /var/www/html/index.php の場合
echo $entry->getPathname(); // /var/www/html/index.php
```

> `getPath()` vs `getPathname()` vs `getFilename()` の比較
>
> | メソッド          | 返り値の例              |
> |-------------------|-------------------------|
> | `getPath()`       | `/var/www/html`         |
> | `getFilename()`   | `index.php`             |
> | `getPathname()`   | `/var/www/html/index.php` |

---

### `getExtension(): string`
- ファイルの拡張子を返す（ドットなし）
- ディレクトリや拡張子なしファイルは空文字を返す

```php
// index.php の場合
echo $entry->getExtension(); // php

// README（拡張子なし）の場合
echo $entry->getExtension(); // ""（空文字）
```

---

### `getSize(): int`
- ファイルのサイズをバイト単位で返す
- ディレクトリに使うと 0 または OS 依存の値になる

```php
$size = $entry->getSize(); // 例: 2048

// KB 単位に変換
echo round($size / 1024, 2) . ' KB';
```

---

### `getATime(): int`
- ファイルへの**最終アクセス日時**を UNIX タイムスタンプで返す（Access Time）

```php
$atime = $entry->getATime();
echo date('Y-m-d H:i:s', $atime); // 例: 2026-03-01 09:00:00
```

---

### `getMTime(): int`
- ファイルの**最終更新日時**を UNIX タイムスタンプで返す（Modification Time）
- ファイルの中身が変更された日時

```php
$mtime = $entry->getMTime();
echo date('Y-m-d H:i:s', $mtime); // 例: 2026-02-15 14:30:00
```

---

### `getCTime(): int`
- ファイルの**iノード変更日時**を UNIX タイムスタンプで返す（Change Time）
- 中身の変更だけでなく、パーミッションや所有者の変更も反映される
- Windows では「作成日時」として扱われることがある

```php
$ctime = $entry->getCTime();
echo date('Y-m-d H:i:s', $ctime);
```

> `getATime()` / `getMTime()` / `getCTime()` の違い
>
> | メソッド     | 意味                                     |
> |--------------|------------------------------------------|
> | `getATime()` | 最終アクセス日時（読み込みでも更新）     |
> | `getMTime()` | 最終更新日時（ファイルの中身の変更）     |
> | `getCTime()` | iノード変更日時（中身・権限・所有者など） |

---

## よく使うパターン

### 特定の拡張子のファイルだけ取得する

```php
$dir = new DirectoryIterator('/path/to/dir');

foreach ($dir as $entry) {
    if ($entry->isDot()) continue;
    if ($entry->isFile() && $entry->getExtension() === 'php') {
        echo $entry->getFilename() . PHP_EOL;
    }
}
```

---

### ファイルとディレクトリを分けてリストアップ

```php
$files = [];
$dirs  = [];

foreach (new DirectoryIterator('/path/to/dir') as $entry) {
    if ($entry->isDot()) continue;

    if ($entry->isFile()) {
        $files[] = $entry->getFilename();
    } elseif ($entry->isDir()) {
        $dirs[] = $entry->getFilename();
    }
}
```

---

### ファイルサイズと更新日時を一覧表示

```php
foreach (new DirectoryIterator('/path/to/dir') as $entry) {
    if ($entry->isDot() || !$entry->isFile()) continue;

    printf(
        "%-30s %8s KB  %s\n",
        $entry->getFilename(),
        round($entry->getSize() / 1024, 1),
        date('Y-m-d H:i', $entry->getMTime())
    );
}
```

---

## サブディレクトリも含めて再帰的に処理する場合

`DirectoryIterator` は指定ディレクトリの直下のみを対象とする。
サブディレクトリも含めて再帰的に処理したい場合は `RecursiveDirectoryIterator` と `RecursiveIteratorIterator` を組み合わせる。

```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator('/path/to/dir', FilesystemIterator::SKIP_DOTS)
);

foreach ($iterator as $entry) {
    echo $entry->getPathname() . PHP_EOL; // フルパスで出力
}
```
