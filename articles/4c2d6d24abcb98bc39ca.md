---
title: "Azureでマイクラ鯖をホストする"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["minecraft", "azure", "docker", "アドベントカレンダー2020"]
published: false
---

この記事は[Azureアドベントカレンダー]()および[愚者っとCorp.アドベントカレンダー]()の 20 日目の記事です。

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
- とりあえずマイクラをみんなでやりたいけど自分で管理もしたい
- Azure Container Instances が気になる
- Azure で何かやってみたい

そもそも自分が Docker、Azure、マイクラ鯖を運用すること自体が初めてだったので、結構初心者向けになるはずです。

## 検証環境

検証環境は以下の通りです。といっても作業は全部クラウドでやるのであまり関係ないですね。

|||
|:--:|:--:|
|Azure サブスク|Azure for Student|
|マイクラ鯖用コンテナ|[itzg/docker-minecraft-server](https://github.com/itzg/docker-minecraft-server)|
|マイクラバージョン|`1.16.4`(執筆時点の Latest)|

# Azureでマイクラ鯖をデプロイ

## 技術構成やプラン

今回の記事ですが、クラウドデベロッパーチャンネル(通称くらでべ)の[この動画](https://www.youtube.com/watch?v=-D9kfLLCZys)で紹介されている内容が元ネタだったりします。
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

色々書いてありますが、大雑把に呼んでみると以下の手順を実行することにより
マイクラ鯖をホストできることが分かります。

1. リソースグループの作成
2. Storage Account の作成
3. ファイル共有の作成(設定)
4. Container Instances の作成

ということで、これらをやっていきましょう。構成は以下の通りです。
![diagram](https://storage.googleapis.com/zenn-user-upload/4pujmjmki8v555a1y5tp63ijznjd)

## リソースグループの作成

デプロイスクリプトで言うところの下の部分に当たります。

```bash
az group create -l $ACI_RES_LOC -g $ACI_RES_GRP -o tsv --query "properties.provisioningState"
```

## Storage Accoutの作成と設定

デプロイスクリプトで言うところの下記の部分に該当します。

```bash
az storage account create -g $ACI_RES_GRP -n $ACI_STR_AN -l $ACI_RES_LOC --sku Premium_LRS --kind FileStorage -o tsv --query "provisioningState"
```

```bash
az storage share create -n $ACI_STR_SH_NAME --account-name $ACI_STR_AN -o tsv --query "created"
```

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

# おわりに

素人なりに、ひととおり Azure Container Instances にマイクラ鯖をデプロイしてホストするまでをやりました。
ACI 自体が一時的なタスクを実行するシナリオを想定されていそうなサービスなので、
がっつり運用するなら Azure VM 使ったほうがよさそうですね。価格とかも変わってくるんだろうか。

最後までご覧いただきありがとうございました。
何かのご参考になったら幸いです。
