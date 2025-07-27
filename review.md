# Laravel Lesson レビュー①

## Todo一覧機能

### Todoモデルのallメソッドで実行しているSQLは何か
- $todos = $todo->all();
- SELECT * FROM todos;

- todosテーブルのレコードを全件取得している


### Todoモデルのallメソッドの返り値は何か
- $todos = $todo->all();
- Illuminate\Database\Eloquent\Collectionクラスのインスタンス

- dd($todos);の結果
- Illuminate\Database\Eloquent\Collection {#261 ▼
  #items: array:2 [▼
    0 => App\Todo {#262 ▶}
    1 => App\Todo {#263 ▶}
  ]
 }

### 配列の代わりにCollectionクラスを使用するメリットは
- まず、CollectionはLaravelで用意されているクラスで配列操作に特化しているクラス

- Collectionはメソッドチェーンによって処理を連結でき、for文などを使わずにデータ操作ができるため、
  処理の流れが明確になり、コードの可読性が向上する。
  また、多くのメソッドは元のデータを変更せずに新しいインスタンスを返すため、安全性も高まる。
  Laravel内の他の機能とも親和性が高く、コントローラーからビューへのデータ渡しやレスポンスの生成などでもスムーズに活用できるので、
  開発効率とコードの品質が大きく向上するのがCollectionクラスを使用する大きなメリット。

<!-- - コード例
  普通の配列操作
    $prices = [1000, 1500, 2000];
    $total = 0;
    foreach($prices as $price){
      $total += $price * 1.1;
    }

  Collectionで書くと
  $prices = collect([1000, 1500, 2000]);
  $total = $prices->map(fn($price) => $price * 1.1)->sum(); -->


### view関数の第1・第2引数の指定と何をしているか
- return view('todo.index', ['todos' => $todos]);

- 第一引数：表示したいビューの名前を指定。
          'todo.index' は、resources/views/todo/index.blade.php に対応
          .はディレクトリの区切りを意味するので、todo.index → todo/index.blade.php という意味になる。

- 第二引数：ビューに渡したいデータを指定し、連想配列の形式で渡す。
          'todos' はビューで使う変数名
          $todosは$todos = $todo->all();で取得したデータが代入されている
          この記述によって、todo/index.blade.php の中で{{ route('todo.create') }}のように$todosを使ってデータを表示できるようになる。

- このview関数は、todo/index.blade.phpビューを$todosという変数を渡している。(表示はまだ)

- Laravelでは、Controllerから、HTMLを表すbladeファイルへデータを渡すためにview関数を使用する。
  view関数は画面に表示したいbladeファイルを第一引数で指定し、第二引数に渡したいデータを連想配列の形で渡すことができる。
  view関数の第二引数の連想配列は、[blade内での変数名 => 代入したい値]を意味する。


### index.blade.phpの$todos・$todoに代入されているものは何か
- @foreach ($todos as $todo)
    <div class="d-flex align-items-center p-2">
      <span class="col-9">{{ $todo->content }}</span>
    </div>
  @endforeach

- $todosには、Controllerにて取得したCollectionインスタンスが代入されている
- $todoはCollectionインスタンスに格納されているTodoインスタンスを一つずつ取り出したもの


## Todo作成機能

### Requestクラスのallメソッドは何をしているか


### fillメソッドは何をしているか

### $fillableは何のために設定しているか

### saveメソッドで実行しているSQLは何か

### redirect()->route()は何をしているか

## その他

### テーブル構成をマイグレーションファイルで管理するメリット

### マイグレーションファイルのup()、down()は何のコマンドを実行した時に呼び出されるのか

### Seederクラスの役割は何か

### route関数の引数・返り値・使用するメリット

### @extends・@section・@yieldの関係性とbladeを分割するメリット

### @csrfは何のための記述か

### {{ }}とは何の省略系か
