# Netflix Architecture Reference

A reference guide to Netflix's microservices and cloud architecture patterns.

---

## Overview

Netflix serves **700+ million streaming hours** per day across 190+ countries using a fully cloud-native microservices architecture hosted on AWS.

---

## Core Architecture Layers

```
┌─────────────────────────────────────────────┐
│              Client Devices                  │
│     (TV, Mobile, Browser, Console)          │
└────────────────────┬────────────────────────┘
                     │
┌────────────────────▼────────────────────────┐
│              AWS CloudFront CDN              │
│           (Static Assets + OCA)             │
└────────────────────┬────────────────────────┘
                     │
┌────────────────────▼────────────────────────┐
│               API Gateway                    │
│            (Zuul / Netflix Edge)            │
└────────────────────┬────────────────────────┘
                     │
┌────────────────────▼────────────────────────┐
│           Microservices Layer                │
│  (100s of independent services on AWS EC2)  │
└────────────────────┬────────────────────────┘
                     │
┌────────────────────▼────────────────────────┐
│              Data Layer                      │
│   (Cassandra, MySQL, S3, Elasticsearch)     │
└─────────────────────────────────────────────┘
```

---

## Key Components

### 1. API Gateway — Zuul
- Acts as the front door for all client requests
- Handles routing, filtering, authentication, and rate limiting
- Runs on the edge of the cloud

### 2. Service Discovery — Eureka
- Each microservice registers itself with Eureka on startup
- Other services query Eureka to locate dependencies
- Enables dynamic scaling without hardcoded IPs

### 3. Load Balancing — Ribbon
- Client-side load balancer
- Works with Eureka to distribute traffic across service instances
- Supports round-robin, weighted, and availability-filtered strategies

### 4. Circuit Breaker — Hystrix
- Prevents cascading failures across services
- Falls back to cached/default responses when a service is down
- Monitors failure rates and opens the circuit automatically

### 5. Inter-Service Communication — Feign
- Declarative HTTP client for calling other microservices
- Integrates with Ribbon (load balancing) and Hystrix (circuit breaking)

### 6. Messaging — Apache Kafka
- Event streaming between services
- Used for real-time analytics, recommendations, and activity tracking
- Handles billions of events per day

### 7. Configuration — Archaius
- Dynamic, distributed configuration management
- Allows config changes without redeployment
- Supports A/B testing via feature flags

---

## Data Storage Strategy

| Store | Usage |
|---|---|
| **Apache Cassandra** | User viewing history, ratings, billing data |
| **Amazon S3** | Video content, backups, data lake |
| **MySQL (RDS)** | Billing and subscriber data |
| **Elasticsearch** | Search and logging |
| **EVCache (Memcached)** | Distributed caching layer |
| **Apache Iceberg** | Big data / data warehouse queries |

---

## Content Delivery — Open Connect (OCA)

Netflix built its own CDN called **Open Connect Appliances (OCA)**:
- Physical servers placed inside ISP data centers worldwide
- Pre-loads popular content during off-peak hours
- Reduces bandwidth costs and improves streaming quality
- Serves **95%+ of video traffic** directly from ISPs

---

## Resilience Patterns

### Chaos Engineering — Chaos Monkey
Netflix intentionally kills random services in production to:
- Test system resilience
- Ensure automatic failover works
- Build confidence in the system

Tools in the **Simian Army**:
- `Chaos Monkey` — kills random instances
- `Latency Monkey` — introduces artificial delays
- `Conformity Monkey` — checks instances follow best practices
- `Chaos Gorilla` — simulates full AWS Availability Zone outage

---

## Recommendation Engine

```
User Activity → Kafka → Spark Streaming → ML Models → Personalized Homepage
```

- Tracks every play, pause, rewind, search, and rating
- Uses collaborative filtering + deep learning models
- Runs on **Apache Spark** for batch processing
- Serves recommendations via dedicated microservices

---

## CI/CD Pipeline

```
Code Commit → Spinnaker (CI/CD) → Canary Deployment → Full Rollout
```

- **Spinnaker** — open-source multi-cloud CD platform (built by Netflix)
- **Canary deployments** — roll out to 1% of users first, monitor, then expand
- **Red/Black deployments** — zero-downtime switchover between versions
- Deploys **1000s of times per day**

---

## Observability Stack

| Tool | Purpose |
|---|---|
| **Atlas** | Time-series metrics at scale |
| **Zipkin** | Distributed tracing |
| **Kibana + ELK** | Log aggregation and search |
| **Mantis** | Real-time stream processing for ops data |

---

## Key Takeaways for .NET Microservices

| Netflix Pattern | .NET Equivalent |
|---|---|
| Eureka (Service Discovery) | Consul / Azure Service Fabric |
| Ribbon (Load Balancing) | YARP / Nginx |
| Hystrix (Circuit Breaker) | Polly |
| Zuul (API Gateway) | Ocelot / YARP |
| Feign (HTTP Client) | Refit / HttpClientFactory |
| Kafka (Messaging) | MassTransit + RabbitMQ / Azure Service Bus |
| Spinnaker (CD) | GitHub Actions + Azure DevOps |

---

## References

- [Netflix Tech Blog](https://netflixtechblog.com)
- [Netflix OSS GitHub](https://github.com/Netflix)
- [Chaos Engineering Book by Netflix](https://www.oreilly.com/library/view/chaos-engineering/9781491988459/)
- [Open Connect CDN](https://openconnect.netflix.com)
