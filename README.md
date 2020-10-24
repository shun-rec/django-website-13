# djangoチュートリアル #12

## 管理サイトをカスタマイズしよう！前編 〜全体設定編〜

djangoの大きな特徴の１つは自動生成された管理サイトがついてくることです。  
サービス管理者が内部で使ったり、カスタマイズをすれば一般ユーザーも使うことが可能です！

## 完成版プロジェクト

<https://github.com/shun-rec/django-website-12>

## 事前準備

### 新規サーバーを立ち上げる

[Paiza](https://paiza.cloud)で作業している方はdjangoにチェックを入れて新規サーバーを立ち上げて下さい。  
自分のマシンで作業している方はdjangoが使える作業用フォルダを作成して下さい。

### ブログを作ったプロジェクトをダウンロード

※ 新規djangoプロジェクトを作成しても構いません。その場合は、テスト用のモデルを追加して下さい。

ターミナルを開いて、以下を実行。

```sh
git clone https://github.com/shun-rec/django-website-07
```

フォルダを移動

```sh
cd django-website-07
```

### マイグレートしてDBを作成

```sh
python manage.py migrate
```

### スーパーユーザーを作成

```sh
python manage.py createsuperuser
```

* ユーザー名: admin
* メールアドレス: （無し）
* パスワード: admin

## 管理サイトのタイトルなど全体設定をカスタマイズしよう

最も簡単でよく使われる方法は、全体URL設定（`pj_blog/urls.py`）の`admin`オブジェクトを直接編集することです。  

`pj_blog/urls.py`に以下を追記。

```py
from django.contrib.auth.models import Group

admin.site.site_title = '匿名ブログ 内部管理サイト'
admin.site.site_header = '匿名ブログ 内部管理サイト'
admin.site.index_title = 'メニュー'
admin.site.unregister(Group)
admin.site.disable_action('delete_selected')
```

* `site_title` ブラウザのタブに表示されるタイトル（<title>）
* `site_header` ヘッダ部分に表示されるタイトル（<h1>）
* `index_title` トップページタイトル
* `unregister` 管理サイトに登録済みのモデルを解除する
* `disable_action` 指定したアクションを使用不可にする（`delete_selected`は削除不可）

変更可能な値の一覧は以下の公式ドキュメントに一覧されています。

<https://docs.djangoproject.com/en/3.1/ref/contrib/admin/#adminsite-objects>

このカスタマイズで十分ではない場合には独自の`AdminSite`クラスを作ることも出来ます。

### 動かしてみよう

開発サーバーを起動して管理サイトにアクセスしましょう。  
サイト名が変わっていることを確認出来たらOKです。

```sh
python manage.py runserver
```

## 管理サイト全体のテンプレート（HTML）を変更しよう

djangoの管理サイトのテンプレートは細かいパーツに分かれています。  
そのため、好きな部分だけを選んでカスタマイズすることが可能です。

### 最も優先されるテンプレートフォルダの設定

まずは、プロジェクト共通のテンプレートフォルダを作ります。  
その上で、そのフォルダ内のテンプレートが最優先で使われるように設定します。  

djangoは同名のテンプレートがある場合、先に見つけたほうだけを使用します。  
この仕組を利用して、自作のテンプレートに置き換えます。

1. プロジェクト直下に`templates`フォルダを新規作成します。  
2. その中にさらに`admin`フォルダを新規作成します。
3. 全体設定 `pj_blog/settings.py`の`TEMPLATES`に1で作成した`templates`フォルダを指定します。

```py
'DIRS': [os.path.join(BASE_DIR, 'templates')],
```

`DIRS`に設定したテンプレートフォルダは最優先で探されます。

これで、`templates/admin`以下に同名のファイルがある場合はそれが使われます。  
無い場合にはデフォルトのファイルが使用されます。

### WELCOMEメッセージを削除してみよう

ヘッダー右側のWELCOMEは少し古い感じがするので削除して、ユーザー名だけが表示されるようにしましょう。  

管理サイト全体で共通部分のテンプレートはデフォルトでは`admin/base_site.html`というファイルです。  
なので、同名のファイルを先程作成した`templates/admin`フォルダに新規作成します。

`templates/admin/base_site.html`

```py
{% extends 'admin/base_site.html' %}

{% block welcome-msg %}
    <strong>{{ user }}</strong>
{% endblock %}
```

※ テンプレートの文法が分からないという方は第2回を参照して下さい。  

この自作のテンプレートではまず1行目の`extends`でデフォルトの`base_site.html`をすべてコピーしてきています。  
そのうえで、`welcome-msg`というブロックの中身だけを、ユーザー名だけを表示するように上書きしています。

デフォルトのファイル名や用意されているブロックはdjangoのソースコードを見ることで知ることが出来ます。  
例えば、`welcome-msg`は`base_site.html`がさらに`extends`している`base.html`に定義されています。

<https://github.com/django/django/blob/master/django/contrib/admin/templates/admin/base.html>

※ `base.html`自作することも出来ますが、djangoのアップデート時に頻繁に更新されるため自作のテンプレートが壊れやすくなります。

### 動かしてみよう

開発サーバーを起動して管理サイトにアクセスしましょう。  
WELCOMEメッセージが削除されていたらOKです。

## 管理サイト全体のデザイン（CSS）を変更しよう

### プロジェクト共通の静的ファイルフォルダを作成しよう

まずは静的ファイルフォルダをプロジェクト直下に作成して、全体設定でそのフォルダを指定します。  

プロジェクト直下に `static` というフォルダを新規作成します。

全体設定 `pj_blog/settings.py`に以下を追記します。  
これで先程作成した`static`フォルダがdjangoから使えるようになります。

```py
STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]
```

### 全体のデザインを変えてみよう

まずは全体のデザインを変更するCSSを`static`フォルダ内に以下の内容で作成します。

`static/base_site.css`

```css
#header {
  background: #2e8bc0;
  color: white;
}

body {
  background: #b1d4e0;
}

h1 {
  color: #0c2d48;
}

#header a:link, #header a:visited {
  color: white;
}
#branding h1, #branding h1 a:visited, #branding h1 a:link {
  color: white;
}

.module h2, .module caption, .inline-group h2 {
  background: #2e8bc0;
  color: white;
}

a.section:link, a.section:visited {
  color: white;
}
```

次にこのCSSを管理サイト全体から読み込まれるように設定しましょう。

管理サイト全体のHTMLの変更は先程使用した`base_site.html`で行います。

CSSを記述するためのブロック`extrastyle`が用意されているので、この中に追記します。  
追記の場合にはブロックの先頭に`{{ block.super }}`と記述します。

`templates/admin/base_site.html`

```
{% load static %}

{% block extrastyle %}
    {{ block.super }}
    <link rel="stylesheet" type="text/css" href="{% static 'base_site.css' %}" />
{% endblock %}
```

### 動かしてみよう

開発サーバーを起動して管理サイトにアクセスしましょう。  
デザインが大きく変わってたらOKです。  

### CSSでスタイルを当てる要素をどのように見つけるか？

FirefoxやChromeには開発ツールがついています。  
調べたい要素を右クリックをして「要素を調査」というような項目をクリックすると、その要素のCSS指定子が表示されます。  
あとはそれをCSSにコピーするだけです。

## 管理サイトのURLを変更しよう

セキュリティ上も、デフォルトの`admin/`よりは変えておいた方が良いでしょう。

全体URL設定のadminのところを好きな文字列に変更します。

```py
    path('staff-admin/', admin.site.urls),
```

## 管理サイトをもう１つ追加しよう

内部で管理者が使う管理サイトとは別の管理サイトも複数追加することが出来ます。  
今回はブログサービスのダッシュボードのような一般ユーザーが使える管理サイトを追加してみましょう。

`/mypage/`というURLでブログ、カテゴリ、タグだけが編集出来る管理サイトを作ります。  
他のユーザー情報は見せたくないので非表示とします。  
そして、スタッフでなくてもログイン出来ます。

### 管理サイトクラスを作ろう

`blog/admin.py`に以下の内容を追記します。  
`pj_blog/admin.py`でも構いませんが、今回は`blog`アプリの方に追加します。

```py
from django.contrib.auth.forms import AuthenticationForm
from django.contrib.admin import AdminSite

class BlogAdminSite(AdminSite):
    site_header = 'マイページ'
    site_title = 'マイページ'
    index_title = 'ホーム'
    site_url = None
    login_form = AuthenticationForm

    def has_permission(self, request):
        return request.user.is_active


mypage_site = BlogAdminSite(name="mypage")

mypage_site.register(models.Post)
mypage_site.register(models.Tag)
mypage_site.register(models.Category)
```

デフォルトで用意されている`admin.site`と同等の`mypage_site`という管理サイトを自作しました。  
設定出来る値は先程と同様です。  
`site_url`というのを`None`に設定することで、`サイトを表示`というリンクを削除することが出来ます。

### URLを登録しよう

`blog/urls.py`のURLパターンに以下を追記します。

```py
from .admin import mypage_site

path('mypage/', mypage_site.urls),
```

### スタッフではないユーザーを管理サイトから追加しよう

スタッフではないユーザーを適当に（例: `user1`）作成してブログへの権限をすべて与えておきます。  
※ 権限システムについては次回以降解説します。

### 動かしてみよう

開発サーバーを起動して管理サイトにアクセスしましょう。  
デフォルトの管理サイトが残ったままで、`/mypage/`にアクセスすると別の管理サイトも使えるはずです。  
`user1`は`/mypage/`にはログイン出来てブログの投稿は出来ますが、`/staff-admin/`にはログインできません。
