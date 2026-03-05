# PHPの関数
- https://www.php.net/manual/ja/function.pg-escape-string.php
- 正規表現
```php
// 形式: NNNN_人物名_YYYYMM分
	// 先頭4桁がナンバリング、末尾6桁+分が使用月
	if (!preg_match('/^(\d{4})_.+_(\d{6})分(?:_(未納分|滞納分))?$/u', $name_without_ext, $matches)) {
		return false;
	}
```

# AWS S3へのファイルアクセス
- 以下のメソッドはどのライブラリを読み込めば、使用できるようになる?
- ファイル取得
```php
$res = $s3Client->getObject([
    'Bucket' => $バケット,
    'Key'    => $S3キー,
    'SaveAs' => $path
]);
```

- ファイル保存時
```php
$s3Client->putObject([
    'Bucket' => 'my-system-bucket',
    'Key' => 'upload/sample.csv',
    'Body' => fopen($file, 'r')
]);
```
