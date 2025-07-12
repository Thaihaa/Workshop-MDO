---
title: "2. Requirements Analysis & Problem Definition"
date: 2023-07-12T11:02:05+06:00
weight: 20
chapter: false
---

## Learning Objective
Học cách thu thập và document business requirements từ stakeholders để build compelling business case.

## Stakeholder Interview Simulation (5 phút)

**📸 IMAGE NEEDED: Stakeholder Personas**
*Chụp slide với 3 persona cards showing CTO, Operations Manager, CFO với their key concerns*

### Setup Role-play Exercise:
```markdown
Instructor Setup:
1. Chia nhóm thành 3 teams: Business Analysts, Stakeholders, Observers
2. Provide stakeholder persona cards
3. Set timer cho mỗi interview session (1.5 phút each)
4. Record key requirements on whiteboard
```

### CTO Persona - Technical Requirements
```
"We need a solution that can:
- Deploy 4 microservices (auth, order, payment, notification) reliably
- Handle dependency management automatically  
- Provide rollback capability within 2 minutes
- Support multi-environment deployments (dev, staging, prod)
- Integrate with our existing CI/CD pipeline"

Questions to ask:
- What's the current failure rate?
- How often do you deploy?
- What's the impact of deployment downtime?
- What compliance requirements do we have?
```

### Operations Manager Persona - Operational Requirements
```
"Current pain points include:
- 2-3 staff members tied up for 4 hours per deployment
- Weekend deployment windows impact customer service
- Manual coordination errors cause system outages
- No visibility into deployment progress or issues"

Questions to ask:
- How many people involved in current process?
- What's the cost of deployment windows?
- What monitoring tools do you currently use?
- What training would your team need?
```

### CFO Persona - Business Requirements
```
"Show me numbers. Current deployments cost us:
- Developer overtime: $2,000 per deployment
- Weekend premium pay: $1,500 per deployment  
- Lost revenue during downtime: $5,000 per incident
- Customer service complaints: $500 per incident

We deploy 26 times per year. I need to see clear ROI
within 12 months và operational cost reduction."

Questions to ask:
- What's the budget range for this project?
- What ROI percentage would be acceptable?
- Are there compliance costs to consider?
- How do you measure business impact?
```

## Requirements Documentation Workshop (10 phút)

**📸 IMAGE NEEDED: Requirements Gathering Template**
*Chụp Google Sheets template với columns for Requirement ID, Description, Source, Priority, Acceptance Criteria*

### Hands-on Exercise: Requirements Matrix Creation

#### Step 1: Functional Requirements (3 phút)
```markdown
Participants sẽ fill out template:

| ID | Requirement | Stakeholder | Priority | Acceptance Criteria |
|----|-------------|-------------|----------|-------------------|
| FR-001 | Automated deployment | CTO | Critical | Deploy all 4 services in <10 min |
| FR-002 | Dependency management | CTO | Critical | Auto-resolve service dependencies |
| FR-003 | Rollback capability | CTO | Critical | Rollback within 2 minutes |
| FR-004 | Health monitoring | Ops | High | Real-time service health status |
| FR-005 | Multi-environment | Ops | Medium | Support dev/staging/prod |
| FR-006 | Cost tracking | CFO | High | Detailed cost breakdown per deployment |

Add more requirements based on interview notes...
```

#### Step 2: Non-Functional Requirements (3 phút)
```markdown
| Category | Current State | Target State | Business Impact |
|----------|---------------|-------------|-----------------|
| Performance | 4 hours deployment | <10 minutes | $156K annual savings |
| Reliability | 70% success rate | 98% success rate | Reduce incidents by 95% |
| Availability | 95% uptime | 99.9% uptime | $65K reduced downtime costs |
| Scalability | 1 concurrent deployment | 10+ concurrent | Support business growth |
| Security | Manual approvals | Automated with audit | Compliance requirement |
```

#### Step 3: Requirements Prioritization (2 phút)
```markdown
Priority Framework:
Critical: Must have for MVP (deal breakers)
High: Should have for launch  
Medium: Nice to have for v1.1
Low: Future consideration

Participants vote on each requirement using dot stickers:
🔴 Critical
🟡 High  
🟢 Medium
⚪ Low
```

#### Step 4: Constraints & Assumptions (2 phút)
```markdown
Document key constraints:
- Budget: $150K maximum
- Timeline: 16 weeks
- Team: 3 developers, 1 ops engineer
- Technology: Must use AWS
- Compliance: SOC 2 Type II required
- Integration: Must work with existing CI/CD

Document assumptions:
- Current AWS infrastructure available
- Team has basic AWS knowledge
- No major org changes during project
- Stakeholder availability for testing
``` 