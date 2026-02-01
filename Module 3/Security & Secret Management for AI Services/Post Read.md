# Security & Secret Management for AI Services: Comprehensive Lecture Notes

## What You'll Learn

In this lesson, you'll learn to:

- **Explain how OAuth 2.0 and API keys work** together to secure access to AI services, and understand when to use each one
- **Implement secrets rotation strategies** using modern tools like HashiCorp Vault and cloud secret managers to keep credentials fresh and secure
- **Apply the principle of least privilege** to AI services so each component only gets the exact permissions it needs—nothing more
- **Design secure authorization patterns** that protect your production AI systems from unauthorized access and credential compromise

---

## Detailed Explanation

### A. What Is Secure Access and Secret Management?

Imagine you're building a digital apartment complex. Your residents (users and services) need keys to access different rooms, but you can't give everyone a master key. Some residents need access to the gym, others to the library, and only the maintenance team needs the boiler room. That's essentially what secure access and secret management do for AI services and production systems.

**Secure access and secret management** is the practice of controlling who or what can access your systems, protecting the sensitive information (like passwords, API keys, and tokens) that grant that access, and ensuring those credentials stay fresh, valid, and impossible for attackers to misuse.

In the context of AI services specifically, this becomes even more critical. Modern AI systems often need to call multiple APIs, access databases, authenticate with third-party services, and coordinate with other microservices—all while running automatically without a human typing in a password. This creates a unique challenge: how do you give an AI service the credentials it needs without accidentally exposing them to attackers?

The answer involves three interconnected pieces: **authentication** (proving who you are), **authorization** (determining what you're allowed to do), and **secret management** (keeping sensitive credentials safe and up-to-date). Together, these form the foundation of production-grade AI security.

---

### B. Why This Matters for Production AI Services

Let's ground this in reality. Consider an AI-powered customer support chatbot running on your company's servers. This chatbot needs to:

- Authenticate with your customer database to look up account information
- Call the payment processing API to check transaction history
- Access your company's internal knowledge base
- Send notifications through an email service API
- Log activity to a security monitoring system

Each of these integrations requires credentials. If a single API key gets exposed—perhaps through a poorly secured configuration file, a developer's GitHub commit, or a supply chain attack—an attacker now has the same level of access as your chatbot. They could read customer data, process unauthorized transactions, or damage your knowledge base.

**The stakes are higher with AI services** because these systems often run continuously and autonomously. Unlike a human employee who might notice something suspicious and alert security, an AI service will happily continue executing commands with compromised credentials until someone notices unusual activity in the logs—which might be days or weeks later.

By implementing proper secret management and access control, you reduce the "blast radius" of a compromise. If one credential is leaked, you can rotate it quickly without affecting other services. If an AI agent gets compromised, it can only access the specific resources it needs, not your entire infrastructure.

---

### C. Understanding OAuth 2.0: The Modern Standard for Delegated Access

**What is OAuth 2.0?**

OAuth 2.0 is a widely-adopted standard for allowing users or services to grant access to their resources without sharing passwords. Think of it like giving a valet a special key that only opens your car doors and trunk—not your house or office. The valet can do their job without ever knowing your master key.

OAuth 2.0 is particularly powerful for AI services because it solves a fundamental problem: how can you let a service access another service's API without storing the user's actual password in your system? Instead of storing passwords, you store **tokens**—temporary credentials that grant specific access for a limited time.

**The OAuth 2.0 Flow: Step by Step**

Let's walk through a concrete example. Imagine your AI customer support chatbot needs to access a user's Gmail inbox to help with email support tickets.

**Step 1: The User Initiates Authorization**

Your chatbot UI shows a button: "Connect Your Gmail Account." The user clicks it. Behind the scenes, your application redirects the user to Google's authorization server (not your own servers). This is crucial—the user's Gmail password never touches your application.

**Step 2: The User Authenticates with the Provider**

The user sees Google's login screen and enters their credentials. Google verifies the user is who they claim to be. Your application never sees this password.

**Step 3: The Provider Asks for Permission**

Google shows the user a consent screen: "The XYZ Chatbot wants to access your emails. This will allow it to read your messages and reply on your behalf." The user can see exactly what permissions are being requested. This is transparency—the user knows what they're granting.

**Step 4: The Provider Issues a Token**

If the user approves, Google generates an **access token**—a temporary credential that says "This token is authorized to read and send emails for this specific user, and it expires on January 15, 2026 at 3 PM." Google also issues a **refresh token**, which is a longer-lived credential that can be used to get a new access token when the first one expires.

**Step 5: Your Application Uses the Token**

Google redirects the user back to your application with the access token. Your chatbot stores this token (securely, which we'll discuss later) and uses it to call Gmail's API. The API call looks like this:

```
GET /gmail/v1/users/me/messages
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

The `Bearer` keyword followed by the token tells Gmail's API: "This request is authorized by this token."

**Step 6: Token Expiration and Refresh**

After a few hours, the access token expires. Instead of asking the user to re-authenticate (which would be annoying), your application uses the refresh token to get a new access token:

```
POST /token
client_id: your_app_id
client_secret: your_app_secret
refresh_token: the_refresh_token_you_stored
grant_type: refresh_token
```

Google validates the refresh token and issues a new access token. This happens automatically in the background—the user doesn't need to do anything.

**Why OAuth 2.0 Is Better Than Storing Passwords**

- **Passwords are never stored in your system**: If your database gets breached, attackers don't get the user's Gmail password—they get a token that can be revoked immediately.
- **Granular permissions**: Instead of "full access to everything," you can request "read Gmail, send emails, but don't access calendar or contacts."
- **Easy revocation**: Users can revoke access from their Google account settings anytime. Your application doesn't need to change anything.
- **Audit trails**: The provider logs who accessed what and when. This is crucial for security investigations.

---

### D. API Keys: The Simpler Alternative for Service-to-Service Communication

While OAuth 2.0 is powerful for user-facing scenarios, it's overkill for service-to-service communication. When your AI chatbot needs to call your own internal API, you don't need the complexity of redirects and consent screens. This is where **API keys** come in.

**What Is an API Key?**

An API key is a unique identifier that your service presents to prove it's authorized to access an API. It's simpler than OAuth 2.0 but less flexible. Think of it as a passcode rather than a key—it's fixed, doesn't expire automatically, and grants broad access.

**A Simple API Key Example**

Your AI chatbot needs to call your internal recommendation engine API. You create an API key like this:

```
api_key_12345_abcde_xyz789
```

When the chatbot makes a request, it includes this key:

```
GET /api/recommendations?product_id=42
X-API-Key: api_key_12345_abcde_xyz789
```

The API server checks: "Is this a valid API key? Yes. Is it still active? Yes. What permissions does it have? It can read recommendations. Approved." The request goes through.

**API Keys vs. OAuth 2.0: When to Use Each**

| Scenario | Use API Keys | Use OAuth 2.0 |
|----------|-------------|--------------|
| Service calling your own internal API | ✓ | ✗ |
| Third-party app accessing user data | ✗ | ✓ |
| AI service calling external APIs (like Stripe or Twilio) | ✓ | (unless provider supports OAuth) |
| Mobile app accessing your backend | ✗ | ✓ |
| Microservices communicating | ✓ | (sometimes) |

**The Problem with Traditional API Keys**

API keys have a critical weakness: they don't expire. If you create an API key today and store it in a configuration file, that key remains valid forever—or at least until you manually rotate it. In practice, many organizations never rotate their API keys, which means a leaked key can provide access indefinitely.

This is why modern systems combine API keys with **automatic rotation strategies** and **secret management systems** (which we'll cover next).

---

### E. Secrets Rotation: Keeping Credentials Fresh and Secure

**The Rotation Problem**

Imagine you have a single API key that your AI service uses to access your database. This key is stored in a configuration file on your server. Now imagine:

1. A developer accidentally commits this configuration file to GitHub (yes, this happens constantly)
2. A GitHub crawler finds it within hours
3. An attacker now has your database credentials

How long until you notice? Days? Weeks? Months? During that time, the attacker has full access to your customer data.

**Secrets rotation** is the practice of regularly replacing old credentials with new ones. Instead of hoping a leaked credential never gets used, you assume it will be leaked and ensure it becomes useless quickly.

**How Secrets Rotation Works**

Here's a practical example with a database password:

**Day 1: Initial Setup**
Your AI service is configured with password: `MySecurePass123`

**Day 15: Automatic Rotation**
Your secrets management system (like HashiCorp Vault or AWS Secrets Manager) automatically:
1. Generates a new password: `NewPass456SecurelyGenerated`
2. Updates the database to accept both the old and new passwords
3. Updates your AI service's configuration to use the new password
4. Waits for all active connections using the old password to close
5. Disables the old password

**Day 16: The Old Password is Useless**
If an attacker somehow obtained `MySecurePass123` yesterday, they can't use it today. The database rejects it.

**The Rotation Strategy**

Most organizations implement a rotation schedule based on risk:

- **High-risk credentials** (database passwords, root API keys): Rotate every 30 days
- **Medium-risk credentials** (service API keys): Rotate every 90 days
- **Low-risk credentials** (read-only API keys): Rotate every 180 days

This is configurable and depends on your threat model. A healthcare company handling patient data might rotate everything every 7 days. A hobby project might rotate annually.

---

### F. Secret Management Systems: The Central Repository for Credentials

Managing secrets manually—storing them in configuration files, environment variables, or developer notebooks—is a recipe for disaster. A centralized secret management system is the solution.

**What Is a Secret Management System?**

A secret management system is a secure vault that stores all your credentials in one place, controls who can access them, automatically rotates them, and provides an audit trail of who accessed what and when.

**HashiCorp Vault: A Detailed Example**

HashiCorp Vault is one of the most popular open-source secret management systems. Here's how it works:

**Architecture Overview**

```
┌─────────────────────────────────────────────┐
│         HashiCorp Vault Server              │
│  (Encrypted database of all secrets)        │
│                                             │
│  ├─ Database passwords                     │
│  ├─ API keys                               │
│  ├─ OAuth tokens                           │
│  ├─ SSH keys                               │
│  └─ Certificates                           │
└─────────────────────────────────────────────┘
         ↑                    ↑
         │                    │
    Your AI Service      Your Backup System
    (requests secrets)   (requests secrets)
```

**Initialization: Setting Up Vault**

When you first start Vault, it's sealed—locked and encrypted. You initialize it with a master key split into multiple parts (called "key shards"). This is like a safe deposit box that requires multiple people to unlock.

```
vault operator init -key-shares=5 -key-threshold=3
```

This command generates 5 key shards, and you need any 3 of them to unseal Vault. You distribute these shards to trusted team members. If one person leaves the company, they can't unseal Vault alone. If one shard gets compromised, the attacker needs two more to be dangerous.

**Storing Secrets in Vault**

Once Vault is unsealed and running, you store your secrets. For example, storing a database password:

```
vault kv put secret/database/prod \
  username=admin \
  password=SuperSecurePassword123
```

Vault encrypts this data at rest. Even if someone steals the server's hard drive, they can't read the secrets without the encryption keys.

**Retrieving Secrets: The AI Service**

Your AI service needs the database password. It doesn't read a configuration file. Instead, it authenticates with Vault:

```
POST /auth/kubernetes/login
{
  "role": "my-ai-service",
  "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

Vault verifies that the request is coming from the legitimate AI service (using Kubernetes authentication, for example). If valid, Vault returns a token:

```
{
  "auth": {
    "client_token": "hvs.CAESIIv..."
  }
}
```

The AI service now uses this token to request the database password:

```
GET /v1/secret/database/prod
Authorization: Bearer hvs.CAESIIv...
```

Vault returns the password, and the AI service uses it to connect to the database. Critically, the password was never stored in the AI service's configuration—it was retrieved on-demand from Vault.

**Automatic Rotation in Vault**

Vault can automatically rotate secrets on a schedule. For database passwords, Vault can:

1. Connect to your database with admin credentials
2. Generate a new password
3. Update the database user's password
4. Update the stored secret in Vault
5. Notify all services that use this secret

All of this happens automatically. Your AI service doesn't need to know about the rotation—it just requests the current password from Vault each time it needs it.

**Audit Logging**

Vault logs every access:

```
Time: 2026-01-15T14:23:45Z
User: my-ai-service
Action: Read secret
Path: secret/database/prod
Result: Success
```

If you suspect a breach, you can check: "When was this secret accessed? From what IP address? By what service?" This audit trail is invaluable for incident response.

**AWS Secrets Manager and Azure Key Vault: Cloud Alternatives**

If you're running on AWS or Azure, you don't necessarily need to manage Vault yourself. These cloud providers offer managed secret management services:

**AWS Secrets Manager:**
- Stores secrets encrypted in AWS's managed database
- Automatically rotates secrets using Lambda functions
- Integrates seamlessly with EC2, ECS, Lambda, and other AWS services
- Provides fine-grained IAM policies to control who can access what

**Azure Key Vault:**
- Similar to Secrets Manager but for Azure
- Stores secrets, keys, and certificates
- Integrates with Azure services like App Service, Functions, and Container Instances
- Provides hardware security module (HSM) options for extra protection

The core concept is the same across all of them: centralized storage, automatic rotation, and audit logging.

---

### G. The Principle of Least Privilege (PoLP): Giving Each Service Only What It Needs

**What Is the Principle of Least Privilege?**

The principle of least privilege states: **Every user, service, and process should have the minimum permissions necessary to do their job—nothing more.**

This principle dramatically reduces the damage of a compromise. If an attacker gains access to a service, they can only do what that service is authorized to do.

**A Real-World Analogy**

Imagine a restaurant. The dishwasher needs access to the kitchen and storage room but shouldn't have keys to the office safe or the manager's private office. The manager needs access to the office and vault but doesn't need keys to the walk-in freezer. The chef needs access to the kitchen and storage but not the cash register. Each person has exactly the keys they need—nothing more.

**PoLP in AI Services: Practical Examples**

**Example 1: The Recommendation Engine**

Your AI recommendation service needs to:
- Read product data from the database
- Read user purchase history
- Write recommendations to a cache
- Call the pricing API

Using PoLP, you'd create an API key or OAuth token with exactly these permissions:

```
{
  "service": "recommendation-engine",
  "permissions": [
    "database:read:products",
    "database:read:purchase_history",
    "cache:write:recommendations",
    "api:call:pricing"
  ],
  "restrictions": {
    "ip_whitelist": ["10.0.1.50"],
    "rate_limit": "1000 requests/minute"
  }
}
```

Notice what's NOT included:
- No access to user personal information (addresses, phone numbers)
- No write access to products (can't modify product data)
- No access to payment information
- No access to other services

If this API key gets compromised, an attacker can't access customer payment data or modify products. They can only read products and purchase history, and write recommendations.

**Example 2: The Notification Service**

Your notification service needs to send emails and SMS. It should NOT need to access customer data beyond what's required for the message. Using PoLP:

```
{
  "service": "notification-service",
  "permissions": [
    "email:send",
    "sms:send",
    "database:read:user_contact_info"
  ],
  "restrictions": {
    "rate_limit": "10000 messages/hour",
    "allowed_templates": ["welcome", "password_reset", "order_confirmation"]
  }
}
```

The service can only send messages using approved templates. It can't create custom messages that might be used for phishing. It can't read customer payment history or medical information.

**Example 3: The Logging Service**

Your logging service needs to write logs but shouldn't need to read them (that's for administrators). Using PoLP:

```
{
  "service": "logging-service",
  "permissions": [
    "logs:write"
  ],
  "restrictions": {
    "rate_limit": "100000 log entries/minute"
  }
}
```

If this service gets compromised, an attacker can write fake logs to cover their tracks, but they can't read existing logs to find evidence of other attacks. This is a deliberate trade-off—we accept that an attacker might cover their tracks in exchange for preventing them from reading sensitive logs.

**Role-Based Access Control (RBAC): Implementing PoLP**

RBAC is a practical way to implement the principle of least privilege. Instead of assigning permissions to individual services, you assign them to roles. Services are then assigned to roles.

**Example RBAC Structure:**

```
Roles:
├─ database-reader
│  └─ permissions: [database:read:*]
│
├─ api-caller
│  └─ permissions: [api:call:external]
│
├─ cache-writer
│  └─ permissions: [cache:write:*]
│
└─ admin
   └─ permissions: [*]

Services:
├─ recommendation-engine → roles: [database-reader, cache-writer, api-caller]
├─ notification-service → roles: [database-reader]
├─ logging-service → roles: [logs-writer]
└─ admin-dashboard → roles: [admin]
```

**Benefits of RBAC:**

- **Easier management**: Instead of configuring permissions for 50 services individually, you define 5 roles and assign each service to the appropriate role.
- **Consistency**: All services with the same role have the same permissions.
- **Scalability**: When you add a new service, you just assign it to an existing role.
- **Compliance**: Many regulatory frameworks (like SOC 2, ISO 27001) require documented access controls, and RBAC makes this easy.

**The Challenge: AI Agents and Non-Human Identities**

Here's where things get interesting in 2026. As AI agents become more autonomous and powerful, they represent a new category of "insider"—a non-human identity that can perform actions at machine speed.

Consider an AI agent that autonomously manages your cloud infrastructure. If this agent gets compromised or manipulated, it could potentially:
- Spin up expensive resources and rack up bills
- Modify security groups and expose databases
- Delete critical data
- Change billing information

This is why applying PoLP to AI agents is critical. An AI infrastructure management agent should have permission to:
- Create and modify resources in specific namespaces
- Scale existing services
- Restart services

But NOT:
- Delete production databases
- Modify billing information
- Access other teams' resources
- Change security policies

---

### H. Putting It All Together: A Complete Security Architecture

Let's design a complete, production-ready security architecture for an AI customer support system.

**System Overview:**

```
┌─────────────────────────────────────────────────────────────┐
│                    Customer Browser                          │
│  (User logs in with email/password via OAuth 2.0)           │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
        ┌────────────────────────────────┐
        │    Your Authentication Service  │
        │  (Verifies user, issues JWT)   │
        └────────────────────────────────┘
                         │
                         ↓
        ┌────────────────────────────────┐
        │   API Gateway (Rate limiting,  │
        │    logging, request routing)   │
        └────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   ┌─────────┐   ┌──────────────┐   ┌──────────────┐
   │ AI Chat │   │ Recommendation│   │ Notification │
   │ Service │   │ Engine       │   │ Service      │
   └────┬────┘   └──────┬───────┘   └──────┬───────┘
        │               │                   │
        └───────────────┼───────────────────┘
                        │
                        ↓
        ┌────────────────────────────────┐
        │   HashiCorp Vault / AWS        │
        │   Secrets Manager              │
        │  (Central credential store)    │
        └────────────────┬───────────────┘
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   ┌──────────┐   ┌──────────┐   ┌──────────┐
   │ Database │   │ Cache    │   │ External │
   │ Password │   │ Auth Key │   │ API Keys │
   └──────────┘   └──────────┘   └──────────┘
```

**Authentication Flow:**

1. **User Login**: Customer visits your website and clicks "Login with Google" (OAuth 2.0)
2. **Token Issuance**: Your auth service receives an OAuth token from Google and issues a JWT token to the user
3. **API Request**: User's browser sends the JWT in the Authorization header
4. **Gateway Verification**: API Gateway verifies the JWT signature and checks permissions
5. **Service Authorization**: The specific service (Chat, Recommendation, etc.) receives the request with the user's identity

**Service-to-Service Communication:**

1. **AI Chat Service needs database access**: It requests the database credentials from Vault using its service account token
2. **Vault Authentication**: Vault verifies that the request is coming from the legitimate AI Chat Service (using Kubernetes authentication)
3. **Credential Retrieval**: Vault returns the current database password
4. **Database Connection**: AI Chat Service connects to the database using the retrieved password
5. **Automatic Rotation**: Every 30 days, Vault automatically generates a new database password and updates all services

**Secret Rotation Timeline:**

```
Day 1:  Vault generates password_v1 → Stored in Vault
Day 30: Vault generates password_v2 → Both v1 and v2 work
Day 31: Vault disables password_v1 → Only v2 works
Day 60: Vault generates password_v3 → Both v2 and v3 work
Day 61: Vault disables password_v2 → Only v3 works
```

**Incident Response: What If a Secret Leaks?**

Suppose an API key for the external payment API gets exposed on GitHub:

1. **Detection**: Your monitoring system alerts you (or a security researcher reports it)
2. **Immediate Action**: You revoke the leaked key in the secret manager
3. **Service Impact**: The notification service can no longer access the payment API
4. **Rotation**: Vault generates a new API key and updates the notification service
5. **Recovery Time**: 5-10 minutes from detection to full recovery
6. **Damage Assessment**: Because the key had limited permissions (PoLP), the attacker could only access payment notification APIs, not customer payment data

---

### I. Common Mistakes and How to Avoid Them

**Mistake 1: Storing Secrets in Configuration Files**

❌ Wrong:
```
# config.yaml
database_password: "MySecurePass123"
api_key: "sk_live_51234567890"
```

This file often gets committed to version control, copied to servers, and backed up—creating many copies of your secrets that are hard to rotate.

✓ Right:
```
# config.yaml
vault_addr: "https://vault.company.com"
vault_role: "my-service"
# Secrets are retrieved at runtime from Vault
```

**Mistake 2: Long-Lived API Keys Without Rotation**

❌ Wrong:
```
api_key_created_2020_still_valid_in_2026
```

A key that's been valid for 6 years has a high probability of being leaked somewhere.

✓ Right:
```
# Automatic rotation every 30 days
api_key_created_2026_01_15_expires_2026_02_15
api_key_created_2026_02_15_expires_2026_03_15
```

**Mistake 3: Overprivileged Service Accounts**

❌ Wrong:
```
{
  "service": "notification-service",
  "permissions": ["*"]  # Full access to everything
}
```

If this service gets compromised, an attacker has access to your entire infrastructure.

✓ Right:
```
{
  "service": "notification-service",
  "permissions": [
    "email:send",
    "sms:send",
    "database:read:contact_info"
  ]
}
```

**Mistake 4: Sharing Secrets Across Environments**

❌ Wrong:
```
# Using the same API key in development, staging, and production
api_key_shared_everywhere: "sk_prod_12345"
```

If a developer accidentally exposes the key while testing, production is compromised.

✓ Right:
```
# Separate keys for each environment
api_key_dev: "sk_dev_12345"
api_key_staging: "sk_staging_12345"
api_key_prod: "sk_prod_12345"
```

Each environment has its own credentials, so a leak in development doesn't affect production.

**Mistake 5: No Audit Logging**

❌ Wrong:
```
# No record of who accessed what secrets
```

If a breach occurs, you have no way to know what was accessed or when.

✓ Right:
```
# Complete audit log
2026-01-15T14:23:45Z | Service: ai-chat | Action: read | Secret: database-password | Result: success
2026-01-15T14:24:12Z | Service: notification | Action: read | Secret: email-api-key | Result: success
2026-01-15T14:25:30Z | Service: unknown-service | Action: read | Secret: admin-key | Result: denied (service not authorized)
```

---

### J. Practical Implementation: Step-by-Step Guide

**Step 1: Choose Your Secret Management Tool**

Evaluate based on your needs:

- **HashiCorp Vault**: Open-source, self-hosted, highly flexible, requires operational overhead
- **AWS Secrets Manager**: Managed service, integrates with AWS, pay-per-secret pricing
- **Azure Key Vault**: Managed service for Azure, integrates with Azure services
- **GCP Secret Manager**: Managed service for Google Cloud

For a small team, a managed service is usually better. For large enterprises with complex requirements, Vault is often the choice.

**Step 2: Audit Your Current Secrets**

List all your secrets:
- Database passwords
- API keys
- OAuth tokens
- SSH keys
- Certificates
- Encryption keys

For each, note:
- Where it's currently stored
- Who has access
- How often it's rotated
- When it was last rotated

**Step 3: Migrate Secrets to Your Chosen Tool**

Move all secrets to your secret manager. This is often the most time-consuming step because it requires updating all services to retrieve secrets from the manager instead of configuration files.

**Step 4: Implement Automatic Rotation**

For each secret, configure automatic rotation:
- Database passwords: Every 30 days
- API keys: Every 90 days
- OAuth tokens: Automatic (usually handled by the provider)
- Certificates: 30 days before expiration

**Step 5: Set Up RBAC**

Define roles based on your services:
- AI Chat Service: Can read database credentials, read cache keys, call external APIs
- Notification Service: Can read database credentials, call email/SMS APIs
- Admin Dashboard: Can read all secrets

**Step 6: Implement Audit Logging**

Configure your secret manager to log all access. Send logs to a centralized logging system for analysis and alerting.

**Step 7: Test Incident Response**

Simulate a compromised secret:
1. Revoke the secret
2. Verify services fail gracefully
3. Generate a new secret
4. Verify services recover
5. Document the time to recovery

---

## Key Takeaways

- **OAuth 2.0 and API Keys** serve different purposes: OAuth 2.0 for user-facing authentication and delegated access, API keys for service-to-service communication. OAuth 2.0 is more secure for user scenarios because passwords are never stored in your system.

- **Secrets Rotation** is not optional in production systems. Assume every secret will eventually be compromised, and rotate credentials regularly (30-90 days depending on risk). Automatic rotation through tools like HashiCorp Vault or AWS Secrets Manager prevents human error and ensures consistency.

- **The Principle of Least Privilege** dramatically reduces the impact of a security breach. Each service should have only the permissions it needs to do its job. If a service gets compromised, the attacker is limited to what that service can do, not your entire infrastructure.

- **Centralized Secret Management** (Vault, Secrets Manager, Key Vault) is the foundation of modern security architecture. It provides one place to store, rotate, audit, and control access to all credentials. This makes incident response faster and gives you visibility into who accessed what and when.

- **Non-Human Identities (AI Agents)** are the new insider threat. As AI agents become more autonomous and capable, they need careful access control. Apply PoLP rigorously to AI services—they should have the minimum permissions necessary, with strong monitoring and audit logging.

- **Think of this like a security perimeter**: OAuth 2.0 and tokens are your gates, API keys are your badges, secret management is your vault, PoLP is your security guard who checks each badge, and audit logging is your security camera. Together, they create defense-in-depth that protects your production AI services.

---

## How This Connects to Your AI Journey

As you build and deploy AI services, security becomes increasingly important. The concepts in this lesson form the foundation of production-grade AI systems. You'll use OAuth 2.0 when your AI services need to access user data on behalf of users. You'll use API keys and secret management when your AI services need to call other APIs or databases. You'll apply the principle of least privilege to ensure that if one component gets compromised, the damage is limited.

In future lessons, we'll build on these concepts by exploring monitoring and alerting (detecting when secrets are misused), compliance frameworks (meeting regulatory requirements), and advanced patterns like zero-trust architecture (assuming every request is potentially malicious until proven otherwise).