# djangoチュートリアル

## 仮想環境

仮想環境を作成する

```
$ python3 -m venv myvenv
```
### 仮想環境の開始
```
$ source myvenv/bin/activate
```
Djanogのインストール

pipを最新版にする
```
(myvenv) ~$ python3 -m pip install --upgrade pip
```
## requirementsファイルによってパッケージをインストールする

requirements.txtを作成する
```
django-template
├── myvenv
│   └── ...
└───requirements.txt
```

requirements.txt
```
Django~=2.2.4
```

Djangoをインストールする
```
(myvenv) ~$ pip3 install -r requirements.txt
```

## プロジェクトを作成する
```
(myvenv) ~$ django-admin startproject mysite .
```

## 設定変更

mysite/settings.pyに変更を加える

```python:mysite/settings.py
ALLOWED_HOSTS = ['*']

LANGUAGE_CODE = 'ja'
TIME_ZONE = 'Asia/Tokyo'

STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```

## データベースをセットアップ

```
(myvenv) ~$ python3 manage.py migrate
```

## Webサーバーを起動する

```
(myvenv) ~$ python3 manage.py runserver
```
URLにアクセスすると、Webページが表示されます。  
http://127.0.0.1:8000/

Webサーバーを停止するには、Ctrl + Cを同時に押すと停止します。

![Django](img/first.png)

## 新しいアプリケーションの作成
```
(myvenv) ~$ python3 manage.py startapp blog
```
```
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

```python:mysite/settings.py
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


```python:blog/models.py
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

```
(myvenv) ~$ python3 manage.py makemigrations blog
(myvenv) ~$ python3 manage.py migrate blog
```

## Django Admin

モデルをAdminページ(管理画面)上で見えるようにします。

```python:blog/admin.py
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

管理ユーザー作成

```
(myvenv) ~$ python3 manage.py createsuperuser
```
ユーザー名、メールアドレス、パスワードを入力します。  
パスワードは見えないので、間違えずに入力して下さい。

Webサーバー開始
```
(myvenv) ~$ python3 manage.py runserver
```

ユーザー名とパスワードを入力すると、ダッシュボードが見れます。

![Admin](img/admin.png)

PostsをクリックしてPOSTを追加ボタンで、記事を追加する。

![Post](img/post.png)

