# Amazon DynamoDB 殴り書き

## Amazon DynamoDB: 仕組み 
- DynamoDB の基本的な概念を学習します。

### DynamoDB コアコンポーネント
#### プライマリキー

- 2種類の異なるプライマリキー
`パーティションキー`
`パーティションとソートキー`


#### セカンダリインデックス
- 2 種類のインデックス
`グローバルセカンダリインデックス`GSI
テーブルと異なるパーティションキーとソートキーを持つインデックス
`ローカルセカンダリインデックス`LSI
テーブルと同じパーティションキーと、異なるソートキーを持つインデックス
|Feature  |GSI  |LSI  |
|---|---|---|
|Key,Scheme  |PK<br>PK&SK  |PK&SK  |
|読み込み整合性  |結果整合性のみ  |結果整合性または強い整合性  |


**DynamoDB のテーブル制限**:
20 グローバルセカンダリーインデックス(デフォルトの制限) と、テーブルあたり 5 ローカルセカンダリーインデックスに制限

|DynamoDB API  |methods  |
|---|---|
|ControlPlane  |CreateTable<br>DescribeTable<br>ListTables<br>UpdateTable<br>DeleteTable|
|DataPlane**Write**|PutItem<br>BatchWriteItem<br>
|DataPlane**Read**|GetItem<br>BatchGetItem<br>Query<br>Scan|
|DataPlane**Update**|UpdateItem|
|DataPlane**Delete**|DeleteItem<br>BatchWriteItem|
|Stream|ListStreams<br>DescribeStream<br>GetShardIterator<br>GetRecords|
|Transaction|TransactWriteItems <br>TransactGetItems|
**DynamoDB のデータ型**:
- スカラー型 
- ドキュメント型
- セット型

### 複合キーテーブル

table作成時に2つのattributeを選び、1つをハッシュキーとして、もう一つをレンジキーと呼ばれるキーとして宣言する

```
// Forum
{
  "Name": "DynamoDB",
  "Category": "Amazon Web Services",
  "Threads": 3,
  "Messages": 4,
  "Views": 1000,
  "LastPostBy": "User A",
  "LastPostDateTime": "2012-01-03T00:40:57.165Z"
}
{     
  "Name": "Amazon S3",
  "Category": "AWS",
  "Threads": 1
}

```
```
// Thread
{
  "ForumName": "DynamoDB",
  "Subject": "DynamoDB Thread 1",
  "Message": "DynamoDB thread 1 message text",
  "LastPostedBy": "User A",
  "Views": 0,
  "Replies": 0,
  "Answered": 0,
  "Tags": [ "index", "primarykey", "table" ],
  "LastPostDateTime": "2012-01-03T00:40:57.165Z"
}
{
  "ForumName": "DynamoDB",
  "Subject": "DynamoDB Thread 2",
  "Message": "DynamoDB thread 2 message text",
  "LastPostedBy": "User A",
  "Views": 0,
  "Replies": 0,
  "Answered": 0,
  "Tags": [ "index", "primarykey", "rangekey" ],
  "LastPostDateTime": "2012-01-03T00:40:57.165Z"
}
{
  "ForumName": "Amazon S3",
  "Subject": "Amazon S3 Thread 1",
  "Message": "Amazon S3 Thread 1 message text",
  "LastPostedBy": "User A",
  "Views": 0,
  "Replies": 0,
  "Answered": 0,
  "Tags": [ "largeobject", "multipart upload" ],
  "LastPostDateTime": "2012-01-03T00:40:57.165Z"
}
```
```
// Reply
{
  "Id": "DynamoDB#DynamoDB Thread 1",
  "ReplyDateTime": "2011-12-11T00:40:57.165Z",
  "Message": "DynamoDB Thread 1 Reply 1 text",
  "PostedBy": "User A"
}
{
  "Id": "DynamoDB#DynamoDB Thread 1",
  "ReplyDateTime": "2011-12-18T00:40:57.165Z",
  "Message": "DynamoDB Thread 1 Reply 1 text",
  "PostedBy": "User A"
}
{
  "Id": "DynamoDB#DynamoDB Thread 1",
  "ReplyDateTime": "2011-12-25T00:40:57.165Z",
  "Message": "DynamoDB Thread 1 Reply 3 text",
  "PostedBy": "User B"
}
{
  "Id": "DynamoDB#DynamoDB Thread 2",
  "ReplyDateTime": "2011-12-25T00:40:57.165Z",
  "Message": "DynamoDB Thread 2 Reply 1 text",
  "PostedBy": "User A"
}
{
  "Id": "DynamoDB#DynamoDB Thread 2",
  "ReplyDateTime": "2012-01-03T00:40:57.165Z",
  "Message": "DynamoDB Thread 2 Reply 2",
  "PostedBy": "User A"
}
```
**POINT**

Threadtableの場合
- ForumNameをハッシュキー
- Subjectをレンジキー

Replytableの場合
- Idをハッシュキー
- ReplyDateTimeをレンジキー

2つの値の組み合わせによって、1つのitemを特定
` ForumName = 'DynamoDB' AND Subject = 'DynamoDB Thread 1'`

ハッシュキーだけではitemを1つに特定できません
ThreadtableにおいてForumName = 'DynamoDB'

#### Scan, GetItem, Query *
Scanはハッシュキーテーブルと同様
GetItemはハッシュキーの値とレンジキーの値2つで0個または1個のitemを返す
Queryでは
- Threadtable ForumName = 'DynamoDB'という条件でQuery
「DynamoDBフォーラムのスレッド一覧画面」を取得
- Replytable Idがハッシュキー、ReplyDateTimeがレンジキー
`Id = 'DynamoDB#DynamoDB Thread 1' AND ReplyDateTime BETWEEN '2011-12-15T00:00:00.000Z' AND '2011-12-31T23:59:59.999Z'`

### LSI
*複合キーテーブルにしか定義できません*
テーブルのハッシュキーと同じハッシュキーでしか定義できません。つまり、PostedByをハッシュキー、IdをレンジキーとなるLSIを定義はできません。ハッシュキーは必ずIdである必要がある。
> 「ローカル」というのは「同じパーティション内」

### GSI 


