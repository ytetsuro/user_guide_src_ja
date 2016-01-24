###################################
2.2.x から 3.0.0 へのアップグレード
###################################

アップグレードを行う前に、 index.php ファイルを静的ページに置き換えて、サイトをオフラインにする必要があります。

*******************************************
Step 1: CodeIgniter ファイルを更新する
*******************************************

*system/* ディレクトリのすべてのファイルと index.php を新しいものに
**置き換えてください**。index.php に何か変更を加えていた場合、
新しいファイルに再度適用する必要があります。

.. important:: まず、古い *system/* ディレクトリを削除し、
	それから新しいものを配置する必要があります。単にコピーペーストすると、 
	問題が生じる可能性があります。

.. note:: もし独自に開発したファイルをこれらのフォルダに入れているなら、
	新しいもので置き換える前にコピーを取ってください。

**************************************
Step 2: クラスファイルの名前を更新する
**************************************

CodeIgniter 3.0 以降、全てのクラスファイル
（ライブラリ、ドライバ、コントローラ、そしてモデル）の名前は Ucfirst 形式、
つまり大文字で始まるようにしなければなりません。

例えば、あなたが次のようなライブラリを持っているなら:

	application/libraries/mylibrary.php

このようにファイル名を変更します:

	application/libraries/Mylibrary.php

ドライバライブラリ、それから CodeIgniter 付属の
コアクラスやライブラリを拡張や置き換えしている場合も同様です。

	application/libraries/MY_email.php  
	application/core/MY_log.php

これらのファイル名は次のように変更します:

	application/libraries/MY_Email.php  
	application/core/MY_Log.php

コントローラ:

	application/controllers/welcome.php	->	application/controllers/Welcome.php

モデル:

	application/models/misc_model.php	->	application/models/Misc_model.php

このルールはディレクトリ名、設定ファイル、ビュー、ヘルパー、フック等には適用
されません。クラスのみが対象です。

まとめると、次のシンプルな 1 つのルールになります。
クラス名は Ucfirst、それ以外は小文字です。

**************************************
Step 3: config/mimes.php を置き換える
**************************************

より多くの MIME タイプを含むように更新されています。
これを **application/config/mimes.php** にコピーしてください.

**************************************************************
Step 4: config/autoload.php から $autoload['core'] を削除する
**************************************************************

``$autoload['core']`` は CodeIgniter 1.4.1 で非推奨になっていましたが、ついに削除されました。
もし何か指定されていれば ``$autoload['libraries']`` に移動してください。

***************************************************
Step 5: Log クラスへの拡張・置き換えを移動する
***************************************************

Log クラスは "core" クラスとして扱われるようになったため、
**system/core/** ディレクトリに移動されました。もし Log クラスを拡張・置き換えて
いた場合、それらのファイルを **application/core/** に移動してください:

	application/libraries/Log.php -> application/core/Log.php  
	application/libraries/MY_Log.php -> application/core/MY_Log.php

**********************************************
Step 6: セッションライブラリの使い方を更新する
**********************************************

:doc:`セッションライブラリ </libraries/sessions>` は CodeIgniter 3 で刷新され、
様々な新機能が追加されています。
しかし同時に、注意の必要な変更も加えられています。

最も大きな変更は、（暗号化された）クッキーの代わりに
独立したストレージを使うようになったことです。
データの格納先としてのクッキーは完全に廃止されたので、
今後はサーバ上の何らかのストレージエンジンを使わなければなりません。
デフォルトではファイルシステムが使われます。

新しいセッションクラスは PHP のカスタムセッションハンドラを利用しています。
これによりスーパーグローバルの ``$_SESSION`` からも
セッションデータにアクセスできるようになりました
（これまで同様 "userdata" としても使えます）。

設定オプションも追加・削除されています。
詳しくは :doc:`セッションライブラリのマニュアル </libraries/sessions>` を
すべて読まれることをお勧めしますが、
ここでもいくつかのポイントを説明します。

  - ``$config['sess_driver']`` を設定する

    デフォルトは 'files' ですが、もしあなたが
    ``$config['sess_use_database']`` を設定していた場合は
    'database' になります。

  - ``$config['sess_save_path']`` を設定する

    'database' ドライバでは ``$config['sess_table_name']`` に
    フォールバックしますが、それ以外のドライバではマニュアルを参照して
    選択したドライバでの適切な値を設定する必要があります。

  - ``ci_sessions`` テーブルを変更する ('database' ドライバのみ)

    テーブルの構造が少し変更されています。具体的には:

      - ``session_id`` 列は ``id`` に名前が変わりました
      - ``user_agent`` 列は削除されました
      - ``user_data`` 列は ``data`` に名前が変わり、 MySQL では BLOB 型が使われます
      - ``last_activity`` 列は ``timestamp`` に名前が変わりました

    これに伴いインデックスの更新も必要です。
    詳しくは `Session Database ドライバ
    <../libraries/sessions.html#database-driver>`_ の説明を参照してください。

    .. important:: 現在公式にサポートしているのは MySQL と PostgreSQL のみです。
    	他のデータベースでも動作するかもしれませんが、助言ロックがないために
    	同時アクセスに対して安全ではありません。
    	この場合は他のドライバを検討してください。

  - ``$config['sess_match_useragent']`` を削除する

    ユーザエージェント文字列はブラウザから提供される値、つまりクライアントからの
    入力値です。これを検証しても意味がないので、
    この機能は廃止されました。

  - ``$config['sess_encrypt_cookie']`` を削除する

    前述の通り、今後クッキーにセッションデータが格納されることは
    ありませんから、このオプションもお役御免になりました。

  - ``$config['sess_expire_on_close']`` を削除する

    このオプションはまだ使えますが、後方互換性のためだけに残されています。
    同じ効果は ``$config['sess_expiration']`` を 0 に設定することで
    得ることができます。

  - "flashdata" と "userdata" の衝突をチェックする

    フラッシュデータは次のリクエストで削除するようにマークされている以外は
    ただの "userdata" になりました。つまり、同じ名前の "userdata" と
    "flashdata" を別々に扱うことはできません。

  - セッションメタデータをチェックする

    これまでは 'session_id' 、'ip_address' 、'user_agent' 、'last_activity'
    といったメタデータにも userdata としてアクセスできました。
    今後はできませんので、もしこれらのデータを利用しているなら、
    `セッションメタデータ <../libraries/sessions.html#accessing-session-metadata>`_
    に関する説明を参照してください。

  - ``unset_userdata()`` の使い方をチェックする

    これまでは ``'key' => 'dummy value'`` といった連想配列で、
    一度に複数のキーを削除することができました。
    しかし、この方法は無意味なので、
    今後はキー *だけ* を配列の要素として渡さなければなりません。

    ::

    	// 以前
    	$this->session->unset_userdata(array('item' => '', 'item2' => ''));

    	// 今後
    	$this->session->unset_userdata(array('item', 'item2'));

最後に、もしセッションを拡張して使っていたら、
*application/libraries/Session/* ディレクトリに移動してください。
たぶん、コードの書き換えも必要でしょうが。

***************************************
Step 7: config/database.php を更新する
***************************************

3.0.0 で Active Record から Query Builder に名前が変更されたので、
**config/database.php** の ``$active_record`` 変数も
``$query_builder`` に変更する必要があります::

	$active_group = 'default';
	// $active_record = TRUE;
	$query_builder = TRUE;

**************************************
Step 8: エラーテンプレートを置き換える
**************************************

CodeIgniter 3.0 ではエラーテンプレートもビューとして扱われ、
**application/views/errors** ディレクトリに移動されました。

また、CLI 向けにプレインテキストのエラーテンプレートを
サポートするようになりました。これも新たに分ける必要があります。

これまでのテンプレートは **application/errors** から **application/views/errors/html** にそのまま移動して構いませんが、
新しい **application/views/errors/cli** ディレクトリは CodeIgniter のアーカイブからコピーしてください。

********************************************
Step 9: config/routes.php ファイルを更新する
********************************************

:any を含むルート
======================

歴史的に、 CodeIgniter はこれまでずっとワイルドカード **:any** を
提供してきました。これは URI セグメント **内** で
任意の文字列にマッチします。

しかし実際の **:any** は **.+** という正規表現のエイリアスとして
処理されていました。この正規表現は URI セグメントの
区切り文字である / (スラッシュ) にもマッチするため、
意図しない挙動を生むことになります。これはバグと考えられます。

CodeIgniter 3 では **:any** が **[^/]+** を表わすようになり、
スラッシュにマッチすることはなくなりました。

一方でこのバグを活用していた開発者もたくさんいることでしょう。
もしあなたがその一人で、引き続きスラッシュにマッチさせたいなら、
正規表現 **.+** を使ってください::

	(.+)	// 全ての文字にマッチする
	(:any)	// '/' 以外、全ての文字にマッチする

ディレクトリと 'default_controller'、'404_override'
====================================================

ご存じのように ``$route['default_controller']`` と
``$route['404_override']`` の設定は、コントローラ名だけでなく、
*コントローラ/メソッド* のペアも受け取ります。しかしながら、ルーティングロジック
にバグがあり、
*ディレクトリ/コントローラ* として使っているユーザがみられます。

この振る舞いは意図したものではなく、偶然であり文書化もされていません。
もしあなたのアプリケーションがこれを使っていたら、CodeIgniter 3.0 では動かなく
なります。

バージョン 3 における他の言及すべき変更は、'default_controller' と
'404_override' が *ディレクトリ毎に* 適用されることです。説明のために、
以下の例を用います::

	$route['default_controller'] = 'main';

あなたの Web サイトが *example.com* にあるとすると、ユーザが
``http://example.com/`` にアクセスすると、上記の設定が
'Main' コントローラをロードします。

ところで、もし *application/controllers/admin/* ディレクトリがあり、
ユーザが ``http://example.com/admin/`` にアクセスしたらどうなるでしょう?
CodeIgniter 3 では、ルータが admin/ ディレクトリの下の 'Main' コントローラ
を探します。見つからなければ、Not Found (404) が起こります。

同じルールが '404_override' 設定にも適用されます。

**************************************************************************************
Step 10: 多くの関数が要素がないとき FALSE ではなく NULL を返すようになりました
**************************************************************************************

多くのメソッドと関数は、要求された要素が存在しないとき、FALSE ではなく NULL を返すようになりました:

 - :doc:`共通関数 <../general/common_functions>`

   - config_item()

 - :doc:`Config クラス <../libraries/config>`

   - config->item()
   - config->slash_item()

 - :doc:`Input クラス <../libraries/input>`

   - input->get()
   - input->post()
   - input->get_post()
   - input->cookie()
   - input->server()
   - input->input_stream()
   - input->get_request_header()

 - :doc:`Session クラス <../libraries/sessions>`

   - session->userdata()
   - session->flashdata()

 - :doc:`URI クラス <../libraries/uri>`

   - uri->segment()
   - uri->rsegment()

 - :doc:`Array クラス <../helpers/array_helper>`

   - element()
   - elements()

*******************************
Step 11: XSS フィルタの使い方
*******************************

CodeIgniter の多くの関数は、論理値のパラメータを渡すことで、
XSS フィルタリング機能を使うことができます。
そのデフォルト値は以前は FALSE でしたが、今は NULL に変更され、
``$config['global_xss_filtering']`` の値により動的に
決定されます。

もし、手動で論理値を ``$xss_filter`` パラメータに渡していたり、
``$config['global_xss_filtering']`` に FALSE を設定していた場合は、
この変更は影響しません。

そうでない場合は、次の関数の使い方を確認してください:

 - :doc:`Input ライブラリ <../libraries/input>`

   - input->get()
   - input->post()
   - input->get_post()
   - input->cookie()
   - input->server()
   - input->input_stream()

 - :doc:`Cookie ヘルパー <../helpers/cookie_helper>` :php:func:`get_cookie()`

.. important:: 他の関連する変更点として、 ``$_GET``、``$_POST``、
	``$_COOKIE`` そして ``$_SERVER`` スーパーグローバルは、グローバル XSS
	フィルタリングがオンの場合に、自動的に上書きされなくなりました。

******************************************************
Step 12: URI にまつわる潜在的な XSS 問題をチェックする
******************************************************

:doc:`URI ライブラリ <../libraries/uri>` は以前は特定の "問題のある文字"
が URI セグメントに含まれるとき、HTML エンティティに自動的に
変換していました。

これは、``$config['permitted_uri_chars']`` の設定に加えて、
自動 XSS 保護の提供を目的としていましたが、問題があることが判明したため、
CodeIgniter 3.0 で削除されました。

もし、あなたのアプリケーションがこの機能に頼っているなら、
出力時に ``$this->security->xss_clean()`` で URI セグメントを
フィルタするように更新するべきです。

****************************************************************
Step 13: バリデーションルール 'xss_clean' の使い方をチェックする
****************************************************************

XSS クリーニングについて広く知られていないルールとして、
入力データに対してではなく、*出力にのみ適用すべき* があります。

我々は自動グローバル XSS クリーニング機能 (上の XSS についてのステップを参照)
においてミスを犯しました。そのため、現在、そのような実践に反対するために、
:doc:`フォームバリデーション <../libraries/form_validation>` のルールの
公式サポートリストから、'xss_clean' を削除しています。

何故なら、:doc:`フォームバリデーションライブラリ <../libraries/form_validation>`
は一般的に *入力* データを検証するため、
'xss_clean' ルールは単純にそこに属さないからです。

もし本当に本当にそのルールを適用する必要があるなら、
通常の関数として ``xss_clean()`` が含まれる
:doc:`Security ヘルパー <../helpers/security_helper>` をロードしてください。
通常の関数なら検証ルールとして今でも使用できます。

***********************************************************
Step 14: 入力クラスの get_post() メソッドの使い方を更新する
***********************************************************

以前は、 :doc:`Input クラス <../libraries/input>` の ``get_post()``
メソッドは最初に POST データを探し、それから GET データを探しました。
このメソッドは、その名前が示すように、GET を先に探しその後 POST を
探すように変更されました。

以前と同じようにするために、POST を先に探し GET を探す ``post_get()`` メソッドが追加されました。

**********************************************************************
Step 15: ディレクトリヘルパーの directory_map() 関数の使い方を更新する
**********************************************************************

結果の配列の中で、ディレクトリはディレクトリセパレータ (通常はスラッシュ)
で終わるようになりました。

***********************************************************************
Step 16: データベースフォージの drop_table() メソッドの使い方を更新する
***********************************************************************

今まで ``drop_table()`` は IF EXISTS 句をデフォルトで追加していましたが、
いくつかのドライバでは全く動作しませんでした。
CodeIgniter 3.0 では、IF EXISTS 条件はデフォルトでは追加されず、
オプションの第 2 引数が追加され、デフォルトは FALSE です。

もし、アプリケーションが IF EXISTS を使っているなら、使い方を変更する必要があります。

::

	// 今は DROP TABLE `table_name` だけを生成します
	$this->dbforge->drop_table('table_name');

	// DROP TABLE IF EXISTS `table_name` を生成します
	$this->dbforge->drop_table('table_name', TRUE);

.. note:: この例では MySQL 固有の文法を使っていますが、ODBC 以外のすべての
	ドライバで機能します。

***********************************************************
Step 17: Email ライブラリでの複数メールの扱いを変更する
***********************************************************

:doc:`Email ライブラリ <../libraries/email>` は、メール送信が成功した後に、
自動的に設定されたパラメータをクリアします。
この振る舞いを上書きしたい場合は、``send()`` メソッドの第 1 引数に FALSE
を渡します:

::

	if ($this->email->send(FALSE))
 	{
 		// パラメータはクリアされません
 	}

***************************************************
Step 18: Form_validation の言語ファイルを更新する
***************************************************

:doc:`Form Validation ライブラリ <../libraries/form_validation>` の
:doc:`言語 <../libraries/language>` ファイルとエラーメッセージのフォーマットに
2 つの改良が加えられました:

 - :doc:`Language ライブラリ <../libraries/language>` の行のキーは、衝突を
   避けるために、接頭辞 **form_validation_** を付けなければなりません::

	// 以前
	$lang['rule'] = ...

	// 今後
	$lang['form_validation_rule'] = ...

 - エラーメッセージのフォーマットが名前付きパラメータを使うように変更されました。
   `sprintf()` よりも柔軟にするためです::

	// 以前
	'The %s field does not match the %s field.'

	// 今後
	'The {field} field does not match the {param} field.'

.. note:: 古いフォーマットもまだ動作しますが、接頭辞のない行のキーは廃止予定
	であり、CodeIgniter 3.1+ で削除される予定です。
	よって、早めに更新することが推奨されます。

************************************************************
Step 19: 'base_url' の設定値が空でないことを確認する
************************************************************

``$config['base_url']`` が設定されていないとき、CodeIgniter は Web サイトの
ベース URL を自動的に検知しようとします。これは、純粋に新しいアプリケーションの
開発を開始する際、便利なようになされているものです。

自動検知は決して信頼できるものではなく、セキュリティと関連します。
だから、**いつでも** 手動で設定すべきものです。

CodeIgniter 3.0.3 での変更点の 1 つは、自動検知がどう機能するかです。
より具体的に言えば、クライアントによりリクエストされたホスト名ではなく、
サーバの IP アドレスにフォールバックするようになりました。ゆえに、
もしあなたが自動検知に頼っている場合、Web サイトの振る舞いが変更されるでしょう。

例えば、複数のドメインを使っていたり、http:// と https:// の両方のプリフィックス
をリクエストに応じて動的に使っている場合、
*application/config/config.php* は PHP スクリプトであることを思い出してください。
例えば、次のような数行のコードでロジックを記述できます::

	$allowed_domains = array('domain1.tld', 'domain2.tld');
	$default_domain  = 'domain1.tld';

	if (in_array($_SERVER['HTTP_HOST'], $allowed_domains, TRUE))
	{
		$domain = $_SERVER['HTTP_HOST'];
	}
	else
	{
		$domain = $default_domain;
	}

	if ( ! empty($_SERVER['HTTPS']))
	{
		$config['base_url'] = 'https://'.$domain;
	}
	else
	{
		$config['base_url'] = 'http://'.$domain;
	}


****************************************************************
Step 20: (以前から) 廃止予定の機能を削除する
****************************************************************

``$autoload['core']`` の設定以外にも、CodeIgniter 3.0.0 で削除された機能は
たくさんあります。

SHA1 ライブラリ
================

以前から廃止予定だった SHA1 ライブラリが削除されました。
SHA1 ハッシュを生成するには、PHP のネイティブの ``sha1()`` 関数を使うようにコードを変更してください。

加えて、:doc:`Encrypt ライブラリ <../libraries/encrypt>` の ``sha1()`` メソッド も削除されました。

EXT 定数
================

PHP 4 のサポートをドロップして以来、``EXT`` 定数の使用は推奨されていません。この
新しい CodeIgniter のバージョンで、もはや異なるファイル名の拡張子を保持する必要は
ありません。``EXT`` 定数は削除されました。代わりに、単に '.php' を使って下さい。

Smiley ヘルパー
===============

:doc:`Smiley ヘルパー <../helpers/smiley_helper>` は EllisLab の製品
ExpressEngine 由来のレガシーな機能です。CodeIgniter のような一般的な用途の
フレームワークにとっては、具体的すぎるため、廃止予定となりました。

また、以前から廃止予定の ``js_insert_smiley()`` (バージョン 1.7.2 から) は削除されました。

Encrypt ライブラリ
===================

多数の脆弱性報告に従い、 :doc:`Encrypt ライブラリ <../libraries/encrypt>` は
廃止予定となり、新たな :doc:`Encryption ライブラリ <../libraries/encryption>`
が代わりに追加されました。

新しいライブラリは `MCrypt 機能拡張 <http://php.net/mcrypt>`_ 
(そして /dev/urandom が利用可能なこと) または PHP 5.3.3 と 
`OpenSSL 機能拡張 <http://php.net/openssl>`_ を必要とします。
これは少々不便ですが、暗号の機能を適切に実装するための必要条件です。

.. note:: :doc:`Encrypt ライブラリ <../libraries/encrypt>` はまだ後方互換の
	ために使用可能です。

.. important:: できるだけ早く新しい :doc:`Encryption ライブラリ
	<../libraries/encryption>` に移行することが強く推奨されます!

Cart ライブラリ
================

:doc:`Cart ライブラリ <../libraries/cart>` は :doc:`Smiley ヘルパー
<../helpers/smiley_helper>` と同じように、CodeIgniter にとっては具体的
過ぎます。現在、廃止予定であり、CodeIgniter 3.1+ で削除される予定です。

.. note:: このライブラリはまだ使用可能ですが、早急に使用を止めることを強く
	推奨します。

データベースドライバ 'mysql'、'sqlite'、'mssql'、'pdo/dblib'
============================================================

**mysql** ドライバは、古いコードベースと多くの低レベルでの問題で知られる、古い 'mysql' PHP 機能拡張を利用します。
この機能拡張は PHP 5.5 で廃止予定となり、CodeIgniter ではバージョン 3.0 で
廃止予定となり、デフォルトの MySQL ドライバ設定は **mysqli** に変更されました。

'mysqli' または 'pdo/mysql' ドライバのいずれかを使用してください。古い
'mysql' ドライバは将来削除されます。

**sqlite**、**mssql** そして **pdo/dblib** (pdo/mssql または pdo/sybase としても知られる) ドライバはすべて、さまざまな理由から PHP 5.3 には存在しない PHP 機能拡張に依存しています。

それゆえ、それらのドライバも廃止予定であり、CodeIgniter の次のバージョンで
削除されます。
より進歩した **sqlite3**、**sqlsrv** または **pdo/sqlsrv** ドライバを
使うべきです。

.. note:: これらのドライバはまだ使用可能ですが、早急に他のドライバに
	変更することを強く推奨します。

Security ヘルパーの do_hash()
===============================

:doc:`Security ヘルパー <../helpers/security_helper>` の関数 ``do_hash()`` は
現在 PHP のネイティブ ``hash()`` 関数の単なるエイリアスです。
廃止予定であり、CodeIgniter 3.1+ で削除される予定です。

.. note:: このライブラリはまだ使用可能ですが、早急に使用を止めることを強く
	推奨します。

$config['global_xss_filtering'] 設定
===========================================

上記ですでに説明したように、XSS フィルタリングは入力データに対しするものではなく、
出力にすべきものです。ゆえに、自動的に *入力* データをフィルタする
``$config['global_xss_filtering']`` は悪い実践だと考えられ、廃止予定とされました。

Instead, you should manually escape any user-provided data via the
:php:func:`xss_clean()` function when you need to output it, or use a
library like `HTML Purifier <http://htmlpurifier.org/>`_ that does that
for you.

.. note:: The setting is still available, but you're strongly encouraged to
	remove its usage sooner rather than later.

File ヘルパーの read_file()
=============================

:doc:`File Helper <../helpers/file_helper>` function ``read_file()`` is now just an alias for
PHP's native ``file_get_contents()`` function. It is deprecated and scheduled for removal in
CodeIgniter 3.1+.

.. note:: This function is still available, but you're strongly encouraged to remove its usage sooner
	rather than later.

String ヘルパーの repeater()
============================

:doc:`String Helper <../helpers/string_helper>` function :php:func:`repeater()` is now just an alias for
PHP's native ``str_repeat()`` function. It is deprecated and scheduled for removal in CodeIgniter 3.1+.

.. note:: This function is still available, but you're strongly encouraged to remove its usage sooner
	rather than later.

String ヘルパーの trim_slashes()
================================

:doc:`String Helper <../helpers/string_helper>` function :php:func:`trim_slashes()` is now just an alias
for PHP's native ``trim()`` function (with a slash passed as its second argument). It is deprecated and
scheduled for removal in CodeIgniter 3.1+.

.. note:: This function is still available, but you're strongly encouraged to remove its usage sooner
	rather than later.

Form ヘルパーの form_prep()
=============================

:doc:`Form Helper <../helpers/form_helper>` function :php:func:`form_prep()`
is now just an alias for :doc:`common function </general/common_functions>`
:func:`html_escape()`. It is deprecated and will be removed in the future.

Please use :php:func:`html_escape()` instead.

.. note:: This function is still available, but you're strongly encouraged
	to remove its usage sooner rather than later.

Email ヘルパーの関数
======================

:doc:`Email Helper <../helpers/email_helper>` only has two functions

 - :php:func:`valid_email()`
 - :php:func:`send_email()`

Both of them are now aliases for PHP's native ``filter_var()`` and ``mail()`` functions, respectively.
Therefore the :doc:`Email Helper <../helpers/email_helper>` altogether is being deprecated and
is scheduled for removal in CodeIgniter 3.1+.

.. note:: These functions are still available, but you're strongly encouraged to remove their usage
	sooner rather than later.

Date ヘルパーの standard_date()
================================

:doc:`Date Helper <../helpers/date_helper>` function ``standard_date()`` is being deprecated due
to the availability of native PHP `constants <http://php.net/manual/en/class.datetime.php#datetime.constants.types>`_,
which when combined with ``date()`` provide the same functionality. Furthermore, they have the
exact same names as the ones supported by ``standard_date()``. Here are examples of how to replace
its usage:

::

	// Old way
	standard_date(); // defaults to standard_date('DATE_RFC822', now());

	// Replacement
	date(DATE_RFC822, now());

	// Old way
	standard_date('DATE_ATOM', $time);

	// Replacement
	date(DATE_ATOM, $time);

.. note:: This function is still available, but you're strongly encouraged to remove its usage sooner
	rather than later as it is scheduled for removal in CodeIgniter 3.1+.

HTML ヘルパーの nbs()、br()
============================

:doc:`HTML Helper <../helpers/html_helper>` functions ``nbs()`` and ``br()`` are just aliases
for the native ``str_repeat()`` function used with ``&nbsp;`` and ``<br >`` respectively.

Because there's no point in just aliasing native PHP functions, they are now deprecated and
scheduled for removal in CodeIgniter 3.1+.

.. note:: These functions are still available, but you're strongly encouraged to remove their usage
	sooner rather than later.

Pagination ライブラリの 'anchor_class' 設定
=============================================

The :doc:`Pagination Library <../libraries/pagination>` now supports adding pretty much any HTML
attribute to your anchors via the 'attributes' configuration setting. This includes passing the
'class' attribute and using the separate 'anchor_class' setting no longer makes sense.
As a result of that, the 'anchor_class' setting is now deprecated and scheduled for removal in
CodeIgniter 3.1+.

.. note:: This setting is still available, but you're strongly encouraged to remove its usage sooner
	rather than later.

String ヘルパーの random_string() のタイプ 'unique' と 'encrypt'
==================================================================

When using the :doc:`String Helper <../helpers/string_helper>` function :php:func:`random_string()`,
you should no longer pass the **unique** and **encrypt** randomization types. They are only
aliases for **md5** and **sha1** respectively and are now deprecated and scheduled for removal
in CodeIgniter 3.1+.

.. note:: These options are still available, but you're strongly encouraged to remove their usage
	sooner rather than later.

URL ヘルパーの url_title() のセパレータ 'dash' and 'underscore'
===============================================================

When using the :doc:`URL Helper <../helpers/url_helper>` function :php:func:`url_title()`, you
should no longer pass **dash** or **underscore** as the word separator. This function will
now accept any character and you should just pass the chosen character directly, so you
should write '-' instead of 'dash' and '_' instead of 'underscore'.

**dash** and **underscore** now act as aliases and are deprecated and scheduled for removal
in CodeIgniter 3.1+.

.. note:: These options are still available, but you're strongly encouraged to remove their usage
	sooner rather than later.

Session ライブラリの all_userdata() メソッド
=============================================

As seen in the :doc:`Change Log <../changelog>`, :doc:`Session Library <../libraries/sessions>`
method ``userdata()`` now allows you to fetch all userdata by simply omitting its parameter::

	$this->session->userdata();

This makes the ``all_userdata()`` method redudant and therefore it is now just an alias for
``userdata()`` with the above shown usage and is being deprecated and scheduled for removal
in CodeIgniter 3.1+.

.. note:: This method is still available, but you're strongly encouraged to remove its usage
	sooner rather than later.

Database Forge の AFTER 句を伴う add_column() メソッド 
=======================================================

If you have used the **third parameter** for :doc:`Database Forge <../database/forge>` method
``add_column()`` to add a field for an AFTER clause, then you should change its usage.

That third parameter has been deprecated and scheduled for removal in CodeIgniter 3.1+.

You should now put AFTER clause field names in the field definition array instead::

	// Old usage:
	$field = array(
		'new_field' => array('type' => 'TEXT')
	);

	$this->dbforge->add_column('table_name', $field, 'another_field');

	// New usage:
	$field = array(
		'new_field' => array('type' => 'TEXT', 'after' => 'another_field')
	);

	$this->dbforge->add_column('table_name', $field);

.. note:: The parameter is still available, but you're strongly encouraged to remove its usage
	sooner rather than later.

.. note:: This is for MySQL and CUBRID databases only! Other drivers don't support this
	clause and will silently ignore it.

URI ルーティングのメソッド fetch_directory()、fetch_class()、fetch_method()
============================================================================

プロパティ ``CI_Router::$directory``、``CI_Router::$class`` そして ``CI_Router::$method``
は public であり、対応する ``fetch_*()`` はそれらのプロパティを返すだけです。
なので、これらのメソッドを持つ意味はありません。

それらはすべて内部的な文書化されていないメソッドですが、もしものため、
後方互換性を維持するため、それらを廃止予定とします。
もし、それらのメソッドを使っている人は、代わりにプロパティにアクセスしてください::

	$this->router->directory;
	$this->router->class;
	$this->router->method;

.. note:: これらのメソッドはまだ利用可能ですが、早急に使用を止めることを強く
	推奨します。

Input ライブラリの is_cli_request() メソッド
=============================================

Calls to the ``CI_Input::is_cli_request()`` method are necessary at many places
in the CodeIgniter internals and this is often before the :doc:`Input Library
<../libraries/input>` is loaded. Because of that, it is being replaced by a common
function named :php:func:`is_cli()` and this method is now just an alias.

The new function is both available at all times for you to use and shorter to type.

::

	// Old
	$this->input->is_cli_request();

	// New
	is_cli();

``CI_Input::is_cli_request()`` is now now deprecated and scheduled for removal in
CodeIgniter 3.1+.

.. note:: This method is still available, but you're strongly encouraged to remove its usage
	sooner rather than later.

Config ライブラリの system_url() メソッド
=========================================

Usage of ``CI_Config::system_url()`` encourages insecure coding practices.
Namely, your CodeIgniter *system/* directory shouldn't be publicly accessible
from a security point of view.

Because of this, this method is now deprecated and scheduled for removal in
CodeIgniter 3.1+.

.. note:: This method is still available, but you're strongly encouraged to remove its usage
	sooner rather than later.

=========================
The Javascript ライブラリ
=========================

:doc:`Javascript ライブラリ <../libraries/javascript>` は今までずっと '実験' 
状態であり、そんなに役に立たず、また、適切な解決策でもありませんでした。

現在、廃止予定であり、CodeIgniter 3.1+ で削除される予定です。

.. note:: このライブラリはまだ使えますが、早急に使用を中止することを
	強く推奨します。

**************************************************************
Step 21: Text ヘルパーの highlight_phrase() の使い方を確認する
**************************************************************

The default HTML tag used by :doc:`Text Helper <../helpers/text_helper>` function
:func:`highlight_phrase()` has been changed from ``<strong>`` to the new HTML5
tag ``<mark>``.

Unless you've used your own highlighting tags, this might cause trouble
for your visitors who use older web browsers such as Internet Explorer 8.
We therefore suggest that you add the following code to your CSS files
in order to avoid backwards compatibility with old browsers::

	mark {
		background: #ff0;
		color: #000;
	};
