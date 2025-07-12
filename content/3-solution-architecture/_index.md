---
title: "3. Solution Architecture Design"
date: 2023-07-12T11:02:05+06:00
weight: 30
chapter: false
---

## Learning Objective  
Learn systematic approach to design technical architecture that addresses business requirements.

## Architecture Decision Framework (7 phút)

**📸 IMAGE NEEDED: Decision Framework Diagram**
*Chụp flowchart showing: Requirements → Constraints → Options → Evaluation → Decision → Documentation*

### Hands-on: Technology Selection Workshop

#### Step 1: Service Mapping Exercise (2 phút)
```markdown
Draw service relationship diagram on whiteboard:

Authentication Service
├── Required by: Order, Payment
├── Dependencies: None
├── Criticality: High
└── Deployment time: 2 min

Order Service  
├── Required by: Payment, Notification
├── Dependencies: Authentication
├── Criticality: High
└── Deployment time: 3 min

Payment Service
├── Required by: Notification
├── Dependencies: Authentication, Order
├── Criticality: Critical
└── Deployment time: 2 min

Notification Service
├── Required by: None
├── Dependencies: Order, Payment (optional)
├── Criticality: Medium  
└── Deployment time: 1 min
```

#### Step 2: Orchestration Technology Evaluation (3 phút)

**📸 IMAGE NEEDED: Technology Comparison Matrix**
*Chụp comparison table giữa AWS Step Functions, Jenkins, GitHub Actions, custom solution*

```markdown
Evaluation Matrix:

| Criteria | AWS Step Functions | Jenkins Pipeline | GitHub Actions | Custom Lambda |
|----------|-------------------|------------------|----------------|---------------|
| Visual Workflow | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐ |
| Error Handling | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| AWS Integration | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| Cost | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Maintenance | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| Learning Curve | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐ |
| Scalability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |

Scoring: 5=Excellent, 4=Good, 3=Average, 2=Below Average, 1=Poor

Total Scores:
- AWS Step Functions: 32/35
- Jenkins Pipeline: 19/35  
- GitHub Actions: 24/35
- Custom Lambda: 23/35

Selected: AWS Step Functions
```

#### Step 3: Architecture Decision Record (2 phút)
```markdown
Template ADR-001: Orchestration Service Selection

Status: Proposed → Under Review → Approved → Superseded

Context:
Need workflow orchestration for complex microservice deployments
Current manual process is error-prone và time-consuming
Team has moderate AWS experience

Decision:
Use AWS Step Functions as primary orchestration engine

Alternatives Considered:
1. Jenkins Pipeline - rejected due to infrastructure overhead
2. GitHub Actions - rejected due to limited AWS integration  
3. Custom Lambda solution - rejected due to development complexity

Consequences:
Positive:
+ Visual workflow representation aids debugging
+ Built-in retry logic và error handling
+ Serverless reduces operational overhead
+ Native AWS service integration
+ Pay-per-use pricing model

Negative:
- AWS vendor lock-in
- Learning curve for Step Functions DSL
- State machine size limits
- Cold start latency for infrequent workflows

Implementation Notes:
- Use Express Workflows for high-frequency operations
- Implement comprehensive logging for debugging
- Create reusable state machine templates
```

## High-Level Architecture Design (8 phút)

**📸 IMAGE NEEDED: Architecture Sketching Process**
*Chụp sequence: whiteboard sketch → digital diagram → AWS architecture icons*

### Hands-on: Collaborative Architecture Design

#### Step 1: Component Identification (2 phút)
```markdown
Brainstorm essential components:

Orchestration Layer:
├── Step Functions State Machine
├── Lambda Functions (controllers)
└── CloudWatch Events (triggers)

Application Layer:
├── ECS Services (microservices)
├── Application Load Balancer
└── Target Groups

Data Layer:
├── RDS (if needed)
├── ElastiCache (if needed)
└── S3 (artifacts, logs)

Monitoring Layer:
├── CloudWatch Logs
├── CloudWatch Metrics  
├── CloudWatch Dashboards
└── SNS Notifications

Security Layer:
├── IAM Roles và Policies
├── VPC và Security Groups
└── AWS Secrets Manager
```

#### Step 2: Data Flow Mapping (3 phút)

**📸 IMAGE NEEDED: Data Flow Diagram**
*Chụp diagram showing request flow từ trigger đến completion*

```markdown
Deployment Trigger Flow:

1. Git Push → CodePipeline → S3 Artifact
2. S3 Event → Lambda Trigger → Step Functions
3. Step Functions → Deployment Initializer Lambda
4. Initializer → ECS Service Updates (parallel)
5. ECS → Health Check Lambda (each service)
6. Health Check → Success/Failure → Next State
7. Final State → Notification Lambda → SNS
8. SNS → Email/Slack → Stakeholders

Error Flow:
Any Step Fails → Rollback State → Previous Versions → Notification
```

#### Step 3: Architecture Diagram Creation (3 phút)

**📸 IMAGE NEEDED: Digital Architecture Diagram Tool**
*Chụp draw.io hoặc Lucidchart với AWS architecture symbols*

```markdown
Participants create digital diagram using:
- draw.io (free, web-based)
- AWS Architecture Icons
- Standard notation for data flow
- Color coding for different layers

Required elements:
☐ All identified components
☐ Data flow arrows với labels
☐ Security boundaries (VPC, subnets)
☐ External integrations
☐ Monitoring touchpoints
☐ Error handling paths
```

## Component Selection Matrix (10 phút)

**📸 IMAGE NEEDED: Technology Comparison Matrix**
*Chụp comparison table giữa different AWS services và alternatives*

```markdown
| Component | Option 1 | Option 2 | Option 3 | Selected | Rationale |
|-----------|----------|----------|----------|----------|-----------|
| Orchestration | Step Functions | Jenkins Pipeline | GitHub Actions | Step Functions | Serverless, visual, error handling |
| Compute | Lambda | ECS Fargate | EC2 | Lambda | Serverless, cost-effective |
| Container Orchestration | ECS | EKS | Fargate | ECS | Managed, cost-effective |
| Monitoring | CloudWatch | Datadog | New Relic | CloudWatch | Native integration, cost |
| Notifications | SNS | Slack API | Email | SNS | Multi-channel, reliable |
```

## Detailed Component Specification (5 phút)

**📸 IMAGE NEEDED: Component Specification Template**
*Chụp detailed spec sheet template với technical parameters*

### Lambda Functions Specification:
```yaml
# lambda_functions/deployment_initializer.yaml
name: deployment-initializer
runtime: python3.9
memory: 512MB
timeout: 300 seconds
environment_variables:
  LOG_LEVEL: INFO
  ECS_CLUSTER: restaurant-cluster
permissions:
  - ecs:UpdateService
  - ecs:DescribeServices
  - logs:CreateLogGroup
triggers:
  - step_functions
```

### Step Functions Workflow Specification:
```json
{
  "Comment": "Restaurant Microservices Deployment",
  "StartAt": "InitializeDeployment",
  "States": {
    "InitializeDeployment": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:region:account:function:deployment-initializer",
      "Next": "DeployAuthService",
      "Retry": [
        {
          "ErrorEquals": ["States.TaskFailed"],
          "IntervalSeconds": 5,
          "MaxAttempts": 3,
          "BackoffRate": 2.0
        }
      ],
      "Catch": [
        {
          "ErrorEquals": ["States.ALL"],
          "Next": "HandleFailure"
        }
      ]
    }
  }
}
``` 