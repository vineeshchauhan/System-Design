# System Design Concepts

## 1. Scalability

### Definition
Scalability is the ability of a system to handle increasing traffic (users, requests, data) without breaking by adding resources (horizontal or vertical scaling).

### Key Principles

a. **System-wide Property**: Scalability is the property of the entire system. The system is as scalable as its least scalable part.

b. **End-to-End Consideration**: A backend server can scale to handle 10x traffic, but the database must also be scalable to support the increased connection requests.

c. **Measurable Capacity**: Scalability should ideally be expressed as measurable capacity + how that capacity changes when resources increase.

### Scalability Metrics Example

| Metric | 1× load | 10× load |
|--------|---------|----------|
| Traffic | 10K RPS | 100K RPS |
| App servers | 5 | 50 |
| p99 latency | 150 ms | 180 ms |
| Error rate | 0.05% | 0.08% |
| DB CPU | 55% | 65% |
| DB capacity | 15K RPS | 120K RPS |

---

## 2. Load Balancer

### Overview
A load balancer distributes incoming requests across multiple servers.

### Key Functions

a. **Request Distribution**: Distributes incoming requests across multiple servers.

b. **Health Checks**: Performs health checks on backend servers.

c. **Horizontal Scaling**: Supports horizontal scaling by enabling addition of more servers.

### Load Balancer Types

#### L4 (Layer 4) - TCP/UDP Level
- Routes based on network information
- L4 sees: Source IP, Destination IP, Source Port, Destination Port, TCP/UDP

#### L7 (Layer 7) - HTTP Level
- Routing decision based on HTTP method, headers, paths, host, and cookies

### Load Balancer Algorithms

1. **Round Robin**: Distributes requests sequentially across servers
2. **Least Connections**: Sends request to server with fewest active connections
3. **IP Hash**: Uses client IP address to determine target server
4. **Consistent Hashing**: Maintains cache coherence when servers are added/removed

### Load Balancer Capacity

| Metric | Capacity |
|--------|----------|
| Requests/sec | 1 million RPS |
| New connections/sec | 100K/sec |
| Concurrent connections | 5 million |
| Throughput | 20 Gbps |
| TLS handshakes/sec | 50K/sec |
| P99 Latency | < 20 ms |

---

### Rate Limiting Architecture

Load balancers often implement a two-layer rate limiting approach:

#### a. Edge Protection (at Load Balancer)
- DDoS protection
- Abusive IPs
- Traffic floods
- Obviously malicious traffic

#### b. API/Business Rate Limiting (at API Gateway)
Protects API based on:
- User
- API keys
- Tenant
- Endpoints
- Subscription tier
- Business rules

### System Architecture Diagram

```mermaid
flowchart TD
    Internet[INTERNET] --> Edge[DDoS / WAF / Edge]
    Edge --> LoadBalancer[LOAD BALANCER]
    
    subgraph Backend[Backend]
        LoadBalancer --> App1[App1]
        LoadBalancer --> App2[App2]
        LoadBalancer --> App3[App3]
        
        App1 --> Database
        App2 --> Database
        App3 --> Database
    end
    
    Database[Database]
```

## 3. Latency

### Definition
Latency is the time between sending a request and receiving a response. Latency is measured in percentiles.

### Percentile-based Measurement

- **p50** = 50th percentile (median)
- **p95** = 95th percentile
- **p99** = 99th percentile

#### Example
```
p50 = 50 ms
p95 = 120 ms
p99 = 300 ms
```

**Interpretation**: 1% of requests are taking more than 300 ms.

#### When to use Average vs Percentiles?
- **Average latency** can hide tail latency. Use average to understand overall behavior.
- **Percentiles** help understand user experience and tail latency.

### Latency Components

Latency is composed of multiple components:

| Component | Description |
|-----------|-------------|
| Network Latency | Time for data to travel across the network |
| LB Processing | Load balancer processing time |
| Application Processing | Server application logic execution |
| Redis Latency | Cache lookup/update time |
| Database Latency | Database query execution |

#### Example Breakdown
```
Network        30 ms
LB              5 ms
Application    20 ms
Redis           2 ms
DB             50 ms
-------------------
Total         107 ms
```

### Key Questions to Consider

a. Can my server process the required traffic while keeping p95/p99 latency within our SLO?

b. Every network hop adds latency.

c. **How to reduce latency:**
   - Keep the server/DB near to the user
   - Use caching

d. **Reasons for high p99 latency (e.g., 500ms):**
   - User far from server (network latency)
   - High traffic - CPU/threads/connections become saturated
   - Background processing - CPU contention, thread contention or GC pause
   - DB background tasks - DB contentions, locks, I/O, connection pool exhaustion, slow queries
   - Downstream service latency

### Latency Diagram

```mermaid
flowchart TD
    LATENCY --> Network
    LATENCY --> Compute
    LATENCY --> Waiting

    subgraph Network
        Distance
        TCP_TLS
        Network_hops
    end

    subgraph Compute
        CPU_DB
        Redis
        Code
    end

    subgraph Waiting
        Queues_locks
        Thread_pools
        Connection_pools
    end
```

---
