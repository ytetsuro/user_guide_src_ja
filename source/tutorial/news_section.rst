##################
ニュースセクション
##################

前回我々はフレームワークの基本的な部分を静的ページをロードするクラスを
書きながら学んでいきました。独自のルーティング規則を追加する事により URI
を綺麗にすることもしました。
次は動的コンテンツについて学び、データベースを使ってみましょう。

モデルの準備
------------

データベース操作をコントローラに直接各書くより、クエリは後で再利用
しやすくなるようにモデル内に記述されるべきです。モデルとは、データベースや
他のデータ格納場所からデータを抽出したり、追加したり、更新する場所です。
モデルはデータを表現します。

*application/models/* ディレクトリを開き、新しく *News_model.php* という
ファイルを作成して、次のコードを記述してください。データベースの設定が :doc:`ここ <../database/configuration>` で記述されているように適切であることも
確認してください。

::

	<?php
	class News_model extends CI_Model {

		public function __construct()
		{
			$this->load->database();
		}
	}

このコードは以前使ったコントローラのコードに似ています。ここでは ``CI_Model``
を継承し新しいモデルを作成しており、Database ライブラリをロードして
います。これにより、Database クラスが ``$this->db`` オブジェクトを通して
利用可能になります。

データベースにクエリを投げる前に、データベースのスキーマを作らなければいけません。
自分のデータベースに接続し、下記の SQL コマンド (MySQL) を実行してください。
それに加えてレコードもいくつか挿入してみてください。

::

	CREATE TABLE news (
		id int(11) NOT NULL AUTO_INCREMENT,
		title varchar(128) NOT NULL,
		slug varchar(128) NOT NULL,
		text text NOT NULL,
		PRIMARY KEY (id),
		KEY slug (slug)
	);

データベースとモデルの準備ができたら、次に必要なのはデータベースから全ての記事を
取得するためのメソッドです。そのためには CodeIgniter に含まれているデータベース
抽象化レイヤ — :doc:`Query Builder <../database/query_builder>` — を使います。
これにより、'クエリ' を一度書くだけで
:doc:`サポート済みの全データベースシステム <../general/requirements>` 上で
動かすことが出来ます。
次のコードをモデルに追加してください。

::

	public function get_news($slug = FALSE)
	{
		if ($slug === FALSE)
		{
			$query = $this->db->get('news');
			return $query->result_array();
		}

		$query = $this->db->get_where('news', array('slug' => $slug));
		return $query->row_array();
	}

このコードにより二種類のクエリを実行することが出来ます。全ての news レコードを
取得するか、`slug <#>`_ を使って特定の news 記事を取得できます。``$slug`` 変数
がクエリを実行する前にサニタイズされてないことに気づいたかもしれませんが、
:doc:`Query Builder <../database/query_builder>` がこれを代わりに行なってくれています。

ニュースを表示する
------------------

クエリを書き終えたので、今度はニュース記事を表示するビューとモデルとを紐づけて
ユーザに表示しなければいけません。これを行なうには以前作成した ``Pages`` 
コントローラを使う事も出来ますが、わかりやすくするために新たに ``News`` 
コントローラを定義しましょう。
*application/controllers/News.php* に新しいコントローラを作成してください。

::

	<?php
	class News extends CI_Controller {

		public function __construct()
		{
			parent::__construct();
			$this->load->model('news_model');
			$this->load->helper('url_helper');
		}

		public function index()
		{
			$data['news'] = $this->news_model->get_news();
		}

		public function view($slug = NULL)
		{
			$data['news_item'] = $this->news_model->get_news($slug);
		}
	}

コードを見てみると、以前作られたファイルといくつか共通点があるのに気づくかも
しれません。まず、``__construct()`` メソッド: これは親クラス　(``CI_Controller``) のコンストラクタを呼んだ後に、
モデルをロードし、このコントローラ内の全てのメソッドで利用できるようにします。
また、:doc:`URL ヘルパー <../helpers/url_helper>` 関数の集合をロードします。
ヘルパー関数の 1 つを後でビューの中で使うためです。

次に、2 つのメソッドがあります。1 つは全ニュースを閲覧するためで、1 つは特定の
ニュース記事のためです。2 番目のメソッドでは ``$slug`` 変数がモデルのメソッドに
渡されているのが見えます。
モデルはこの slug を使い、どのニュース記事を返すかを見つけます。

これでコントローラはモデル経由でデータを取得できますが、
まだ何も表示されていません。
次にするべきは、このデータをビューに渡すことです。

::

	public function index()
	{
		$data['news'] = $this->news_model->get_news();
		$data['title'] = 'News archive';

		$this->load->view('templates/header', $data);
		$this->load->view('news/index', $data);
		$this->load->view('templates/footer');
	}

上記のコードは全 news レコードを取得し、変数に代入します。タイトルの値も
``$data['title']`` 要素に代入されており、全てのデータはビューに渡されます。
今度はニュース記事を描画するためのビューを作成する必要があります。
*application/views/news/index.php* を作成し、
次のコードを追加してください。

::

	<h2><?php echo $title; ?></h2>
	
	<?php foreach ($news as $news_item): ?>

		<h3><?php echo $news_item['title']; ?></h3>
		<div class="main">
			<?php echo $news_item['text']; ?>
		</div>
		<p><a href="<?php echo site_url('news/'.$news_item['slug']); ?>">View article</a></p>

	<?php endforeach; ?>

ここでは、1 つ 1 つのニュース記事がループされながらユーザに表示されます。
このテンプレート内では PHP が HTML に混ざって記述されていることを確認できます。
もしテンプレート言語を使いたい場合、CodeIgniter の 
`テンプレートパーサ <../libraries/parser>` クラスを使うか、
サードパーティのパーサを使うことが出来ます。

ニュースの概要ページはこれで完成しましたが、特定のニュース記事を表示するページは
まだ存在しません。先ほど作られたモデルはこの処理を簡単に行なえるように作られて
います。あなたがやらなければいけないのはコントローラに多少のコードを追記するのと、
新規のビューを作成する事だけです。
``News`` コントローラに戻り、``view()`` を次のように更新してください:

::

	public function view($slug = NULL)
	{
		$data['news_item'] = $this->news_model->get_news($slug);

		if (empty($data['news_item']))
		{
			show_404();
		}

		$data['title'] = $data['news_item']['title'];

		$this->load->view('templates/header', $data);
		$this->load->view('news/view', $data);
		$this->load->view('templates/footer');
	}

``get_news()`` メソッドをパラメータなしで呼び出す代わりに、``$slug`` 変数が
渡され、特定のニュース記事が返ってくるようになります。あとは、これに対応した
ビューを *application/views/news/view.php* に作成するだけです。
次のコードをこのファイルに記述してください。

::

	<?php
	echo '<h2>'.$news_item['title'].'</h2>';
	echo $news_item['text'];

ルーティング
------------

以前作ったワイルドカードルーティング規則のために、作成したコントローラのための
新たなルートが必要です。ルーティングファイル (*application/config/routes.php*)
を下記のように修正してください。
これによりリクエストが ``Pages`` コントローラではなく、正しく ``News`` 
コントローラに到達することが保証されます。最初の行は slug 付きの URI を
``News`` コントローラの ``view()`` メソッドにルーティングしてくれます。

::

	$route['news/(:any)'] = 'news/view/$1';
	$route['news'] = 'news';
	$route['(:any)'] = 'pages/view/$1';
	$route['default_controller'] = 'pages/view';

ブラウザであなたのドキュメントルートに行き、
そのあとに index.php/news にアクセスしてニュースページを見てみてください。
