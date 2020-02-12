---
description: description
---

# django Startup Template

## 仮想環境

test

仮想環境を作成する



```text
$ python3 -m venv myvenv
```

### 仮想環境の開始

```text
$ source myvenv/bin/activate
```

Djanogのインストール

pipを最新版にする

```text
(myvenv) ~$ python3 -m pip install --upgrade pip
```

## requirementsファイルによってパッケージをインストールする

requirements.txtを作成する

```text
django-template
├── myvenv
│   └── ...
└───requirements.txt
```

requirements.txt

```text
Django~=2.2.4
```

Djangoをインストールする

```text
(myvenv) ~$ pip3 install -r requirements.txt
```

## プロジェクトを作成する

```text
(myvenv) ~$ django-admin startproject mysite .
```

## 設定変更

mysite/settings.pyに変更を加える

mysite/settings.py

```text
ALLOWED_HOSTS = ['*']

LANGUAGE_CODE = 'ja'
TIME_ZONE = 'Asia/Tokyo'

STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```

## データベースをセットアップ

```text
(myvenv) ~$ python3 manage.py migrate
```

## Webサーバーを起動する

```text
(myvenv) ~$ python3 manage.py runserver
```

URLにアクセスすると、Webページが表示されます。  
[http://127.0.0.1:8000/](http://127.0.0.1:8000/)

Webサーバーを停止するには、Ctrl + Cを同時に押すと停止します。

![Django](.gitbook/assets/first.png)

## 新しいアプリケーションの作成

```text
(myvenv) ~$ python3 manage.py startapp blog
```

```text
├── blog
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── db.sqlite3
├── manage.py
├── mysite
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── myvenv
│   └── ...
└── requirements.txt
```

Djangoにアプリケーションを使えるように設定する

mysite/settings.py

```text
# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog.apps.BlogConfig',
]
```

## モデルの作成

blog/models.py

```text
from django.conf import settings
from django.db import models
from django.utils import timezone


class Post(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(default=timezone.now)
    published_date = models.DateTimeField(blank=True, null=True)

    def publish(self):
        self.published_date = timezone.now()
        self.save()

    def __str__(self):
        return self.title
```

## データベースにモデルのためのテーブルを作成する

```text
(myvenv) ~$ python3 manage.py makemigrations blog
(myvenv) ~$ python3 manage.py migrate blog
```

## Django Admin

モデルをAdminページ\(管理画面\)上で見えるようにします。

blog/admin.py

```text
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

管理ユーザー作成

```text
(myvenv) ~$ python3 manage.py createsuperuser
```

ユーザー名、メールアドレス、パスワードを入力します。  
パスワードは見えないので、間違えずに入力して下さい。

Webサーバー開始

```text
(myvenv) ~$ python3 manage.py runserver
```

ユーザー名とパスワードを入力すると、ダッシュボードが見れます。

![Admin](.gitbook/assets/admin.png)

PostsをクリックしてPOSTを追加ボタンで、記事を追加する。

![Post](.gitbook/assets/post.png)

## URL追加

mysite/urls.py

```text
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
]
```

## blogのURL追加

urls.pyファイルを作成

blog/urls.py

```text
from django.urls import path
from . import views

urlpatterns = [
    path('', views.post_list, name='post_list'),
]
```

## View追加

blog/views.py

```text
from django.shortcuts import render

def post_list(request):
    return render(request, 'blog/post_list.html', {})
```

## テンプレート追加

templatesフォルダとblogフォルダを追加する。

```text
blog
└───templates
    └───blog
```

作成したblogフォルダにpost\_list.htmlファイルを追加する。

blog/templates/blog/post\_list.html

```text
<html>
<body>
    <p>Hello!</p>
    <p>This is working.</p>
</body>
</html>
```

Webサーバー開始

```text
(myvenv) ~$ python3 manage.py runserver
```

[http://127.0.0.1:8000/](http://127.0.0.1:8000/)

ページが表示されました。

## テンプレート内の動的データ

blog/views.py

```text
from django.shortcuts import render
from django.utils import timezone
from .models import Post

def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})
```

## Djangoテンプレート

blog/templates/blog/post\_list.html

```text
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Django Startup Template</title>
</head>
<body>
  <div>
    <h1><a href="/">Django Startup Template</a></h1>
  </div>

  {% for post in posts %}
    <div>
      <p>published: {{ post.published_date }}</p>
      <h2><a href="">{{ post.title }}</a></h2>
      <p>{{ post.text|linebreaksbr }}</p>
    </div>
  {% endfor %}
</body>
</html>
```

管理サイトでPostをPublishします。

Published dataを追記します。

![Post](.gitbook/assets/publish.png)

Webサーバー開始

```text
(myvenv) ~$ python3 manage.py runserver
```

[http://127.0.0.1:8000/](http://127.0.0.1:8000/)

投稿した内容が表示されます。

![Post](.gitbook/assets/hello.png)

