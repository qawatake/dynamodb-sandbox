# dynamodb-sandbox

## セットアップ

- [Deploying DynamoDB locally on your computer - Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html)
  - とりあえず、dockerを使う方式で。

```sh
docker compose up -d
aws dynamodb list-tables --endpoint-url http://localhost:8000 --region us-east-1
```

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
    --table-class STANDARD \
    --endpoint-url http://localhost:8000 --region us-east-1

aws dynamodb list-tables --endpoint-url http://localhost:8000 --region us-east-1
{
    "TableNames": [
        "Music"
    ]
}
```

## データ投入

```sh
```
