###############
言語ヘルパー
###############

言語ヘルパーのファイルの中身は、
言語ファイルでの動作に役立つ関数です。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

ヘルパーのロード
===================

このヘルパーは次のコードを使ってロードします::

	$this->load->helper('language');

利用できる機能
===================

次の関数が利用できます:


.. php:function:: lang($line[, $for = ''[, $attributes = array()]])

 	:param	string	$line: Language line key
 	:param	string	$for: HTML "for" attribute (ID of the element we're creating a label for)
 	:param	array	$attributes: Any additional HTML attributes
 	:returns:	HTML-formatted language line label
	:rtype:	string

	この関数は、ビューファイルで言語クラスの ``CI_Lang::line()`` 
	メソッドを呼び出すよりもより簡単な構文で
	ロードされた言語ファイルからのテキスト行を返します。

	例::

		echo lang('language_key', 'form_item_id', array('class' => 'myClass'));
		// Outputs: <label for="form_item_id" class="myClass">Language line</label>