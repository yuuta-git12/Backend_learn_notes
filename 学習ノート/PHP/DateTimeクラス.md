# PHP DateTime クラス

- PHPの標準クラスライブラリ
- 日付・時刻を扱うオブジェクト指向インターフェース
- 引数有無でデータの持ち方が違う（日付/時刻文字列）

## コンストラクタ

```php
new DateTime(string $datetime = 'now', ?DateTimeZone $timezone = null)
```

| 引数        | 内容                                               |
|-------------|----------------------------------------------------|
| `$datetime` | 日付文字列。省略 or `'now'` で現在日時            |
| `$timezone` | タイムゾーン。省略すると `php.ini` の設定値を使用 |

```php
// 現在日時
$now = new DateTime();

// 文字列で指定
$dt = new DateTime('2026-03-01 12:00:00');

// 相対指定も可能
$tomorrow = new DateTime('+1 day');
$lastMonth = new DateTime('first day of last month');
```

---

## DateTime vs DateTimeImmutable

| クラス                | 特徴                                                         |
|-----------------------|--------------------------------------------------------------|
| `DateTime`            | **ミュータブル**。メソッド呼び出しでオブジェクト自体が変わる |
| `DateTimeImmutable`   | **イミュータブル**。メソッドは変更済みの**新しいオブジェクト**を返す |

```php
// DateTime（元のオブジェクトが変わる）
$dt = new DateTime('2026-03-01');
$dt->modify('+1 day');
echo $dt->format('Y-m-d'); // 2026-03-02（元が変わった）

// DateTimeImmutable（元のオブジェクトは変わらない）
$dt = new DateTimeImmutable('2026-03-01');
$next = $dt->modify('+1 day');
echo $dt->format('Y-m-d');   // 2026-03-01（変わらない）
echo $next->format('Y-m-d'); // 2026-03-02（新しいオブジェクト）
```

> **推奨**: 予期しない副作用を防ぐため、`DateTimeImmutable` を使う方がよい。

---

## メソッド

### `format(string $format): string`
- 日付・時刻を指定フォーマットの文字列に変換する

```php
$dt = new DateTime('2026-03-01 09:30:00');

echo $dt->format('Y-m-d');           // 2026-03-01
echo $dt->format('Y年m月d日');        // 2026年03月01日
echo $dt->format('H:i:s');           // 09:30:00
echo $dt->format('D, d M Y');        // Sun, 01 Mar 2026
echo $dt->format('U');               // UNIXタイムスタンプ
```

**主要フォーマット文字**

| 文字 | 意味              | 例         |
|------|-------------------|------------|
| `Y`  | 4桁の年           | 2026       |
| `m`  | 2桁の月（0埋め）   | 03         |
| `d`  | 2桁の日（0埋め）   | 01         |
| `H`  | 24時間制の時       | 09         |
| `i`  | 分（0埋め）        | 30         |
| `s`  | 秒（0埋め）        | 00         |
| `N`  | 曜日（1=月〜7=日） | 7          |
| `U`  | UNIXタイムスタンプ | 1740823800 |

---

### `modify(string $modifier): static`
- 相対指定で日時を変更する（`DateTime` は自身を変更、`DateTimeImmutable` は新オブジェクトを返す）

```php
$dt = new DateTimeImmutable('2026-03-01');

echo $dt->modify('+1 day')->format('Y-m-d');    // 2026-03-02
echo $dt->modify('-2 months')->format('Y-m-d'); // 2026-01-01
echo $dt->modify('next monday')->format('Y-m-d'); // 次の月曜日
echo $dt->modify('last day of this month')->format('Y-m-d'); // 月末
```

---

### `setDate(int $year, int $month, int $day): static`
- 年・月・日を個別にセットする

```php
$dt = new DateTime('2026-01-01');
$dt->setDate(2026, 3, 15);
echo $dt->format('Y-m-d'); // 2026-03-15
```

---

### `setTime(int $hour, int $minute, int $second = 0, int $microsecond = 0): static`
- 時・分・秒をセットする

```php
$dt = new DateTime('2026-03-01');
$dt->setTime(10, 30, 0);
echo $dt->format('H:i:s'); // 10:30:00
```

---

### `setTimestamp(int $timestamp): static`
- UNIXタイムスタンプ（1970-01-01 00:00:00 UTC からの秒数）でセットする

```php
$dt = new DateTime();
$dt->setTimestamp(0);
echo $dt->format('Y-m-d H:i:s'); // 1970-01-01 09:00:00（JST の場合）
```

---

### `getTimestamp(): int`
- UNIXタイムスタンプを取得する

```php
$dt = new DateTime('2026-03-01 00:00:00');
echo $dt->getTimestamp(); // 例: 1740751200
```

---

### `createFromFormat(string $format, string $datetime, ?DateTimeZone $tz = null): DateTime|false`
- フォーマットを指定して文字列からDateTimeを生成する（静的メソッド）
- `new DateTime()` では解釈できない形式の文字列に対して使う

```php
// スラッシュ区切りや独自フォーマットのパース
$dt = DateTime::createFromFormat('d/m/Y', '01/03/2026');
echo $dt->format('Y-m-d'); // 2026-03-01

// 和暦風フォーマット
$dt = DateTime::createFromFormat('Y年m月d日', '2026年03月01日');
echo $dt->format('Y-m-d'); // 2026-03-01

// 失敗時は false を返すのでチェックが必要
$dt = DateTime::createFromFormat('Y-m-d', '不正な日付');
if ($dt === false) {
    echo 'パース失敗';
}
```

---

## タイムゾーンの指定

### `DateTimeZone` オブジェクトを使用する

```php
$tz = new DateTimeZone('Asia/Tokyo');
$dt = new DateTime('now', $tz);
echo $dt->format('Y-m-d H:i:s T'); // 2026-03-01 09:00:00 JST
```

- タイムゾーン文字列は IANA タイムゾーンデータベースの識別子を使う
- 例: `'Asia/Tokyo'`、`'UTC'`、`'America/New_York'`

### `setTimezone()` で後から変更

```php
$dt = new DateTime('now', new DateTimeZone('UTC'));
echo $dt->format('H:i T'); // UTC の時刻

$dt->setTimezone(new DateTimeZone('Asia/Tokyo'));
echo $dt->format('H:i T'); // JST の時刻（同じ瞬間を別のタイムゾーンで表示）
```

### `php.ini` との関係
- `date.timezone` に設定された値が `DateTime` のデフォルトタイムゾーンになる
- コンストラクタや `setTimezone()` で明示的に指定するとその値が優先される（`php.ini` の設定より強い）
- `date_default_timezone_set()` 関数でスクリプト内のデフォルトタイムゾーンを変更することもできる

```php
// php.ini の date.timezone を上書き（スクリプト全体に影響）
date_default_timezone_set('Asia/Tokyo');

// インスタンス単位で上書き（推奨）
$dt = new DateTime('now', new DateTimeZone('Asia/Tokyo'));
```

> **ベストプラクティス**: `php.ini` に依存せず、コンストラクタや `setTimezone()` で明示的に指定するのが安全。

---

## 日付の差分 — DateInterval

- `diff()` メソッドで2つの日時の差を `DateInterval` オブジェクトとして取得できる

```php
$start = new DateTime('2026-01-01');
$end   = new DateTime('2026-03-01');

$diff = $start->diff($end);
echo $diff->days;  // 59（総日数）
echo $diff->m;     // 2（月）
echo $diff->d;     // 0（残りの日）

// フォーマットも可能
echo $diff->format('%y年 %m月 %d日 %H時間'); // 0年 2月 0日 00時間
```

### DateInterval の手動作成

```php
// 'P' から始まる ISO 8601 期間文字列
// P1Y2M3DT4H5M6S → 1年2ヶ月3日4時間5分6秒
$interval = new DateInterval('P1Y2M3D');

$dt = new DateTimeImmutable('2026-01-01');
echo $dt->add($interval)->format('Y-m-d'); // 2027-03-04
echo $dt->sub($interval)->format('Y-m-d'); // 2024-10-29
```

---

## 期間の反復 — DatePeriod

- 開始日から終了日まで一定間隔で繰り返すことができる

```php
$start    = new DateTimeImmutable('2026-03-01');
$interval = new DateInterval('P1W'); // 1週間ごと
$end      = new DateTimeImmutable('2026-04-01');

$period = new DatePeriod($start, $interval, $end);

foreach ($period as $date) {
    echo $date->format('Y-m-d') . PHP_EOL;
}
// 2026-03-01
// 2026-03-08
// 2026-03-15
// 2026-03-22
// 2026-03-29
```

---

## よく使うパターン集

```php
// 現在日時（JST）
$now = new DateTimeImmutable('now', new DateTimeZone('Asia/Tokyo'));

// 月の初日・末日
$firstDay = new DateTimeImmutable('first day of this month');
$lastDay  = new DateTimeImmutable('last day of this month');

// 日付の比較
$dt1 = new DateTime('2026-03-01');
$dt2 = new DateTime('2026-06-01');
var_dump($dt1 < $dt2);  // bool(true)
var_dump($dt1 == $dt2); // bool(false)

// 文字列 → DateTime → 別フォーマットで出力
$input = '01-03-2026';
$dt = DateTime::createFromFormat('d-m-Y', $input);
echo $dt->format('Y/m/d'); // 2026/03/01
```
