---
title: "4. Phát triển Prototype"
date: 2023-07-12T11:02:05+06:00
weight: 40
chapter: false
---

Trong phần này, chúng ta sẽ xây dựng một prototype hoạt động để chứng minh tính khả thi của giải pháp tự động hóa triển khai microservices. Chúng ta sẽ sử dụng AWS Step Functions để điều phối quy trình triển khai và AWS Lambda để thực hiện các nhiệm vụ cụ thể.

## 1. Tạo State Machine với AWS Step Functions

### Bước 1: Truy cập AWS Step Functions Console

1. Đăng nhập vào AWS Management Console
2. Tìm và chọn dịch vụ "Step Functions"
3. Nhấp vào nút "Create state machine"

![Tạo State Machine](/images/prototype/create-state-machine.png)

### Bước 2: Thiết kế Workflow

1. Chọn "Design your workflow visually" (Thiết kế workflow trực quan)
2. Đặt tên cho state machine của bạn: `MicroservicesDeploymentOrchestrator`
3. Chọn loại "Standard"

### Bước 3: Xây dựng Sơ đồ Workflow

Chúng ta sẽ xây dựng một workflow với các bước sau:

1. **Xác thực tham số đầu vào**: Kiểm tra các tham số đầu vào
2. **Sao lưu trạng thái hiện tại**: Sao lưu trạng thái hiện tại
3. **Triển khai thay đổi cơ sở dữ liệu**: Cập nhật schema cơ sở dữ liệu
4. **Triển khai dịch vụ Backend**: Triển khai các microservices backend
5. **Chạy kiểm thử tích hợp**: Chạy kiểm thử tích hợp
6. **Triển khai Frontend**: Triển khai frontend
7. **Xác minh triển khai**: Xác minh toàn bộ hệ thống
8. **Khôi phục (nếu cần)**: Khôi phục lại trạng thái trước nếu có lỗi

Sử dụng giao diện kéo thả để tạo workflow như sau:

```json
{
  "Comment": "Quy trình điều phối triển khai Microservices",
  "StartAt": "ValidateInput",
  "States": {
    "ValidateInput": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:ValidateInputFunction",
      "Next": "BackupCurrentState",
      "Catch": [
        {
          "ErrorEquals": ["ValidationError"],
          "Next": "DeploymentFailed"
        }
      ]
    },
    "BackupCurrentState": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:BackupStateFunction",
      "Next": "DeployDatabaseChanges",
      "Catch": [
        {
          "ErrorEquals": ["BackupError"],
          "Next": "DeploymentFailed"
        }
      ]
    },
    "DeployDatabaseChanges": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:DeployDBFunction",
      "Next": "DeployBackendServices",
      "Catch": [
        {
          "ErrorEquals": ["DBDeployError"],
          "Next": "RollbackChanges"
        }
      ]
    },
    "DeployBackendServices": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "DeployAuthService",
          "States": {
            "DeployAuthService": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:DeployAuthServiceFunction",
              "End": true
            }
          }
        },
        {
          "StartAt": "DeployOrderService",
          "States": {
            "DeployOrderService": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:DeployOrderServiceFunction",
              "End": true
            }
          }
        },
        {
          "StartAt": "DeployInventoryService",
          "States": {
            "DeployInventoryService": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:DeployInventoryServiceFunction",
              "End": true
            }
          }
        }
      ],
      "Next": "RunIntegrationTests",
      "Catch": [
        {
          "ErrorEquals": ["ServiceDeployError"],
          "Next": "RollbackChanges"
        }
      ]
    },
    "RunIntegrationTests": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:RunTestsFunction",
      "Next": "DeployFrontend",
      "Catch": [
        {
          "ErrorEquals": ["TestFailedError"],
          "Next": "RollbackChanges"
        }
      ]
    },
    "DeployFrontend": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:DeployFrontendFunction",
      "Next": "VerifyDeployment",
      "Catch": [
        {
          "ErrorEquals": ["FrontendDeployError"],
          "Next": "RollbackChanges"
        }
      ]
    },
    "VerifyDeployment": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:VerifyDeploymentFunction",
      "Next": "DeploymentSucceeded",
      "Catch": [
        {
          "ErrorEquals": ["VerificationError"],
          "Next": "RollbackChanges"
        }
      ]
    },
    "RollbackChanges": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:RollbackFunction",
      "Next": "DeploymentFailed"
    },
    "DeploymentSucceeded": {
      "Type": "Succeed"
    },
    "DeploymentFailed": {
      "Type": "Fail",
      "Error": "DeploymentError",
      "Cause": "Triển khai thất bại, xem nhật ký thực thi để biết chi tiết"
    }
  }
}
```

![Sơ đồ Step Functions](/images/prototype/step-functions-workflow.png)

### Bước 4: Tạo và Lưu State Machine

1. Nhấp vào nút "Create state machine"
2. Xác nhận IAM role cho state machine (tạo role mới hoặc sử dụng role hiện có)
3. Nhấp vào "Create state machine" để hoàn tất

## 2. Tạo Lambda Functions

Bây giờ chúng ta cần tạo các Lambda functions được tham chiếu trong state machine. Chúng ta sẽ tạo một function làm ví dụ:

### Tạo ValidateInputFunction

1. Truy cập AWS Lambda console
2. Nhấp vào "Create function"
3. Chọn "Author from scratch"
4. Nhập thông tin:
   - Function name: `ValidateInputFunction`
   - Runtime: Node.js 14.x
   - Architecture: x86_64
   - Permissions: Tạo role mới với quyền Lambda cơ bản
5. Nhấp vào "Create function"

![Tạo Lambda Function](/images/prototype/create-lambda.png)

6. Trong phần code editor, thay thế code mặc định bằng:

```javascript
exports.handler = async (event) => {
    console.log('Đang xác thực tham số triển khai:', JSON.stringify(event));
    
    // Kiểm tra các tham số bắt buộc
    if (!event.version || !event.services || !event.environment) {
        throw new Error('ValidationError: Thiếu tham số bắt buộc');
    }
    
    // Kiểm tra môi trường hợp lệ
    const validEnvironments = ['dev', 'staging', 'production'];
    if (!validEnvironments.includes(event.environment)) {
        throw new Error('ValidationError: Môi trường không hợp lệ');
    }
    
    // Kiểm tra các dịch vụ cần triển khai
    const validServices = ['auth', 'order', 'inventory', 'frontend'];
    for (const service of event.services) {
        if (!validServices.includes(service)) {
            throw new Error(`ValidationError: Dịch vụ không hợp lệ: ${service}`);
        }
    }
    
    // Trả về input đã được xác thực
    return {
        ...event,
        timestamp: new Date().toISOString(),
        validationPassed: true
    };
};
```

7. Nhấp vào "Deploy" để lưu function

### Tạo các Lambda Functions khác

Lặp lại các bước tương tự để tạo các Lambda functions còn lại được tham chiếu trong state machine. Mỗi function sẽ thực hiện một nhiệm vụ cụ thể trong quy trình triển khai.

## 3. Kiểm thử Workflow

### Bước 1: Chạy thử State Machine

1. Quay lại AWS Step Functions console
2. Chọn state machine `MicroservicesDeploymentOrchestrator` của bạn
3. Nhấp vào "Start execution"
4. Nhập JSON input sau:

```json
{
  "version": "1.0.0",
  "environment": "dev",
  "services": ["auth", "order", "inventory", "frontend"],
  "rollbackOnFailure": true,
  "notifyOnCompletion": true,
  "notificationEmail": "admin@example.com"
}
```

5. Nhấp vào "Start execution"

### Bước 2: Theo dõi thực thi

1. Quan sát quá trình thực thi trực quan trên giao diện Step Functions
2. Kiểm tra từng bước để đảm bảo chúng thực thi đúng
3. Xem logs của từng Lambda function để debug nếu cần

![Trực quan hóa thực thi](/images/prototype/execution-visualization.png)

## 4. Tích hợp với CI/CD Pipeline

Để tích hợp workflow này vào CI/CD pipeline, chúng ta có thể sử dụng AWS CodePipeline:

1. Tạo một CodePipeline mới
2. Cấu hình source stage (ví dụ: GitHub, CodeCommit)
3. Thêm stage để gọi Step Functions state machine
4. Cấu hình các thông báo và cổng phê duyệt

## 5. Tạo API Gateway để kích hoạt Workflow

Để kích hoạt workflow từ bên ngoài:

1. Tạo một API trong API Gateway
2. Tạo một resource và method (POST /deploy)
3. Tích hợp với Step Functions state machine
4. Triển khai API và kiểm thử

## Tổng kết

Trong phần này, chúng ta đã:

1. Tạo một Step Functions state machine để điều phối quy trình triển khai microservices
2. Phát triển các Lambda functions để thực hiện các nhiệm vụ cụ thể
3. Kiểm thử workflow để đảm bảo nó hoạt động đúng
4. Tìm hiểu cách tích hợp với CI/CD pipeline và API Gateway

Prototype này chứng minh tính khả thi của giải pháp tự động hóa triển khai microservices, giúp giảm thời gian triển khai từ hàng giờ xuống còn vài phút và giảm thiểu lỗi do con người.

Trong phần tiếp theo, chúng ta sẽ xây dựng business case để thuyết phục các bên liên quan về giá trị của giải pháp này. 