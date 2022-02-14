# Casper NCTL - Docker Container

[NCTL](https://github.com/casper-network/casper-node/tree/release-1.4.3/utils/nctl) は、CLIアプリケーションであり、1つ以上のCasperネットワークをローカルでコントロールする為のものです。殆どの開発者は、ブロックチェーンにデプロイする前に、小規模なテストネットワークを起動させてローカルでテストを実施したいと思うものです。

## このイメージの使い方

### 5つのノードでネットワークを起動させます

下記の通りに書いて、NCTLイメージのコンテナを作成してください。

```bash
docker run --rm -it --name mynctl -d -p 11101:11101 -p 14101:14101 -p 18101:18101 makesoftware/casper-nctl
```

`mynctl`は、コンテナの名前です。ネットワーク内の1つめのノードのポートは、ホストに公開されます。

### `nctl-*` コマンドのアクティベート

下記コマンドをbashコンソールで実行し、`nctl-*` コマンドをローカルホストでアクティベートしてください。

```bash
source nctl-activate.sh mynctl
```

Powershellターミナル内で、下記を実行してください。

```bash
. .\nctl-activate.ps1 mynctl
```

`mynctl` は、コンテナの名前です。

ホストマシーンにて`nctl-view-faucet-account`, `nctl-transfer-native` などのコマンドを実行できるようになりました。全てのコマンドリストは、[こちら](https://github.com/casper-network/casper-node/blob/release-1.4.3/utils/nctl/docs/commands.md)です。

たまに、faucetやノード、もしくは事前に定義されたユーザーの秘密鍵が必要となることがありますが、`nctl-*` コマンドをアクティベート後、下記コマンドを実行し取得できます。

```bash
nctl-view-faucet-secret-key
```

```bash
nctl-view-node-secret-key node=1
```

```bash
nctl-view-user-secret-key user=3
```

### 事前に定義したアカウント鍵でコンテナを実行する方法

コンテナを起動する度に、NCTLをランダムに生成したアカウント鍵にて実行させます。下記の追加パラメーターをコンテナで実行させ、事前に定義、および生成されたアカウント鍵のセットを使用します。

```bash
docker run --rm -it --name mynctl -d -p 11101:11101 makesoftware/casper-nctl /bin/bash -c "/home/casper/restart-with-predefined-accounts.sh"
```

### コンテナの停止

コンテナを停止する際は、下記を実行してください。

```bash
docker stop mynctl
```

### コンテナのシェルへのアクセス

docker exec コマンドは、Dockerコンテナ内でのコマンド実行を可能とします。下記コマンドラインによって、NCTLコンテナ内にてbashシェルの利用が可能になります。

```bash
docker exec -it mynctl bash
```

コンテナのシェルでは、`casper-client` ツールや`nctl-*`のコマンド一式を使えます。


## Dockerイメージを使ってGitHubでできること

一つの例として、GitHub ActionにてNCTLをコンテナサービスとして起動させ、インテグレーションテストを行います。

```yaml
name: CI

on:
  # Actionsタブからマニュアルにて、このワークフローを実行できます。
  workflow_dispatch:

jobs:
  integration-test:
    # ジョブが実行されるランナータイプ
    runs-on: ubuntu-latest
    
    # `runner-job`で実行するサービスコンテナ
    services:
      # サービスコンテナへのアクセスに使用されるラベル
      casper-nctl:
        # Docker Hub image
        image: makesoftware/casper-nctl:latest
        options: --name casper-nctl
        ports:
          # Opens RPC, REST and events ports on the host and service container
          - 11101:11101
          - 14101:14101
          - 18101:18101
          
    steps:
      - uses: actions/checkout@v2
      - name: Setup .NET
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: 5.0.x
      - name: Obtain faucet secret key from container
        run: docker exec -t casper-nctl cat /home/casper/casper-node/utils/nctl/assets/net-1/faucet/secret_key.pem > Casper.Network.SDK.Test/TestData/faucetact.pem
      - name: Restore dependencies
        run: dotnet restore
      - name: Build
        run: dotnet build --no-restore
      - name: Test
        run: dotnet test --no-build --verbosity normal --settings Casper.Network.SDK.Test/test.runsettings --filter="TestCategory=NCTL" 
```
