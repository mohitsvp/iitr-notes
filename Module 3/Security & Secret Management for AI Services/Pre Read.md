# Security & Secret Management for AI Services: Pre-Class Notes

## What You'll Learn

In this pre-read, you'll discover:

- **Why keeping secrets secure matters**: Understand how unauthorized access to passwords, tokens, and API keys can expose your entire AI system to attackers
- **How OAuth 2.0 and API keys work**: Learn the difference between these two authentication methods and when to use each one
- **The art of secret rotation**: Discover why regularly changing your credentials is like changing your house locks—and how modern tools automate this process
- **The Principle of Least Privilege**: Learn how to give your AI services only the minimum permissions they need to do their job, limiting damage if something goes wrong

---

## Introduction: What Is Security & Secret Management?

Imagine you're running a restaurant. You don't give every employee a master key to the cash register, the safe, and the back office. Instead, you give the cashier only access to the register, the chef only access to the kitchen, and the manager a slightly broader set of keys. Each person gets exactly what they need—nothing more.

**Security and secret management is the practice of protecting sensitive information (like passwords, API keys, and authentication tokens) and controlling who gets access to what resources, following the same principle as our restaurant example.**

When you build AI services that connect to databases, call third-party APIs, or access cloud resources, these services need "credentials"—digital proof that says "I'm allowed to do this." The challenge is keeping those credentials safe while making sure the right services can access the right resources.

---

## Importance: Why Does This Matter?

### 1. **Preventing Unauthorized Access and Data Breaches**

Think of your API keys and passwords as the keys to your house. If someone steals them, they can walk right in and take whatever they want. In 2026, data breaches are more sophisticated than ever. Attackers use stolen credentials to access your AI models, steal training data, or manipulate outputs. By properly managing and protecting secrets, you make it dramatically harder for attackers to gain entry. Even if one credential is compromised, proper security practices ensure the damage is limited to just that one service, not your entire system.

### 2. **Meeting Compliance and Regulatory Requirements**

If you're building AI services for healthcare, finance, or government sectors, you're likely subject to regulations like HIPAA, PCI-DSS, or GDPR. These regulations require you to prove that you're protecting sensitive data and controlling access properly. Modern secret management practices aren't optional—they're mandatory. Organizations that fail to implement proper security face hefty fines, legal liability, and loss of customer trust. By implementing these practices now, you're building a foundation that keeps your organization compliant.

### 3. **Enabling Safe Scaling and Team Collaboration**

As your AI service grows, more developers, data scientists, and engineers need access to different parts of your system. Without proper secret management, you'd end up sharing passwords via email or Slack—a nightmare waiting to happen. Modern secret management tools let you grant fine-grained access to specific people and services without ever sharing the actual secrets. This means your team can collaborate safely, and you can audit exactly who accessed what and when.

---

## Building Understanding: From Known to New

Let's start with a problem you probably recognize: managing passwords.

### The Painful Way (Without Proper Secret Management)

Imagine you have an AI service that needs to connect to three different systems: a database, an email service, and a payment processor. Each requires a password. Without proper management:

- You might hardcode passwords directly into your code (terrible idea—anyone who reads your code has access)
- You might store them in a `.env` file and commit it to version control (now it's in your entire Git history)
- You might write them on a sticky note or share them via email (anyone with access to those systems now has your secrets)
- When you need to change a password, you have to update it everywhere manually (and you'll inevitably forget one place)
- You have no way to audit who accessed what secret or when
- If someone leaves your team, you have to manually change every password they knew about

This approach is chaotic, insecure, and doesn't scale.

### The Modern Way (With Secret Management)

Now imagine the same scenario with proper secret management:

- All secrets are stored in a **centralized, encrypted vault** (like AWS Secrets Manager or HashiCorp Vault)
- Your AI service requests credentials from the vault using a secure, temporary token
- The vault logs exactly who requested what secret and when
- You set an expiration date on each secret, and the vault **automatically rotates** (changes) them before they expire
- Different services get different credentials with different permission levels
- When someone leaves your team, you revoke their access with a single command
- Your entire team can safely access secrets through the vault without ever seeing the actual values

This is secure, auditable, and scales beautifully.

---

## Core Components: The Three Pillars of Secure Secret Management

### 1. **Authentication: Proving You Are Who You Say You Are**

Before your AI service can access any resource, it needs to prove its identity. Think of this like showing your ID at a bank.

**OAuth 2.0** is the modern standard for authentication. It's a framework that allows your service to request access without ever sharing the actual password with the service that's granting access. Here's the basic flow:

1. Your AI service says: "I'd like to access the user's data"
2. An authorization server verifies: "Yes, I know this service is legitimate"
3. The authorization server gives your service a temporary **access token**—a piece of digital proof that expires after a short time (typically 15-30 minutes)
4. Your service uses this token to access the resource
5. When the token expires, your service gets a new one

**API Keys** are simpler but less flexible. They're like a long-term password that your service includes with every request. The problem: if an API key is exposed, it remains valid until someone manually revokes it. In 2026, best practices recommend using OAuth 2.0 whenever possible, and only using API keys for simple, non-critical integrations.

**JWT (JSON Web Tokens)** are a specific type of token often used with OAuth 2.0. A JWT is like a digitally signed document that contains information about who you are and what you're allowed to do. Because it's signed, the receiving service can verify it hasn't been tampered with.

### 2. **Authorization: Controlling What You Can Access**

Authentication answers "Who are you?" Authorization answers "What are you allowed to do?"

**Role-Based Access Control (RBAC)** is the traditional approach. You assign people or services to roles (like "Admin," "Editor," "Viewer"), and each role has a predefined set of permissions. For example:

- The "Database Admin" role can read, write, and delete data
- The "Data Analyst" role can only read data
- The "AI Model" role can only read training data and write predictions

RBAC is simple to understand and implement, but it's not very flexible. What if you want to let someone read data only from 9 AM to 5 PM? Or only from a specific location? That's where RBAC falls short.

**Attribute-Based Access Control (ABAC)** is more sophisticated. Instead of assigning roles, you define policies based on attributes: who the person is, what they're trying to do, when they're trying to do it, where they're accessing from, and what resource they're accessing. For example:

- "Allow the AI Model to read training data only from the secure network, during business hours, and only if the data is tagged as 'non-sensitive'"

ABAC is more powerful but also more complex to set up. In 2026, many organizations use a hybrid approach: RBAC for basic structure, with ABAC for fine-grained control where it matters most.

### 3. **Secret Storage and Rotation: Keeping Secrets Safe and Fresh**

This is where tools like **HashiCorp Vault**, **AWS Secrets Manager**, and **Azure Key Vault** come in. These are specialized services designed to:

- **Encrypt secrets at rest**: Your passwords, API keys, and tokens are stored in encrypted form. Even if someone breaks into the server, they can't read the secrets without the encryption keys
- **Control access to secrets**: Only authorized services and people can request secrets
- **Log all access**: Every time someone or something accesses a secret, it's recorded for auditing
- **Automatically rotate secrets**: The vault can change passwords and API keys on a schedule, and inform your services about the new values
- **Generate temporary credentials**: For some services (like databases), the vault can generate temporary usernames and passwords that expire after a few hours

**Secrets rotation** is critical. Imagine you discover that someone might have seen your database password. With rotation, you don't have to panic—your vault automatically generates a new password, updates the database, and tells your AI service to use the new one. All of this can happen without any manual intervention.

---

## The Principle of Least Privilege: Your Safety Net

The **Principle of Least Privilege (PoLP)** is simple but powerful: every user, service, and application should have the minimum permissions needed to do their job, and nothing more.

### Why This Matters

Imagine your AI service is compromised by an attacker. If that service has been given admin access to your entire database, the attacker can steal all your data. But if that service only has permission to read specific tables and write results to a specific output folder, the attacker's damage is limited.

PoLP is your safety net. It ensures that even if something goes wrong, the blast radius is contained.

### How to Implement PoLP

**For People:**
- A data scientist needs to read training data and run experiments, but shouldn't be able to delete data or access production systems
- A junior developer should have limited access to staging environments, not production
- An operations person should only be able to restart services, not modify code

**For Services:**
- Your AI model training service should only be able to read data from the training dataset, not access customer data
- Your inference service (the one making predictions) should only be able to read the model and write predictions, not access the database directly
- Your monitoring service should only be able to read logs, not modify them

**For AI Systems:**
- An AI agent that handles customer support should only be able to access customer information for that specific customer, not all customers
- An AI model that generates reports should only be able to read the data needed for that report, not sensitive financial data
- A language model API should only be able to call specific endpoints, not all endpoints

### Real Example: The Contained Breach

Let's say your AI service gets hacked. Here's what happens with PoLP:

**Without PoLP:** The service has admin access to everything. The attacker steals all your customer data, deletes your backups, and modifies your models. Disaster.

**With PoLP:** The service only has permission to read training data and write predictions to a specific folder. The attacker can see training data (which is less sensitive) and write malicious predictions (which you'll notice and fix). Your customer data, backups, and models are all safe.

---

## Step-by-Step: How Modern Secret Management Works in Practice

### Step 1: Your Service Starts Up

Your AI service launches and needs to access the database. Instead of having a hardcoded password, it has a **service identity**—a certificate or token that proves it's a legitimate service.

```
AI Service: "I'm starting up. I'm the 'training-pipeline' service."
```

### Step 2: Request Credentials

Your service contacts the secret vault and says: "I need database credentials."

```
AI Service → Vault: "I'm 'training-pipeline'. Give me database credentials."
```

### Step 3: The Vault Verifies Identity

The vault checks: "Is this really the training-pipeline service? Does it have permission to access database credentials?" It verifies the service's identity using cryptographic certificates or tokens.

```
Vault: "Yes, I recognize this service. It has permission to access database credentials."
```

### Step 4: Get a Temporary Token

The vault doesn't give the actual database password. Instead, it gives the service a **temporary token** that proves it's authorized. This token expires after a short time.

```
Vault → AI Service: "Here's your temporary database token. It expires in 30 minutes."
```

### Step 5: Use the Credentials

Your AI service uses this token to connect to the database and do its work.

```
AI Service → Database: "Here's my token. I'd like to read training data."
Database: "Token is valid. Here's your data."
```

### Step 6: Token Expires and Refreshes

After 30 minutes, the token expires. If your service is still running, it automatically requests a new one from the vault. The old token is now useless—even if an attacker finds it, they can't use it.

```
AI Service → Vault: "My token is about to expire. Give me a new one."
Vault → AI Service: "Here's a new token. It expires in 30 minutes."
```

---

## Key Features of Modern Secret Management in 2026

### 1. **Dynamic Secret Generation**

Instead of storing static passwords that never change, modern vaults can generate temporary credentials on-the-fly. For example, when your AI service requests database access, the vault can:

- Create a temporary database user just for this service
- Set an expiration time (e.g., 1 hour)
- Automatically delete the user when it expires

This is far more secure than traditional passwords because the credentials are automatically cleaned up and can't be reused.

### 2. **Audit Logging and Compliance**

Every access to a secret is logged:

- Who requested the secret (which service or person)
- When they requested it
- What they did with it
- Whether the request was approved or denied

This creates a complete audit trail that satisfies regulatory requirements and helps you detect suspicious activity.

### 3. **Encryption and Key Management**

Secrets are encrypted using industry-standard encryption (AES-256) both in transit and at rest. The encryption keys themselves are protected by the vault and rotated regularly. In 2026, many vaults use **envelope encryption**, where data is encrypted with a key, and that key is encrypted with another key, creating layers of protection.

---

## Putting It All Together: A Real-World Example

Let's trace through a complete scenario: an AI service that analyzes customer data and generates reports.

### The Setup

- **Service**: Report Generator (an AI service that reads customer data and creates reports)
- **Resources**: Customer Database, Report Storage, Email Service
- **Security Requirements**: The service should only read customer data, write reports, and send emails. Nothing else.

### The Implementation

**1. Define Permissions with PoLP**

```
Report Generator Service:
├── Database: Read-only access to customer_data table
├── Report Storage: Write-only access to /reports/generated/ folder
└── Email Service: Send-only access to send reports
```

**2. Set Up Secret Storage**

All credentials are stored in a vault:

```
Vault Secrets:
├── db_password (encrypted)
├── email_api_key (encrypted)
└── storage_access_token (encrypted)
```

**3. Service Starts and Requests Credentials**

```
Report Generator → Vault: "I'm the Report Generator. I need credentials."
Vault: "Verifying your identity... ✓ Approved. Here are your credentials."
```

**4. Service Uses Credentials Safely**

```
Report Generator → Database: "Here's my token. Get me customer data."
Database: "✓ Token valid. Here's the data."

Report Generator: [Processes data and generates report]

Report Generator → Storage: "Here's my token. Store this report."
Storage: "✓ Token valid. Report stored."

Report Generator → Email: "Here's my token. Send this report."
Email: "✓ Token valid. Report sent."
```

**5. Credentials Automatically Rotate**

Behind the scenes, the vault is managing credential rotation:

```
Vault (daily): "Time to rotate secrets. Generating new credentials..."
Vault → Database: "Update password for Report Generator user"
Vault → Report Generator: "Your new database token is ready"
Report Generator: [Automatically uses new token for next request]
```

**6. Audit Trail**

```
Audit Log:
- 2026-01-15 09:00:00: Report Generator requested db_password ✓
- 2026-01-15 09:00:01: Report Generator requested email_api_key ✓
- 2026-01-15 09:05:30: Report Generator requested storage_access_token ✓
- 2026-01-15 17:30:00: Vault rotated all credentials ✓
```

This entire process is secure, auditable, and automated. If an attacker somehow gets one of the tokens, it expires in 30 minutes and is useless after that.

---

## Common Tools and Platforms in 2026

### **HashiCorp Vault**

An open-source, cloud-agnostic secrets management platform. It's highly flexible and can run anywhere—your data center, AWS, Google Cloud, or Azure. It supports dynamic secrets, multiple authentication methods, and sophisticated policy engines.

**Best for:** Organizations that want maximum flexibility and are willing to manage their own infrastructure.

### **AWS Secrets Manager**

A fully managed service from Amazon that integrates seamlessly with AWS services. It handles automatic rotation, encryption, and audit logging. If your infrastructure is already on AWS, this is often the easiest choice.

**Best for:** Organizations already using AWS who want a managed solution.

### **Azure Key Vault**

Microsoft's secrets management service. Like AWS Secrets Manager, it's fully managed and integrates with Azure services. It also has good support for managing cryptographic keys, not just passwords.

**Best for:** Organizations using Azure infrastructure.

### **Google Cloud Secret Manager**

Google's managed secrets service. Similar to AWS and Azure, it's fully managed and integrates with Google Cloud services.

**Best for:** Organizations using Google Cloud infrastructure.

In 2026, a common pattern is to use a cloud provider's managed secrets service (like AWS Secrets Manager) for most secrets, and potentially a separate tool like Vault for more complex scenarios or multi-cloud environments.

---

## OAuth 2.0 in Action: The Authorization Code Flow

Let's understand the most common OAuth 2.0 flow, which you'll encounter when building AI services that access user data.

### The Scenario

Your AI service wants to access a user's Google Drive to analyze documents. You don't want to ask the user for their Google password (that would be insecure). Instead, you use OAuth 2.0.

### The Flow

**1. User Clicks "Connect to Google Drive"**

```
User → Your AI Service: "I want to connect my Google Drive"
```

**2. Your Service Redirects to Google**

```
Your AI Service → Google: "This user wants to authorize me to access their Drive"
```

**3. Google Shows a Permission Screen**

```
Google: "Your AI Service wants to access your Google Drive. 
         It will be able to read documents. Do you allow this?"
User: "Yes, I allow it"
```

**4. Google Gives Your Service an Authorization Code**

```
Google → Your AI Service: "Here's an authorization code. It's only valid for 10 minutes."
```

**5. Your Service Exchanges the Code for an Access Token**

```
Your AI Service → Google: "I have an authorization code. Here's also my secret."
Google: "Code is valid and your secret is correct. Here's an access token."
```

**6. Your Service Uses the Token**

```
Your AI Service → Google Drive: "Here's my access token. Get me the user's documents."
Google Drive: "✓ Token valid. Here are the documents."
```

### Why This Is Secure

- The user's password is never shared with your service
- The authorization code is short-lived
- The access token is also short-lived (usually 1 hour)
- Google can revoke access at any time
- The user can see exactly what permissions they granted

This is why OAuth 2.0 is the standard for modern applications.

---

## API Keys: When and How to Use Them

API keys are simpler than OAuth 2.0 but less flexible. They're useful for service-to-service communication where there's no user involved.

### Example: Your AI Service Calling a Weather API

```
Your AI Service → Weather API: "Here's my API key: sk_live_abc123xyz789"
Weather API: "✓ Key is valid. Here's today's weather."
```

### Best Practices for API Keys in 2026

- **Rotate regularly**: Change your API keys every 90 days or when someone leaves your team
- **Use environment-specific keys**: Have different keys for development, staging, and production
- **Never commit to version control**: Store API keys in environment variables or a secrets vault, never in code
- **Monitor usage**: Set up alerts if your API key is used in unexpected ways
- **Scope permissions narrowly**: If the API supports it, create keys with limited permissions (e.g., read-only)
- **Use short expiration times**: If the API supports it, set API keys to expire after a certain time

### API Key vs. OAuth 2.0

| Aspect | API Key | OAuth 2.0 |
|--------|---------|----------|
| **Complexity** | Simple | More complex |
| **Security** | Medium | High |
| **User involvement** | None | Yes (user grants permission) |
| **Best for** | Service-to-service | User-facing applications |
| **Token expiration** | Manual | Automatic |
| **Revocation** | Manual | Can be automatic |

---

## Secrets Rotation: Why and How

### Why Rotate Secrets?

Imagine you discover that someone might have seen your database password. With rotation:

- You don't have to panic
- The vault automatically generates a new password
- Services automatically start using the new password
- The old password is invalidated
- Even if someone has the old password, they can't use it

Rotation happens on a schedule (e.g., every 30 days for highly sensitive secrets, every 90 days for less sensitive ones) or manually when needed.

### How Automatic Rotation Works

**1. Schedule is set** (e.g., "rotate every 30 days")

```
Vault Configuration:
- Secret: database_password
- Rotation: Every 30 days
- Last rotated: 2026-01-01
- Next rotation: 2026-01-31
```

**2. Rotation time arrives**

```
Vault (2026-01-31): "Time to rotate database_password"
```

**3. Vault generates new credentials**

```
Vault → Database: "Generate new credentials for the AI service"
Database: "✓ New password created: NewSecurePassword123"
```

**4. Vault updates the secret**

```
Vault: [Stores new password in encrypted storage]
```

**5. Services automatically get the new secret**

```
Vault → AI Service: "Your database password has been updated. 
                     Use this new one on your next request."
AI Service: [Automatically uses new password]
```

**6. Audit log records everything**

```
Audit Log:
- 2026-01-31 02:00:00: Rotation started for database_password
- 2026-01-31 02:00:05: New credentials generated
- 2026-01-31 02:00:10: Secret updated
- 2026-01-31 02:00:15: AI Service notified of rotation
- 2026-01-31 02:00:20: Rotation completed successfully
```

### Rotation Strategies

**Time-based rotation**: Rotate every X days (most common)

**Event-based rotation**: Rotate when someone leaves the team, after a security incident, or when a service is compromised

**Manual rotation**: An administrator manually triggers rotation when needed

In 2026, best practice is a combination: automatic time-based rotation for routine security, plus the ability to manually rotate immediately if needed.

---

## Implementing PoLP in Your AI Service

### For Database Access

**Without PoLP:**
```
AI Service has: Full admin access to entire database
Risk: If compromised, attacker can steal all data, delete everything, modify data
```

**With PoLP:**
```
AI Service has: 
├── Read-only access to training_data table
├── Write-only access to predictions table
└── No access to user_personal_info, financial_data, or other tables
Risk: If compromised, attacker can only see training data and write predictions
```

### For Cloud Resources

**Without PoLP:**
```
AI Service has: Full access to all S3 buckets, all EC2 instances, all databases
Risk: If compromised, attacker can access everything
```

**With PoLP:**
```
AI Service has:
├── Read access to s3://training-data-bucket/
├── Write access to s3://predictions-bucket/
└── No access to other buckets or any EC2 instances
Risk: If compromised, attacker can only access the specific buckets needed
```

### For API Access

**Without PoLP:**
```
AI Service has: Admin API token for payment processor
Risk: If compromised, attacker can process refunds, access transaction history, modify accounts
```

**With PoLP:**
```
AI Service has: Read-only API token for payment processor
Risk: If compromised, attacker can only read transaction history, cannot process refunds or modify accounts
```

### Implementing PoLP: Step by Step

1. **Identify what your service needs to do**: List every action it takes (read data, write results, call APIs, etc.)

2. **Map to resources**: For each action, identify exactly which resources it needs to access (which databases, which folders, which APIs)

3. **Define minimal permissions**: Create permissions that allow exactly those actions, nothing more

4. **Test thoroughly**: Verify that your service works with these limited permissions

5. **Monitor and adjust**: As your service evolves, update permissions to match new requirements

6. **Review regularly**: Every quarter, review permissions and remove any that are no longer needed

---

## Security Best Practices for AI Services

### 1. **Never Hardcode Secrets**

❌ **Bad:**
```python
DATABASE_PASSWORD = "mySecurePassword123"
API_KEY = "sk_live_abc123xyz789"
```

✅ **Good:**
```python
# Get secrets from vault
database_password = vault.get_secret("database_password")
api_key = vault.get_secret("api_key")
```

### 2. **Use Environment Variables for Local Development**

```bash
# .env file (never commit this!)
DATABASE_PASSWORD=dev_password_123
API_KEY=dev_api_key_456
```

### 3. **Implement Proper Logging**

Log authentication and authorization events:
- Who accessed what
- When they accessed it
- Whether access was granted or denied
- Any anomalies or suspicious activity

### 4. **Use HTTPS/TLS for All Communication**

All communication between your service and the vault, database, or APIs should be encrypted in transit.

### 5. **Implement Rate Limiting**

Prevent brute force attacks by limiting how many times someone can try to authenticate:

```
After 5 failed login attempts, lock the account for 15 minutes
```

### 6. **Regular Security Audits**

Periodically review:
- Who has access to what
- Which secrets are being used
- Whether any suspicious access patterns exist
- Whether permissions align with PoLP

---

## Practice Exercises

### Exercise 1: Identify the Security Issue

**Scenario:** Your AI service needs to connect to a database and send emails. A junior developer hardcodes both the database password and email API key directly in the code and commits it to GitHub.

**Question:** What security vulnerabilities does this create? What could go wrong?

**Think about:** Who can see the code? What can an attacker do with these credentials? How long does it take to discover the breach? How do you fix it?

---

### Exercise 2: Design Permissions with PoLP

**Scenario:** You're building an AI service that:
1. Reads customer data from a database
2. Processes the data with a machine learning model
3. Writes results to a reporting database
4. Sends notifications via email

**Question:** What should this service's permissions look like?

**Think about:** What does it need to read? What does it need to write? What should it NOT be able to do? What happens if it's compromised?

---

### Exercise 3: Trace the OAuth 2.0 Flow

**Scenario:** A user wants to use your AI service to analyze their Gmail inbox. Your service uses OAuth 2.0 to request permission.

**Question:** Trace the complete flow from start to finish. What messages are sent? Who sends them? What does each party verify?

**Think about:** Why doesn't your service ask for the user's Gmail password? What stops an attacker from using a stolen token? How long is the token valid?

---

### Exercise 4: Secrets Rotation Challenge

**Scenario:** Your organization has 50 AI services, each connecting to 3-4 different systems (databases, APIs, storage). You need to implement automatic secrets rotation.

**Question:** What challenges do you face? How do you ensure no service loses access during rotation?

**Think about:** What happens if a service tries to use a secret while it's being rotated? How do you coordinate rotation across multiple systems? How do you handle failures?

---

### Exercise 5: Spot the PoLP Violation

**Scenario:** You review your organization's access controls and find:

- The data pipeline service has read access to all customer personal data (not just training data)
- The inference service has write access to the customer database (not just read access)
- The monitoring service has admin access to all systems (not just read access to logs)
- The junior developer has production database access (same as senior developers)

**Question:** For each case, explain why it violates PoLP and what the proper permissions should be.

**Think about:** What is each service's actual job? What permissions does it really need? What damage could occur if each service is compromised?

---

## Key Takeaways

- **Secret management is foundational**: Properly protecting and managing credentials is essential for any production AI system
- **OAuth 2.0 is the modern standard**: For applications that involve users, OAuth 2.0 provides better security than API keys alone
- **Secrets should be rotated regularly**: Automatic rotation reduces the window of vulnerability if a secret is compromised
- **The Principle of Least Privilege is your safety net**: Limiting permissions to only what's needed contains damage if something goes wrong
- **Modern tools automate the hard parts**: Services like AWS Secrets Manager, HashiCorp Vault, and Azure Key Vault handle encryption, rotation, and auditing
- **Audit logging is critical**: You can't secure what you can't see—comprehensive logging helps you detect and respond to security incidents

---

## What's Next?

In the upcoming class, we'll dive deeper into:

- **Hands-on implementation**: Setting up a secrets vault for your first AI service
- **Real-world scenarios**: How to handle secrets rotation during deployments, secrets management in containerized environments (Docker/Kubernetes), and multi-cloud secret synchronization
- **Troubleshooting**: Common mistakes and how to fix them
- **Advanced topics**: Dynamic secrets for databases, secrets management for AI agents, and securing LLM API keys

You now have the conceptual foundation. In the next class, we'll build the practical skills to implement these concepts in production systems.

---

## Quick Reference: Tools and Commands

### Getting Started with Secret Vaults

**AWS Secrets Manager:**
```bash
# Store a secret
aws secretsmanager create-secret --name database-password --secret-string "myPassword123"

# Retrieve a secret
aws secretsmanager get-secret-value --secret-id database-password
```

**HashiCorp Vault:**
```bash
# Start Vault
vault server -dev

# Store a secret
vault kv put secret/database-password value="myPassword123"

# Retrieve a secret
vault kv get secret/database-password
```

### Environment Variables (Local Development)

```bash
# Create a .env file
echo "DATABASE_PASSWORD=dev_password_123" > .env
echo "API_KEY=dev_api_key_456" >> .env

# Add to .gitignore so it's never committed
echo ".env" >> .gitignore

# Load in your application
source .env
```

### OAuth 2.0 Token Example

```
Authorization header:
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

This JWT contains:
{
  "sub": "user123",
  "exp": 1672531200,
  "iat": 1672444800,
  "scopes": ["read:drive"]
}
```

### Audit Log Example

```
2026-01-15 09:00:00 | INFO  | Service: ai-training | Action: SECRET_REQUESTED | Secret: database-password | Result: GRANTED
2026-01-15 09:00:01 | INFO  | Service: ai-training | Action: SECRET_REQUESTED | Secret: email-api-key | Result: GRANTED
2026-01-15 09:05:30 | INFO  | Service: ai-training | Action: SECRET_REQUESTED | Secret: storage-token | Result: GRANTED
2026-01-15 17:30:00 | INFO  | Vault: ROTATION_STARTED | Secret: database-password
2026-01-15 17:30:15 | INFO  | Vault: ROTATION_COMPLETED | Secret: database-password
```

---

## Glossary of Key Terms

**Access Token:** A temporary credential that proves you have permission to access a resource. Usually expires after 15 minutes to 1 hour.

**API Key:** A long-term credential used to authenticate service-to-service communication. Less secure than OAuth 2.0 but simpler to implement.

**Authentication:** The process of verifying who you are (proving your identity).

**Authorization:** The process of determining what you're allowed to do (granting permissions).

**Credential:** Any piece of information that proves your identity, such as a password, API key, or token.

**Encryption:** The process of converting readable information into a code that only authorized parties can decode.

**JWT (JSON Web Token):** A digitally signed token that contains information about who you are and what you're allowed to do.

**OAuth 2.0:** A modern framework for secure authentication and authorization, especially useful when users are involved.

**Principle of Least Privilege (PoLP):** The security practice of giving users and services only the minimum permissions they need to do their job.

**Role-Based Access Control (RBAC):** A system where permissions are assigned based on predefined roles (like "Admin" or "Editor").

**Attribute-Based Access Control (ABAC):** A more flexible system where permissions are based on attributes like time, location, and resource sensitivity.

**Secret:** Any sensitive information that should be protected, such as passwords, API keys, or tokens.

**Secret Rotation:** The process of regularly changing credentials to reduce the window of vulnerability if they're compromised.

**Vault:** A secure, centralized storage system for secrets that handles encryption, access control, and rotation.

**Zero Trust:** A security philosophy that assumes no one and nothing should be trusted by default—all access must be verified and authorized.