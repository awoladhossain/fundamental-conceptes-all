# 🏗️ System Design Learning Notes

## 📂 Topics Covered

| File | Description | Status |
|------|-------------|--------|
| [Learn 01](learn01.md) | System Design Basics | ✅ Done |
| [Event Driven Architecture](Event_driven_architecture.md) | EDA Concepts | ✅ Done |

## 🎯 Quick Reference

### Key Concepts

#### Scalability
- **Vertical Scaling** - Add more power (CPU, RAM)
- **Horizontal Scaling** - Add more machines

#### Load Balancing
- Round Robin
- Least Connections
- IP Hash
- Weighted

#### Caching
- **Client-side** - Browser cache
- **CDN** - Content Delivery Network
- **Server-side** - Redis, Memcached
- **Database** - Query cache

#### Database
- **SQL** - Structured, ACID compliant
- **NoSQL** - Flexible, eventually consistent
- **Sharding** - Horizontal partitioning
- **Replication** - Master-Slave, Master-Master

### Design Patterns

#### Microservices
```
┌─────────────┐     ┌─────────────┐
│   API GW    │────▶│  Service A  │
└─────────────┘     └─────────────┘
       │            ┌─────────────┐
       └───────────▶│  Service B  │
                    └─────────────┘
```

#### Event-Driven Architecture
```
Producer ──▶ Message Queue ──▶ Consumer
             (Kafka/RabbitMQ)
```

### Common Estimations
- 1 Million users = ~100 requests/sec
- 1 KB per request = 100 KB/sec = 8.6 GB/day
- Read:Write ratio typically 100:1

## 📚 Resources
- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [Designing Data-Intensive Applications](https://dataintensive.net/)
- [ByteByteGo](https://bytebytego.com/)

---
*Last Updated: March 2026*
