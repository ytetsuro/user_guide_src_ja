#############
Cookie ヘルパー
#############

クッキーヘルパーのファイルは、クッキーを処理するのに役立つ関数で構成さ
れています。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

ヘルパーをロードする
===================

このヘルパーは次のコードを使ってロードします::

	$this->load->helper('cookie');

利用できる機能
===================

次の関数が利用できます:


.. php:function:: set_cookie($name[, $value = ''[, $expire = ''[, $domain = ''[, $path = '/'[, $prefix = ''[, $secure = FALSE[, $httponly = FALSE]]]]]]])

	:param	mixed	$name: Cookie name *or* associative array of all of the parameters available to this function
	:param	string	$value: Cookie value
	:param	int	$expire: Number of seconds until expiration
	:param	string	$domain: Cookie domain (usually: .yourdomain.com)
	:param	string	$path: Cookie path
	:param	string	$prefix: Cookie name prefix
	:param	bool	$secure: Whether to only send the cookie through HTTPS
	:param	bool	$httponly: Whether to hide the cookie from JavaScript
	:rtype:	void

	This helper function gives you friendlier syntax to set browser
	cookies. Refer to the :doc:`Input Library <../libraries/input>` for
	a description of its use, as this function is an alias for
	``CI_Input::set_cookie()``.

.. php:function:: get_cookie($index[, $xss_clean = NULL])

	:param	string	$index: Cookie name
	:param	bool	$xss_clean: Whether to apply XSS filtering to the returned value
	:returns:	The cookie value or NULL if not found
	:rtype:	mixed

	This helper function gives you friendlier syntax to get browser
	cookies. Refer to the :doc:`Input Library <../libraries/input>` for
	detailed description of its use, as this function acts very
	similarly to ``CI_Input::cookie()``, except it will also prepend
	the ``$config['cookie_prefix']`` that you might've set in your
	*application/config/config.php* file.

.. php:function:: delete_cookie($name[, $domain = ''[, $path = '/'[, $prefix = '']]])

	:param	string	$name: Cookie name
	:param	string	$domain: Cookie domain (usually: .yourdomain.com)
	:param	string	$path: Cookie path
	:param	string	$prefix: Cookie name prefix
	:rtype:	void

	Lets you delete a cookie. Unless you've set a custom path or other
	values, only the name of the cookie is needed.
	::

		delete_cookie('name');

	この関数は、値や有効期限のパラメータがない以外は、 ``set_cookie()`` と同じ
	ものです。第1引数に配列で渡すか、個別パラメータで値を渡すことができま
	す。
	::

		delete_cookie($name, $domain, $path, $prefix);
