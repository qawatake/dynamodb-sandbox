# dynamodb-sandbox

## セットアップ

- [Deploying DynamoDB locally on your computer - Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html)
  - とりあえず、dockerを使う方式で。

```sh
docker compose up -d
aws dynamodb list-tables
```

## ローカル用のconfigファイルの

- [共有 config ファイルと 共有 credentials ファイルの場所 - AWSSDK とツール](https://docs.aws.amazon.com/ja_jp/sdkref/latest/guide/file-location.html)
  - configファイルのパスを変更できるよ。
- [[アップデート] AWSのサービスエンドポイントを表す環境変数とプロファイル設定が増えました | DevelopersIO](https://dev.classmethod.jp/articles/aws-endpoint-url-environment-varable-is-supported-on-sdks/)
  - endpointもconfigファイルに記載できるよ。

## テーブル作成

```sh
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=5,WriteCapacityUnits=5 \
    --table-class STANDARD

aws dynamodb list-tables
{
    "TableNames": [
        "Music"
    ]
}
```

## データ投入

```sh
```
