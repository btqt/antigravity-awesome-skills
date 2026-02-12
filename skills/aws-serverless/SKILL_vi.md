---
name: aws-serverless
description: "Kỹ năng chuyên biệt để xây dựng các ứng dụng không máy chủ (serverless) sẵn sàng cho sản xuất trên AWS. Bao gồm các hàm Lambda, API Gateway, DynamoDB, các mẫu hướng sự kiện SQS/SNS, triển khai SAM/CDK và tối ưu hóa khởi động lạnh (cold start)."
source: vibeship-spawner-skills (Apache 2.0)
---

# AWS Serverless

## Các Mẫu thiết kế (Patterns)

### Mẫu Lambda Handler

Cấu trúc hàm Lambda đúng với xử lý lỗi

**Khi nào sử dụng**: ['Bất kỳ triển khai hàm Lambda nào', 'API handlers, bộ xử lý sự kiện, tác vụ được lên lịch']

```javascript
// Node.js Lambda Handler
// handler.js

// Khởi tạo bên ngoài handler (được sử dụng lại qua các lần gọi)
const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
const { DynamoDBDocumentClient, GetCommand } = require("@aws-sdk/lib-dynamodb");

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

// Hàm Handler
exports.handler = async (event, context) => {
  // Tùy chọn: Không đợi event loop xóa (Node.js)
  context.callbackWaitsForEmptyEventLoop = false;

  try {
    // Phân tích đầu vào dựa trên nguồn sự kiện
    const body =
      typeof event.body === "string" ? JSON.parse(event.body) : event.body;

    // Logic nghiệp vụ
    const result = await processRequest(body);

    // Trả về phản hồi tương thích với API Gateway
    return {
      statusCode: 200,
      headers: {
        "Content-Type": "application/json",
        "Access-Control-Allow-Origin": "*",
      },
      body: JSON.stringify(result),
    };
  } catch (error) {
    console.error(
      "Error:",
      JSON.stringify({
        error: error.message,
        stack: error.stack,
        requestId: context.awsRequestId,
      }),
    );

    return {
      statusCode: error.statusCode || 500,
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        error: error.message || "Lỗi máy chủ nội bộ",
      }),
    };
  }
};

async function processRequest(data) {
  // Logic nghiệp vụ của bạn ở đây
  const result = await docClient.send(
    new GetCommand({
      TableName: process.env.TABLE_NAME,
      Key: { id: data.id },
    }),
  );
  return result.Item;
}
```

```python
# Python Lambda Handler
# handler.py

import json
import os
import logging
import boto3
from botocore.exceptions import ClientError

# Khởi tạo bên ngoài handler (được sử dụng lại qua các lần gọi)
logger = logging.getLogger()
logger.setLevel(logging.INFO)

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def handler(event, context):
    try:
        # Phân tích đầu vào...
```

### Mẫu Tích hợp API Gateway

Tích hợp REST API và HTTP API với Lambda

**Khi nào sử dụng**: ['Xây dựng REST API được hỗ trợ bởi Lambda', 'Cần các điểm cuối HTTP cho các hàm']

```yaml
# template.yaml (SAM)
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    Runtime: nodejs20.x
    Timeout: 30
    MemorySize: 256
    Environment:
      Variables:
        TABLE_NAME: !Ref ItemsTable

Resources:
  # HTTP API (được khuyến nghị cho các trường hợp sử dụng đơn giản)
  HttpApi:
    Type: AWS::Serverless::HttpApi
    Properties:
      StageName: prod
      CorsConfiguration:
        AllowOrigins:
          - "*"
        AllowMethods:
          - GET
          - POST
          - DELETE
        AllowHeaders:
          - "*"

  # Lambda Functions
  GetItemFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/get.handler
      Events:
        GetItem:
          Type: HttpApi
          Properties:
            ApiId: !Ref HttpApi
            Path: /items/{id}
            Method: GET
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref ItemsTable

  CreateItemFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/create.handler
      Events:
        CreateItem:
          Type: HttpApi
          Properties:
            ApiId: !Ref HttpApi
            Path: /items
            Method: POST
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref ItemsTable

  # DynamoDB Table
  ItemsTable:
    Type: AWS::DynamoDB::Table
    Properties:
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST

Outputs:
  ApiUrl:
    Value: !Sub "https://${HttpApi}.execute-api.${AWS::Region}.amazonaws.com/prod"
```

```javascript
// src/handlers/get.js
const { getItem } = require('../lib/dynamodb');

exports.handler = async (event) => {
  const id = event.pathParameters?.id;

  if (!id) {
    return {
      statusCode: 400,
      body: JSON.stringify({ error: 'Thiếu tham số id' })
    };
  }

  const item = // ...
```

### Mẫu SQS Hướng Sự kiện (Event-Driven SQS Pattern)

Lambda được kích hoạt bởi SQS để xử lý không đồng bộ đáng tin cậy

**Khi nào sử dụng**: ['Xử lý không đồng bộ, tách biệt', 'Cần logic thử lại và DLQ', 'Xử lý tin nhắn theo lô']

```yaml
# template.yaml
Resources:
  ProcessorFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/processor.handler
      Events:
        SQSEvent:
          Type: SQS
          Properties:
            Queue: !GetAtt ProcessingQueue.Arn
            BatchSize: 10
            FunctionResponseTypes:
              - ReportBatchItemFailures # Xử lý lỗi lô một phần

  ProcessingQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 180 # 6x thời gian chờ Lambda
      RedrivePolicy:
        deadLetterTargetArn: !GetAtt DeadLetterQueue.Arn
        maxReceiveCount: 3

  DeadLetterQueue:
    Type: AWS::SQS::Queue
    Properties:
      MessageRetentionPeriod: 1209600 # 14 ngày
```

```javascript
// src/handlers/processor.js
exports.handler = async (event) => {
  const batchItemFailures = [];

  for (const record of event.Records) {
    try {
      const body = JSON.parse(record.body);
      await processMessage(body);
    } catch (error) {
      console.error(`Không thể xử lý tin nhắn ${record.messageId}:`, error);
      // Báo cáo mục này bị lỗi (sẽ được thử lại)
      batchItemFailures.push({
        itemIdentifier: record.messageId,
      });
    }
  }

  // Trả về các mục bị lỗi để thử lại
  return { batchItemFailures };
};

async function processMessage(message) {
  // Logic xử lý của bạn
  console.log("Đang xử lý:", message);

  // Mô phỏng công việc
  await saveToDatabase(message);
}
```

```python
# Phiên bản Python
import json
import logging

logger = logging.getLogger()

def handler(event, context):
    batch_item_failures = []

    for record in event['Records']:
        try:
            body = json.loads(record['body'])
            process_message(body)
        except Exception as e:
            logger.error(f"Không thể xử lý {record['messageId']}: {e}")
            batch_item_failures.append({
                'itemIdentifier': record['messageId']
            })

    return {'batchItemFailures': batch_item_failures}
```

## Các Mẫu Sai lầm (Anti-Patterns)

### ❌ Monolithic Lambda (Lambda nguyên khối)

**Tại sao tệ**: Gói triển khai lớn gây ra khởi động lạnh chậm.
Khó mở rộng các hoạt động riêng lẻ.
Cập nhật ảnh hưởng đến toàn bộ hệ thống.

### ❌ Large Dependencies (Phụ thuộc lớn)

**Tại sao tệ**: Tăng kích thước gói triển khai.
Làm chậm khởi động lạnh đáng kể.
Hầu hết SDK/thư viện có thể không được sử dụng.

### ❌ Gọi Đồng bộ trong VPC

**Tại sao tệ**: Lambda gắn VPC có chi phí thiết lập ENI.
Tra cứu DNS hoặc kết nối chặn làm tồi tệ thêm khởi động lạnh.

## ⚠️ Các Vấn đề Tiềm ẩn (Sharp Edges)

| Vấn đề | Mức độ nghiêm trọng | Giải pháp                                 |
| ------ | ------------------- | ----------------------------------------- |
| Vấn đề | cao                 | ## Đo lường giai đoạn INIT của bạn        |
| Vấn đề | cao                 | ## Đặt thời gian chờ (timeout) phù hợp    |
| Vấn đề | cao                 | ## Tăng phân bổ bộ nhớ                    |
| Vấn đề | trung bình          | ## Xác minh cấu hình VPC                  |
| Vấn đề | trung bình          | ## Bảo Lambda không đợi event loop        |
| Vấn đề | trung bình          | ## Đối với tải lên tệp lớn                |
| Vấn đề | cao                 | ## Sử dụng các buckets/prefixes khác nhau |
