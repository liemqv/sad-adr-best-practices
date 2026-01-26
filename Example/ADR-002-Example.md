# ADR-002: PostgreSQL for Transactional Data

**Status**: Accepted  
**Date**: 2026-01-26  
**Deciders**: Architecture Team, Database Team, CTO  
**Tags**: database, postgresql, transactional, acid, data-persistence

---

## Context

**What is the issue or problem we're facing?**

Following the decision to adopt microservices architecture (ADR-001), we need to select a database solution for transactional data storage. The RideShare platform requires:

- **ACID Compliance**: Critical for financial transactions (payments, driver payouts)
- **Strong Consistency**: User data, trip records, and payment records must be consistent
- **Geospatial Support**: Need to store and query location data (coordinates, distances, geospatial searches)
- **Relational Data Model**: Our data has clear relationships (User → Ride → Payment, Driver → Vehicle, etc.)
- **Transaction Support**: Need to support complex transactions across multiple tables
- **Performance**: Must handle 100,000+ transactions per day with sub-100ms query times
- **Scalability**: Support read replicas for read-heavy workloads
- **Maturity & Reliability**: Proven track record for production systems

**Current State:**
- No existing database (new system)
- Need to support multiple services (User Service, Trip Service, Payment Service)
- Each service will have its own database (database per service pattern)

**Constraints:**
- Must be open-source or have reasonable licensing costs
- Team has experience with SQL databases
- Must support cloud-managed services (AWS RDS)
- Need to support automated backups and point-in-time recovery

---

## Decision

**What did we decide?**

We will use **PostgreSQL** as the primary database for all transactional data in the RideShare platform. This includes:

- User accounts and profiles
- Trip records and history
- Payment transactions
- Rating records
- Address information

PostgreSQL will be deployed as:
- **Primary Database**: AWS RDS PostgreSQL with Multi-AZ deployment for high availability
- **Read Replicas**: 2+ read replicas for read-heavy operations (analytics, reporting, trip history queries)
- **PostGIS Extension**: Enabled for geospatial queries and location-based operations

Each microservice that requires transactional data will have its own PostgreSQL database instance to maintain loose coupling (database per service pattern).

---

## Considered Options

**What alternatives did we consider?**

### Option 1: PostgreSQL

**Pros:**
- ✅ ACID compliance (critical for financial transactions)
- ✅ Strong consistency guarantees
- ✅ PostGIS extension for geospatial queries
- ✅ Mature and battle-tested (25+ years)
- ✅ Excellent performance for complex queries
- ✅ Rich feature set (JSON support, full-text search, arrays)
- ✅ Strong community and ecosystem
- ✅ AWS RDS managed service available
- ✅ Excellent documentation
- ✅ Supports read replicas for scaling reads

**Cons:**
- ❌ Vertical scaling limitations (single-node writes)
- ❌ Requires read replicas for horizontal read scaling
- ❌ More complex than NoSQL for simple key-value operations
- ❌ Schema changes require migrations (can be complex)
- ❌ Higher operational overhead than serverless options

### Option 2: MySQL

**Pros:**
- ✅ ACID compliance
- ✅ Strong consistency
- ✅ Mature and widely used
- ✅ AWS RDS managed service available
- ✅ Good performance
- ✅ Large community

**Cons:**
- ❌ Limited geospatial support (less mature than PostGIS)
- ❌ Weaker JSON support compared to PostgreSQL
- ❌ Less advanced features (window functions, CTEs are less mature)
- ❌ Licensing concerns (Oracle ownership)
- ❌ Team has less experience with MySQL

### Option 3: MongoDB

**Pros:**
- ✅ Flexible schema (good for evolving requirements)
- ✅ Horizontal scaling (sharding)
- ✅ Good performance for read-heavy workloads
- ✅ Document model fits some use cases
- ✅ MongoDB Atlas managed service available

**Cons:**
- ❌ No ACID transactions across documents (until v4.0, limited)
- ❌ Eventual consistency (not suitable for financial transactions)
- ❌ Limited geospatial support compared to PostGIS
- ❌ No joins (must denormalize or use application-level joins)
- ❌ Less suitable for relational data model
- ❌ Team has less experience with MongoDB

### Option 4: Amazon Aurora PostgreSQL

**Pros:**
- ✅ PostgreSQL compatible
- ✅ Better performance than standard RDS
- ✅ Auto-scaling storage
- ✅ Serverless option available
- ✅ Multi-region replication
- ✅ Faster failover

**Cons:**
- ❌ Vendor lock-in to AWS
- ❌ Higher cost than standard RDS
- ❌ More complex pricing model
- ❌ May be overkill for initial scale

### Option 5: CockroachDB

**Pros:**
- ✅ PostgreSQL compatible
- ✅ Distributed SQL (horizontal scaling)
- ✅ Strong consistency
- ✅ ACID transactions
- ✅ Multi-region support

**Cons:**
- ❌ Newer technology (less battle-tested)
- ❌ Higher operational complexity
- ❌ May be overkill for initial requirements
- ❌ Team has no experience
- ❌ Higher cost

---

## Decision Outcome

**Why did we choose this option?**

We chose **PostgreSQL (Option 1)** because:

1. **ACID Compliance**: Critical requirement for payment processing and financial transactions. PostgreSQL provides strong ACID guarantees that are essential for our payment service.

2. **Geospatial Requirements**: PostGIS extension provides industry-leading geospatial capabilities, which are essential for:
   - Storing driver and passenger locations
   - Calculating distances and ETAs
   - Geospatial queries (finding nearby drivers)
   - Route optimization

3. **Relational Data Model**: Our data has clear relationships (User → Ride → Payment). PostgreSQL's relational model with foreign keys, joins, and transactions fits perfectly.

4. **Maturity & Reliability**: PostgreSQL has 25+ years of production use, proven reliability, and excellent documentation. This reduces risk for a critical system component.

5. **Team Experience**: Our team has strong PostgreSQL experience, reducing learning curve and operational risk.

6. **Cloud Support**: AWS RDS PostgreSQL provides managed service with:
   - Automated backups
   - Point-in-time recovery
   - Multi-AZ deployment for high availability
   - Read replicas for scaling

7. **Performance**: PostgreSQL handles complex queries efficiently and supports read replicas for read-heavy workloads.

8. **Cost-Effective**: Open-source with reasonable AWS RDS costs. Aurora would be more expensive without clear benefits for our initial scale.

While PostgreSQL has vertical scaling limitations, we can address this with:
- Read replicas for read scaling
- Connection pooling
- Query optimization
- Caching layer (Redis)

For future horizontal write scaling, we can consider:
- Database sharding (if needed)
- Moving to Aurora or CockroachDB
- CQRS pattern (separate read/write models)

---

## Consequences

**What are the positive and negative impacts of this decision?**

### Positive

- ✅ **ACID Compliance**: Strong transactional guarantees for financial data
- ✅ **Data Integrity**: Foreign keys, constraints, and transactions ensure data consistency
- ✅ **Geospatial Support**: PostGIS provides powerful location-based query capabilities
- ✅ **Mature Ecosystem**: Rich tooling, libraries, and community support
- ✅ **Team Familiarity**: Team can be productive immediately
- ✅ **Cloud Managed**: AWS RDS reduces operational overhead
- ✅ **Read Scaling**: Read replicas allow horizontal read scaling
- ✅ **Complex Queries**: Supports sophisticated SQL queries for analytics and reporting

### Negative

- ❌ **Vertical Scaling Limits**: Single-node write bottleneck (mitigated with read replicas)
- ❌ **Schema Migrations**: Schema changes require careful migration planning
- ❌ **Operational Overhead**: Requires database administration, backups, monitoring
- ❌ **Connection Management**: Need connection pooling to handle concurrent connections
- ❌ **Cost**: RDS costs scale with instance size (but reasonable for our scale)

### Neutral / Notes

- **Database per Service**: Each service has its own database, ensuring loose coupling
- **Future Considerations**: May need to evaluate sharding or distributed databases if write scale becomes a bottleneck
- **Caching Strategy**: Will use Redis for frequently accessed data to reduce database load

---

## Architecture Impact

**This decision affects the data layer architecture.**

### Context Diagram Changes

**Does this decision change the System Context?**

No changes to System Context. PostgreSQL is an internal implementation detail.

### Container Diagram Changes

**Does this decision change the Container/Logical View?**

Yes, this decision defines the database containers in the architecture.

**Changes:**
- **Added containers**: 
  - PostgreSQL Primary Database (User Service)
  - PostgreSQL Primary Database (Trip Service)
  - PostgreSQL Primary Database (Payment Service)
  - PostgreSQL Read Replicas (2+ per primary)
- **Database per Service**: Each service has its own database instance

### Logical Flow Changes

**Does this decision change the data flow or process flow?**

No changes to logical flow. Services continue to read from and write to their databases, but now using PostgreSQL instead of other database options.

**Changes:**
- **Database Protocol**: Services use JDBC/PostgreSQL protocol
- **Connection Pooling**: Services use connection pools (HikariCP, pgBouncer)
- **Read Replicas**: Read-heavy operations route to read replicas

---

## Data Model Impact

**This decision defines the database technology but the ERD structure was already designed.**

### ERD Changes

**Does this decision change the Entity Relationship Diagram?**

The ERD structure remains the same, but now implemented in PostgreSQL.

**Changes:**
- **No schema changes**: Entity relationships remain as designed
- **PostgreSQL-specific features**:
  - Foreign key constraints enforced at database level
  - Indexes on foreign keys for performance
  - Check constraints for data validation
  - Unique constraints for business rules
- **PostGIS extension**: 
  - Location columns use `GEOGRAPHY` or `GEOMETRY` types
  - Spatial indexes (GIST) for location queries
  - Geospatial functions for distance calculations

### Schema Migration Notes

- **Migration tool**: Flyway or Liquibase for version-controlled migrations
- **Migration location**: `/database/migrations/` per service
- **Data migration required**: No (new system, no existing data)
- **Rollback strategy**: 
  - All migrations are reversible
  - Backup before major migrations
  - Feature flags for gradual schema rollouts

---

## Security Impact

**This decision has security implications for data storage.**

### Security Analysis

**What security concerns were analyzed?**

- **Threat**: Unauthorized database access
  - **Risk Level**: High
  - **Mitigation**: 
    - VPC with private subnets (no public internet access)
    - Security groups restricting access to application servers only
    - Database credentials stored in AWS Secrets Manager
    - Encrypted connections (SSL/TLS)

- **Threat**: Data breach
  - **Risk Level**: Critical
  - **Mitigation**: 
    - Encryption at rest (AES-256)
    - Encrypted backups
    - Access logging and monitoring
    - Regular security audits

- **Threat**: SQL injection
  - **Risk Level**: High
  - **Mitigation**: 
    - Parameterized queries (ORM usage)
    - Input validation
    - Least privilege database users
    - Regular security testing

- **Threat**: Data loss
  - **Risk Level**: Critical
  - **Mitigation**: 
    - Automated daily backups
    - Point-in-time recovery (7-day retention)
    - Multi-AZ deployment for high availability
    - Cross-region backup replication

### Security Measures

**What security measures are implemented or required?**

- **Encryption**: 
  - Encryption at rest (AES-256) enabled on RDS
  - Encryption in transit (SSL/TLS) for all connections
  - Encrypted backups

- **Access Control**: 
  - Database users with least privilege
  - No direct database access from applications (connection pooling)
  - Database credentials in AWS Secrets Manager
  - IAM database authentication (optional)

- **Network Security**: 
  - Databases in private subnets
  - Security groups restrict access to application tier only
  - No public internet access

- **Monitoring**: 
  - Database access logging
  - Failed login attempt monitoring
  - Unusual query pattern detection
  - Regular security audits

- **Compliance**: 
  - GDPR compliance (data encryption, right to deletion)
  - SOC 2 controls
  - Regular compliance audits

### Security Trade-offs

- **Positive**: 
  - Strong access controls
  - Encryption at rest and in transit
  - Audit logging capabilities
  - Compliance-friendly

- **Negative**: 
  - Additional operational overhead for security management
  - Performance impact from encryption (minimal with modern hardware)

---

## Performance Impact

**This decision significantly affects database performance.**

### Performance Analysis

**How does this decision affect performance?**

- **Query Performance**: 
  - Before: N/A (new system)
  - After: < 100ms for standard queries (p95)
  - Optimization: Indexes, query optimization, connection pooling

- **Write Throughput**: 
  - Before: N/A
  - After: 10,000+ writes per second per database (with proper indexing)
  - Limitation: Single-node write bottleneck (addressed with read replicas for reads)

- **Read Throughput**: 
  - Before: N/A
  - After: 50,000+ reads per second (with read replicas)
  - Scaling: Horizontal read scaling via read replicas

- **Connection Handling**: 
  - Before: N/A
  - After: Connection pooling (HikariCP) supports 100+ concurrent connections per service
  - Optimization: pgBouncer for connection pooling at database level

### Performance Optimizations

**What optimizations are implemented or planned?**

- **Indexing Strategy**: 
  - Primary keys (auto-indexed)
  - Foreign key indexes
  - Composite indexes for common query patterns
  - Spatial indexes (GIST) for PostGIS columns
  - Partial indexes for filtered queries

- **Query Optimization**: 
  - EXPLAIN ANALYZE for query plan analysis
  - Query tuning based on execution plans
  - Avoid N+1 queries (use JOINs, batch loading)
  - Use prepared statements

- **Connection Pooling**: 
  - Application-level: HikariCP (20-50 connections per service)
  - Database-level: pgBouncer (optional, for connection multiplexing)

- **Caching**: 
  - Redis cache for frequently accessed data
  - Application-level caching for read-heavy queries
  - Database query result caching (PostgreSQL 13+)

- **Read Replicas**: 
  - 2+ read replicas per primary database
  - Route read queries to replicas
  - Use primary only for writes and critical reads

### Performance Monitoring

**What metrics will be monitored?**

- **Key metrics**: 
  - Query execution time (p50, p95, p99)
  - Database connections (active, idle, waiting)
  - Read/write throughput
  - Replication lag (for read replicas)
  - Cache hit rates
  - Slow query log

- **Alert thresholds**: 
  - Query time > 500ms (p95)
  - Connection pool exhaustion (> 80% utilization)
  - Replication lag > 1 second
  - Database CPU > 80%
  - Database memory > 80%

- **Performance testing**: 
  - Load testing with realistic data volumes
  - Query performance testing
  - Connection pool stress testing
  - Replication lag testing

---

## Dependencies & Libraries

**This decision introduces database drivers and ORM dependencies.**

### New Dependencies

**What new dependencies or libraries are introduced?**

| Dependency | Version | Purpose | License |
|------------|---------|---------|---------|
| PostgreSQL JDBC Driver | 42.x | Java database connectivity | BSD 2-Clause |
| node-postgres (pg) | 8.x | Node.js PostgreSQL client | MIT |
| go-pg / pgx | Latest | Go PostgreSQL client | MIT / Apache 2.0 |
| HikariCP | 5.x | Java connection pool | Apache 2.0 |
| TypeORM | 0.3.x | TypeScript/Node.js ORM | MIT |
| GORM | Latest | Go ORM | MIT |
| Flyway | 9.x | Database migration tool | Apache 2.0 |
| PostGIS | 3.x | Geospatial extension | GPL v2 |

### Dependency Management

- **Package managers**: 
  - npm (Node.js services)
  - Maven (Java services)
  - Go modules (Go services)

- **Version pinning strategy**: 
  - Exact versions for database drivers (compatibility critical)
  - Minor version updates for ORMs (with testing)
  - Major version updates require ADR

- **Update policy**: 
  - Security patches: Immediate
  - Minor updates: Monthly
  - Major updates: Quarterly review

### Dependency Risks

- **Security**: 
  - PostgreSQL JDBC driver has good security track record
  - Regular security updates
  - CVE monitoring required

- **Maintenance**: 
  - All dependencies are actively maintained
  - Strong community support
  - PostgreSQL has long-term support

- **License**: 
  - Most dependencies are MIT/Apache 2.0 (permissive)
  - PostGIS is GPL v2 (acceptable for our use case)
  - No license conflicts

- **Size**: 
  - Database drivers are lightweight
  - ORMs add some overhead but acceptable
  - Total impact: Minimal

---

## Deployment Impact

**This decision requires database infrastructure setup.**

### Infrastructure Changes

**What infrastructure changes are required?**

- **New Services**: 
  - AWS RDS PostgreSQL instances (3 primary databases: User, Trip, Payment)
  - Read replicas (2 per primary = 6 total)
  - Database parameter groups (custom configuration)

- **Configuration Changes**: 
  - Database connection strings per service
  - Connection pool configuration
  - PostGIS extension enabled
  - Backup configuration (daily, 7-day retention)
  - Multi-AZ deployment enabled

### Deployment Process Changes

**How does deployment change?**

- **Before**: N/A (new system)

- **After**: 
  - Database provisioning via Infrastructure as Code (Terraform/CloudFormation)
  - Database migrations run as part of deployment pipeline
  - Migration rollback procedures
  - Database backup before migrations

- **Migration Steps**: 
  1. Provision RDS PostgreSQL instances (Terraform)
  2. Enable PostGIS extension
  3. Run initial schema migrations (Flyway)
  4. Configure read replicas
  5. Set up automated backups
  6. Configure monitoring and alerts
  7. Test connection pooling
  8. Performance testing

### Deployment Considerations

- **Rollback Strategy**: 
  - All migrations are reversible
  - Database backups before major migrations
  - Feature flags for gradual schema rollouts
  - Point-in-time recovery available

- **Zero-Downtime**: 
  - Schema changes that don't lock tables can be zero-downtime
  - Some migrations may require brief maintenance window
  - Read replicas allow reads during primary maintenance

- **Database Migrations**: 
  - Flyway for version-controlled migrations
  - Migrations run automatically in deployment pipeline
  - Manual approval for major migrations
  - Rollback scripts prepared

- **Feature Flags**: N/A (database changes are infrastructure-level)

- **Monitoring**: 
  - CloudWatch metrics for RDS
  - Database performance insights
  - Slow query logging
  - Connection pool monitoring

### Infrastructure Requirements

- **Compute**: 
  - Primary databases: db.r5.xlarge (4 vCPU, 32GB RAM) initially
  - Read replicas: db.r5.large (2 vCPU, 16GB RAM) initially
  - Auto-scaling based on CPU/memory metrics

- **Storage**: 
  - Initial: 500GB SSD per database
  - Auto-scaling storage (up to 64TB)
  - Provisioned IOPS for performance (3000 IOPS initially)

- **Network**: 
  - Databases in private subnets
  - Security groups for access control
  - VPC endpoints for AWS services

- **Third-party Services**: 
  - AWS RDS PostgreSQL (managed service)
  - AWS Secrets Manager (database credentials)
  - AWS CloudWatch (monitoring)

---

## Implementation Notes

**How will we implement this decision?**

1. **Database Provisioning**: 
   - Use Terraform to provision RDS PostgreSQL instances
   - Enable Multi-AZ deployment for high availability
   - Configure automated backups (daily, 7-day retention)

2. **PostGIS Setup**: 
   - Enable PostGIS extension on all databases
   - Configure spatial reference systems
   - Create spatial indexes for location columns

3. **Schema Design**: 
   - Design schemas per service (User, Trip, Payment)
   - Define foreign key relationships
   - Create indexes for performance
   - Set up constraints for data integrity

4. **Migration Setup**: 
   - Set up Flyway for each service
   - Create initial migration scripts
   - Establish migration workflow in CI/CD

5. **Connection Pooling**: 
   - Configure HikariCP for Java services
   - Configure connection pools for Node.js services
   - Set appropriate pool sizes (20-50 connections)

6. **Read Replicas**: 
   - Create 2 read replicas per primary database
   - Configure application routing to replicas for reads
   - Monitor replication lag

7. **Monitoring**: 
   - Set up CloudWatch dashboards
   - Configure alerts for performance issues
   - Enable slow query logging
   - Set up database performance insights

8. **Security**: 
   - Store database credentials in AWS Secrets Manager
   - Enable encryption at rest
   - Configure SSL/TLS for connections
   - Set up security groups
   - Enable audit logging

---

## References

- [ADR-001: Use Microservices Architecture](./ADR-001-Example.md) - Database per service pattern
- [ADR-003: MongoDB for Driver Documents](./ADR-003-Example.md) - Document database choice
- [ADR-004: Redis for Real-Time Location Tracking](./ADR-004-Example.md) - Caching layer
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostGIS Documentation](https://postgis.net/documentation/)
- [AWS RDS PostgreSQL](https://aws.amazon.com/rds/postgresql/)
- [Database per Service Pattern](https://microservices.io/patterns/data/database-per-service.html)

---

## Notes

**Additional context, follow-up decisions, or changes**

- **2026-01-26**: Initial decision made during architecture planning
- **Future Considerations**: 
  - May need to evaluate database sharding if write scale becomes bottleneck
  - Consider Aurora PostgreSQL if performance requirements increase
  - Evaluate CockroachDB for multi-region requirements
- **Review Date**: Review this decision in 6 months or if we encounter scaling issues
- **Related Decisions**: This decision supports ADR-001 (microservices) and influences ADR-005 (Kafka for eventual consistency)

---

**Last Updated**: 2026-01-26  
**Related ADRs**: [ADR-001](./ADR-001-Example.md), [ADR-003](./ADR-003-Example.md), [ADR-004](./ADR-004-Example.md)
