# OpenWebUI-Keycloak

## 目的

Open WebUI に Keycloak を使った OAuth(OIDC) 認証を導入して、動作確認を行う最小構成のサンプルプロジェクトです。

## 動作環境

Windows11 + WSL2 (Ubuntu 22.04)

WSL2で起動し、Windows側のブラウザからOpen WebUIとKeycloakにアクセスします。
Macなどでの動作確認はしていません。

## 起動手順

### 1. コンテナ起動

Ollama、Keycloak、Open WebUI の各コンテナを起動します。
起動までに数分かかる場合があります。

```shell
cp .env.example .env
docker compose up -d
```

### 2. Ollama モデルダウンロード

使うモデルは何でもOKですが、ここでは `gemma:2b` を例にします。

```shell
docker exec -it ollama ollama pull gemma:2b
```

### 3. Keycloak セットアップ

1. Realm 作成
    1. ブラウザで `http://keycloak.localtest.me:8080` にアクセス
    2. 管理者権限でログイン (admin / admin)
    3. 左上ハンバーガーメニューから `Manage realms` > `Create realm` を選択
    4. Realm名に `open-webui` と入力
    5. **Create** をクリック
2. OAuth Client 作成
    1. 左メニューから **Clients** → **Create client**
    2. 入力する項目は以下の通り：
        - Client Type: `OpenID Connect`
        - Client ID: `open-webui`
        - Client Authentication: `ON`
        - Root URL: `http://localhost:8888/`
        - Valid redirect URIs: `http://localhost:8888/oauth/oidc/callback`
    3. 最後まで進めて **Save** をクリック
    4. 作成後、**Credentials** タブから **Client Secret** をコピー
    5. `.env` ファイルを編集し、以下を追加：

   ```env
   OIDC_CLIENT_SECRET=<コピーしたClient Secret>
   ```

3. ユーザー作成
    1. 左メニューから **Users** → **Create new user**
    2. 設定する項目は以下の通り：
        - Username: `admin`
        - Email: `admin@example.com`
        - First Name: `Admin`
        - Last Name: `User`
    3. **Create**
    4. **Credentials** タブ → **Set password**
    5. Password を設定し、**Temporary** は OFF

### 4. Open WebUI にアクセス

1. `.env`の変更を反映するため、Open WebUI コンテナを再起動します。

    ```shell
    docker compose up -d --force-recreate open-webui
    ```

2. OpenWeb UIが起動後にブラウザで `http://localhost:8888` にアクセスし、ログイン画面に移動
3. `Continue with keycloak` をクリック
4. 作成した`admin`ユーザーでログイン

最初にログインしたユーザーが Open WebUI の管理者になります。
