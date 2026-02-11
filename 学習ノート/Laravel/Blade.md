#Laravel 

- Laravelで使用するテンプレートエンジン
	- HTMLを穴あきにして、穴の部分をプログラムによって動的出力する仕組み
	- テンプレートの穴は`{{　}}`で囲まれた部分
	- `{{　}}`の中には変数や式を記述することができる
	- `{{　}}`の中身が出力される際、HTMLエスケープが実行される
	- エスケープしたくない場合は`{!! !!}`で囲う
		- ＊エスケープしない場合XSS脆弱性のリスクがある
## Bladeディレクティブ
- `@`から始まるキーワードのこと
- 条件分岐・繰り返しによる複数回出力を行いたい場合に使用する
- 例
```
// 条件分岐
@if(session('message'))
<div style="color: blue">
{{ session('message') }}
</div>
@endif

// 繰り返し
@foreach($books as $book)
<tr>
<td>{{$book->category->title}}</td>
<td>{{$book->title}}</td>
<td>{{$book->price}}</td>
</tr>
@endforeach
```
- 認証済み、未認証の場合を判定するディレクティブも存在する
	- `@auth`,`@guest`
- 繰り返しの処理に関して`$loop`変数を使ってインデックスの参照、繰り返しの回数などの参照ができる
- 属性を追加するディレクティブもある
	- `@selected`：`selected`属性を追加
	- `@checked`：`checked`属性を追加
	- `@disabled`：`diabled`属性を追加
	- `@readonly`：`readonly`属性を追加
	- `@required`：`required`属性を追加