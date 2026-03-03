- CSVインジェクション対策のコードスニペット
```php
function sanitizeCsvField($value): string{
    // null対応
    if ($value === null) {
        $value = '';
    }

    // 文字列化
    $value = (string) $value;

    // タブ・改行除去（セル分割防止）
    $value = preg_replace("/[\t\r\n]/", " ", $value);

    // 数式インジェクション対策
    if (preg_match('/^[=+\-@]/', $value)) {
        $value = "'" . $value;
    }

    // ダブルクォートをエスケープ
    $value = str_replace('"', '""', $value);

    // 全体をダブルクォートで囲む
    return $value;
}
```
