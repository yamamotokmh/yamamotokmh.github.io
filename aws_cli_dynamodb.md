# AWS CLI で DynamoDB を操作する

## テーブルの設定の取得

### テーブルの一覧

- コマンド

```
aws dynamodb list-tables
```

- 出力結果

```
{
    "TableNames": [
        "ProductCatalog",
        "sample-table",
        "test-table"
    ]
}
```

### テーブルの設定値

- コマンド

```
aws dynamodb describe-table --table-name ProductCatalog
```

- 出力結果

```
{
    "Table": {
        "AttributeDefinitions": [
            {
                "AttributeName": "Id",
                "AttributeType": "N"
            }
        ],
        "TableName": "ProductCatalog",
        "KeySchema": [
            {
                "AttributeName": "Id",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": 1564333991.55,
        "ProvisionedThroughput": {
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 5,
            "WriteCapacityUnits": 5
        },
        "TableSizeBytes": 1075,
        "ItemCount": 8,
        "TableArn": "arn:aws:dynamodb:ap-northeast-1:xxxxxxxxxxxx:table/ProductCatalog",
        "TableId": "55e6fe03-c2cb-4044-8669-c8290650cff4"
    }
}
```

## データの取得

公式にあるサンプルデータの ProductCatalog を使用
https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/SampleData.LoadData.html

### テーブルのデータを全件取得する

```
aws dynamodb scan \
     --table-name ProductCatalog
```

### データの条件つき

商品カタログから商品区分が本であるデータを取得する

```
aws dynamodb scan \
     --table-name ProductCatalog \
     --filter-expression "ProductCategory = :sample" \
     --expression-attribute-values '{":sample":{"S":"Book"}}'
```

### データの条件つき＋表示する項目を指定

商品カタログから商品区分が本である ID を取得する

- コマンド

```
aws dynamodb scan \
     --table-name ProductCatalog \
     --filter-expression "ProductCategory = :sample" \
     --expression-attribute-values '{":sample":{"S":"Book"}}' \
     --query Items[].Id[]
```

- 出力結果

```
[
    {
        "N": "102"
    },
    {
        "N": "103"
    },
    {
        "N": "101"
    }
]
```

### データの条件つき＋表示する項目を指定(複数)＋テキスト形式で表示

商品カタログから商品区分が本であるデータの ID とタイトル、値段を取得し、テキスト形式で表示する

- コマンド

```
aws dynamodb scan \
     --table-name ProductCatalog \
     --filter-expression "ProductCategory = :sample" \
     --expression-attribute-values '{":sample":{"S":"Book"}}' \
     --query 'Items[].{id:Id,title:Title,price:Price}'\
     --output text
```

- 出力結果

```
ID      102
PRICE   20
TITLE   Book 102 Title
ID      103
PRICE   2000
TITLE   Book 103 Title
ID      101
PRICE   2
TITLE   Book 101 Title
```

### AND 条件

商品カタログから商品区分が本かつ公開フラグが true であるデータの ID と商品区分、公開フラグ を取得し、テキスト形式で表示する

- コマンド

```
aws dynamodb scan \
     --table-name ProductCatalog \
     --filter-expression "ProductCategory = :arg1 and InPublication = :arg2" \
     --expression-attribute-values '{":arg1":{"S":"Book"}, ":arg2":{"BOOL":true}}' \
     --query 'Items[].{id:Id.N,productCategory:ProductCategory.S,inPublication:InPublication.BOOL}' \
     --output text
```

- 出力結果

```
102     True    Book
101     True    Book
```

### OR 条件

商品カタログからタイトルが 21-Bike-202"または"18-Bike-204"であるデータの ID とタイトルを取得し、テキスト形式で表示する

- コマンド

```
aws dynamodb scan \
     --table-name ProductCatalog \
     --filter-expression "Title = :arg1 or Title = :arg2" \
     --expression-attribute-values '{":arg1":{"S":"21-Bike-202"}, ":arg2":{"S":"18-Bike-204"}}' \
     --query 'Items[].{id:Id.N,title:Title.S}' \
     --output text
```

- 出力結果

```
205     18-Bike-204
202     21-Bike-202
204     18-Bike-204
```

### 特定の項目のみを表示する

商品カタログからタイトルが 21-Bike-202"または"18-Bike-204"であるデータの ID と値段、ブランドを JSON 形式で表示する

- コマンド

```
aws dynamodb scan \
     --table-name ProductCatalog \
     --filter-expression "Title = :arg1 or Title = :arg2" \
     --expression-attribute-values '{":arg1":{"S":"21-Bike-202"}, ":arg2":{"S":"18-Bike-204"}}' \
     --query 'Items[].{id:Id,price:Price,brand:Brand}'
```

- 出力結果

```
[
    {
        "id": {
            "N": "205"
        },
        "price": {
            "N": "500"
        },
        "brand": {
            "S": "Brand-Company C"
        }
    },
    {
        "id": {
            "N": "202"
        },
        "price": {
            "N": "200"
        },
        "brand": {
            "S": "Brand-Company A"
        }
    },
    {
        "id": {
            "N": "204"
        },
        "price": {
            "N": "400"
        },
        "brand": {
            "S": "Brand-Company B"
        }
    }
]
```

### 特定の項目のみを表示する（"N"等の余計なものは表示しない）

商品カタログからタイトルが 21-Bike-202"または"18-Bike-204"であるデータの ID と値段、ブランドを JSON 形式で表示する

- コマンド

```
aws dynamodb scan \
     --table-name ProductCatalog \
     --filter-expression "Title = :arg1 or Title = :arg2" \
     --expression-attribute-values '{":arg1":{"S":"21-Bike-202"}, ":arg2":{"S":"18-Bike-204"}}' \
     --query 'Items[].{id:Id.N,price:Price.N,brand:Brand.S}'
```

- 出力結果

```
[
    {
        "id": "205",
        "price": "500",
        "brand": "Brand-Company C"
    },
    {
        "id": "202",
        "price": "200",
        "brand": "Brand-Company A"
    },
    {
        "id": "204",
        "price": "400",
        "brand": "Brand-Company B"
    }
]
```

### ソート（表示項目１つだけの場合）

商品カタログからタイトルが 21-Bike-202"または"18-Bike-204"である ID を取得し、ID でソートしてテキスト形式で表示する

- コマンド

```
aws dynamodb scan \
     --table-name ProductCatalog \
     --filter-expression "Title = :arg1 or Title = :arg2" \
     --expression-attribute-values '{":arg1":{"S":"21-Bike-202"}, ":arg2":{"S":"18-Bike-204"}}' \
     --query 'sort(Items[].Id.N)' \
     --output text
```

- 出力結果

```
202     204     205
```

### ソート（表示項目が複数の場合）

商品カタログからタイトルが 21-Bike-202"または"18-Bike-204"であるデータの ID と値段、ブランドを取得し、ID でソートして JSON 形式で表示する

- コマンド

```
aws dynamodb scan \
     --table-name ProductCatalog \
     --filter-expression "Title = :arg1 or Title = :arg2" \
     --expression-attribute-values '{":arg1":{"S":"21-Bike-202"}, ":arg2":{"S":"18-Bike-204"}}' \
     --query 'Items[].{id:Id.N, price:Price.N, brand:Brand.S}|sort_by(@,&id)'
```

- 出力結果

```
[
    {
        "id": "202",
        "price": "200",
        "brand": "Brand-Company A"
    },
    {
        "id": "204",
        "price": "400",
        "brand": "Brand-Company B"
    },
    {
        "id": "205",
        "price": "500",
        "brand": "Brand-Company C"
    }
]
```
