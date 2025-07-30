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
- $inputs = $request->all();

- フォームから送信された全てのデータを連想配列の形で取得している
- $requestは、Illuminate\Http\Requestクラスのインスタンス


### fillメソッドは何をしているか
- $todo->fill($inputs);

- 連想配列で取得した値を->fill()を使用して、Todoインスタンスの各プロパティに一括で代入している
- ->fill()は$todo->{連想配列のkey} = {連想配列のvalue}を配列の全ての要素に対して行ってくれる


### $fillableは何のために設定しているか
- protected $fillable = [
    'content',
  ];

- $fillableというプロパティは、名前の通り->fill()によってModelに代入可能なプロパティを記述するもの。
  一括代入には脆弱性があるため$fillableを定義して代入できる項目に制限をかける必要がある。
  対策をせずに使用すると悪意のあるユーザに攻撃されてしまう恐れがある。


### saveメソッドで実行しているSQLは何か
- $todo->save();

- INSERT INTO todos (content) VALUES (:content);
- Todoインスタンスの'->save()'を実行してオブジェクトの状態をDBに保存するINSERT文を実行


### redirect()->route()は何をしているか
- return redirect()->route('todo.index');

- ToDoが新規作成された後に、白い画面ではなく一覧画面を表示させたいので、一覧画面にリダイレクトするルートを定義している
- redirect()->route('ルート名')とすることでリダイレクトさせることができる


## その他

### テーブル構成をマイグレーションファイルで管理するメリット
- SQLを知らなくても、PHPコードでテーブル操作ができるため学習コストが不要であり、
  マイグレーションファイルをGitで共有することで、開発者全員が同じテーブルを作成することができる(開発者同士のテーブル構成を統一させることができる)

- マイグレーションは、データベースのテーブル構造（例：カラムの追加や削除、テーブルの作成など）をPHPのコードとして定義・管理できる仕組み
  データベースの設計変更をソースコードと同様にバージョン管理することができ、チームでの開発や環境の再構築が非常に効率的になる

  マイグレーションを使うことで、データベースに直接SQLを実行して構造を変更する必要がなくなり、誰がどのような変更を行ったかが明確になる。
  開発環境・本番環境・テスト環境などで同じ構成のデータベースを自動で作成できるため、環境ごとの不整合も防ぐことができる。


### マイグレーションファイルのup()、down()は何のコマンドを実行した時に呼び出されるのか
- up()：php artisan migrate を実行した時に呼び出される
        データベースに新しいテーブル、カラム、またはインデックスを追加するために使用

  down()：php artisan migrate:rollback を実行した時に呼び出される
          upメソッドによって実行する操作と逆の操作を実装し、以前の状態へ戻す必要がある時に使用


### Seederクラスの役割は何か
- Seederクラスは、Laravelのデータベースに初期データを自動で投入するための仕組み
- Seederクラスは、テーブルに対して任意のデータを挿入（INSERT）するための処理を記述する箇所
  開発やテストのために、あらかじめダミーデータやテストデータを用意しておきたい場合に使用する

- Seederを使用するメリットは下記の２つ
- SQLを知らなくても、PHPコードでテーブル操作ができるため学習コストが不要
  シーダーファイルをgit管理することで、テストデータを開発者間で共有できる


### route関数の引数・返り値・使用するメリット
- Route::get('/todo', function () {
    echo 'Hello World!';
  });

- 引数：URLを直接書かずに、ルート名を記述
       することで変更に柔軟に対応できる
  返り値：文字列（string）として、ルート名に対応するURLを返す
　       URLの管理がしやすく、メンテナンス性・安全性が上がる

- Laravelにおけるroute()関数は、定義済みのルート名から対応するURLを生成するための関数
  これを使うことで、URLを直接記述することなく、ルート名をもとに安全かつ柔軟にリンクの生成ができる


### @extends・@section・@yieldの関係性とbladeを分割するメリット
- @extends・@section・@yield は、LaravelのテンプレートエンジンBladeでレイアウトを継承するための3つの主要なディレクティブ(命令)
- bladeファイルはテンプレートエンジンの一つで、HTMLに比べて簡単に様々な機能を実施することができる

- @extends：親テンプレートを指定する(子テンプレートに記載)
- @section：親テンプレートに渡す中身(コンテンツ)を定義する(子テンプレートに記載)
- @yield：親テンプレート内で子テンプレートの中身を表示する場所を定義する(親テンプレートに記載)

- @extends('Bladeファイルのパス')を使用することで他のBladeファイルを継承することができる
  @section('任意の文字列') ~ @endsectionで囲った部分を継承したBladeファイルの@yield('任意の文字列')の箇所に挿入される

- Bladeファイルを分割することで、共通部分の再利用が可能になり、コードの可読性が向上する。
  さらに、修正や追加が簡単になり、チーム開発でも効率的に作業を分担できるようになる。


### @csrfは何のための記述か
- @csrfとはLaravelのフォーム送信時にセキュリティ対策として必要なCSRFトークンを自動で埋め込むディレクティブ
  POSTなどのリクエストで外部からの不正な送信を防ぐ役割を果たしている

- @csrfと追記するだけでCSRF対策が完了する
- CSRF対策のためのトークンが含まれたinputタグが生成され、CSRF対策のトークンも一緒に送信され、Laravel側でトークンの検証も自動的に実施してくれる。


### {{ }}とは何の省略系か
- {{ }}はPHPのechoを省略した書き方
  自動でHTMLエスケープ処理されるため、もし$nameに危険なHTMLが入っていても、勝手に安全に変換してくれる

- 以下の２つは同じ意味になる
  {{ $name }}　　　　　　　　← Blade構文（Laravelでよく使う）
  <?php echo $name; ?>　　 ← PHP構文

- bladeでは、PHPを記述する際に{{ }}を使用する。
  この波括弧で囲ってあげることで、その部分がPHPの処理として認識される。

- {!! !!} はエスケープしないでそのまま出力する方法
  HTMLをそのまま表示できるが、悪意あるコードが含まれていると、XSS攻撃の原因になる
