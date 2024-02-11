# dynamodb-sandbox

## dynamodbがなにかざっくり知る

- [Amazon DynamoDB Deep Dive | AWS Summit Tokyo 2019 - YouTube](https://www.youtube.com/watch?v=16RYHfe89WY)
	- わかりやすかった。
- [Core components of Amazon DynamoDB - Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html)
	- 動画のあとに読んだらいい感じ。

## ローカル用のaws configファイルを参照するように設定

```sh
# ローカルのaws config/credentialsを参照するように環境変数を設定
cp .envrc.sample .envrc
direnv allow
```

- [共有 config ファイルと 共有 credentials ファイルの場所 - AWSSDK とツール](https://docs.aws.amazon.com/ja_jp/sdkref/latest/guide/file-location.html)
  - configファイルのパスを変更できるよ。
- [[アップデート] AWSのサービスエンドポイントを表す環境変数とプロファイル設定が増えました | DevelopersIO](https://dev.classmethod.jp/articles/aws-endpoint-url-environment-varable-is-supported-on-sdks/)
  - endpointもconfigファイルに記載できるよ。

## セットアップ

- [Deploying DynamoDB locally on your computer - Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html)
  - とりあえず、dockerを使う方式で。

```sh
docker compose up -d
aws dynamodb list-tables
```

## テーブル作成

[ステップ 1: テーブルを作成します - Amazon DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/getting-started-step-1.html)

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

[ステップ 2: コンソールまたは AWS CLI を使用して、テーブルにデータを書き込みます - Amazon DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/getting-started-step-2.html)

```sh
aws dynamodb put-item \
    --table-name Music  \
    --item \
        '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}, "Awards": {"N": "1"}}'

aws dynamodb put-item \
    --table-name Music  \
    --item \
        '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Howdy"}, "AlbumTitle": {"S": "Somewhat Famous"}, "Awards": {"N": "2"}}'

aws dynamodb put-item \
    --table-name Music \
    --item \
        '{"Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}, "AlbumTitle": {"S": "Songs About Life"}, "Awards": {"N": "10"}}'

aws dynamodb put-item \
    --table-name Music \
    --item \
        '{"Artist": {"S": "Acme Band"}, "SongTitle": {"S": "PartiQL Rocks"}, "AlbumTitle": {"S": "Another Album Title"}, "Awards": {"N": "8"}}'
```

## データ取得

[ステップ 3: テーブルからデータを読み込みます - Amazon DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/getting-started-step-3.html)

```sh
aws dynamodb get-item --consistent-read \
    --table-name Music \
    --key '{ "Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}'
```

## データ更新

[ステップ 4: テーブルのデータを更新します - Amazon DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/getting-started-step-4.html)

```sh
aws dynamodb update-item \
    --table-name Music \
    --key '{ "Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}' \
    --update-expression "SET AlbumTitle = :newval" \
    --expression-attribute-values '{":newval":{"S":"Updated Album Title"}}' \
    --return-values ALL_NEW
```

## クエリ

[ステップ 5: テーブルのデータをクエリします - Amazon DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/getting-started-step-5.html)

```sh
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :name" \
    --expression-attribute-values  '{":name":{"S":"Acme Band"}}'

aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :x and begins_with(SongTitle, :name)" \
    --expression-attribute-values  '{":name":{"S":"H"}, ":x": {"S": "Acme Band"}}'
```

## GSI

[ステップ 6: グローバルセカンダリインデックスを作成します - Amazon DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/getting-started-step-6.html)

```sh
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"AlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

## GSI クエリ

[ステップ 7: グローバルセカンダリインデックスをクエリします - Amazon DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/getting-started-step-7.html)

```sh
aws dynamodb query \
    --table-name Music \
    --index-name AlbumTitle-index \
    --key-condition-expression "AlbumTitle = :name" \
    --expression-attribute-values  '{":name":{"S":"Somewhat Famous"}}'
```

## NoSQL Workbench

[項目の操作を行う｜NoSQL Workbenchで学ぶAmazon DynamoDB](https://zenn.dev/nobkovskii/books/d160fe953e0121/viewer/84ffcc)

DynamoDB localを動かしながら、connection localを選択する。

## 設計

- [[AWS Black Belt Online Seminar] Amazon DynamoDB Advanced Design Pattern 資料及び QA 公開 | Amazon Web Services ブログ](https://aws.amazon.com/jp/blogs/news/webinar-bb-dynamodb-advanced-design-pattern-2018/)
  - 動画で触れられてた。見れてないけど。
- [Modeling data with Amazon DynamoDB - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/dynamodb-data-modeling/welcome.html)
  - ベストプラクティスがまとまっているみたい。
  - ちなみに、これは[AWS 規範的ガイダンス](https://aws.amazon.com/jp/prescriptive-guidance/?apg-all-cards.sort-by=item.additionalFields.sortDate&apg-all-cards.sort-order=desc&awsf.apg-new-filter=*all&awsf.apg-content-type-filter=*all&awsf.apg-code-filter=*all&awsf.apg-category-filter=*all&awsf.apg-rtype-filter=*all&awsf.apg-isv-filter=*all&awsf.apg-product-filter=*all&awsf.apg-env-filter=*all&apg-all-cards.q=dynamodb&apg-all-cards.q_operator=AND&awsm.page-apg-all-cards=1)でdynamodbと検索すると出てきた。
