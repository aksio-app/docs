# ADR-003: Cloud Provider Selection

> **Technical Note**: All cost estimates, performance metrics, and numerical comparisons in this document are rough approximations based on publicly available pricing and benchmarks as of August 2025. These numbers are provided for relative comparison only and represent a small part of our decision criteria. Actual costs and performance will vary significantly based on usage patterns, specific configurations, and regional factors. The primary decision drivers are strategic alignment with our educational mission and developer experience, not marginal cost differences.

## Status
- 2025-08-14 - Draft
- 2025-08-14 - **Accepted**

## Context

Aksio is an intelligent educational platform that revolutionizes how students organize and master their curriculum through AI-powered learning optimization. We need to select a cloud provider that aligns with our technical requirements, educational sector needs, and core principles of Security by Default and Transparency by Default.

Our platform consists of:
- Mobile application (iOS and Android)
- Web application (Next.js/TypeScript)
- Backend API service (Go)
- Database layer (to be determined in ADR-004)
- Infrastructure as Code (Terraform)
- AI/ML components for personalized learning (planned)

Key considerations:
- Primary market: European educational sector (GDPR compliance critical)
- Target users: Students and educational institutions
- Scale: From individual students to entire universities
- Budget: Cost-conscious due to educational sector pricing expectations

## Decision

We will use **Google Cloud Platform (GCP)** as our primary cloud provider.

## Rationale

### 1. Educational Sector Alignment

**GCP Advantages:**
- **Google Workspace Integration**: Native integration with Google Classroom, Drive, and Docs - used by 170M+ students globally
- **Chrome OS Dominance**: 50M+ students use Chromebooks in education
- **Educational Programs**: Google for Education provides credits and resources
- **Single Sign-On**: Students already have Google accounts through their institutions

**AWS Limitations:**
- Limited educational tool integration
- No native classroom management tools
- Requires separate identity management

**Azure Considerations:**
- Strong with Microsoft 365 Education
- Teams for Education integration
- However, less prevalent in K-12 compared to Google

### 2. AI/ML Infrastructure for Custom Development

Since Aksio will develop proprietary AI/ML algorithms in-house, we need robust compute infrastructure rather than pre-built AI services. All major cloud providers offer similar ML infrastructure:

**Infrastructure Comparison:**

| Capability | GCP | AWS | Azure |
|------------|-----|-----|-------|
| **Kubernetes** | GKE | EKS | AKS |
| **GPU Instances** | A100, T4, V100 | Same GPUs | Same GPUs |
| **Custom Chips** | TPUs (unique) | Inferentia/Trainium | None |
| **Object Storage** | Cloud Storage | S3 | Blob Storage |
| **Data Warehouse** | BigQuery | Redshift | Synapse |
| **Notebooks** | Vertex Workbench | SageMaker Studio | ML Studio |

**Key Differentiators:**

**TPU Availability (GCP Only):**
- TPUs can be 2-5x more cost-effective than GPUs for specific workloads
- However, requires code optimization for TPU architecture
- Most ML frameworks are GPU-first, TPU support varies

**GPU Pricing Reality:**
- All providers use the same NVIDIA GPUs (A100, V100, T4)
- Pricing varies by 10-20%, not dramatically different
- Spot/preemptible instances available on all platforms

**AWS Advantages:**
- Largest selection of instance types
- Inferentia chips for inference (though less flexible than GPUs)
- More mature spot instance market

**Azure Advantages:**
- Strong PyTorch partnership
- Good integration if using GitHub (GitHub Actions)
- Competitive GPU pricing in some regions

**GCP Moderate Advantages:**
- TPU option for compatible workloads
- Slightly simpler pricing model
- Good balance of features without overwhelming complexity

### 3. Cost Analysis with Verified Pricing (2025)

**Detailed Cost Comparison Based on Actual Published Rates:**

| Service | GCP | AWS | Azure | Sources |
|---------|-----|-----|-------|------|
| **Serverless Compute** | | | | |
| - Pricing per million requests | $0.40 | $0.20 | $0.20 | [1][2][3] |
| - Compute pricing | $0.0000024/vCPU-s | $0.0000166667/GB-s | $0.000016/GB-s | [1][2][3] |
| - Free tier (monthly) | 2M requests, 180k vCPU-s | 1M requests, 400k GB-s | 1M requests, 400k GB-s | [1][2][3] |
| - Monthly estimate (10M req, 400ms avg) | **$13.69** | **~$15-20** | **~$18-25** | Calculated |
| | | | | |
| **Database (PostgreSQL - Smallest Production)** | | | | |
| - Instance type | db-f1-micro | db.t3.micro | B1ms | |
| - Specs | 0.6 vCPU, 0.6GB RAM | 2 vCPU, 1GB RAM | 1 vCore, 2GB RAM | |
| - Base monthly cost | $15-30 | $21.90 | $12.41 | [4][5][6] |
| - Storage (20GB) | ~$3.40 | ~$2.30 | ~$2.50 | [4][5][6] |
| - Monthly estimate with storage | **$18-33** | **$24.20** | **$14.91** | Calculated |
| | | | | |
| **ML/GPU Compute (Dev - 100 hrs/month)** | | | | |
| - NVIDIA T4 hourly | $0.75 | ~$0.90 (g4dn) | ~$1.20 | [7][8] |
| - NVIDIA V100 hourly | $2.55-2.88 | $3.06 (p3.2xl) | ~$3.50 | [7][8] |
| - NVIDIA A100 hourly | $3.67 | ~$5-8 (p4d) | N/A | [7][8] |
| - Monthly (100 hrs T4) | **$75** | **$90** | **$120** | Calculated |
| | | | | |
| **Storage & CDN** | | | | |
| - Object storage ($/GB/month) | $0.020 | $0.023 | $0.0184 | Official docs |
| - CDN ($/GB transferred) | $0.08-0.20 | $0.085-0.20 | $0.081-0.15 | Official docs |
| - Monthly (100GB storage, 50GB CDN) | **$6-12** | **$6.5-12.5** | **$6-10** | Calculated |
| | | | | |
| **TOTAL MVP ESTIMATE** | **$113-130** | **$136-143** | **$159-170** | |

**Sources:**
1. [Google Cloud Run Pricing](https://cloud.google.com/run/pricing)
2. [AWS Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
3. [Azure Functions Pricing](https://azure.microsoft.com/pricing/details/functions/)
4. [Cloud SQL PostgreSQL Pricing](https://cloud.google.com/sql/docs/postgres/pricing)
5. [AWS RDS PostgreSQL Pricing](https://aws.amazon.com/rds/postgresql/pricing/)
6. [Azure Database PostgreSQL Pricing](https://azure.microsoft.com/pricing/details/postgresql/flexible-server/)
7. [GCP GPU Pricing](https://cloud.google.com/compute/gpus-pricing)
8. [AWS EC2 GPU Instances](https://aws.amazon.com/ec2/instance-types/)

**Key Findings from Actual Pricing:**

1. **Serverless**: AWS and Azure charge less per request but GCP's free tier is more generous
2. **Database**: Azure is cheapest for basic PostgreSQL
3. **GPU/ML**: GCP has competitive T4 pricing, AWS P4 instances offer better A100 value
4. **Real difference**: ~15-30% between providers, not dramatic

**Important Caveats:**
- Prices vary by region (these are US/Europe baseline prices)
- Committed use/reserved instances can reduce costs by 25-52%
- Network egress costs can add significantly to bills
- Support plans not included (can add $100-1000+/month)

### 4. Technical Capabilities

**Cloud Run Superiority:**
- Industry-leading serverless container platform
- Automatic HTTPS with managed certificates
- Built-in authentication (IAP)
- Traffic splitting for canary deployments
- WebSockets support
- Sub-500ms cold starts

**Database Services:**
- Cloud SQL with full PostgreSQL support including extensions
- Automatic backups and point-in-time recovery
- High availability with automatic failover
- Read replicas for scaling
- Support for both relational and graph databases

### 5. GDPR Compliance and European Market Focus

**Reality Check: All major providers offer comprehensive GDPR compliance**

| Feature | GCP | AWS | Azure |
|---------|-----|-----|-------|
| **Data Processing Agreement** | Cloud DPA with SCCs | DPA with SCCs | DPA with SCCs |
| **EU Data Residency** | Yes (multiple regions) | Yes (multiple regions) | Yes + EU Data Boundary |
| **ISO Certifications** | 27001, 27017, 27018 | 27001, 27017, 27018, 27701 | 27001, 27017, 27018 |
| **Data Subject Rights Tools** | Limited automation | Via AWS Artifact | DSR Portal (automated) |
| **Audit Logging** | Cloud Audit Logs | CloudTrail | Azure Monitor |
| **Encryption** | Default + CMEK | Default + KMS | Default + Key Vault |
| **Compliance Resources** | GDPR Resource Center | GDPR Center | Compliance Manager (free) |

**Key Differences:**
- **Azure**: Most mature with EU Data Boundary (completed Feb 2025) - all data including support stays in EU
- **AWS**: Most comprehensive documentation and compliance reports via AWS Artifact
- **GCP**: Simplest to implement but fewer automated compliance tools

**For Aksio's Needs:**
All three providers meet GDPR requirements. The choice isn't about compliance capability but about:
- Azure has the strongest EU data sovereignty commitment
- AWS has the most mature compliance ecosystem
- GCP has adequate compliance with simpler implementation

### 6. Norwegian Presence and Data Sovereignty

**Google's Investment in Norway:**

**New Data Center in Skien (2024-2026):**
- **€600 million investment** in first Norwegian data center
- **Location**: Skien, 85 miles southwest of Oslo  
- **Capacity**: 240MW initially (20MW → 100MW → 120MW phases)
- **Timeline**: Construction started 2024, operational by 2026
- **Jobs**: 2,000 during construction, 100+ permanent positions
- **Sustainability**: 99% carbon-free energy, waste heat recovery

**Strategic Importance for Aksio:**
- **Data sovereignty**: Norwegian data can stay in Norway after 2026
- **Lower latency**: ~5-10ms vs 20-30ms to other European regions
- **Norwegian law**: Potential for data residency under Norwegian jurisdiction
- **Local support**: Google establishing Norwegian presence and partnerships
- **Green profile**: Aligns with Norwegian sustainability values

**Comparison with Competitors:**
- **Azure**: Has Norway East/West regions (operational since 2019)
- **AWS**: No Norwegian regions (nearest is Stockholm)
- **GCP**: Will have the newest, most modern Norwegian infrastructure

While Azure currently has operational Norwegian regions, Google's upcoming state-of-the-art facility represents a major commitment to the Norwegian market, providing future-proof infrastructure for Aksio's growth.

### 7. Developer Experience and Personal Perspective

**Personal Experience (Olav Selnes Lorentzen):**

Having worked extensively with both Azure and Google Cloud, the developer experience difference is significant:

**Azure Experience:**
- Portal feels cluttered with enterprise features we don't need
- Resource groups add unnecessary complexity for our scale
- ARM templates are verbose compared to Terraform on GCP
- Azure DevOps feels heavy for a small team
- Constant context switching between blade interfaces

**Google Cloud Experience:**
- Clean, intuitive console that doesn't overwhelm
- gcloud CLI is more intuitive than az CLI
- Cloud Shell with built-in editor is incredibly convenient
- Cloud Run deployment is literally one command
- Logs are centralized and easy to search

**Google Workspace Integration:**
As we already use Google Workspace for organizational administration:
- Single identity provider for both cloud and productivity tools
- Seamless IAM integration with Google Groups
- Cloud Identity unifies user management
- No need for separate admin portals
- Team already familiar with Google's UX patterns

**Developer Productivity Gains:**
- Cloud Run: Deploy in seconds vs. minutes with Azure Container Instances
- Cloud Build: Simple YAML vs. complex Azure Pipelines
- Firebase integration for future mobile features
- Better VS Code integration through Cloud Code extension

## Alternatives Considered

### Amazon Web Services (AWS)

**Pros:**
- Largest ecosystem and marketplace
- Most mature services
- Extensive third-party integrations
- Strong enterprise adoption

**Cons:**
- 30-50% more expensive for our use case
- Steeper learning curve
- Complex IAM and networking
- Lambda cold starts slower than Cloud Run
- Weaker educational sector integration

**Verdict**: More complex than needed, higher operational overhead for our team size

### Microsoft Azure

**Pros:**
- Microsoft 365 Education integration
- Azure for Students program
- Strong in higher education
- Good European presence

**Cons:**
- Most expensive option (~1.3x GCP costs)
- Smaller ecosystem than AWS
- Functions less mature than Cloud Run
- Complex pricing model
- Less suitable for K-12 market

**Verdict**: Strong option for Microsoft ecosystems, but less aligned with our target market's existing tools

## Consequences

### Positive Consequences

1. **Native Google Workspace integration** crucial for education sector
2. **Verified cost savings** (~15-30% based on actual pricing)
3. **Cloud Run scale-to-zero** saves money during school off-hours
4. **Simpler platform** reduces operational complexity
5. **Adequate infrastructure** for our custom ML development needs

### Negative Consequences

1. **Smallest ecosystem** of the three major providers
2. **Fewer third-party tools** and marketplace options
3. **Smallest talent pool** (harder to hire GCP experts)
4. **Less enterprise adoption** (may affect partnerships)
5. **Potential vendor lock-in** (though true for all clouds)
6. **Fewer availability zones** than AWS globally
7. **Less mature** in some enterprise features

### Risk Mitigation

1. **Vendor Lock-in Mitigation:**
   - Use Terraform for infrastructure (portable)
   - Containerize applications (run anywhere)
   - Use PostgreSQL (portable database)
   - Abstract GCP-specific services behind interfaces
   - Estimated migration effort: 2-3 months if needed

2. **Talent Shortage Mitigation:**
   - Provide GCP training for team
   - Use familiar tools (Terraform, Docker, PostgreSQL)
   - Leverage GCP's excellent documentation
   - Consider GCP certified partners for specialized needs

3. **Ecosystem Limitations:**
   - Build vs buy evaluation for missing tools
   - Use open-source alternatives where possible
   - Leverage GCP Marketplace growing offerings

## Next Steps

**Immediate Actions:**
1. Continue using existing GCP setup with Terraform
2. Document GCP-specific conventions in team wiki
3. Set up cost monitoring and alerts

**Future Considerations:**
- Evaluate committed use discounts when usage stabilizes
- Plan migration to Norwegian data center when operational (2026)
- Create abstraction layers for GCP-specific services to maintain portability

## Monitoring and Review

**Success Metrics:**
- Monthly cloud costs < $500 for MVP
- API response time < 200ms (p95)
- 99.9% uptime SLA achievement
- Zero security incidents
- Developer satisfaction score > 8/10

**Review Schedule:**
- Quarterly cost analysis
- Annual provider evaluation
- Continuous monitoring of new services

## Conclusion

We choose Google Cloud Platform based on a balanced evaluation:

**Primary Decision Factors:**
1. **Educational Integration** (Decisive): Google Workspace is used by 170M+ students - this native integration is unmatched
2. **Developer Experience** (Important): Personal experience shows significantly smoother development on GCP vs Azure
3. **Google Workspace Integration** (Important): We already use it for organizational admin - unified identity and management
4. **Norwegian Data Center** (Future advantage): €600M investment in Skien facility (2026) shows commitment to Norway
5. **Cloud Run's Scale-to-Zero** (Cost benefit): Matches school traffic patterns perfectly
6. **Simplicity** (Team fit): Less complex than AWS, cleaner than Azure for our small team

**Cost Reality (Based on Actual Pricing):**
- GCP: ~$113-130/month for MVP
- AWS: ~$136-143/month for MVP  
- Azure: ~$159-170/month for MVP
- **Actual difference: 15-30%**, not dramatic but meaningful for education sector

**Acknowledging Trade-offs:**
- Azure actually has cheaper PostgreSQL (B1ms at $12.41/month)
- AWS has better GPU variety and free tier for first year
- GCP has the smallest ecosystem and talent pool

**Why This Makes Sense for Aksio:**
- Google account integration is critical for student adoption
- Personal experience proves faster development velocity on GCP
- Already using Google Workspace - natural extension to GCP
- Norwegian data center coming in 2026 provides local data residency
- Cloud Run's scale-to-zero matches school traffic patterns
- Simpler platform reduces operational overhead for small team

This is a pragmatic choice based on our specific context - an educational platform where Google Workspace integration and cost-efficiency during off-peak hours outweigh the ecosystem limitations.

This decision provides the foundation for all subsequent infrastructure decisions, including our database architecture (ADR-004), which will leverage GCP's managed services for optimal performance and cost efficiency.

## Related Decisions

- ADR-001: Go for Backend Development
- ADR-002: Gin HTTP Router Package
- **ADR-004: Database Architecture** (builds on this GCP decision)

## References

- [Google Cloud for Education](https://cloud.google.com/edu)
- [Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Vertex AI for Education Use Cases](https://cloud.google.com/vertex-ai/docs/start/introduction-unified-platform)
- [GCP GDPR Compliance](https://cloud.google.com/privacy/gdpr)
- [Cloud Provider Comparison - 2025 Gartner Report](https://www.gartner.com/en/documents/cloud-comparison)
- [Educational Technology Cloud Adoption Study](https://educause.edu/cloud-study-2025)

## Appendix: Detailed Service Mapping

| Requirement | GCP Service | AWS Service | Azure Service |
|-------------|------------|-------------|---------------|
| Serverless Compute | Cloud Run | Lambda/Fargate | Container Instances |
| PostgreSQL | Cloud SQL | RDS | Database for PostgreSQL |
| Object Storage | Cloud Storage | S3 | Blob Storage |
| CDN | Cloud CDN | CloudFront | Azure CDN |
| Secrets | Secret Manager | Secrets Manager | Key Vault |
| ML Infrastructure | GKE, TPUs, GPUs | EKS, EC2 GPUs | AKS, VM GPUs |
| Data Processing | Dataflow, BigQuery | Kinesis, Redshift | Stream Analytics, Synapse |
| Authentication | Identity Platform | Cognito | Azure AD B2C |
| Monitoring | Cloud Monitoring | CloudWatch | Azure Monitor |
| CI/CD | Cloud Build | CodeBuild | Azure DevOps |

## Approval
**Decision Maker(s):** Olav Selnes Lorentzen  
**Date Approved:** 2025-08-14