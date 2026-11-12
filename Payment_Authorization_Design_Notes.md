# Payment Authorization Platform - Interview Notes

## Top-Down Approach
Business Requirement -> Customer Journey -> Domain Services -> API Design -> Event Flow -> Security -> Observability -> Scalability -> Technology

## Core Services
- API Gateway
- Orchestrator Service
- Authorization Service
- Credit Card Payment Service
- Fraud Service
- Notification Service
- Audit Service
- Webhook Service
- Crypto Service

## HLD
Merchant -> API Gateway -> Orchestrator -> Kafka

Topics:
- payment-auth-request
- payment-auth-result
- payment-authorized
- payment-declined

Consumers:
- Fraud Service
- Credit Card Service
- Authorization Service
- Notification Service
- Audit Service
- Webhook Service

## Security
- OAuth2 / JWT
- TLS
- AWS Secrets Manager
- AWS KMS
- Role-based authorization
- Mask PAN / PII

## Observability
- Structured Logging
- Metrics
- Distributed Tracing
- Correlation IDs
- CloudWatch / OpenTelemetry

## Resilience
- Retry
- Circuit Breaker
- Bulkhead
- Dead Letter Queue
- Idempotency Keys

## AWS
- ECS/EKS
- RDS PostgreSQL
- MSK Kafka
- Secrets Manager
- KMS
- CloudWatch
- X-Ray

## Trade-offs
Kafka:
+ Decoupling, scalability, replayability
- Operational complexity, eventual consistency

Orchestrator:
+ Centralized workflow
- Additional component and potential bottleneck
