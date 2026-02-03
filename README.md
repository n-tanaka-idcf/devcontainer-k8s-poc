# devcontainer-k8s-poc

VS Code Dev Container 上で Kind Kubernetes クラスタを立ち上げ、ツール類は Aqua で宣言的に管理するための検証用サンドボックスです。

## 前提

- Docker が利用できること
- VS Code + Dev Containers 拡張が利用できること
- リポジトリ直下に `.env` が必要（`compose.yml` の build args: `UID`/`GID` 用、git 管理外）

```console
printf "UID=$(id -u)\nGID=$(id -g)\n" > .env
```

## Dev Container を起動

- VS Code でこのリポジトリを開き、Dev Containers: Reopen in Container を実行します。
- 設定は `.devcontainer/kubernetes/devcontainer.json` を利用します。

起動後、Aqua により必要な CLI は `postCreateCommand` でインストールされます。

## ツール確認

```console
cd /workspaces/kubernetes
task environment:check
```

## Kind クラスタを作成して kubectl を使う

このリポジトリの Kind 設定は `kubernetes/kind-config.yaml`（control-plane 1 + worker 2）です。

```console
cd /workspaces/kubernetes

# クラスタ作成
kind create cluster --config kind-config.yaml

# Dev Container から kind ネットワークへ到達させる
docker network connect kind "$HOSTNAME"

# kubeconfig を Dev Container 内から使える形で書き出し
kind get kubeconfig --internal > ~/.kube/config

# 確認
kubectl get nodes
```

削除する場合:

```console
kind delete cluster
```

## 補足（管理ファイル）

- `compose.yml`: Dev Container サービス定義（リポジトリを `/workspaces` にマウント）
- `kubernetes/Dockerfile`: Dev Container のベースイメージ（Ubuntu + `vscode` ユーザー）
- `.devcontainer/kubernetes/aqua.yaml`: コンテナ内 CLI ツール定義
- `kubernetes/Taskfile.yml`: 環境チェックなどの Task 定義

## CI について

- `kubernetes/Dockerfile` を `hadolint` でチェック
- Dev Container をビルドし、`task environment:check` を実行
- GitHub Actions の workflow YAML を `actionlint` でチェック
