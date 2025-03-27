ステップ 1: 必要なツールのインストール
VS Codeのインストール

VS Codeのインストール

Dockerのインストール

Dockerのインストール

Gitのインストール

Gitのインストール

MySQLのインストール

MySQLのインストール

Pythonのインストール

Pythonのインストール

Hugging Faceライブラリのインストール

bash
コードをコピーする
pip install transformers
ステップ 2: プロジェクトの準備
GitHubでリポジトリを作成

GitHubで新しいリポジトリを作成

VS Codeで新しいプロジェクトを作成

プロジェクトディレクトリを作成し、VS Codeで開きます。

bash
コードをコピーする
mkdir ai-chatbot
cd ai-chatbot
code .
ステップ 3: Djangoのセットアップ
Djangoプロジェクトの作成

bash
コードをコピーする
pip install django
django-admin startproject chatbot
cd chatbot
MySQLの設定

MySQLに接続し、データベースを作成します。

sql
コードをコピーする
CREATE DATABASE chatbot_db;
DjangoのMySQL設定 chatbot/settings.pyを編集し、MySQLを使用するように設定します。

python
コードをコピーする
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'chatbot_db',
        'USER': 'root',
        'PASSWORD': 'your_password',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
Djangoマイグレーションの実行

bash
コードをコピーする
python manage.py migrate
ステップ 4: AIモデルの準備 (Hugging Face)
Hugging Faceモデルの使用

Hugging Faceのトランスフォーマーモデルを使って自然言語処理を行います。例えば、DistilGPT-2を使います。

python
コードをコピーする
from transformers import pipeline
chatbot = pipeline("text-generation", model="distilgpt2")
AIチャットボット機能の実装

ユーザーの入力に基づいてAIが応答を生成するビューを作成します。

python
コードをコピーする
from django.http import JsonResponse
from transformers import pipeline

chatbot = pipeline("text-generation", model="distilgpt2")

def chat_view(request):
    user_message = request.GET.get('message', '')
    if user_message:
        response = chatbot(user_message)
        return JsonResponse({'response': response[0]['generated_text']})
    return JsonResponse({'response': 'Hello! How can I help you?'})
urls.pyにルートを設定

python
コードをコピーする
from django.urls import path
from .views import chat_view

urlpatterns = [
    path('chat/', chat_view, name='chat'),
]
ステップ 5: Docker環境の構築
Dockerfileの作成 プロジェクトルートにDockerfileを作成します。

dockerfile
コードをコピーする
# 使用するベースイメージ
FROM python:3.9

# 作業ディレクトリを設定
WORKDIR /app

# 必要なパッケージをインストール
COPY requirements.txt .
RUN pip install -r requirements.txt

# ソースコードをコピー
COPY . .

# ポートを開放
EXPOSE 8000

# Djangoアプリを起動
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
docker-compose.ymlの作成 Docker ComposeでMySQLとDjangoを連携させます。

yaml
コードをコピーする
version: '3'
services:
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: chatbot_db
    ports:
      - "3306:3306"
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
コンテナの起動

bash
コードをコピーする
docker-compose up --build
ステップ 6: GitHubリポジトリにプッシュ
Gitの設定

プロジェクトのルートで以下を実行します。

bash
コードをコピーする
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/yourusername/yourrepo.git
git push -u origin master
ステップ 7: 完成とテスト
ブラウザでテスト

http://localhost:8000/chat/?message=Helloにアクセスして、AIチャットボットが動作することを確認します。

