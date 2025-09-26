
# WSL設定

- [WSL設定](#wsl設定)
  - [1. WSL の準備](#1-wsl-の準備)
    - [WSLのインストール](#wslのインストール)
    - [WSL で systemd を有効化しておく](#wsl-で-systemd-を有効化しておく)
  - [2. WSL上でDockerのインストール](#2-wsl上でdockerのインストール)
  - [公式推奨: Docker CE を導入](#公式推奨-docker-ce-を導入)
    - [動作確認](#動作確認)
  - [WSL構築時トラブルシューティング手順](#wsl構築時トラブルシューティング手順)
    - [LANネットワークを使えない場合](#lanネットワークを使えない場合)
    - [解決策](#解決策)



## 1. WSL の準備

ref. https://code.visualstudio.com/docs/remote/wsl

まず、WSLが有効になっているか確認します。

### WSLのインストール

コマンドプロンプトで以下を実行：

```cmd
wsl.exe --list --online
```

```cmd
wsl.exe --install Ubuntu-22.04
```

初回セットアップでユーザー名とパスワードを設定します。


### WSL で systemd を有効化しておく

``` bash
cat /etc/wsl.conf
[boot]
systemd=true
```

となっていない場合は記載追加

``` bash
# WSL 内（Ubuntu）で
sudo bash -c 'cat >/etc/wsl.conf' <<'EOF'
[boot]
systemd=true
EOF

# Windows 側で WSL を落として再起動
wsl.exe --shutdown
# その後、また Ubuntu を起動してログイン
```



## 2. WSL上でDockerのインストール

Ubuntuを想定しています。




## 公式推奨: Docker CE を導入

``` bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ca-certificates curl gnupg lsb-release

### GPG キー登録
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# リポジトリ追加
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker Engine インストール
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# ユーザーを docker グループに追加
sudo usermod -aG docker $USER
newgrp docker   # 現シェルに反映
```



### 動作確認

``` bash
docker version
docker info
docker run --rm hello-world
```


## WSL構築時トラブルシューティング手順

### LANネットワークを使えない場合

物理LANが 172.17.0.0/16 を使用している場合
WSL内で Docker を導入すると、デフォルトの docker0 も 172.17.0.0/16 を割り当てる
その結果、WSL 内から外部DBへ接続できず No route to host が発生する

### 解決策

手動ルート追加

1. IP確認

    ``` bash
    ip route | grep ^default
    default via 172.24.112.1 dev eth0 proto kernel
    ```

    ここでは`172.24.112.1`が割り当てられていると想定

1. WSL セッションを開いたら、外部用のルートを明示的に設定する。
  172.17.0.3につなぎたい場合

   ```bash
   sudo ip route add 172.17.0.3/32 via 172.24.112.1 dev eth0
   ```

1. 動作確認

    ```bash
    ip route get 172.17.0.3
    nc -vz 172.17.0.3 3306
    ```

   - `Connection to 172.17.0.3 3306 port [tcp/mysql] succeeded!` と表示されればOK。

1. 恒久化（自動化）
   /etc/wsl.conf で自動実行
   WSL起動時に自動でルートを設定する。

    ```bash
    [boot]
    command="ip route add 172.17.0.3/32 via 172.24.112.1 dev eth0"
    ```

    適用するには：

    ```bash
    wsl --shutdown
    ```

    → 再起動後に反映。



