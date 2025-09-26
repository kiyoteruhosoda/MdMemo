# QuickStart

VSCODEを起動しターミナルに接続

```ps
wsl
```


```vscode
>WSL: Connect to WSL
```

```bash
prj="<myproject>"
```


```bash
cd ~/project
mkdir $prj && cd $prj

cat <<'EOF' > Dockerfile
FROM ubuntu:22.04

# 基本ツール
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    curl ca-certificates gnupg lsb-release \
    python3 python3-venv python3-pip \
    openjdk-21-jdk \
 && rm -rf /var/lib/apt/lists/*

# NodeSource 公式リポジトリで Node.js をインストール（LTS/Current はURLで切替）
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - \
 && apt-get install -y nodejs \
 && rm -rf /var/lib/apt/lists/*

# バージョン確認（キャッシュ固定の目印、任意）
RUN node -v && npm -v && java -version && python3 --version

WORKDIR /workspace
EOF

cat <<'EOF' > .dockerignore
.git
.gitignore
.env
*.pem
*.key
*.crt
.mvn/

# ビルド生成物
target/
build/
dist/
node_modules/
.vscode/
.venv/
__pycache__/
.DS_Store
EOF

mkdir -p .devcontainer
cat <<'JSON' > .devcontainer/devcontainer.json
{
  "name": "python-dev",
  "build": { "dockerfile": "../Dockerfile" },
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind",
  "workspaceFolder": "/workspace",
  "customizations": {
    "vscode": {
      "settings": {
        "python.defaultInterpreterPath": "/workspace/.venv/bin/python",
        "python.terminal.activateEnvironment": true
      },
      "extensions": [
        "ms-python.python",
        "ms-toolsai.jupyter"
      ]
    }
  },
  "postCreateCommand": "python3 -m venv .venv && . .venv/bin/activate && python -V"
}
JSON

cat <<'EOF' > $prj.code-workspace
{
  "folders": [
    {
      "path": "."
    }
  ],
  "settings": {
    // ワークスペース専用設定
    "files.exclude": {
      "**/__pycache__": true,
      "**/.venv": true,
      "**/.mypy_cache": true
    }
  },
  "extensions": {
    "recommendations": [
      "ms-python.python",
      "ms-toolsai.jupyter",
      "ms-azuretools.vscode-docker"
    ]
  }
}
EOF

```

起動

```bash
docker build -t $prj .
docker run -it --rm -v "$(pwd)":/workspace $prj bash
```

>デーモン起動して後から入る場合
>```bash
>docker run -d --name "$prj" -v "$(pwd)":/workspace "$prj" sleep infinity
>docker exec -it "$prj" bash
>```
>



既存の実行中コンテナにVSで接続

VSCode で $prj.code-workspaceを開く

> 手動なら
> `Dev Containers: Attach to Running Container...`
> コマンドを利用

作業が終わったらターミナルで`exit`

>デーモンの場合：
>```bash
>docker stop "$prj"
>docker rm "$prj"
>```
