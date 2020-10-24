# djangoチュートリアル #13

## 管理サイトをカスタマイズしよう！後編 〜個別モデル設定編〜

djangoの大きな特徴の１つは自動生成された管理サイトがついてくることです。  

## 完成版プロジェクト

<https://github.com/shun-rec/django-website-13>

## 事前準備

### 新規サーバーを立ち上げる

[Paiza](https://paiza.cloud)で作業している方はdjangoにチェックを入れて新規サーバーを立ち上げて下さい。  
自分のマシンで作業している方はdjangoが使える作業用フォルダを作成して下さい。

### 前回管理サイトをカスタマイズしたプロジェクトをダウンロード

ターミナルを開いて、以下を実行。

```sh
git clone https://github.com/shun-rec/django-website-12
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

## モデル一覧にフィールドを表示しよう

ブログ一覧に「タイトル」「カテゴリ」「作成日」「更新日」を表示するようにしましょう。  
`ModelAdmin`のlist_display`を使用します。

`blog/admin.py`の`PostAdmin`に以下を追加します。

```py
    list_display = ('title', 'category', 'created', 'updated')
```

`ForeignKey`フィールド`OneToOne`フィールドの場合だけ注意が必要で、加えて`list_select_related`にも追加する必要があります。  
そうしないと、N+1問題というバグが発生します。

※ N+1問題・・・DBアクセスが大量に発生し、サイトの負荷増大と速度低下を招く問題。

今回は`category`が`ForeignKey`フィールドなのでさらに以下を追加します。

```py
    list_select_related = ('category', )
```

### 動かしてみよう

開発サーバーを起動して管理サイトにアクセスしてみましょう。  

```sh
python manage.py runserver
```

ブログ一覧画面で「タイトル」「カテゴリ」「作成日」「更新日」カラムが表示されたらOKです。

## モデル一覧にManyToManyフィールド（多対多）を表示しよう

多対多のフィールドはそのままでは表示出来ません。  
今回は`tags`フィールドがその例です。  
複数あるタグをどのように表示するかを指示する必要があります。

今回はカンマ区切りで表示してみます。    
名前は何でも良いですが、`tags_summary`という名前でタグをカンマ区切りに整形するメソッドを定義します。

`blog/admin.py`の`PostAdmin`に続けて以下を追記します。

```py
    def tags_summary(self, obj):
        qs = obj.tags.all()
        label = ', '.join(map(str, qs))
        return label
```



```py
    def get_queryset(self, request):
        return super().get_queryset(request).prefetch_related('tags')
```








