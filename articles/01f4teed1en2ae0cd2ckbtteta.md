---
title: "Step FunctionsでDynamoDBをAthenaでクエリする環境作成を自動化してみた"
type: "tech"
category: []
description: "非公開"
publish: false
---

# はじめに

DyanmoDBには、S3にエクスポートする機能があります。本機能の概要は下記の記事が参考になります。
本機能は、エクスポートされたデータがS3上に下記のような形で配置されます。

```bash
/
└── AWSDynamoDB
    ├── 01620089105917-bed5879e/data
    ├── 01620089211868-3135f307/data
    └── 01620089105917-bed5879e/data
```

新しくエクスポートされたDyanmoDBのデータをみたい場合、Athenaのテーブルが参照しているS3のパス(location)を変更するがあり、ALTER TABLEを毎回叩く必要があります。

以上より、今回はStep Functionsを利用して以下の作業の自動化してみました。
1. DynamoDBからエクスポート指示
2. エクスポート完了待機
3. ALTERテーブルクエリ実行
4. ALTERテーブルクエリ実行待機
5. 完了

Glue Jobを使うまでもなく、手軽にDynamoDBのデータをAthenaで検索環境を作りたい場合に、おすすめです。

DynamoDBからS3にエクスポートする機能に関しては、下記の記事がおすすめです。
* [New – Export Amazon DynamoDB Table Data to Your Data Lake in Amazon S3, No Code Writing Required](https://aws.amazon.com/jp/blogs/aws/new-export-amazon-dynamodb-table-data-to-data-lake-amazon-s3/)
* [【新機能】Amazon DynamoDB Table を S3 に Export して Amazon Athena でクエリを実行する](https://dev.classmethod.jp/articles/dynamodb-table-export-service/)
* [DynamoDBのData Export機能を使って、データを定期的にS3にエクスポートする](https://dev.classmethod.jp/articles/dynamodb-data-export-s3/)

AWS Glue と Step Functionsを使ってS3にあげる方法は、下記の記事がおすすめです。
[DynamoDB から S3 への定期的なエクスポートの仕組みを AWS Glue と Step Functions を使用して実装してみた](https://dev.classmethod.jp/articles/how-to-export-an-amazon-dynamodb-table-to-amazon-s3/)


# 構成
## ステートマシーン
Step Functionsのステートマシーンの構成は以下の通りです。紫字で各処理の概要を示します。Lambda①〜Lambda④については、後述します。

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1620094051/blog/01f4teed1en2ae0cd2ckbtteta/1_export-dynamo-statemachine-1.png)

## サンプルデータ

サンプルには、馴染みにやすいようにブログサービスを想定して、下記ようなDynamoDBを用意します。(本筋とは関係ないので、なんでも構いません)

テーブル名: Article
|userId(PK)|articleId(SK)|title|category|
|---|---|---|---|
|7155dc64-982a-4559-9f94-5320dc4d003d|01F4TKNKWMH40A749JEWFGHH6K|タイトル1|["cate1", "cate2"]
|7155dc64-982a-4559-9f94-5320dc4d003d|01F4TKPRSMQ56VVBW850B52EGP|タイトル2|["cate1", "cate3"]

:::details DynamoDB CDK定義
```ts
    const articeTable = new dynamodb.Table(this, `Article`, {
      partitionKey: {
        name: 'userId',
        type: dynamodb.AttributeType.STRING,
      },
      sortKey: {
        name: 'articleId',
        type: dynamodb.AttributeType.STRING,
      },
      tableName: `Article`,
      pointInTimeRecovery: true,
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
    });
```

```json:サンプルデータ1
{
  "userId": "7155dc64-982a-4559-9f94-5320dc4d003d",
  "articleId": "XXXXXXX",
  "content": "hoge",
  "createAt": 1577804400
}
```

```json:サンプルデータ2
{
  "userId": "7155dc64-982a-4559-9f94-5320dc4d003d",
  "articleId": "YYYYYY",
  "content": "foo",
  "createAt": 1577804401
}
```
:::
<br>

## 実行方法
実装イメージが湧きやすいように、実行方法を先に説明します。

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1620094540/blog/01f4teed1en2ae0cd2ckbtteta/2_use.png)

入力には、exportしたいテーブルの名前を入れます。
```json
{"tableName":"Article"}
```

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1620094540/blog/01f4teed1en2ae0cd2ckbtteta/3_use-1.png)

## 実行結果
### Step Functions

::: details 実行ログ
|時刻|イベント|
|---|---|
|12:23:56.839|実行開始|
|12:23:56.875|エクスポート開始|
|12:23:57.467|エクスポート完了|
|12:23:56.839|エクスポート待機開始|
|12:28:57.475|エクスポート待機完了|
|12:28:57.488|エクスポートステータス取得|
|12:28:58.009|エクスポートステータス取得完了|
|12:28:58.018|エクスポート完了判定|
|12:28:58.018|エクスポート完了判定完了|
|12:28:58.118|エクスポート待機開始(2回目)|
|12:33:58.118|エクスポート待機完了(2回目)|
|12:33:58.137|エクスポートステータス取得(2回目)|
|12:33:58.653|エクスポートステータス取得完了(2回目)|
|12:33:58.661|エクスポート完了判定(2回目)|
|12:33:58.661|エクスポート完了判定完了(2回目)|
|12:33:58.761|クエリ実行|
|12:34:00.046|クエリ実行完了|
|12:34:00.054|クエリ実行待機|
|12:34:00.054|クエリ実行待機完了|
|12:35:00.071|クエリステータス取得|
|12:35:01.235|クエリステータス取得完了|
|12:35:01.243|クエリステータス判定|
|12:35:01.243|クエリステータス判定完了|
|12:35:01.343|SuceedState|
|12:35:01.343|SuceedState完了|
|12:35:01.343|実行終了|
:::


### S3
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1620101560/blog/01f4teed1en2ae0cd2ckbtteta/4_result-s3.png)

### Athena
無事に内容を取得できました。

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1620101648/blog/01f4teed1en2ae0cd2ckbtteta/5_result-athena.png)



# 実装

## CDK
CDKのバージョンは、`1.101.0`を利用します。全体的に権限が緩めなので、動作確認が済次第絞ることをお勧めします。

:::details Athena,S3,Step Functions,Glue TableのCDK定義
```ts
 cdk from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';
import * as lambda from '@aws-cdk/aws-lambda';
import * as iam from '@aws-cdk/aws-iam';
import * as events from '@aws-cdk/aws-events';
import * as eventsTargets from '@aws-cdk/aws-events-targets';
import * as athena from '@aws-cdk/aws-athena';
import * as glue from '@aws-cdk/aws-glue';
import * as stepfunctions from '@aws-cdk/aws-stepfunctions';
import * as stepfunctionsTasks from '@aws-cdk/aws-stepfunctions-tasks';

export class AnalysisStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const accountId = cdk.Stack.of(this).account;
    const DB_NAME = `ddb`;

    // DynamoDB出力用、Athena検索用のバケットを作成
    const anlysisBucket = new s3.Bucket(this, 'AnalysisBucket', {
      bucketName: `analysis-bucket-${accountId}`,
    });

    // Athenaのワークグループを作成、Lambdaからクエリを実行する際に必要
    const athenWorkGroup = new athena.CfnWorkGroup(this, `AthenaWorkGroup`, {
      name: `work-group`,
      workGroupConfiguration: {
        resultConfiguration: {
          outputLocation: `s3://${anlysisBucket.bucketName}/athena-work-group/`,
        },
      },
    });

    // Lambda①
    const exportDynamoFunction = new lambda.Function(
      this,
      'exportDynamoFunction',
      {
        code: lambda.Code.fromAsset(`dist/exportDynamo`),
        environment: {
          EXPORT_S3_BUCKET_NAME: anlysisBucket.bucketName,
        },
        functionName: `export-article-table`,
        handler: 'index.handler',
        runtime: lambda.Runtime.NODEJS_14_X,
        tracing: lambda.Tracing.ACTIVE,
        timeout: cdk.Duration.seconds(10),
      },
    );
    exportDynamoFunction.addToRolePolicy(
      new iam.PolicyStatement({
        actions: ['dynamodb:exportTableToPointInTime'],
        resources: ['*'],
      }),
    );
    exportDynamoFunction.addToRolePolicy(
      new iam.PolicyStatement({
        actions: ['s3:PutObject'],
        resources: ['*'],
      }),
    );

    // Lambda②
    const getExportDynamoStatusFunction = new lambda.Function(
      this,
      'getExportDynamoStatusFunction',
      {
        code: lambda.Code.fromAsset(`dist/getExportDynamoStatus`),
        environment: {},
        functionName: `get-export-dynamo-status`,
        handler: 'index.handler',
        runtime: lambda.Runtime.NODEJS_14_X,
        tracing: lambda.Tracing.ACTIVE,
        timeout: cdk.Duration.seconds(10),
      },
    );
    getExportDynamoStatusFunction.addToRolePolicy(
      new iam.PolicyStatement({
        actions: ['dynamodb:*'],
        resources: ['*'],
      }),
    );

    // Lambda③
    const relocateTableFunction = new lambda.Function(
      this,
      'relocateTableFunction',
      {
        code: lambda.Code.fromAsset(`dist/relocateTable`),
        environment: {
          DB_NAME: DB_NAME,
          ATHENA_WORK_GROUP: athenWorkGroup.name,
        },
        functionName: `relocate-table`,
        handler: 'index.handler',
        runtime: lambda.Runtime.NODEJS_14_X,
        tracing: lambda.Tracing.ACTIVE,
        timeout: cdk.Duration.seconds(10),
      },
    );
    relocateTableFunction.addToRolePolicy(
      new iam.PolicyStatement({
        actions: ['athena:*', 's3:*', 'glue:*'],
        resources: ['*'],
      }),
    );

    // Lambda④
    const getRelocateTableStatusFunction = new lambda.Function(
      this,
      'getRelocateTableStatusFunction',
      {
        code: lambda.Code.fromAsset(`dist/getRelocateTableStatus`),
        functionName: `relocate-table-status`,
        handler: 'index.handler',
        runtime: lambda.Runtime.NODEJS_14_X,
        tracing: lambda.Tracing.ACTIVE,
        timeout: cdk.Duration.seconds(10),
      },
    );
    getRelocateTableStatusFunction.addToRolePolicy(
      new iam.PolicyStatement({
        actions: ['athena:*', 's3:*'],
        resources: ['*'],
      }),
    );

    // ステートマシーンの作成
    const EXPORT_DYNAMO_WAIT_TIME = 5;
    const RELOCATE_TABLE_WAIT_TIME = 1;

    const exportDynamoTask = new stepfunctionsTasks.LambdaInvoke(
      this,
      'Request export of DynamoDB to S3',
      {
        lambdaFunction: exportDynamoFunction,
        invocationType:
          stepfunctionsTasks.LambdaInvocationType.REQUEST_RESPONSE,
        outputPath: '$.Payload',
      },
    );

    const exportDynamoWait = new stepfunctions.Wait(
      this,
      'Wait to request export of DynamoDB to S3',
      {
        time: stepfunctions.WaitTime.duration(
          cdk.Duration.minutes(EXPORT_DYNAMO_WAIT_TIME),
        ),
      },
    );

    const getExportDynamoStatusTask = new stepfunctionsTasks.LambdaInvoke(
      this,
      'Get export of DynamoDB to S3 status',
      {
        lambdaFunction: getExportDynamoStatusFunction,
        invocationType:
          stepfunctionsTasks.LambdaInvocationType.REQUEST_RESPONSE,
        outputPath: '$.Payload',
      },
    );

    const relocateTableTask = new stepfunctionsTasks.LambdaInvoke(
      this,
      'Query to change location',
      {
        lambdaFunction: relocateTableFunction,
        invocationType:
          stepfunctionsTasks.LambdaInvocationType.REQUEST_RESPONSE,
        outputPath: '$.Payload',
      },
    );

    const getRelocateTableTask = new stepfunctionsTasks.LambdaInvoke(
      this,
      'Get query status',
      {
        lambdaFunction: getRelocateTableStatusFunction,
        invocationType:
          stepfunctionsTasks.LambdaInvocationType.REQUEST_RESPONSE,
        outputPath: '$.Payload',
      },
    );

    const relocateTableWait = new stepfunctions.Wait(this, 'Wait for query', {
      time: stepfunctions.WaitTime.duration(
        cdk.Duration.minutes(RELOCATE_TABLE_WAIT_TIME),
      ),
    });

    const relocateTableSucceed = new stepfunctions.Succeed(
      this,
      'Query succeed',
    );
    const relocateTableFaild = new stepfunctions.Fail(this, 'Query falid');

    const exportDynamoChoice = new stepfunctions.Choice(
      this,
      'Export complete?',
    )
      .when(
        stepfunctions.Condition.stringEquals('$.exportStaus', 'COMPLETED'), // Choice().next()は不可なので、ネストしている
        relocateTableTask
          .next(relocateTableWait)
          .next(getRelocateTableTask)
          .next(
            new stepfunctions.Choice(this, 'Query complete?')
              .when(
                stepfunctions.Condition.stringEquals('$.status', 'FAILED'),
                relocateTableFaild,
              )
              .when(
                stepfunctions.Condition.stringEquals('$.status', 'SUCCEEDED'),
                relocateTableSucceed,
              )
              .otherwise(relocateTableWait),
          ),
      )
      .otherwise(exportDynamoWait);

    const definition = exportDynamoTask
      .next(exportDynamoWait)
      .next(getExportDynamoStatusTask)
      .next(exportDynamoChoice);

    const stateMachine = new stepfunctions.StateMachine(
      this,
      'exportDynamostateMachine',
      {
        stateMachineName: `export-dynamo`,
        definition: definition,
      },
    );

    /* EventBridgeで定期的に更新する(cronの方がよいかも)
    new events.Rule(this, `articleExportDyanmoRule`, {
      ruleName: `article-export-dynamo-rule`,
      schedule: events.Schedule.rate(cdk.Duration.hours(8)),
      targets: [
        new eventsTargets.SfnStateMachine(stateMachine, {
          input: events.RuleTargetInput.fromObject({
            tableName: 'Article',
            s3Bucket: anlysisBucket.bucketName,
          }),
        }),
      ],
    });
    */

    // DynamoDBから出力されたデータをS3に格納し、それを参照するGlueのテーブル定義
    const db = new glue.Database(this, `ExportDynamo`, {
      databaseName: DB_NAME,
    });

    new glue.Table(this, `article`, {
      database: db,
      tableName: `article`,
      bucket: anlysisBucket,
      dataFormat: glue.DataFormat.JSON,
      columns: [
        {
          name: 'Item',
          type: glue.Schema.struct([
            {
              name: 'userId',
              type: glue.Schema.struct([
                {
                  name: 'S',
                  type: glue.Schema.STRING,
                },
              ]),
            },
            {
              name: 'articleId',
              type: glue.Schema.struct([
                {
                  name: 'S',
                  type: glue.Schema.STRING,
                },
              ]),
            },
            {
              name: 'content',
              type: glue.Schema.struct([
                {
                  name: 'S',
                  type: glue.Schema.STRING,
                },
              ]),
            },
            {
              name: 'createAt',
              type: glue.Schema.struct([
                {
                  name: 'N',
                  type: glue.Schema.STRING,
                },
              ]),
            },
          ]),
        },
      ],
    });
  }
}
```
:::

下記に補足します

### 作成するS3バケットの用途

DynamoDBでexportを実行すると、成果物は`s3://${Prefix}/AWSDynamoDB/${ExportArn}`に作成されます。

Prefixはexportを実行する際に指定可能です。Prefixには、複数のDynamoDBをエクスポートし、Glueテーブルの向け先を更新することを想定し、DyanmoDBのテーブル名を指定します。(詳しく後述するLambdaの実装参照)

|S3のパス|用途|
|---|---|
|s3://${DynamoDBテーブル名}/AWSDynamoDB/${ExportArn}/data|Glueテーブルの参照先|
|s3://${DynamoDBテーブル名}/athena-work-group|athenaの実行結果参照先|

### Glueテーブル定義の作成
今回は、AWS Glueクローラー利用せず、事前にテーブル定義を作成しています。クローラーが型解釈を間違えるリスクがないのがメリットです。(クローラーを使うパターンにもメリットがあるので、状況によりけりです)

本Step Functionsは、**入力イベントのテーブル名に対応したテーブル(小文字)を更新するようにしているため**、定期的にAthenaで検索したいテーブルは、全てCDKでテーブル定義の事前作成が必要です。

定義してしまえば、あとはポチポチで自動更新されますし、EventBridgeで定期的にイベントを流しても良いと思います。


## Lambda
※ 実装は、記事と関係ないエラー定義などは削除しているので、動作を保証するものではありません。

:::details Lambda① DynamoDBへエクスポートを指示する
```ts
import * as DynamoDB from '../../infrastructures/dynamodb/dynamodb';

interface Event {
  tableName: string;
}

interface Context {
  invokedFunctionArn: string;
}

interface Result {
  exportArn: string;
  s3Bucket: string;
  tableName: string;
}

export const handler = async (
  event: Event,
  context: Context,
): Promise<Result> => {
  try {
    const AWS_REGION = process.env.AWS_REGION!;
    const EXPORT_S3_BUCKET_NAME = process.env.EXPORT_S3_BUCKET_NAME!; // CDKで作成したS3を指定

    const accountId = context.invokedFunctionArn.split(':')[4];
    const tableArn = `arn:aws:dynamodb:${AWS_REGION}:${accountId}:table/${event.tableName}`;

    const exportRequestResult = await DynamoDB.requestExportToS3({ // 次のコードブロックに記述
      tableArn: tableArn,
      s3Bucket: EXPORT_S3_BUCKET_NAME,
      s3Prefix: event.tableName, // s3://${Prefix}/AWSDynamoDB/${exportArn}/data でエクスポートするように指示
    });

    if (!exportRequestResult.ExportDescription?.ExportArn) {
      throw new ExportToS3RequestHandlerNotFoundExportArnError(event);
    }

    return {
      exportArn: exportRequestResult.ExportDescription.ExportArn,
      s3Bucket: EXPORT_S3_BUCKET_NAME,
      tableName: event.tableName,
    };
  } catch (e) {
    throw e;
  }
};
```

```ts
import * as DynamoDB from 'aws-sdk/clients/dynamodb';

const DYNAMODB_API_VERSION = '2012-10-08';
const REGION = 'ap-northeast-1';

export const dynamodbClient = new DynamoDB({
  apiVersion: DYNAMODB_API_VERSION,
  region: REGION,
});

interface ExportToS3Settings {
  tableArn: string;
  s3Bucket: string;
  s3Prefix?: string;
}

export const requestExportToS3 = async (
  exportToS3Settings: ExportToS3Settings,
): Promise<DynamoDB.ExportTableToPointInTimeOutput> => {
  const response = await dynamodbClient
    .exportTableToPointInTime({
      TableArn: exportToS3Settings.tableArn,
      S3Bucket: exportToS3Settings.s3Bucket,
      S3Prefix: exportToS3Settings.s3Prefix,
      ExportFormat: 'DYNAMODB_JSON',
    })
    .promise();

  return response;
};
```

:::

:::details Lambda②  エクスポートした状況を確認する
```ts
import * as DynamoDB from '../../infrastructures/dynamodb/dynamodb';

interface Event {
  exportArn: string;
  s3Bucket: string;
  tableName: string;
}

interface Result {
  exportArn: string;
  exportStaus: string;
  s3Bucket: string;
  tableName: string;
}

export const handler = async (event: Event): Promise<Result> => {
  try {
    const exportRequests = await DynamoDB.getExportToS3Status(); // 次のコードブロックに記述
    const filteredExportSummary = exportRequests.ExportSummaries?.filter(
      (exportSummary) => {
        return exportSummary.ExportArn === event.exportArn;
      },
    );

    if (
      !filteredExportSummary ||
      !filteredExportSummary[0].ExportArn ||
      !filteredExportSummary[0].ExportStatus
    ) {
      throw new GetExportDynamoStatusUnknownError(event);
    }

    return {
      exportArn: event.exportArn,
      exportStaus: filteredExportSummary[0].ExportStatus,
      s3Bucket: event.s3Bucket,
      tableName: event.tableName,
    };
  } catch (e) {
    throw e;
  }
};

```

```ts
import * as DynamoDB from 'aws-sdk/clients/dynamodb';

const DYNAMODB_API_VERSION = '2012-10-08';
const REGION = 'ap-northeast-1';

export const dynamodbClient = new DynamoDB({
  apiVersion: DYNAMODB_API_VERSION,
  region: REGION,
});

export const getExportToS3Status = async (): Promise<DynamoDB.ListExportsOutput> => {
  const exportsRequest = await dynamodbClient.listExports().promise();

  return exportsRequest;
};
```

:::

:::details Lambda③ ALTER TABLEでテーブルのlocationを変更する
```ts
import * as Athena from 'aws-sdk/clients/athena';

interface Event {
  exportArn: string;
  s3Bucket: string;
  tableName: string;
}

interface Result {
  queryExecutionId: string;
}

const DB_NAME = process.env.DB_NAME!;

const API_VERSION = '2017-05-18';
const REGION = process.env.AWS_REGION!;
const ATHENA_WORK_GROUP = process.env.ATHENA_WORK_GROUP!;

export const athenaClient = new Athena({
  apiVersion: API_VERSION,
  region: REGION,
});

export const handler = async (event: Event): Promise<Result> => {
  try {
    const splitExportArn = event.exportArn.split('/');

   // CDKでテーブル名はLowerCaseにしているので変換
    const athenaTableName = event.tableName.toLowerCase();
    const exportId = splitExportArn[3];

    const queryLocation = `ALTER TABLE ${DB_NAME}.${athenaTableName}
SET LOCATION 's3://${event.s3Bucket}/${event.tableName}/AWSDynamoDB/${exportId}/data/';`;

    const response = await athenaClient
      .startQueryExecution({
        QueryString: queryLocation,
        QueryExecutionContext: {
          Database: DB_NAME,
        },
        WorkGroup: ATHENA_WORK_GROUP, // CDKで作成したAtehnaのワークグループ
      })
      .promise();

    return {
      queryExecutionId: response.QueryExecutionId!, // null undefined判定したほうがよいです..
    };
  } catch (e) {

    throw e;
  }
};
```
:::

:::details Lambda④ クエリの状況を確認する
```ts
import * as Athena from 'aws-sdk/clients/athena';

interface Event {
  queryExecutionId: string;
}

interface Result {
  status: string;
}

const API_VERSION = '2017-05-18';
const REGION = process.env.AWS_REGION!;

export const athenaClient = new Athena({
  apiVersion: API_VERSION,
  region: REGION,
});

export const handler = async (event: Event): Promise<Result> => {
  try {
    const response = await athenaClient
      .getQueryExecution({
        QueryExecutionId: event.queryExecutionId,
      })
      .promise();

    if (response.QueryExecution?.Status?.State !== 'SUCCEEDED') {
      return {
        status: response.QueryExecution?.Status?.State ?? 'UNKNOWN',
      };
    }

    return {
      status: response.QueryExecution?.Status.State,
    };
  } catch (e) {
    throw e;
  }
};
```
:::


# 最後に
本記事では、DynamoDBの内容を定期的にS3にあげてAthenaでクエリできる環境をStep Functionsで自動化しました。AthenaのPrestoは非常に強力なので、DynamoDBを使っていて検索まわりで壁にぶつかったら、本Step Functionsを導入してまず何ができる試してみることをお勧めします。

ソースコードがもっと詳しくみたい！とかありましたら、[こちら](https://github.com/hozi-dev/article/issues)までご連絡ください。
