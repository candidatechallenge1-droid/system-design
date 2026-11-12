## Design Philosophy

When approaching a system design problem, I use a top-down approach to ensure architecture decisions are driven by business requirements rather than technology preferences.

```text
Business Requirements
        ↓
Customer Journey
        ↓
Domain Services
        ↓
API Design
        ↓
Data Model
        ↓
Integration Pattern
        ↓
Event Design
        ↓
Security
        ↓
Resilience
        ↓
Observability
        ↓
Scalability
        ↓
AWS / Cloud Deployment
        ↓
Trade-Off Analysis
```

---

# Payment Authorization Platform Example

## Business Requirement

Allow merchants to securely submit payment authorization requests and receive approval or decline decisions.

### Functional Requirements

```text
Authorize payments

Perform fraud checks

Integrate with card processors

Notify merchant

Maintain audit trail
```

### Non-Functional Requirements

```text
High Availability

Low Latency

Scalability

Security

PCI Compliance

Observability

Auditability
```

---

# Customer Journey

```text
Customer clicks Pay
        ↓
Merchant submits payment request
        ↓
Payment Authorization begins
        ↓
Fraud Checks
        ↓
Card Authorization
        ↓
Authorization Decision
        ↓
Merchant receives response
        ↓
Webhook Notification
```

---

# Domain Services

```text
API Gateway

Orchestrator Service

Authorization Service

Fraud Service

Credit Card Payment Service

Notification Service

Webhook Service

Audit Service

Crypto Service
```

### Responsibility Principle

```text
Each service owns one business capability.
```

---

# High-Level Architecture

```text

                Merchant / Client

                        |
                        v

                +---------------+
                | API Gateway   |
                +---------------+

                        |
                        v

                +---------------+
                | Orchestrator  |
                | Service       |
                +---------------+

                        |
                        v

====================================================
                    Kafka
====================================================

                payment-auth-request

                        |
                        |
       +----------------+----------------+
       |                                 |
       v                                 v

 +--------------+               +---------------+
 | Fraud        |               | Credit Card   |
 | Service      |               | Service       |
 +--------------+               +---------------+

                        |
                        |
                        v

                payment-auth-result

                        |
                        v

              +------------------+
              | Authorization    |
              | Service          |
              +------------------+

                        |
          +-------------+-------------+
          |                           |
          v                           v

 payment-authorized        payment-declined

          |                           |

   +------+------+------+      +------+------+
   |      |      |      |      |      |      |
   v      v      v      v      v      v      v

Settlement Notification Audit Webhook Notification Audit
```

---

# API Design

## Create Authorization Request

```http
POST /v1/payments/authorize
```

### Request

```json
{
  "paymentReference":"ORD123",
  "merchantId":"M001",
  "amount":100.00,
  "currency":"AUD"
}
```

### Response

```json
{
  "paymentId":"PAY123",
  "status":"PENDING"
}
```

### Design Considerations

```text
Versioning

Validation

Idempotency

Error Handling

Security
```

---

# Data Model

## Payment

```sql
payment_id
payment_reference
merchant_id
customer_id
amount
currency
status
created_date
updated_date
```

### Idempotency Protection

```sql
UNIQUE(payment_reference)
```

Prevents duplicate payment processing.

---

# Integration Pattern

## External Clients

```text
REST API
```

Reason:

```text
Immediate acknowledgement required.
```

---

## Internal Services

```text
Kafka
```

Reason:

```text
Loose coupling

Independent scaling

Replay capability

Failure isolation
```

---

# Event Design

## Topic

```text
payment-auth-request
```

Example:

```json
{
  "eventId":"UUID",
  "paymentId":"PAY123",
  "customerId":"CUST001",
  "amount":100.00
}
```

---

## Topic

```text
payment-auth-result
```

Example:

```json
{
  "paymentId":"PAY123",
  "service":"FRAUD",
  "decision":"PASS"
}
```

---

## Topic

```text
payment-authorized
```

Consumers:

```text
Settlement Service

Notification Service

Audit Service

Webhook Service
```

---

## Topic

```text
payment-declined
```

Consumers:

```text
Notification Service

Audit Service

Webhook Service
```

---

# Kafka Design

## Producers

```text
Orchestrator Service

Fraud Service

Credit Card Service

Authorization Service
```

---

## Consumers

```text
Fraud Service

Credit Card Service

Authorization Service

Settlement Service

Notification Service

Audit Service

Webhook Service
```

---

## Partition Key

```text
CustomerId

or

AccountId
```

Reason:

```text
Ordering Guarantee
```

Example:

```text
Payment Created

Payment Authorized

Payment Settled

Payment Refunded
```

All processed in order.

---

## Kafka Cluster

```text
Broker-1
Broker-2
Broker-3
```

Replication Factor:

```text
3
```

Benefits:

```text
High Availability

Fault Tolerance
```

---

# Security Design

## Authentication

```text
OAuth2

OpenID Connect

JWT
```

---

## Authorization

```text
Role Based Access Control
```

Examples:

```text
payment.read

payment.write
```

---

## Encryption

### In Transit

```text
TLS
```

### At Rest

```text
AWS KMS
```

---

## Secret Management

```text
AWS Secrets Manager
```

---

## Sensitive Data Protection

Never expose:

```text
PAN

CVV

PII
```

Mask fields:

```text
************1234
```

---

# Crypto Service

Responsibilities:

```text
Tokenization

Encryption

Decryption

Hashing

Signature Verification
```

Purpose:

```text
Keep cryptographic concerns separate
from business services.
```

---

# Resilience

## Retry

Used for transient failures.

Example:

```text
Card Processor Timeout
```

---

## Circuit Breaker

Used when downstream systems fail.

Example:

```text
Issuer API unavailable
```

---

## Bulkhead

Isolate failures.

Example:

```text
Fraud Service failure
should not affect
Notification Service.
```

---

## Dead Letter Queue

```text
payment-auth-request-dlq
```

Stores poison messages.

---

## Idempotency

Every payment request includes:

```text
Idempotency-Key
```

Benefits:

```text
Retry Safe

No Double Charging
```

---

# Observability

## Logging

Structured JSON logs.

Include:

```text
CorrelationId

PaymentId

MerchantId
```

---

## Metrics

Monitor:

```text
Authorization Success %

Authorization Failure %

TPS

P95 Latency

Kafka Consumer Lag
```

---

## Distributed Tracing

Track requests across:

```text
API Gateway
↓
Orchestrator
↓
Kafka
↓
Fraud Service
↓
Authorization Service
```

Tools:

```text
OpenTelemetry

AWS X-Ray
```

---

# Scalability

## Horizontal Scaling

```text
More service instances
```

instead of:

```text
Larger servers
```

---

## Kafka Scaling

```text
More partitions

More consumers
```

---

## Redis

Used for:

```text
Caching

Reference Data

Session Data
```

---

## Database Scaling

```text
Read Replicas

Connection Pooling

Partitioning
```

---

# AWS Deployment

## Compute

```text
ECS Fargate

or

EKS
```

---

## Messaging

```text
Amazon MSK
```

---

## Database

```text
Amazon RDS PostgreSQL
```

---

## Monitoring

```text
CloudWatch

OpenTelemetry

AWS X-Ray
```

---

## Security

```text
IAM

Secrets Manager

KMS
```

---

# Trade-Off Discussion

## Why Kafka?

### Benefits

```text
Loose Coupling

Replay Capability

Scalability

Durability
```

### Trade-Offs

```text
Operational Complexity

Eventual Consistency

Harder Debugging
```

---

## Why Async Processing?

### Benefits

```text
Higher Throughput

Failure Isolation

Independent Scaling
```

### Trade-Offs

```text
Eventual Consistency

Longer End-to-End Workflow
```

---

## Why Orchestrator Pattern?

### Benefits

```text
Centralized Workflow

Operational Visibility

Business Flow Control
```

### Trade-Offs

```text
Additional Service

Potential Bottleneck

Additional Operational Overhead
```

---

# Interview Closing Statement

> My approach is to start with business requirements and customer journey before discussing technology. Once the domain services, APIs, and data ownership are identified, I define integration patterns, security controls, resilience strategies, observability requirements, and scalability considerations. Finally, I select technologies such as Spring Boot, Kafka, Redis, PostgreSQL, AWS, and OpenTelemetry and explain the trade-offs behind each decision.