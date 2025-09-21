
# WSL設定

- [WSL設定](#wsl設定)
  - [1. WSL の準備](#1-wsl-の準備)
    - [WSLのインストール](#wslのインストール)
    - [WSL で systemd を有効化しておく](#wsl-で-systemd-を有効化しておく)
  - [2. WSL上でDockerのインストール](#2-wsl上でdockerのインストール)
  - [公式推奨: Docker CE を導入](#公式推奨-docker-ce-を導入)
    - [動作確認](#動作確認)



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

