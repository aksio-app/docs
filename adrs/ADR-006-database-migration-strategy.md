# ADR-006: Database Migration Strategy

> **Technical Note**: This document outlines the database migration strategy for Aksio's educational platform. Implementation details, security configurations, and operational procedures are maintained in internal documentation.

## Status
- 2025-08-14 - Draft
- 2025-08-14 - **Accepted**

## Context

As Aksio scales from hundreds to millions of students, we need a robust database migration strategy that ensures data integrity, enables zero-downtime deployments, and maintains compliance with GDPR and DSA requirements. Our PostgreSQL database with Apache AGE extension requires careful migration planning to preserve both relational and graph data structures.

### Key Requirements

**Operational Requirements:**
- Zero-downtime deployments for 24/7 student access
- Rollback capability for failed migrations
- Support for PostgreSQL extensions (AGE, pgAudit, uuid-ossp)
- Integration with GitHub Actions CI/CD
- Work with Google Cloud SQL managed service

**Security & Compliance:**
- Audit trail for all schema changes
- Security of processing (GDPR Article 32)
- Protection of sensitive student data during migrations
- Secure handling of migration credentials
- Compliance with data protection regulations

**Team & Scale Considerations:**
- Small development team requires automation
- Must handle growth from hundreds to millions of records
- Developer-friendly with minimal operational overhead
- Clear rollback procedures for production safety

## Decision

We will use **golang-migrate/migrate** as our database migration tool, implementing capabilities progressively as we scale.

> **Implementation Note**: As a small startup, we will implement these practices in phases. Not all procedures described in this document are required from day one. See "Implementation Phases" section for our pragmatic rollout plan.

## Rationale

### Why golang-migrate/migrate?

After evaluating multiple options with input from architecture, security, and operations perspectives, golang-migrate/migrate provides the optimal balance of:

1. **Production Maturity**: 14,000+ GitHub stars, used by major companies
2. **PostgreSQL Excellence**: Full SQL control, extension support, native features
3. **Security Features**: Checksum validation, version locking, atomic transactions
4. **Operational Simplicity**: Single binary, no dependencies, clean CLI
5. **CI/CD Integration**: First-class GitHub Actions support, Docker-friendly
6. **Google Cloud Native**: Works seamlessly with Cloud SQL and Cloud Storage

### Migration Philosophy: Expand-Contract Pattern

For zero-downtime deployments, we adopt the expand-contract pattern:

```
1. EXPAND: Add new columns/tables (backward compatible)
2. DEPLOY: Roll out new application code
3. MIGRATE: Transform existing data
4. CONTRACT: Remove deprecated columns/tables
```

This ensures old and new code versions can coexist during deployment.

### Security-First Approach

All migrations must:
- Use static SQL files (no dynamic generation)
- Include checksums for integrity verification
- Execute with minimal-privilege service accounts
- Generate audit logs for compliance
- Create automatic backups before execution

## Implementation Strategy

### Migration Organization

Migrations follow a numbered sequence with up/down pairs for reversibility. Each migration is atomic and tested for both application and rollback scenarios.

### Migration Lifecycle (Current MVP Process)

**Development:**
1. Developer creates migration files locally
2. Test with `migrate up` and `migrate down`
3. Submit PR with migration files

**Production:**
1. Review migration in PR
2. Ensure Cloud SQL backup is recent
3. Run migration manually
4. Quick smoke test
5. Monitor for issues

### Rollback Strategy

Every migration must be reversible:
- **Immediate rollback**: For failed migrations (< 5 minutes)
- **Data-safe rollback**: For issues discovered later (< 24 hours)
- **Point-in-time recovery**: For critical failures (Cloud SQL backups)

### Monitoring & Alerting

Track key metrics:
- Migration execution time and rows affected
- Database performance (connections, locks, replication lag)
- Application health (error rates, response times)
- Resource utilization (CPU, memory, disk)
- Schema version consistency across environments

## Alternatives Considered

### pressly/goose
**Verdict**: Good tool but lacks checksum validation and has smaller ecosystem. The ability to mix Go and SQL migrations adds unnecessary complexity.

### jackc/tern
**Verdict**: PostgreSQL-specific but limited community (500 stars) and lacks enterprise features we need for compliance.

### ORM-based Migrations (GORM/ent)
**Verdict**: Poor support for PostgreSQL extensions, especially Apache AGE. Hidden SQL generation makes security audits difficult.

### Custom Solution
**Verdict**: Would consume significant engineering resources better spent on core product. Well-solved problem with mature tools available.

## Operational Procedures

### Large-Scale Migration Considerations

For tables with millions of rows:
- Use batched operations to minimize lock time
- Implement concurrent index creation where possible
- Consider maintenance windows for major schema changes
- Monitor replication lag and connection pool saturation
- Test migrations against production-sized datasets in staging

### Apache AGE Graph Migrations

Graph data requires special handling:
- Separate graph schema migrations from relational changes
- Validate vertex and edge integrity after migrations
- Test graph query performance post-migration
- Maintain compatibility with AGE version updates

### CI/CD Integration

**GitHub Actions Workflow:**
1. Validate migration syntax
2. Check for backward compatibility
3. Test rollback procedures
4. Deploy to appropriate environment
5. Monitor and alert on issues

### Environment Management

- **Development**: Automatic migrations on every deploy
- **Staging**: Migrations require approval, automatic rollback on failure
- **Production**: Manual approval, phased rollout, monitoring required

### Disaster Recovery

**Recovery Objectives:**
- Recovery Time Objective (RTO): 15 minutes
- Recovery Point Objective (RPO): 5 minutes

**Backup Strategy:**
- Cloud SQL automated backups (30-day retention)
- Point-in-time recovery capability
- Cross-region backup replication for critical data
- Monthly disaster recovery testing
- Documented runbooks for common failure scenarios

## Security Controls

### Access Management
- Dedicated migration service account per environment
- Temporary credentials (1-hour maximum)
- IP whitelist from CI/CD runners only
- Audit logging for all migration executions

### Data Protection

**GDPR Compliance:**
- Data Protection Impact Assessment (DPIA) for schema changes affecting personal data
- Audit trail retention for 7 years (GDPR Article 30)
- Backup exemptions documented under Article 17(3)(b)
- Privacy by design principles in migration planning

**Technical Measures:**
- Automatic backup before migrations
- Encryption in transit and at rest
- No personal data in migration files
- Secure credential management via Google Secret Manager
- Automated detection of PII in migration scripts

## Success Metrics

**Current MVP Metrics:**
- **Migration success rate**: > 95%
- **Rollback capability**: 100% (all migrations must have down scripts)
- **Recovery time**: < 30 minutes (via Cloud SQL backup if needed)

**Future Target Metrics (as we scale):**
- **Zero-downtime achievement**: 99.9%
- **Mean time to rollback**: < 5 minutes
- **Audit compliance**: 100% coverage

## Risks and Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| Data loss during migration | Critical | Automatic backups, tested rollback procedures |
| Extended downtime | High | Expand-contract pattern, feature flags |
| Graph data corruption | High | AGE-specific testing, separate graph migrations |
| Compliance violation | High | DPIA process, audit logging, secure procedures |
| Large table migrations | High | Batched operations, performance testing |
| AGE compatibility issues | Medium | Version testing, compatibility matrix |
| Team knowledge gap | Medium | Documentation, training, pair programming |

## Related Decisions

- ADR-001: Go for Backend Development
- ADR-003: Google Cloud Platform
- ADR-004: PostgreSQL with Apache AGE
- ADR-005: Authentication Strategy

## Implementation Phases

### Phase 1: MVP (Current - Q4 2025) ✅
**What we're actually doing now:**
- Basic golang-migrate setup with up/down migrations
- Simple GitHub Actions for syntax validation
- Manual migrations in production with basic backup
- Two environments: development and production
- Manual rollback if needed

**Minimum viable practices:**
```yaml
environments:
  development:
    auto_migrate: true
    backup: optional
  production:
    manual_approve: true
    backup: required (Cloud SQL automated)
    rollback: manual via down migrations
```

### Phase 2: Early Growth (Q1 2026)
**Add when we have paying customers:**
- Staging environment for testing migrations
- Basic monitoring (execution time, success/failure)
- Documented rollback procedures
- GDPR compliance for audit trails

### Phase 3: Growth (Q2 2026)
**Add when approaching 100+ active users:**
- Automated backup verification
- Performance baseline testing
- Enhanced error handling and alerting
- Migration approval workflow

### Phase 4: Scaling (Q3 2026)
**Add when approaching 1000+ users:**
- Zero-downtime expand-contract pattern
- Automated performance testing
- DPIA process for schema changes
- Read replica considerations

### Phase 5: Enterprise (Q4 2026+)
**Add when approaching 10,000+ users:**
- Cross-region disaster recovery
- Blue-green deployments
- Advanced migration orchestration
- Full compliance automation

## Scalability Considerations

The architecture described in this document supports growth from hundreds to millions of users. We will implement each capability as needed, avoiding premature optimization while maintaining a clear upgrade path.

## References

- [golang-migrate Documentation](https://github.com/golang-migrate/migrate)
- [Zero-Downtime Migrations](https://www.brunton-spall.co.uk/post/2014/05/06/database-migrations-done-right/)
- [PostgreSQL AGE Extension](https://age.apache.org/)
- [Cloud SQL Migration Best Practices](https://cloud.google.com/sql/docs/postgres/migration-overview)
- [GDPR Article 32 - Security of Processing](https://gdpr-info.eu/art-32-gdpr/)
- [GDPR Article 17 - Right to Erasure](https://gdpr-info.eu/art-17-gdpr/)

## Approval

**Decision Maker(s):** Olav Selnes Lorentzen  
**Date Approved:** 2025-08-14