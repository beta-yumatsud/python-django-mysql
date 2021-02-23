# python-django-mysql
Djangoのちょい学習。MySQLを使いつつ、docker composeで動かす。

# やったことメモ
* Dockerfileの作成
* requirements.txtの用意
    * Django
    * PyMySQL
* docker-compose.yml
* Djangoプロジェクトの作成
```
$ docker-compose run django django-admin.py startproject study .
```
* `study/settings.py`にMySQLの設定を記載
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'django_db',
        'USER': 'django_user',
        'PASSWORD': 'django_pass',
        'HOST': 'mysql',
        'PORT': '3306'
    }
}
```
* `manage.py` にMySQLの設定を追加
```python
import pymysql
pymysql.install_as_MySQLdb()
```
* 起動
```
$ docker-compose up -d
```
* `http://localhost:8000/` でアクセス
* DBマイグレーション
```
$ docker exec -it django.web /bin/bash
root@418eb811bba8:/code# python3 manage.py migrate
```
* admin用スーパーユーザの作成
```
$ docker exec -it django.web /bin/bash                       
root@418eb811bba8:/code# python3 manage.py createsuperuser
ユーザー名 (leave blank to use 'root'): admin
メールアドレス: admin@example.com
Password: hogefuga
Password (again): hogafuga
Superuser created successfully.
```
* アプリケーションの作成
```
$ docker exec -it django.web /bin/bash                       
root@418eb811bba8:/code# python3 manage.py startapp polls
```
* `views.py` にviewを追加し、 `polls/urls.py` で `polls/` 配下のpathを設定
* `study/urls.py` で上記の `polls.urls` を読み込むようにする
* `polls/models.py` でDBのモデルを定義
* `study/settings.py` に `polls` アプリケーションを登録する（ `INSTALLED_APPS` ）
* 作成したmodelをDBに反映
```
$ docker exec -it django.web /bin/bash                       
root@418eb811bba8:/code# python3 manage.py makemigrations polls
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice

root@418eb811bba8:/code# python3 manage.py sqlmigrate polls 0001
-- ＜migrateしようとするSQLの内容を確認できる＞

root@418eb811bba8:/code# python3 manage.py check
System check identified no issues (0 silenced).

root@418eb811bba8:/code# python3 manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK
```
* 下記コマンドでdjangoにshellとして連携できるようになる
```
root@418eb811bba8:/code# python3 manage.py shell
```
* admin上で `polls` アプリケーションを使えるようにするため `polls/admin.py` を修正
* 新しくviewを追加しつつ、DBからデータを引く部分などを追加
* テストコードの追加と実施、その際DBに接続しちゃうのでテスト用DBを用意（これってskipさせるようにできないかは要調査）
```
root@2607680ab802:/# mysql -u root -p
mysql> GRANT ALL PRIVILEGES ON test_django_db.* TO 'django_user'@'%';
Query OK, 0 rows affected (0.02 sec)

$ docker exec -it django.web /bin/bash
root@418eb811bba8:/code# 
root@418eb811bba8:/code# python3 manage.py test polls
```
* 管理画面のカスタマイズ

# 参考
* [Django](https://www.djangoproject.com/)
* [Compose と Django](https://docs.docker.jp/compose/django.html)
