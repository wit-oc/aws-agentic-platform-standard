# One-Page Architecture Bootstrap

## Goal
Give implementation teams a no-fluff starting map for standing up an enterprise agentic platform in AWS.

## Architecture Diagram

```mermaid
flowchart LR
    C[Client/API/ChatOps] --> G[API Gateway]
    G --> R[Agent Runtime]
    R --> O[Step Functions]
    O <--> E[EventBridge + SQS]
    R --> B[Amazon Bedrock]
    R --> T[Tool Gateway]
    R --> D[DynamoDB Session State]
    R --> V[Vector Memory]
    R --> S[S3 Artifacts]
    R --> X[OTel + CloudWatch/X-Ray]

    subgraph ControlPlane[Control Plane]
      A[Tenant + Registry + Policy]
      P[CI/CD + Eval Gates]
      U[Audit + Cost Dashboards]
    end

    A --> R
    P --> R
    X --> U
    T --> Z[Enterprise Systems]
```

## Build Order (Minimal)
1. Runtime contract + single-agent endpoint
2. Step Functions orchestration skeleton
3. Bedrock invocation policy + budget limits
4. Tool gateway with schema + risk class
5. Session state + artifacts
6. Telemetry + alarms
7. CI/CD promotion gates
