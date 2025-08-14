# ADR-005: Authentication and User Management Strategy

> **Technical Note**: This document outlines the authentication strategy for Aksio's educational platform. Implementation details are maintained in separate internal documentation for security purposes.

## Status
- 2025-08-14 - Draft
- 2025-08-14 - **Accepted**

## Context

The Aksio educational platform requires a secure, user-friendly authentication system that accommodates diverse user types in the education sector. With the EU Digital Services Act fully in effect (since February 17, 2024) and the AI Act requirements for educational systems (in force since August 1, 2024), we must balance security, compliance, and user experience while maintaining data sovereignty.

### Key Requirements

**User Base:**
- Individual student sign-ups (16+ age restriction globally)
- Two initial roles: Student and Admin
- Email as primary identifier (no username system)
- Early access/waitlist system for controlled launch

**Authentication Methods Priority:**
1. Google Sign-In (primary for MVP)
2. Email/password (essential for non-Google users)
3. Sign in with Apple (required for App Store, future iteration)
4. SSO/SAML (potential future for institutions)

**Security & Compliance:**
- Age restriction: 16+ globally (covers all EU GDPR requirements)
- Email verification required
- Password reset capability
- Secure session management with token rotation
- Multi-device support with security monitoring
- GDPR-compliant data handling
- Comprehensive audit trail

## Decision

We will implement a **self-managed authentication system** using **secure token-based authentication** with **PostgreSQL as the single source of truth** for all user data, with **16+ age restriction** to avoid complex parental consent requirements.

## Rationale

### Why Self-Managed Authentication?

Following our architectural philosophy established in ADR-004 (choosing PostgreSQL+AGE over dual database systems), we apply the same principle here: **one unified system we fully control** rather than splitting identity management across multiple services.

**Alignment with Our Architecture:**
1. **Single source of truth** - All user data in PostgreSQL (no sync issues)
2. **Full control** - Customize any aspect of authentication
3. **Cost efficiency** - No additional service charges
4. **Data sovereignty** - All data in EU-based infrastructure
5. **Operational simplicity** - One system to monitor, backup, and debug

### Age Restriction Strategy (16+ Global)

**Why 16+ Instead of 13+:**
1. **Covers all EU countries** - Germany, Netherlands require 16+ for GDPR
2. **Simplifies GDPR compliance** - No parental consent needed in any EU country
3. **Australia compliant** - Their limit is 16 for social platforms
4. **Future-proof** - DSA already in effect, EUDI Wallet coming December 2026
5. **Avoids complexity** - Single global age threshold simplifies operations

**Additional benefit**: Also avoids US COPPA requirements (which apply to under-13)

**Legal Requirements in August 2025:**
- EU Digital Services Act has been fully in effect since February 17, 2024
- DSA enforcement and guidance continues to evolve
- Enhanced age verification technically required under DSA
- For 16+ users, date of birth collection with clear Terms of Service remains common practice
- EUDI Wallet not yet available (deployment December 2026, mandatory acceptance November 2027)

### Architecture Overview

Our authentication system follows industry-standard security patterns with multiple layers of protection, comprehensive audit logging, and compliance-focused design. The architecture emphasizes defense in depth, data minimization, and privacy by design principles.

## Future Considerations: Enhanced Age Verification

**Current Status (August 2025):**
- February 17, 2024: DSA became fully applicable to all platforms
- 2025: DSA enforcement continues with evolving guidance
- August 2025 (Now): Platforms implementing enhanced verification per new guidelines
- December 2026: EU member states must provide EUDI Wallets to citizens
- November 2027: Businesses must accept EUDI Wallets for authentication

### EU Digital Identity Wallet (EUDI) Integration

The EUDI Wallet will fundamentally change age verification:
- Privacy-preserving age verification (zero-knowledge proofs)
- User proves they're 16+ without revealing exact age
- No personal data shared with service provider
- Interoperable across all EU member states

**Recommendation**: Continue with self-declaration for 16+ users while monitoring DSA enforcement patterns. Be prepared to integrate specialized KYC providers as regulatory requirements evolve.

## AI Act Considerations

**Note on AI Act Coverage**: The EU AI Act (entered into force August 1, 2024, with AI literacy obligations since February 2, 2025) requirements for educational AI systems are **not covered in this authentication ADR**. These requirements will be addressed in a separate future ADR on Learning Analytics and AI Strategy.

**TODO**: Update this reference with the specific ADR number once the AI Act compliance ADR is created.

## Alternatives Considered

### Firebase Authentication
**Verdict**: Contradicts our unified architecture principle

### Google Identity Platform
**Verdict**: Better than Firebase but still creates dual-system complexity

### Auth0
**Verdict**: Too expensive and complex for education startup

### Supabase Auth
**Verdict**: Interesting but adds unnecessary complexity

## Key Implementation Priorities

**Security**: Industry-standard security controls including secure password storage, input validation, rate limiting, and comprehensive audit logging.

**Compliance**: GDPR-compliant data handling with clear consent mechanisms, data retention policies, and 16+ age restriction enforcement.

**Future Requirements**: Monitoring evolving DSA enforcement, preparing for EUDI Wallet integration (December 2026), and planning for mandatory EUDI acceptance (November 2027).

## Compliance Strategy

Our authentication system is designed to meet current and upcoming regulatory requirements:

- **GDPR**: Full compliance through privacy by design, clear consent, and data minimization
- **DSA**: 16+ restriction simplifies compliance with new minor protection guidelines
- **AI Act**: Learning analytics will be addressed in separate ADR
- **UK AADC**: 16+ restriction avoids most requirements (in effect since September 2021)

## Success Metrics

The system will maintain high security standards while ensuring regulatory compliance and optimal user experience.

## Related Decisions

- ADR-001: Go for Backend Development
- ADR-002: Gin HTTP Router
- ADR-003: Google Cloud Platform
- ADR-004: PostgreSQL Database (single source of truth principle)

## References

- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [EU Digital Identity Wallet](https://commission.europa.eu/strategy-and-policy/priorities-2019-2024/europe-fit-digital-age/european-digital-identity_en)
- [GDPR Age of Consent by Country](https://gdpr-info.eu/art-8-gdpr/)
- [Digital Services Act](https://digital-strategy.ec.europa.eu/en/policies/digital-services-act-package) (In effect since February 17, 2024)
- [EU AI Act](https://artificialintelligenceact.eu/) (In force since August 1, 2024)
- [UK Age Appropriate Design Code](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/) (In effect since September 2, 2021)

## Approval

**Decision Maker(s):** Olav Selnes Lorentzen  
**Date Approved:** 2025-08-14