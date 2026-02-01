# Security & Secret Management for AI Services: Assignment

## Overview

This assignment tests your understanding of secure access, secrets management, and authorization in production AI systems. You'll work through scenarios that challenge you to apply OAuth 2.0, API key management, secrets rotation strategies, and the Principle of Least Privilege (PoLP) in real-world contexts.

---

## SECTION A: OBJECTIVE QUESTIONS (Easy & Moderate Difficulty)

### Easy Questions (Knowledge-Based Understanding)

**Question 1 (MCQ): Understanding OAuth 2.0 vs. API Keys**

Which of the following is the PRIMARY advantage of using OAuth 2.0 access tokens over static API keys in production AI services?

A) OAuth 2.0 tokens are easier to generate and require no setup  
B) OAuth 2.0 tokens automatically expire and can be revoked, limiting exposure window if compromised  
C) OAuth 2.0 tokens never require encryption during transmission  
D) OAuth 2.0 tokens allow unlimited access to all resources without scope restrictions  

**Correct Answer:** B

---

**Question 2 (MCQ): Secrets Rotation Frequency**

According to current security best practices for production AI services, what is the recommended maximum duration for which a secret should remain active before rotation?

A) 1 year  
B) 6 months  
C) 90 days  
D) 30 days  

**Correct Answer:** C

---

**Question 3 (MSQ): Principle of Least Privilege in Action**

Select ALL statements that correctly describe the Principle of Least Privilege (PoLP) in the context of AI services:

A) A data processing microservice should have read-only access to the database table it queries, not write permissions  
B) An AI model inference service should be granted admin access to all cloud resources for flexibility  
C) Service accounts should have access limited to only the specific APIs and resources they need  
D) Users should receive the minimum permissions required to complete their specific tasks  

**Correct Answers:** A, C, D

---

**Question 4 (MCQ): Secret Manager Rotation Strategy**

Your AI service needs to rotate database credentials every 30 days automatically. Which approach best aligns with modern security practices?

A) Store credentials in environment variables and manually update them monthly  
B) Use a dedicated Secret Manager service (like AWS Secrets Manager or HashiCorp Vault) with automated rotation  
C) Hardcode credentials in your application code and change them when breaches occur  
D) Store credentials in version control with a restricted branch policy  

**Correct Answer:** B

---

**Question 5 (MCQ): JWT Token Expiration Best Practice**

For an API serving sensitive AI model inference requests, what is the recommended maximum expiration time for an access token?

A) 7 days  
B) 24 hours  
C) 15-30 minutes  
D) 1 year  

**Correct Answer:** C

---

**Question 6 (MSQ): Zero-Trust Architecture Principles**

Which of the following are core principles of Zero-Trust security for API access in AI services? (Select all that apply)

A) Assume all traffic is trustworthy until proven otherwise  
B) Verify every user and device identity before granting access, regardless of network location  
C) Implement continuous authentication and authorization checks  
D) Grant broad permissions to reduce administrative overhead  

**Correct Answers:** B, C

---

**Question 7 (MCQ): Comparing RBAC and ABAC for AI Services**

Your organization is deploying multiple AI services with varying permission requirements based on user department, resource tags, and time of access. Which access control model is most flexible for this scenario?

A) Role-Based Access Control (RBAC) only  
B) Attribute-Based Access Control (ABAC)  
C) Static API key management  
D) No access control (open access)  

**Correct Answer:** B

---

**Question 8 (MSQ): API Security Posture Components**

Which of the following should be implemented to maintain a secure API security posture for production AI services? (Select all that apply)

A) Enforce TLS 1.3 for all API communications  
B) Verify JWT signatures properly to prevent token forgery  
C) Implement Role-Based Access Control (RBAC) with OAuth 2.0  
D) Allow API keys to be transmitted in plain text for debugging purposes  

**Correct Answers:** A, B, C

---

## SECTION B: SUBJECTIVE QUESTION (Hard Difficulty - Application & Synthesis)

### Question 9 (Subjective): Designing a Secure Secret Management System

**Scenario:**

You are an AI engineer at a fintech company building a production AI fraud detection service. Your system needs to:

1. Call multiple third-party AI model APIs (each with unique API keys)
2. Connect to a PostgreSQL database for training data
3. Store temporary authentication tokens from OAuth 2.0 providers
4. Run scheduled batch jobs that process sensitive financial data
5. Scale across multiple Kubernetes clusters in different regions

**Your Task:**

Design a comprehensive secret management and access control strategy for this system. Your design should address:

1. **Secret Storage & Rotation:** How would you store and rotate the API keys, database credentials, and OAuth tokens? Name the tools/services you'd use and explain the rotation frequency for each type of secret.

2. **Access Control Model:** Would you use RBAC, ABAC, or a combination? Justify your choice based on the complexity of your system. Provide at least 2 specific examples of how you'd apply your chosen model.

3. **Principle of Least Privilege Implementation:** Identify at least 3 specific ways you would apply PoLP to different components of this system (e.g., microservices, batch jobs, developers).

4. **Zero-Trust Architecture:** Describe how you would implement zero-trust principles for inter-service communication within your Kubernetes clusters. What verification mechanisms would you use?

5. **Incident Response:** If an API key was exposed in your Git repository history, what immediate steps would you take to mitigate the breach?

**Submission Requirements:**

- Provide a detailed written response (800-1200 words)
- Include a simple architecture diagram or flowchart showing how secrets flow through your system
- Justify all decisions with references to security best practices

---

## ANSWER KEY AND SOLUTIONS

### Objective Questions Answer Key

**Question 1:** B  
**Explanation:** OAuth 2.0 access tokens have a critical security advantage over static API keys: they automatically expire (typically within 15-30 minutes for sensitive APIs) and can be revoked immediately. This limits the exposure window if a token is compromised. Static API keys, by contrast, remain valid indefinitely until manually rotated, making them a larger security risk in production environments.

**Common Misconceptions:**
- A is incorrect: OAuth 2.0 requires proper setup and configuration
- C is incorrect: Both OAuth tokens and API keys must be encrypted in transit (TLS)
- D is incorrect: OAuth 2.0 uses scopes to restrict access to specific resources

---

**Question 2:** C  
**Explanation:** Industry best practices and compliance standards recommend rotating secrets at least every 90 days. For highly sensitive secrets (like database passwords or API keys for critical services), rotation may occur more frequently (30-60 days). This reduces the risk window if a secret is compromised without being detected.

**Why other options are incorrect:**
- A & B: Too long; increases exposure if a secret is compromised
- D: While more frequent rotation is good, 30 days may be operationally challenging for all secrets

---

**Question 3:** A, C, D  
**Explanation:** PoLP means each component, service, and user gets only the minimum permissions needed to function.
- A is correct: A data processor needs read access, not write
- B is incorrect: Admin access violates PoLP; it's excessive
- C is correct: Service accounts should have scoped, limited permissions
- D is correct: Users should follow the principle of minimum necessary access

---

**Question 4:** B  
**Explanation:** Modern security practices require automated secret rotation using dedicated services like AWS Secrets Manager, HashiCorp Vault, or Azure Key Vault. These tools:
- Automate rotation on a schedule
- Track rotation history
- Enable seamless credential updates without downtime
- Integrate with applications automatically

Manual rotation (A) is error-prone and doesn't scale. Hardcoding (C) and version control (D) are security anti-patterns that expose secrets.

---

**Question 5:** C  
**Explanation:** Best practices recommend JWT access tokens expire within 15-30 minutes for sensitive APIs. This minimizes the window of exposure if a token is stolen. Longer expiration times (A, B, D) increase risk; if a token is compromised, it remains valid for too long.

**Implementation pattern:** Use short-lived access tokens with longer-lived refresh tokens to balance security and user experience.

---

**Question 6:** B, C  
**Explanation:** Zero-Trust is built on "never trust, always verify":
- B is correct: Every access request requires identity verification, regardless of origin
- C is correct: Continuous verification ensures compromised credentials are caught quickly
- A is incorrect: This is the opposite of Zero-Trust
- D is incorrect: Broad permissions contradict Zero-Trust principles

---

**Question 7:** B  
**Explanation:** ABAC (Attribute-Based Access Control) is more flexible than RBAC for complex scenarios. It allows access decisions based on:
- User attributes (department, role, team)
- Resource attributes (tags, classification level)
- Environment attributes (time, location, device type)

RBAC (A) is simpler but less flexible for multi-dimensional access requirements. Options C and D are security anti-patterns.

---

**Question 8:** A, B, C  
**Explanation:** A secure API posture requires:
- A is correct: TLS 1.3 encrypts all data in transit
- B is correct: Verifying JWT signatures prevents token forgery and unauthorized access escalation
- C is correct: RBAC with OAuth 2.0 provides strong authentication and authorization
- D is incorrect: API keys must NEVER be transmitted in plain text; this is a critical security violation

---

### Subjective Question: Model Answer & Evaluation Rubric

**Model Solution Outline (800-1200 words expected):**

**1. Secret Storage & Rotation Strategy:**

For this fintech AI fraud detection service, I would implement a tiered secret management approach:

**API Keys (Third-party AI models):**
- **Storage:** HashiCorp Vault or AWS Secrets Manager with encryption at rest
- **Rotation Frequency:** 90 days (or immediately if compromised)
- **Implementation:** Use Vault's dynamic secrets feature to generate temporary credentials that expire after each use, or AWS Secrets Manager's Lambda-based rotation
- **Access:** Services retrieve secrets via authenticated requests; keys are never stored in application code or environment variables

**Database Credentials (PostgreSQL):**
- **Storage:** AWS Secrets Manager with automatic rotation
- **Rotation Frequency:** 30 days for production, 60 days for non-prod
- **Implementation:** Use AWS Secrets Manager's managed rotation, which automatically updates both the secret and the database user password
- **Access:** Applications use temporary database sessions via credential injection at runtime

**OAuth 2.0 Tokens:**
- **Storage:** In-memory cache with automatic refresh before expiration
- **Rotation Frequency:** Automatic refresh 5 minutes before token expiration
- **Implementation:** Use OAuth 2.0 refresh token flow; never store long-lived tokens
- **Access:** Tokens are generated on-demand and cached temporarily

**Batch Job Credentials:**
- **Storage:** Separate vault instance or namespace in Secrets Manager
- **Rotation Frequency:** 60 days
- **Implementation:** Batch jobs authenticate via service accounts with temporary credentials
- **Access:** Kubernetes ServiceAccounts with IRSA (IAM Roles for Service Accounts) in AWS

---

**2. Access Control Model Selection:**

I would implement a **hybrid RBAC + ABAC approach**:

**RBAC Foundation:**
- Define roles: DataScientist, MLEngineer, DevOps, Analyst, Admin
- Each role has predefined permissions for common tasks
- Simplifies management and auditing

**ABAC Enhancement:**
- Layer attributes on top of roles for fine-grained control
- Example 1: A DataScientist can access training data, but only for the fraud detection project (project attribute), and only during business hours (time attribute)
- Example 2: An MLEngineer can deploy models to staging (environment attribute) but requires additional approval for production deployment

**Why hybrid?** RBAC provides baseline simplicity; ABAC adds flexibility for complex scenarios without overwhelming administrators.

---

**3. Principle of Least Privilege Implementation:**

**Application 1 - Microservices:**
- Fraud detection inference service: Read-only access to real-time transaction data; no access to training data or user PII
- Model training service: Read access to historical transaction data; write access only to model artifacts; no access to payment processing APIs
- Data pipeline service: Read access to raw data sources; write access only to processed datasets; no access to model inference endpoints

**Application 2 - Batch Jobs:**
- Scheduled retraining jobs run with a dedicated service account that has:
  - Read access to training data in S3
  - Write access only to model artifact storage
  - No access to inference endpoints or production databases
  - Time-limited credentials that expire after job completion

**Application 3 - Developer Access:**
- Developers have read access to non-sensitive logs and metrics
- Database access is provided through a read-only replica, never the production database
- Secrets are accessed through a centralized audit log; developers never see raw secret values
- Each developer's access is tagged with their identity for accountability

---

**4. Zero-Trust Architecture for Inter-Service Communication:**

Within Kubernetes clusters, I would implement:

**Mutual TLS (mTLS):**
- All pod-to-pod communication is encrypted and authenticated
- Use a service mesh (Istio or Linkerd) to enforce mTLS automatically
- Certificates are automatically rotated by the service mesh

**Service Accounts & RBAC:**
- Each microservice runs with a unique Kubernetes ServiceAccount
- Network policies restrict which services can communicate with each other
- Example: Only the API gateway can communicate with the fraud detection service; the fraud detection service cannot initiate calls to other services

**Continuous Verification:**
- Every request includes a signed JWT from the calling service
- The receiving service verifies the JWT signature and checks the caller's permissions
- Access logs record every inter-service call for audit trails

**Identity Verification:**
- Services authenticate via SPIFFE/SVID (Secure Production Identity Framework for Everyone)
- Certificates are automatically issued and rotated by the Kubernetes control plane

---

**5. Incident Response for Exposed API Key:**

**Immediate Actions (within 5 minutes):**
1. Revoke the exposed API key in the Secrets Manager
2. Trigger automatic rotation to generate a new key
3. Alert the security team and relevant stakeholders
4. Check logs to determine if the key was used after exposure

**Short-term Actions (within 1 hour):**
1. Audit all API calls made with the exposed key
2. Identify if any unauthorized access occurred
3. Scan the Git repository history to find all instances of the key
4. Use GitGuardian or similar tools to detect if the key appeared in other repositories

**Medium-term Actions (within 24 hours):**
1. Implement pre-commit hooks to detect secrets in code
2. Rotate all related secrets (database credentials, OAuth tokens)
3. Conduct a security review of how the key was exposed
4. Update documentation and training to prevent recurrence

**Long-term Actions:**
1. Implement secret scanning in CI/CD pipeline
2. Use tools like TruffleHog to scan repository history for secrets
3. Enforce secret management policies across all teams
4. Conduct incident postmortem and update security procedures

---

**Architecture Diagram Description:**

```
┌─────────────────────────────────────────────────────────────────┐
│                     Kubernetes Cluster (Multi-region)            │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐         ┌──────────────┐    ┌─────────────┐  │
│  │  API Gateway │ ──mTLS──│ Fraud Detect │    │ Data Pipeline│ │
│  │              │         │  Service     │    │  Service     │  │
│  └──────┬───────┘         └──────┬───────┘    └──────┬──────┘  │
│         │                        │                   │          │
│         └────────────┬───────────┴───────────────────┘          │
│                      │                                           │
│              ┌───────▼──────────┐                               │
│              │  Service Mesh    │ (mTLS enforcement)            │
│              │  (Istio/Linkerd) │                               │
│              └──────────────────┘                               │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
                ┌─────────────────────────┐
                │  AWS Secrets Manager    │
                │  (Encrypted at rest)    │
                │  - API Keys             │
                │  - DB Credentials       │
                │  - OAuth Tokens         │
                └─────────────────────────┘
                            │
         ┌──────────────────┼──────────────────┐
         ▼                  ▼                  ▼
    ┌─────────┐        ┌──────────┐      ┌──────────┐
    │PostgreSQL│        │ S3 Data  │      │Third-party│
    │Database  │        │ Storage  │      │AI APIs    │
    └─────────┘        └──────────┘      └──────────┘
```

---

**Evaluation Rubric for Subjective Question:**

| Criteria | Excellent (90-100) | Good (75-89) | Satisfactory (60-74) | Needs Improvement (<60) |
|----------|-------------------|--------------|----------------------|------------------------|
| Secret Storage & Rotation | Comprehensive tiered approach with specific tools and frequencies | Clear strategy with most details | Basic approach with some missing details | Incomplete or vague strategy |
| Access Control Model | Justified hybrid approach with 2+ specific examples | Clear model choice with 1-2 examples | Basic model selection with limited examples | No clear model or examples |
| PoLP Implementation | 3+ specific applications across different components | 2-3 applications with clear explanations | 2 applications with basic explanations | 1 or fewer applications |
| Zero-Trust Architecture | Detailed mTLS, service accounts, and continuous verification | Clear implementation with most components | Basic zero-trust concepts mentioned | Limited or incorrect understanding |
| Incident Response | Comprehensive 5-step plan with timeline | Clear plan with 4 steps | Basic response plan | Incomplete or reactive response |
| Writing Quality | Clear, professional, well-organized | Generally clear and organized | Somewhat clear but disorganized | Unclear or poorly organized |
| Diagram/Visual | Clear, accurate, helpful architecture diagram | Adequate diagram with most elements | Simple diagram with some elements | No diagram or unclear diagram |

---

## Additional Resources for Learning

- **OAuth 2.0 & JWT:** IETF RFC 6749, RFC 7519
- **Secret Management:** HashiCorp Vault documentation, AWS Secrets Manager guides
- **Zero-Trust:** NIST Zero Trust Architecture (SP 800-207)
- **RBAC/ABAC:** OWASP Authorization Cheat Sheet
- **PoLP:** CIS Controls v8, NIST Cybersecurity Framework