# ADR-007: Secrets Management Strategy

## Status
Accepted

## Dates
- 2025-08-14 - Draft
- 2025-08-14 - **Accepted**

## Context
The Aksio platform requires secure management of sensitive configuration data including API keys, database credentials, OAuth client secrets, encryption keys, and third-party service tokens. These secrets must be:

- Stored securely with encryption at rest
- Accessible only to authorized services
- Rotatable without service disruption
- Auditable for compliance requirements
- Recoverable in disaster scenarios
- Isolated between environments (dev/staging/prod)

Student data protection regulations (FERPA/GDPR) require demonstrable security controls around access to systems that process personal information.

## Decision
We will use Google Secret Manager as our primary secrets management solution, following the principle of least privilege and implementing automatic rotation where possible.

## Rationale
Google Secret Manager provides:
1. **Native GCP Integration** - Seamless integration with our existing GCP infrastructure
2. **Encryption at Rest** - Automatic encryption using Google-managed or customer-managed keys
3. **Fine-grained IAM** - Role-based access control at the secret and version level
4. **Audit Logging** - Complete audit trail through Cloud Audit Logs
5. **Version Management** - Multiple versions with easy rollback capability
6. **Automatic Rotation** - Built-in rotation for supported secret types
7. **High Availability** - Regional and multi-regional replication options

## Options Considered

### Option 1: Google Secret Manager (Selected)
**Pros:**
- Native GCP service with deep integration
- No additional infrastructure to manage
- Pay-per-use pricing model
- Automatic encryption and key management
- Compliance certifications (SOC, ISO, HIPAA, etc.)
- Built-in rotation capabilities

**Cons:**
- Vendor lock-in to GCP
- Requires careful IAM configuration
- Limited to GCP-supported features

### Option 2: HashiCorp Vault
**Pros:**
- Cloud-agnostic solution
- Advanced features (dynamic secrets, PKI)
- Strong community and ecosystem
- Self-hosted option for complete control

**Cons:**
- Additional infrastructure to manage and secure
- Higher operational complexity
- Additional cost for hosting and maintenance
- Requires expertise to configure properly

### Option 3: Environment Variables
**Pros:**
- Simple and familiar to developers
- No additional services required
- Works across all platforms

**Cons:**
- Insecure - visible in process listings
- No rotation capability
- No audit trail
- Difficult to manage across environments
- Not suitable for production

### Option 4: Kubernetes Secrets (if using GKE)
**Pros:**
- Native Kubernetes integration
- Automatic injection into pods
- Supports volume mounts and env vars

**Cons:**
- Base64 encoded, not encrypted by default
- Limited to Kubernetes workloads
- No built-in rotation
- Less granular access control

## Consequences

### Positive
- **Enhanced Security** - Centralized, encrypted storage with access controls
- **Compliance Ready** - Audit logs demonstrate security controls for FERPA/GDPR
- **Operational Efficiency** - Automated rotation reduces manual intervention
- **Disaster Recovery** - Versioning enables quick rollback of compromised secrets
- **Zero-downtime Updates** - Version management allows gradual rollout

### Negative
- **GCP Dependency** - Tighter coupling to Google Cloud Platform
- **Learning Curve** - Team needs to understand Secret Manager patterns
- **Migration Effort** - Existing secrets need to be migrated

### Neutral
- **Cost Impact** - Small additional cost (~$0.06 per secret per month)
- **Architecture Change** - Applications must be updated to fetch secrets

## Security Implications

### Access Control
- Service accounts use Workload Identity for authentication
- Each service has minimal required permissions
- Human access requires additional MFA
- Emergency break-glass procedures documented

### Encryption
- Secrets encrypted at rest using AES-256
- Encryption in transit via TLS 1.3
- Option for customer-managed encryption keys (CMEK)

### Compliance
- Audit logs retained for compliance periods
- Access patterns monitored for anomalies
- Regular access reviews required
- Supports GDPR data residency requirements

### Incident Response
- Immediate secret rotation capability
- Version history for forensics
- Automated alerting on unauthorized access
- Integration with security monitoring tools

## Implementation

High-level implementation approach:
1. Configure Secret Manager in each environment
2. Implement secret rotation policies
3. Update applications to use Secret Manager SDK
4. Migrate existing secrets with proper validation
5. Configure monitoring and alerting
6. Document emergency procedures
7. Train team on proper usage

**Note:** Detailed implementation steps are documented in internal documentation for security reasons.

## Risks and Mitigation

| Risk | Impact | Probability | Mitigation Strategy |
|------|--------|-------------|-------------------|
| Service account compromise | High | Low | Implement principle of least privilege, use Workload Identity, monitor access patterns |
| Secret exposure in logs | High | Medium | Implement log filtering, educate developers, use structured logging |
| Rotation failures | Medium | Low | Test rotation procedures, implement monitoring, maintain rollback procedures |
| GCP service outage | High | Low | Regional redundancy, cached secrets with TTL, disaster recovery plan |
| Misconfiguration | Medium | Medium | Infrastructure as Code, peer review, automated testing |

## Dependencies
- Google Cloud Platform account and projects
- Cloud IAM properly configured
- Workload Identity Federation for service authentication
- Cloud Audit Logs enabled
- Terraform for infrastructure management

## Related Decisions
- ADR-003: Cloud Provider Selection (Google Cloud Platform)
- ADR-005: Authentication Strategy (secure token management)
- ADR-006: Database Migration Strategy (credentials management)

## Notes
This ADR provides the high-level architecture decision for secrets management. Implementation details, specific configurations, and sensitive procedures are documented separately in internal documentation to maintain security.

The decision prioritizes operational simplicity and native cloud integration over portability, aligning with our choice of GCP as our cloud platform. 

## Approval
**Decision Maker(s):** Olav Selnes Lorentzen  
**Date Approved:** 2025-08-14