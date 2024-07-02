---
title: Amplify Gen2 | How to use DynamoDB client in a Lambda function
description: Amplify Gen2 | Some solutions of using DynamoDB client in a Lambda function
last_modified_at: 2024-07-02T22:51:37+09:00
---
# Amplify Gen2: How to use DynamoDB client in a Lambda function
✨ Created on 2024/7/2

I have a project using [Amplify Gen2](https://docs.amplify.aws/), and I want to implement a sequence function on DynamoDB like other relational database such as [Oracle](https://www.oracle.com/database/), [PostgreSQL](https://www.postgresql.org/), etc. So I decided to create a table to store sequence information and a lambda function to increment the sequence then returns the next value by using [DynamoDB client](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/dynamodb/).  
  
At first I defined a table named `Sequence`.  
**amplify/data/resource.ts**
```ts
const schema = a.schema({
  Sequence: a
    .model({
      name: a.string().required(),
      value: a.integer().required()
    })
    .identifier(['name'])
    .authorization(allow => [
      allow.ownerDefinedIn('name').to(['get']), //Prevent direct calls from graphql with meaningless permissions
    ]),
});
```
And then I created a lambda function named `seqNextValue`.  
**amplify/functions/seq-next-value/resource.ts**
```ts
export const seqNextValue = defineFunction();
```
The function needs permission to update the table.  
**amplify/backend.ts**
```ts
import { Function } from 'aws-cdk-lib/aws-lambda';
backend.data.resources.tables['Sequence'].grantWriteData(backend.seqNextValue.resources.lambda);
(backend.seqNextValue.resources.lambda as Function).addEnvironment('SEQUENCE_TABLE_NAME', backend.data.resources.tables['Sequence'].tableName);
```
And then encountered the error as below.  
```diff
- The CloudFormation deployment has failed.  
- Caused By: ❌ Deployment failed: Error [ValidationError]: Circular dependency between resources: [storage～～, auth～～, data～～, function～～]
```
**Yes, that's why there are no examples of using DynamoDB client in the [Amplify Gen2 Documentations](https://docs.amplify.aws/).**  
  
After some research I found two ways to do the stuff.  
  
### Solution 1: Using [Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) and wildcards in permissions of the table.
**amplify/backend.ts**
```ts
import * as iam from 'aws-cdk-lib/aws-iam';
import * as ssm from 'aws-cdk-lib/aws-ssm';
// This is the function of seqNextValue
const snvfunc = backend.seqNextValue.resources.lambda as Function;
// Create a stack for parameter creation.
const stack = backend.createStack('zbstack');
// Create a parameter to store the table name.
const sspnm = `/${stack.artifactId}/SEQUENCE_TABLE_NAME`;
new ssm.StringParameter(stack, 'zbsp', {
  parameterName: sspnm,
  stringValue: backend.data.resources.tables['Sequence'].tableName,
});
// Pass the name of parameter to the function.
snvfunc.addEnvironment('SEQUENCE_SSM', sspnm);

// Grant permission of getting parameter to the function.
let stateSnv = new iam.PolicyStatement({
  actions: ['ssm:GetParameter'],
  resources: [`arn:aws:ssm:${stack.region}:${stack.account}:parameter${sspnm}`],
});
snvfunc.addToRolePolicy(stateSnv);

// Grant permission of updating table to the function.
stateSnv = new iam.PolicyStatement({
  actions: [
    'dynamodb:BatchWriteItem',
    'dynamodb:PutItem',
    'dynamodb:UpdateItem',
    'dynamodb:DeleteItem',
    'dynamodb:DescribeTable'
  ],
  resources: [
    `arn:aws:dynamodb:${stack.region}:${stack.account}:table/Sequence-*`
  ],
});
snvfunc.addToRolePolicy(stateSnv);
```
**amplify/functions/seq-next-value/handler.ts**
```ts
import type { Handler } from 'aws-lambda';
import { SSMClient, GetParameterCommand } from '@aws-sdk/client-ssm';
import { DynamoDBClient, UpdateItemCommand, ReturnValue } from '@aws-sdk/client-dynamodb';

type SeqNextValueEvent = {
  name: string;
};
export const handler: Handler<SeqNextValueEvent, number> = async (event, context) => {
  // Get table name from parameter store
  const ssmClient = new SSMClient();
  const cmdgp = new GetParameterCommand({
    Name: process.env.SEQUENCE_SSM,
    WithDecryption: true,
  });
  const paramstore = await ssmClient.send(cmdgp);
  const table = paramstore.Parameter!.Value;

  // Increment the sequence
  const dbClient = new DynamoDBClient();
  const ucmd = new UpdateItemCommand({
    TableName: table,
    Key: {
      name: {
        S: event.name,
      },
    },
    UpdateExpression: 'add #value :value',
    ExpressionAttributeNames: {
      '#value': 'value',
    },
    ExpressionAttributeValues: {
      ':value': {
        N: '1',
      },
    },
    ReturnValues: ReturnValue.UPDATED_NEW,
  });
  const resp = await dbClient.send(ucmd);
  if (resp.Attributes && resp.Attributes['value']?.N) {
    // Return the next value
    return parseInt(resp.Attributes['value'].N);
  } else {
    throw new Error('Failed to increment the sequence');
  }
};
```

### Solution 2: Give up using table's definition in [data backend](https://docs.amplify.aws/react/build-a-backend/data/set-up-data/) and create table ourselves via [AWS CDK](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_dynamodb-readme.html).
**amplify/backend.ts**
```ts
import * as db from 'aws-cdk-lib/aws-dynamodb';
// This is the function of seqNextValue
const snvfunc = backend.seqNextValue.resources.lambda as Function;
// Create a stack for table creation.
const stack = backend.createStack('zbstack');
// Create the table.
const seqdb = new db.TableV2(stack, 'seqdb', {
  tableName: `Sequence-${stack.artifactId}`,
  partitionKey: {
    name: 'name',
    type: db.AttributeType.STRING,
  },
  removalPolicy: RemovalPolicy.DESTROY,
});
// Pass table name to the function.
snvfunc.addEnvironment('SEQUENCE_TABLE_NAME', seqdb.tableName);
// Grant permission of updating table to the function.
seqdb.grantWriteData(snvfunc);
```
**amplify/functions/seq-next-value/handler.ts**
```ts
import type { Handler } from 'aws-lambda';
import { DynamoDBClient, UpdateItemCommand, ReturnValue } from '@aws-sdk/client-dynamodb';

type SeqNextValueEvent = {
  name: string;
};
export const handler: Handler<SeqNextValueEvent, number> = async (event, context) => {
  // Increment the sequence
  const dbClient = new DynamoDBClient();
  const ucmd = new UpdateItemCommand({
    TableName: process.env.SEQUENCE_TABLE_NAME,
    Key: {
      name: {
        S: event.name,
      },
    },
    UpdateExpression: 'add #value :value',
    ExpressionAttributeNames: {
      '#value': 'value',
    },
    ExpressionAttributeValues: {
      ':value': {
        N: '1',
      },
    },
    ReturnValues: ReturnValue.UPDATED_NEW,
  });
  const resp = await dbClient.send(ucmd);
  if (resp.Attributes && resp.Attributes['value']?.N) {
    // Return the next value
    return parseInt(resp.Attributes['value'].N);
  } else {
    throw new Error('Failed to increment the sequence');
  }
};
```

**Conclusion: I recommend using solution 2 because it is more concise.**

[🚗BACK](/README.html)
