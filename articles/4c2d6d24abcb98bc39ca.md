---
title: "Azureでマイクラ鯖をホストする"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["minecraft", "azure", "docker", "アドベントカレンダー2020"]
published: true
---

この記事は[Azureアドベントカレンダー](https://qiita.com/advent-calendar/2020/azure)および[愚者っとCorp.アドベントカレンダー](https://adventar.org/calendars/5126)の 20 日目の記事です。

# はじめに

どうも、にー兄さんです。
最近 Azure にハマっているので、今日は初めて Azure に関する記事を書きます。
ド素人なのでお手柔らかにお願いします。

## この記事で得られる知見と対象読者

この記事を読んで得られる知見は以下の通りです。

- Azure Container Instances に docker コンテナをデプロイする
- docker を使ってマイクラ鯖をホストする方法
- Azure Container Instances のデータを永続化する方法

以上より、この記事の対象読者は以下の通りです。

- マイクラをプレイしたことがある
- Azure を使ったことがある、もしくはどんなものか知っている
- とりあえずマイクラをみんなでやりたいけど自分でサーバー管理もしたい
- Azure Container Instances が気になる
- Azure で何かやってみたい

そもそも自分が Docker、Azure、マイクラ鯖を運用すること自体が初めてだったので、結構初心者向けになるはずです。

## 検証環境

検証環境は以下の通りです。といっても作業は全部クラウドでやるのであまり関係ないですね。

||環境|
|:--:|:--:|
|Azure サブスク|Azure for Student|
|マイクラ鯖用コンテナ|[itzg/docker-minecraft-server](https://github.com/itzg/docker-minecraft-server)|
|マイクラバージョン|`1.16.4`(執筆時点の Latest)|

# Azureでマイクラ鯖をデプロイ

## 技術構成やプラン

今回の記事ですが、クラウドデベロッパーチャンネル(通称くらでべ)の[この動画](https://www.youtube.com/watch?v=-D9kfLLCZys)が元ネタだったりします。
@[youtube](-D9kfLLCZys)
ということでまずはこの動画について少し説明します。

動画では、りおさんというつよつよエンジニアの方が作った[こちらのデプロイスクリプト](https://github.com/rioriost/deploy_minecraft/blob/master/create_minecraft.sh)を使ってコンテナのデプロイを行っていました。
実際このスクリプトを動かせば今回紹介する内容はすべて実現できるわけですが、正直何が起こってるのかよくわからず実行するのが怖いです。
ということで、スクリプトから作業手順を読み解き、Azure Portal で手を動かしながらやってみようと思いました。

スクリプトの主要部分を引用します。

```sh: create_minecraft.sh
# ...

echo "Creating Resource Group..."
res=$(az group create -l $ACI_RES_LOC -g $ACI_RES_GRP -o tsv --query "properties.provisioningState")
if [ "$res" != "Succeeded" ]; then
	exit
fi

# Create the storage account with the parameters
echo "Creating Storage Account..."
res=$(az storage account create -g $ACI_RES_GRP -n $ACI_STR_AN -l $ACI_RES_LOC --sku Premium_LRS --kind FileStorage -o tsv --query "provisioningState")
if [ "$res" != "Succeeded" ]; then
	az group delete --yes --no-wait -g $ACI_RES_GRP
	exit
fi

# Create the file share
echo "Creating Storage Share..."
res=$(az storage share create -n $ACI_STR_SH_NAME --account-name $ACI_STR_AN -o tsv --query "created")
if [ "$res" != "true" ]; then
	az group delete --yes --no-wait -g $ACI_RES_GRP
	exit
fi
STORAGE_KEY=$(az storage account keys list -g $ACI_RES_GRP --account-name $ACI_STR_AN --query "[0].value" -o tsv)

# Create the container  
echo "Creating Container..."
res=$(az container create --image rioriost/minecraft-server -g $ACI_RES_GRP -n $ACI_CNT_NAME \
	--ip-address Public --ports 25565 25575 \
	--dns-name-label $ACI_CNT_NAME \
	--cpu 2 --memory 8 \
	-e EULA=TRUE ENABLE_RCON=true \
	RCON_PASSWORD=$RCON_PASSWORD \
	--azure-file-volume-account-name $ACI_STR_AN \
	--azure-file-volume-account-key "$STORAGE_KEY" \
	--azure-file-volume-share-name $ACI_STR_SH_NAME \
	--azure-file-volume-mount-path /data/ \
	-o tsv --query "provisioningState")

# ...
```

色々書いてありますが、大雑把に読んでみると以下の手順を実行していることが分かります。

1. リソースグループの作成
2. Azure Storage Account の作成
3. ファイル共有の作成(設定)
4. Azure Container Instances の作成

ということで、これらをやっていきましょう。構成は以下の通りです。
![diagram](https://storage.googleapis.com/zenn-user-upload/4pujmjmki8v555a1y5tp63ijznjd)

## リソースグループの作成

デプロイスクリプトで言うところの下の部分に当たります。

```bash
az group create -l $ACI_RES_LOC -g $ACI_RES_GRP -o tsv --query "properties.provisioningState"
```

まずは今回作るリソースを管理するためのリソースグループを作っていきます。
設定する項目は以下の通りです。

- リージョンの場所
- リソースグループ名

リソースの作成ボタン->「リソースグループ」を検索->作成ボタン
クリックしていき、リソースグループの名前とリージョンを設定します。
今回は東日本リージョンで、「zenn-minecraft-tutorial」というリソースグループを作ります。
![asxa](https://storage.googleapis.com/zenn-user-upload/arhcq46ryaxyd59iusfbpp9i9re8)

## Storage Accoutの作成と設定

デプロイスクリプトで言うところの下記の部分に該当します。

```bash
az storage account create -g $ACI_RES_GRP -n $ACI_STR_AN -l $ACI_RES_LOC --sku Premium_LRS --kind FileStorage -o tsv --query "provisioningState"
```

```bash
az storage share create -n $ACI_STR_SH_NAME --account-name $ACI_STR_AN -o tsv --query "created"
```

Azure Storage Account というものを作ります。Azure Storage Accout とは、Azure 上のリソースで使えるデータを保存する機構のようなものだと思います。
基本コンテナではデータは永続的に保存はできないので、コンテナを停止、再起動したときにちゃんと前回のプレイデータが復元できるようにストレージアカウントに保存しておきます。

Azure Storage Accout にはファイル共有という機能があり、それを使うことによって Container Instances でストレージをマウントします。

まずはストレージアカウントを検索して作成ボタンを押し、デプロイスクリプトに基づいて以下のように設定していきます。

![xsaxa](https://storage.googleapis.com/zenn-user-upload/yoj2gpzge2egzx7q6gu7xcn8eu4k)

設定項目を下記に記します。

|項目|値|
|:--:|:--:|
|リソースグループ|↑で作ったリソースグループを指定|
|ストレージアカウント名|zennminecraft(小文字と数字しか使えないので注意)|
|場所|東日本|
|パフォーマンス|Standard|
|アカウントの種類|汎用 v2|
|レプリケーション|ローカル冗長ストレージ(LRS)|

表にあるように、基本タブ以外はとくにいじらなくて大丈夫です。このまま確認とリソースの作成をしてしまいましょう。
リソースが作成出来たら、ファイル共有の設定をしていきます。
サイドバーにある「ファイル共有」という項目をクリックし、「＋ファイル共有」から適当な名前で作成します。
![あｓぁｓぁ](https://storage.googleapis.com/zenn-user-upload/gvxmmhffewrswcbes4b4pui8qcaq)
今回は「zenn-minecraft-file-share」という名前にしました。
![asdf](https://storage.googleapis.com/zenn-user-upload/pn4m1ng6byf47py63k2qzkekfmo3)


## Container Instancesを使ったコンテナのデプロイ

デプロイスクリプトの下記部分の処理をしていきます。

```bash
az container create --image rioriost/minecraft-server -g $ACI_RES_GRP -n $ACI_CNT_NAME \
	--ip-address Public --ports 25565 25575 \
	--dns-name-label $ACI_CNT_NAME \
	--cpu 2 --memory 8 \
	-e EULA=TRUE ENABLE_RCON=true \
	RCON_PASSWORD=$RCON_PASSWORD \
	--azure-file-volume-account-name $ACI_STR_AN \
	--azure-file-volume-account-key "$STORAGE_KEY" \
	--azure-file-volume-share-name $ACI_STR_SH_NAME \
	--azure-file-volume-mount-path /data/ \
	-o tsv --query "provisioningState"
```

デプロイスクリプトでは、docker イメージ、リソースグループ、コンテナ名、ip、解放ポート、dns ラベル、
リソースのスペック、環境変数、ボリュームのマウントなどを設定しています。
基本的にはこのスクリプトに沿って行くのですが、自分が調べた結果いくつか変更したい箇所があります。

### マイクラ鯖のdockerイメージ

マイクラ鯖用の docker イメージで一番有名なのは、[itzgさんという方のdocker-minecraft-server](https://github.com/itzg/docker-minecraft-server)というリポジトリのモノっぽいです。
https://github.com/itzg/docker-minecraft-server
今回のデプロイスクリプトではりおさんが作られた docker イメージを指定されていますが、こちらのイメージも実は itzg さんの docker イメージをフォークして作られたものでした。
https://github.com/rioriost/deploy_minecraft
りおさんの docker イメージは 2 年前くらいに変更されたものらしく、その後に fork 元のイメージではいろいろ変更が加わっていました。
その結果、環境変数を docker 起動時に変更すればマイクラ鯖の設定を柔軟にいじれるようになったみたいです。
りおさんのイメージではマイクラ鯖のメモリ上限を dockerfile を変更して設定していましたが、普通に環境変数によってそれが可能になっています。
ということで、りおさんの docker イメージではなく itzg さんのイメージを利用することにしました。

### リソースのスペック

スクリプトでは 2 コアの vCPU と 8GB のメモリを指定していますが、まぁそこまでいらないかなぁと思ったので、
1 コア vCPU と 2GB メモリでやってみます。多分大丈夫。

### Container Instances の作成

さていよいよ最後のタスクである Container Instances の作成になります。
他のリソースと同じように Container Instances も Azure Portal から作成できるのですが、今回は Storage Account のボリュームをマウントする必要があるため、普通にできなかっかったです。
(少なくとも自分はやり方を調べても出てきませんでした。もし知ってる方がいらっしゃったら教えてほしいです)

ではどうするのかというと、Resource Manager テンプレートを利用します。
Resource Manager テンプレートとは、リソースに必要な設定を JSON で記述し、それをもとにリソースを作成するシステムのことです。
詳しくは[こちら](https://azure.microsoft.com/ja-jp/services/arm-templates/)をご覧ください。
https://azure.microsoft.com/ja-jp/services/arm-templates/
Container Instances のドキュメントに[ストレージアカウントのボリュームをマウントするときのサンプル](https://docs.microsoft.com/ja-jp/azure/container-instances/container-instances-volume-azure-files#deploy-container-and-mount-volume---resource-manager)が載っているので、そちらを参考に JSON ファイルを作成します。
https://docs.microsoft.com/ja-jp/azure/container-instances/container-instances-volume-azure-files#deploy-container-and-mount-volume---resource-manager

今回作成する Container Instances の設定項目を下記にまとめました。

|項目|設定した値|
|:---:|:---:|
|イメージ|itzg/docker-minecraft-server|
|リソースグループ|さっき作った zenn-minecraft-tutorial|
|コンテナの名前|zenn-minecraft-container|
|IP の設定|Public|
|Port|25565, 25575|
|DNS ラベル|zenn-minecraft-tutorial|
|CPU とメモリ|1 コア vCPU と 2GB メモリ|
|環境変数|いろいろあるので詳細後述|
|ボリュームのマウント設定||
|ファイル共有|さっき作った zenn-miencraft-file-share|
|ストレージアカウント名|さっき作った zennminecraft|
|ストレージアカウントキー|ストレージアカウントのアクセスキー|

ストレージアカウントのアクセスキーは zennminecraft ストレージアカウントの画面サイドバーにある「アクセスキー」というやつの中にある Key1 のキーをコピーしましょう。

Docker コンテナを起動するときの環境変数を下記に記します。
環境変数に関しては、itzg/docker-minecraft-server の GitHub にフルドキュメントがあるみたいなので参考になさってください。

|name|value|備考|
|:--:|:--:|:--|
|EULA|TRUE|マイクラの利用規約的なもの|
|ENABLE_RCON|true|外部からコマンドを使えるようにするやつ|
|RCON_PASSWORD|自分で設定||
|MAX_MEMORY|1G|マイクラが使用するメモリ上限|
|VERSION|お好きなバージョンで|執筆当時は 1.16.4 が最新。Latest で最新バージョンを指定できます。|
|SEED|お好きなシード値||

メモリの上限は、マイクラ鯖以外のプロセスも走ることを考えて 1G にしました。
上限指定しないと結構鯖が不安定になるので、設定しておきましょう。

さて、これらを Resource Manager テンプレートで設定してみましょう。
まずはリソースの作成->「テンプレート」で検索->「テンプレートのデプロイ」をクリック
そうすると、以下のような画面がでるので「エディターで独自のテンプレートを作成する」をクリックします。
![ahaha](https://storage.googleapis.com/zenn-user-upload/8qrb875gf7nrsb7vmbfhl5a73j5i)

するとテキストエディタが開くので、その中に JSON を記述していきます。
その結果を以下に記します。

:::details JSONテンプレート
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "containerGroups_name": {
            "defaultValue": "zenn-minecraft-container",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.ContainerInstance/containerGroups",
            "apiVersion": "2019-12-01",
            "name": "[parameters('containerGroups_name')]",
            "location": "japaneast",
            "properties": {
                "sku": "Standard",
                "containers": [
                    {
                        "name": "[parameters('containerGroups_name')]",
                        "properties": {
                            "image": "itzg/minecraft-server",
                            "ports": [
                                {
                                    "port": 25565
                                },
                                {
                                    "port": 25575
                                }
                            ],
                            "environmentVariables": [
                                {
                                    "name": "EULA",
                                    "value": "TRUE"
                                },
                                {
                                    "name": "ENABLE_RCON",
                                    "value": "true"
                                },
                                {
                                    "name": "RCON_PASSWORD",
                                    "value": "変更してください"
                                },
                                {
                                    "name": "MEMORY",
                                    "value": "1G"
                                },
                                {
                                    "name": "MAX_MEMORY",
                                    "value": "1G"
                                },
                                {
                                    "name": "VERSION",
                                    "value": "1.16.4"
                                },
                                {
                                    "name": "SEED",
                                    "value": "0123456789"
                                }
                            ],
                            "resources": {
                                "requests": {
                                    "memoryInGB": 2,
                                    "cpu": 1
                                }
                            },
                            "volumeMounts": [
                                {
                                    "name": "filesharevolume",
                                    "mountPath": "/data"
                                }
                            ]
                        }
                    }
                ],
                "initContainers": [],
                "ipAddress": {
                    "ports": [
                        {
                            "protocol": "TCP",
                            "port": 25565
                        },
                        {
                            "protocol": "TCP",
                            "port": 25575
                        }
                    ],
                    "type": "Public",
                    "dnsNameLabel": "zenn-minecraft-tutorial"
                },
                "osType": "Linux",
                "volumes": [
                    {
                        "name": "filesharevolume",
                        "azureFile": {
                            "shareName": "zenn-minecraft-file-share",
														"storageAccountName": "zennminecraft",
														"storageAccountKey": "変更してください"
                        }
                    }
                ]
            }
        }
    ]
}
```
:::

保存をするとこのような画面になるので、リソースグループを指定して、作成しましょう。
![asxa](https://storage.googleapis.com/zenn-user-upload/ooput9o0owa7d6571xs0ll50vqve)

デプロイが完了したら、コンテナのページに行って、URL をコピーします。

![asxasx](https://storage.googleapis.com/zenn-user-upload/kjtpz412wcpkhrfohd6d5oilacxs)

そしてこの URL でマイクラ鯖にインしてみると......。

![mine](https://storage.googleapis.com/zenn-user-upload/jc2sm47iw9b8wiy7cpdjowqgiqsd)

やった～～。




# おわりに

素人なりに、ひととおり Azure Container Instances にマイクラ鯖をデプロイしてホストするまでをやりました。
ACI 自体が一時的なタスクを実行するシナリオを想定されていそうなサービスなので、
がっつり運用するなら Azure VM 使ったほうがよさそうですね。価格とかも変わってくるんだろうか。

最後までご覧いただきありがとうございました。
何かのご参考になったら幸いです。
