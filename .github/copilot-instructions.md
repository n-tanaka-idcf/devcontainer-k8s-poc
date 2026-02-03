# Copilot instructions (devcontainer-k8s-poc)

## このリポジトリは何か
- これは主に **VS Code Dev Container + Kind Kubernetes の検証用サンドボックス** です。
- いわゆる「アプリ本体」のコードはなく、変更点の中心は Dev Container/ツール類と Kubernetes 立ち上げ手順です。

## 主要ファイル / 構成
- `compose.yml`: Dev Container サービス `kubernetes` を定義し、リポジトリを `/workspaces` にマウントします。
- `kubernetes/Dockerfile`: Dev Container のベースイメージ (Ubuntu + `vscode` ユーザー) です。
- `.devcontainer/kubernetes/devcontainer.json`: Dev Container の設定 (features/env/extensions 等) です。
- `.devcontainer/kubernetes/aqua.yaml`: コンテナ内で使う CLI ツールを Aqua で宣言的に管理します。
- `kubernetes/kind-config.yaml`: Kind クラスタ構成 (control-plane 1 + worker 2) です。
- `kubernetes/Taskfile.yml`: 環境/ツールのチェックを Task で再現可能にしています。

## 開発ワークフロー (コマンド)
### Dev Container を起動する (Docker Compose)
- ビルド時に `UID` と `GID` を渡すため、リポジトリ直下に `.env` が必要です (`.env` は git 管理外)。
  - ホスト側で作成:
    - `printf "UID=$(id -u)\nGID=$(id -g)\n" > .env`

### コンテナ内ツールの確認
- `/workspaces/kubernetes` で実行:
  - `task environment:check`
- ツールは `.devcontainer/kubernetes/aqua.yaml` で管理し、`.devcontainer/kubernetes/postCreateCommand.sh` で導入されます。

### Kind クラスタを作成/利用する (コンテナ内)
- `/workspaces/kubernetes` で実行:
  - `kind create cluster --config kind-config.yaml`
  - `docker network connect kind "$HOSTNAME"` (Dev Container から Kind ネットワークへ到達させる)
  - `kind get kubeconfig --internal > ~/.kube/config`
  - `kubectl get nodes`

## CI の前提 (壊さないこと)
- `.github/workflows/devcontainer_ci.yml`:
  - `kubernetes/Dockerfile` を `hadolint` でチェックします。
  - `devcontainers/ci` で Dev Container をビルドし、`task environment:check` を実行します。
- `.github/workflows/github_actions_workflow_ci.yml`:
  - workflow YAML を `actionlint` でチェックします。

## このリポジトリ特有の流儀
- ツールの追加/更新は原則 `.devcontainer/kubernetes/aqua.yaml` で行い、場当たり的な `apt-get install` は避けてください。
- 必須 CLI を追加したら `kubernetes/Taskfile.yml` の `environment:check` にも追加して、CI で検証できるようにしてください。

## エージェント向け注意
- `.env` など一部の環境ファイルは意図的に git 管理外です。生成コードがそれらの中身に依存しないようにしてください。
- 変更は Dev Container/Kubernetes 周辺の配線に寄せ、アプリ雛形の追加などスコープ外の作業は避けてください。
