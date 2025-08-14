# ADR-004: Database Architecture - PostgreSQL with ltree on GCP

> **Technical Note**: Performance benchmarks, cost estimates, and technical specifications in this document are rough approximations based on vendor documentation and community benchmarks as of August 2025. Numbers like "80% relational, 20% graph queries" are illustrative estimates, not measured metrics. Actual performance, costs, and query patterns will vary based on implementation, data volume, and usage patterns. The decision to use PostgreSQL with ltree is primarily driven by operational simplicity and unified data management, not marginal performance differences.

## Status
- 2025-08-14 - Draft
- 2025-08-14 - **Accepted**
- 2025-08-14 - **Revised** - Changed from Apache AGE to ltree extension

## Context

Following our decision to use Google Cloud Platform (ADR-003), we need to select a database architecture that leverages GCP's strengths while meeting the complex requirements of an educational platform. The Aksio platform requires sophisticated data storage that can handle both traditional relational data and complex graph relationships between educational entities.

### Core Requirements

**Data Model Needs:**
- **Relational Data**: Traditional structured data for users, courses, schedules, assessments
- **Graph Capabilities**: Complex relationships between topics, prerequisites, learning paths
- **Hierarchical Data**: Organizational structures, curriculum trees, nested content
- **Time-Series Data**: Progress tracking, learning analytics, engagement metrics

**Security & Compliance:**
- **Encryption at Rest**: All data must be encrypted when stored
- **Row-Level Security**: Fine-grained access control based on enrollments and roles
- **GDPR/FERPA Compliance**: Educational privacy requirements
- **Complete Audit Trails**: Track all data modifications for compliance

**Operational Requirements:**
- **GCP Cloud SQL Integration**: Leverage managed database services
- **Cost Optimization**: Scale-to-zero not needed for database, but must be cost-effective
- **High Availability**: Automatic failover and backups
- **Performance**: Support thousands of concurrent students

## Decision

We will use **PostgreSQL with ltree extension** running on **GCP Cloud SQL** as our primary database solution.

> **Implementation Note**: Initially we considered Apache AGE for graph capabilities, but discovered it's not available in Cloud SQL. The ltree extension, which is available, actually provides a better fit for our hierarchical data needs.

## Rationale

### 1. Perfect GCP Cloud SQL Integration

PostgreSQL with ltree on Cloud SQL provides unmatched integration with our chosen cloud platform:

**Native GCP Support:**
- **Fully Managed Service**: Cloud SQL handles patches, backups, replication automatically
- **High Availability**: Automatic failover with 99.95% SLA
- **Point-in-Time Recovery**: Restore to any second within retention period
- **Automated Backups**: Daily backups with 30-day retention
- **Read Replicas**: Scale read operations across multiple zones

**Cost Optimization on GCP:**
- **Shared Core Instances**: Start with db-f1-micro ($15/month) for development
- **Committed Use Discounts**: Up to 52% savings with 1-year commitment
- **Storage Auto-Resize**: Pay only for storage actually used
- **Connection Pooling**: PgBouncer integration reduces connection overhead
- **Estimated Monthly Cost**: $35-75 for MVP, $200-500 at scale

### 2. Unified Solution Advantage

Unlike dual-database architectures (Neo4j + PostgreSQL), our single-database approach provides:

**Operational Simplicity:**
- **Single Backup Strategy**: One backup process, one recovery plan
- **Unified Monitoring**: Single dashboard in Cloud Monitoring
- **One Security Model**: Consistent RBAC/RLS across all data
- **ACID Transactions**: Full transactional integrity across graph and relational data
- **No Synchronization Issues**: No data consistency problems between systems

**Developer Productivity:**
```sql
-- Single query combining relational and hierarchical data
-- ltree paths like 'course.module.topic.subtopic' provide natural hierarchy
WITH learning_path AS (
  SELECT 
    path,
    nlevel(path) as depth
  FROM curriculum
  WHERE path <@ 'course.module'  -- All descendants of module
)
-- Join with relational data
SELECT 
  c.name AS course_name,
  u.progress_percentage,
  lp.path
FROM courses c
JOIN user_progress u ON u.course_id = c.id
JOIN learning_path lp ON lp.curriculum_id = c.topic_id
WHERE u.user_id = current_user_id();
```

### 3. Educational Domain Fit

PostgreSQL with ltree perfectly matches educational data patterns:

**Hybrid Data Model:**
- **Relational**: Student records, grades, schedules (80% of queries)
- **Hierarchical**: Course → Module → Topic → Subtopic structures with ltree
- **JSON/JSONB**: Flexible content storage, quiz questions, learning materials
- **Arrays**: Multiple choice answers, tag systems, skill sets

**Learning Path Optimization:**
```sql
-- Find learning path using ltree hierarchical queries
SELECT path, name
FROM curriculum
WHERE path BETWEEN 'course.module1' AND 'course.module5'
ORDER BY path;
```

### 4. Security & Compliance Excellence

PostgreSQL's mature security features integrate perfectly with GCP:

**Multi-Layer Security:**
```sql
-- Row-Level Security for student data isolation
ALTER TABLE learning_materials ENABLE ROW LEVEL SECURITY;

CREATE POLICY student_access ON learning_materials
  FOR ALL
  USING (
    course_id IN (
      SELECT course_id FROM enrollments 
      WHERE user_id = current_user_id() 
      AND status = 'active'
    )
  );

-- Audit logging with pgAudit
CREATE EXTENSION IF NOT EXISTS pgaudit;
ALTER SYSTEM SET pgaudit.log = 'WRITE, DDL, ROLE';
ALTER SYSTEM SET pgaudit.log_relation = 'on';
```

**GCP Security Integration:**
- **Cloud SQL IAM**: Database users authenticated via Google accounts
- **Private IP**: Database isolated in VPC with Private Service Connect
- **Cloud SQL Proxy**: Encrypted connections without managing SSL certificates
- **Secret Manager Integration**: Automatic password rotation

### 5. Performance Optimization on GCP

**Cloud SQL Performance Features:**
- **SSD Storage**: Default for all Cloud SQL instances
- **Memory Optimization**: Up to 624 GB RAM for large instances
- **Connection Pooling**: Built-in PgBouncer support
- **Query Insights**: Automatic slow query detection and analysis
- **Auto-scaling Storage**: No performance degradation as data grows

**Caching Strategy:**
```go
// Application-level caching with GCP Memorystore
type CourseRepository struct {
    db    *sql.DB
    cache *redis.Client // GCP Memorystore
}

func (r *CourseRepository) GetCourse(ctx context.Context, id int) (*Course, error) {
    // Check cache first
    if cached, err := r.cache.Get(ctx, fmt.Sprintf("course:%d", id)).Result(); err == nil {
        return unmarshalCourse(cached), nil
    }
    
    // Query database with graph relationships
    course, err := r.queryWithPrerequisites(ctx, id)
    if err != nil {
        return nil, err
    }
    
    // Cache for next time
    r.cache.Set(ctx, fmt.Sprintf("course:%d", id), course, 5*time.Minute)
    return course, nil
}
```

### 6. Go Ecosystem Excellence

PostgreSQL has the best Go support among all database options:

**Production-Ready Drivers:**
- **pgx**: High-performance, feature-complete PostgreSQL driver
- **sqlx**: Enhanced database/sql with struct scanning
- **GORM**: Full ORM support if needed (though we prefer raw SQL)
- **migrate**: Database migration tooling

**Type Safety:**
```go
// Strong typing with pgx
type Topic struct {
    ID           int       `db:"id"`
    Name         string    `db:"name"`
    Prerequisites []int     `db:"prerequisites"` // PostgreSQL array type
    Metadata     JSONB     `db:"metadata"`      // JSONB type
}

// Query with full type safety
var topics []Topic
err := pgx.Select(&topics, `
    SELECT id, name, prerequisites, metadata
    FROM topics
    WHERE course_id = $1
`, courseID)
```

## Comparison with Alternatives

### Why Not Neo4j + PostgreSQL?

While Neo4j is the leading graph database, the dual-database approach has significant drawbacks for our use case:

| Aspect | PostgreSQL + ltree | Neo4j + PostgreSQL |
|--------|-------------------|-------------------|
| **Monthly Cost (GCP)** | $35-75 | $200-400 |
| **Operational Complexity** | Single system | Two systems to manage |
| **Transaction Boundaries** | Full ACID across all data | No distributed transactions |
| **Backup/Recovery** | Single process | Must coordinate two systems |
| **Learning Curve** | SQL + ltree operators | SQL + Cypher + Sync logic |
| **GCP Integration** | Native Cloud SQL | Self-managed on GCE |

### Why Not SQL Server or Aurora?

- **SQL Server**: Requires self-managed VMs on GCP, expensive licensing
- **Aurora**: AWS-only, not available on GCP
- **Both**: Would violate our GCP-first strategy from ADR-003

## Next Steps

**Initial Setup:**
- Enable PostgreSQL extensions (ltree, pgAudit, uuid-ossp)
- Design schema that supports both relational and hierarchical patterns
- Implement Row-Level Security for multi-tenancy

**Ongoing Considerations:**
- Monitor query performance and add indexes as needed
- Evaluate read replicas when traffic increases
- Consider specialized databases if requirements change

## Consequences

### Positive Consequences

1. **Reduced Complexity**: Single database system simplifies everything
2. **Cost Savings**: 50-70% lower than dual-database architectures
3. **Better Performance**: No network latency between databases
4. **Stronger Consistency**: ACID transactions across all data
5. **Excellent GCP Integration**: Leverage Cloud SQL's managed features
6. **Future Flexibility**: Can add specialized databases later if needed

### Negative Consequences

1. **Graph Limitations**: ltree handles hierarchies well but not general graphs
2. **Learning Curve**: Team must learn ltree operators
3. **Graph Performance**: Complex graph operations would need workarounds

### Risk Mitigation

**Hierarchical Performance:**
- Most queries (80%) are relational, where PostgreSQL excels
- ltree is optimized for hierarchical queries
- Can add graph database later if requirements evolve
- PostgreSQL's recursive CTEs available for complex cases

**Scaling Strategy:**
- Start with single primary instance
- Add read replicas as traffic grows
- Implement caching layer (Memorystore)
- Consider partitioning for very large datasets

## Monitoring and Success Metrics

**Performance Targets:**
- Query response time: < 100ms (p95)
- Connection pool utilization: < 80%
- Storage growth rate: < 10GB/month initially
- Backup recovery time: < 1 hour

**Cost Targets:**
- Development environment: < $50/month
- Production MVP: < $200/month
- At 10,000 users: < $500/month

## Related Decisions

- **ADR-001**: Go for Backend Development (excellent PostgreSQL support)
- **ADR-002**: Gin HTTP Router Package (works well with PostgreSQL)
- **ADR-003**: Google Cloud Platform (Cloud SQL provides managed PostgreSQL)

## References

- [Cloud SQL for PostgreSQL Documentation](https://cloud.google.com/sql/docs/postgres)
- [PostgreSQL ltree Documentation](https://www.postgresql.org/docs/current/ltree.html)
- [PostgreSQL Row Level Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [pgAudit Documentation](https://www.pgaudit.org/)
- [Cloud SQL Pricing](https://cloud.google.com/sql/pricing)
- [GCP + PostgreSQL Best Practices](https://cloud.google.com/sql/docs/postgres/best-practices)

## Conclusion

PostgreSQL with ltree on GCP Cloud SQL provides the optimal balance of:
- **Simplicity**: Single database for all data types
- **Cost-effectiveness**: 50-70% cheaper than alternatives
- **GCP Integration**: Native Cloud SQL support with managed operations
- **Security**: Enterprise-grade security with RLS and audit logging
- **Flexibility**: Handles relational, hierarchical, and document data models

This decision aligns perfectly with our GCP cloud strategy (ADR-003) and provides a solid foundation for building a scalable, secure, and cost-effective educational platform.

## Approval

**Decision Maker(s):** Olav Selnes Lorentzen  
**Date Approved:** 2025-08-14