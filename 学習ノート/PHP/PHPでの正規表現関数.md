#PHP
# PHP の関数

---

## 正規表現

PHP では `preg_*` 系関数を使って正規表現を扱う。
パターンは `/パターン/フラグ` の形式で記述する（PCRE 形式）。

### `preg_match()`

```php
preg_match(string $pattern, string $subject, array &$matches = []): int|false
```

- パターンに一致すれば `1`、しなければ `0`、エラーの場合 `false` を返す
- `$matches` にキャプチャグループの結果が格納される（`$matches[0]` は一致全体、`$matches[1]` 以降がグループ）

```php
// 形式: NNNN_人物名_YYYYMM分
// 先頭4桁がナンバリング、末尾6桁+分が使用月
if (!preg_match('/^(\d{4})_.+_(\d{6})分(?:_(未納分|滞納分))?$/u', $name_without_ext, $matches)) {
    return false;
}
// $matches[1] → 先頭4桁のナンバリング
// $matches[2] → 6桁の使用月（YYYYMM）
// $matches[3] → "未納分" または "滞納分"（存在する場合）
```

**パターンの解説**

| 部分                          | 意味                                         |
|-------------------------------|----------------------------------------------|
| `^`                           | 文字列の先頭                                 |
| `(\d{4})`                     | 数字4桁をキャプチャ（ナンバリング）          |
| `_`                           | アンダースコアのリテラル                     |
| `.+`                          | 任意の1文字以上（人物名）                    |
| `(\d{6})分`                   | 数字6桁＋「分」をキャプチャ（使用月）        |
| `(?:_(未納分\|滞納分))?`      | `(?:...)` は非キャプチャグループ、`?` で省略可 |
| `$`                           | 文字列の末尾                                 |
| `u` フラグ                    | UTF-8 マルチバイト対応                       |

### よく使う `preg_*` 関数

| 関数              | 説明                                           |
|-------------------|------------------------------------------------|
| `preg_match()`    | 最初の1件だけマッチを確認する                  |
| `preg_match_all()`| すべての一致を取得する                          |
| `preg_replace()`  | 一致した部分を置換する                          |
| `preg_split()`    | パターンで文字列を分割する                      |
| `preg_grep()`     | 配列の中からパターンに一致する要素を取得する    |

```php
// preg_match_all：すべての一致を取得
preg_match_all('/\d+/', '商品A: 100円、商品B: 200円', $matches);
// $matches[0] → ['100', '200']

// preg_replace：一致した部分を置換
$result = preg_replace('/\s+/', ' ', '余分な　スペース   を削除');
// → '余分な スペース を削除'

// preg_split：パターンで分割
$parts = preg_split('/[,、]/', '山田,田中、佐藤');
// → ['山田', '田中', '佐藤']
```

### よく使う正規表現パターン

| パターン                  | 意味                               |
|---------------------------|------------------------------------|
| `\d`                      | 数字 [0-9]                         |
| `\w`                      | 英数字 + アンダースコア            |
| `\s`                      | 空白文字（スペース・タブ・改行）   |
| `.`                       | 任意の1文字（改行を除く）          |
| `*`                       | 0回以上の繰り返し                  |
| `+`                       | 1回以上の繰り返し                  |
| `?`                       | 0回または1回                       |
| `{n}`                     | ちょうど n 回                      |
| `{n,m}`                   | n 回以上 m 回以下                  |
| `^` / `$`                 | 文字列の先頭 / 末尾                |
| `[abc]`                   | a か b か c のいずれか             |
| `[^abc]`                  | a・b・c 以外                       |
| `(abc)`                   | グループ化（キャプチャ）           |
| `(?:abc)`                 | 非キャプチャグループ               |
| `a\|b`                    | a または b                         |

---

## pg_escape_string()

> 参考: https://www.php.net/manual/ja/function.pg-escape-string.php

```php
pg_escape_string(?PgSql\Connection $connection = null, string $data): string
```

- PostgreSQL へのクエリに含める文字列を**エスケープ**する関数
- SQLインジェクション対策として使用する
- `'`（シングルクォート）などの特殊文字を安全な形式に変換する

```php
$conn = pg_connect("dbname=test");

$input = "O'Reilly"; // シングルクォートを含む文字列

// エスケープしてからクエリに組み込む
$escaped = pg_escape_string($conn, $input);
$query = "SELECT * FROM users WHERE name = '{$escaped}'";
// → SELECT * FROM users WHERE name = 'O''Reilly'

pg_query($conn, $query);
```

> **注意**: 現在は `pg_escape_string()` よりも**プリペアドステートメント**（`pg_prepare()` / `pg_execute()`）を使う方がより安全で推奨される。

```php
// プリペアドステートメントを使う場合（推奨）
pg_prepare($conn, "get_user", "SELECT * FROM users WHERE name = $1");
pg_execute($conn, "get_user", [$input]);
```
